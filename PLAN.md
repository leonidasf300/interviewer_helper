# Interviewer Helper — Plan de desarrollo

Asistente de respuestas en vivo para pruebas de `ai-voice-interview-platform`.

## Contexto

`ai-voice-interview-platform` (repo hermano en `C:\Users\ASUS\CodingProjects\ai-voice-interview-platform`) es una plataforma web (React+Vite / FastAPI / Postgres+Redis) donde una IA entrevistadora ("Clara") hace entrevistas técnicas por voz. Hoy, probar el flujo completo requiere que un humano haga de candidato manualmente — hablando respuestas técnicas reales frente al navegador — lo cual es lento y depende de que el tester improvise buenas respuestas técnicas.

Este proyecto (`interviewer-helper`) es una herramienta **independiente y desacoplada** (repo propio, sin compartir código con la plataforma) que asiste a un **tester humano** durante esas pruebas manuales: escucha en tiempo real la pregunta que hace la IA entrevistadora (vía audio real del sistema), genera una respuesta sugerida con un LLM, y la muestra en pantalla para que el humano la lea en voz alta hacia su propio micrófono. El humano sigue siendo quien "contesta" — la herramienta solo le ahorra tener que pensar/improvisar la respuesta técnica, y como es voz humana real (no sintética), también sirve para validar el pipeline de voz real de la plataforma (TTS de Clara + Web Speech Recognition del navegador).

No es un candidato 100% automatizado ni está pensada para entrevistas de trabajo reales — es tooling de apoyo para pruebas manuales internas. La latencia importa porque el tester necesita el texto sugerido rápido para poder leerlo con naturalidad antes de que la plataforma marque timeout o silencio.

La v1 corre en local (misma PC o PC separada en la misma red); el diseño debe permitir moverlo a otra PC/cloud después sin rediseño mayor.

Como el pipeline escucha audio real del sistema (no depende de la API de la plataforma), es **agnóstico a quién hace la pregunta** — funciona igual si la pregunta viene de Clara (voz sintética) o de una persona real en una llamada (Zoom/Meet). Esto se aprovecha para ampliar las pruebas: Clara solo tiene 8 preguntas fijas, así que además de probar contra la plataforma, se harán sesiones de prueba en llamada con un colega ingeniero haciendo de entrevistador (ambos conscientes de que es una prueba/práctica del tool, no una entrevista real) — esto da más variedad de preguntas y un entorno más controlado para validar el pipeline completo.

## Decisiones ya tomadas

- **Un solo modo**: asistente humano-en-el-loop (se descarta, por ahora, un candidato 100% automatizado vía API — puede revisitarse a futuro como herramienta de CI/regresión separada).
- **Fuente de la pregunta: audio real del sistema (loopback) + STT**, no la API de la plataforma — prioriza fidelidad total (así viviría la pregunta un candidato real) y mantiene la herramienta reutilizable contra otras plataformas de entrevista a futuro, no solo esta.
- **GPU**: no confirmada (a determinar/detectar en setup) — afecta si Whisper/LLM local son viables para latencia mínima.
- **Proveedores STT/LLM**: híbrido priorizando latencia — usar APIs cloud de streaming donde den menor latencia (ej. STT tipo Deepgram/AssemblyAI, LLM tipo Anthropic/OpenAI con streaming), con arquitectura "pluggable" para poder cambiar de proveedor o usar local (Whisper/Ollama) sin rediseñar.
- **Stack**: Python — mejor soporte para captura de audio loopback en Windows e integración con proveedores de IA.
- **Idioma MVP**: inglés primero (reduce complejidad de detección de idioma / prompts bilingües); español se agrega después de validar el pipeline.
- **Rol objetivo**: entrevistas de AI Engineering / Data, abierto a otros roles después.

## Objetivo general

Construir un asistente que escuche en tiempo real las preguntas de una IA entrevistadora durante pruebas manuales de `ai-voice-interview-platform`, transcriba la pregunta, genere con un LLM una respuesta técnica sugerida, y la muestre en pantalla lo más rápido posible para que el tester humano la lea en voz alta — minimizando la latencia entre el fin de la pregunta y la aparición de la respuesta sugerida.

## Arquitectura (pipeline)

1. **Captura de audio**: loopback del audio de salida del sistema (lo que suena por parlantes/headset) para escuchar la pregunta de Clara. Mic del tester no es necesario para el pipeline core (el humano habla directo a la plataforma), pero queda como posible mejora futura (ej. detectar cuándo el humano empieza a responder, para saber que la pregunta actual ya se cerró y resetear el estado).
2. **VAD (detección de fin de habla)**: para saber cuándo Clara terminó de hablar y disparar el cierre de la transcripción — clave para no esperar de más ni cortar la pregunta antes de tiempo.
3. **STT (transcripción)**: streaming, de baja latencia — para que al momento de detectar silencio, la transcripción final ya esté (casi) lista.
4. **LLM**: genera la respuesta sugerida en streaming (para que el tester empiece a leer los primeros tokens mientras se termina de generar el resto), con un prompt orientado a entrevistas técnicas de AI/Data Eng.
5. **UI de consola**: muestra la pregunta transcrita y la respuesta sugerida en streaming, de forma simple y legible en tiempo real.
6. **Instrumentación de latencia**: medir y mostrar el tiempo entre fin-de-pregunta y aparición de los primeros tokens de respuesta, para poder tunear cada etapa del pipeline.

Las decisiones de librería específicas (proveedor STT/LLM exacto, librería de VAD, librería de audio loopback) quedan **abiertas** — se deciden en la fase de implementación de cada milestone, no están fijadas en este plan.

## Roadmap

1. **M1 — Captura de audio (PoC)**: capturar el audio de salida del sistema (loopback) en Windows y confirmar que se escucha correctamente la voz de Clara durante una sesión de prueba real de la plataforma.
2. **M2 — STT en tiempo real**: transcripción en vivo de lo capturado, mostrada en consola; validar precisión y latencia (solo inglés).
3. **M3 — VAD / segmentación de preguntas**: detectar fin de pregunta para saber cuándo cerrar la transcripción y disparar el siguiente paso.
4. **M4 — LLM + respuesta sugerida**: enviar la pregunta transcrita (con contexto de entrevista AI/Data Eng) al LLM y mostrar la respuesta en streaming en consola.
5. **M5 — Medición y tuning de latencia**: instrumentar tiempos por etapa, mostrar métrica end-to-end, ajustar para minimizar el delay entre fin-de-pregunta y primera palabra sugerida.
6. **M6 — Pulido de UI de consola**: vista en vivo con pregunta transcrita, respuesta en streaming, e historial de la sesión.
7. **Futuro (fuera de este plan)**: soporte de español/bilingüe; captura de mic del humano para auto-detectar cuándo empezó a responder; modo 100% automatizado vía API (para regresión/CI, sin humano); portar a otra PC/cloud; módulo de práctica para humanos (la IA hace de entrevistador y el humano practica, con feedback post-entrevista) como proyecto/módulo separado.

## Verificación

Dos entornos de prueba, ya que Clara solo cubre 8 preguntas fijas:

1. **Contra la plataforma**: sesión real contra `ai-voice-interview-platform` en local (`docker-compose up`), escuchando las preguntas de Clara.
2. **Llamada con colega**: sesión por Zoom/Meet con un colega ingeniero haciendo de entrevistador (ambos conscientes de que es una prueba/práctica del tool) — da más variedad de preguntas y un entorno controlado para validar el pipeline fuera del set fijo de Clara.

Checks por milestone:
- M1: grabación de una sesión en cada entorno, confirmar que el audio capturado corresponde a la voz del entrevistador (Clara o colega).
- M2: transcripción en vivo revisada manualmente contra lo que realmente se dijo, en al menos 3 preguntas de cada entorno.
- M3: detección de fin de pregunta correcta en pruebas repetidas (sin cortes prematuros ni esperas largas de más), en ambos entornos.
- M4: respuesta sugerida coherente y técnicamente razonable para preguntas típicas de AI/Data Eng, en ambos entornos.
- M5: reporte de latencia end-to-end (fin de pregunta → primeros tokens de respuesta) en al menos 3 corridas por entorno.

## Prerequisitos / dependencias externas

- `ai-voice-interview-platform` corriendo en local (`docker-compose up`) para las pruebas end-to-end.
- API key(s) del/los proveedor(es) de STT y LLM elegidos.
- Dispositivo de audio con salida accesible por loopback (headset/parlantes del PC de pruebas).
