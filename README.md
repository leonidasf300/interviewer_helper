# Interviewer Helper

Asistente de respuestas en vivo para pruebas de `ai-voice-interview-platform`.

## Qué es

Una herramienta que escucha en tiempo real las preguntas de un entrevistador (IA o humano) durante pruebas manuales, genera una respuesta sugerida con LLM, y la muestra en pantalla para que el tester humano la lea en voz alta — ahorrándole al tester tener que improvisar respuestas técnicas complejas mientras valida el pipeline real de voz de la plataforma.

## Por qué

`ai-voice-interview-platform` es una plataforma de entrevistas técnicas por voz. Hoy, probar el flujo completo requiere que un humano haga de candidato manualmente, lo cual es lento y depende de que el tester improvise buenas respuestas. Este proyecto acelera esas pruebas manuales.

## Cómo funciona (arquitectura)

Pipeline de baja latencia:

1. **Captura de audio**: loopback del audio de salida del sistema (lo que suena por parlantes/headset)
2. **VAD**: detecta cuándo el entrevistador terminó de hablar → dispara cierre de transcripción
3. **STT (streaming)**: transcribe la pregunta en tiempo real (bajo latencia)
4. **LLM (streaming)**: genera respuesta sugerida (técnica, contexto AI/Data Eng)
5. **UI de consola**: muestra pregunta + respuesta en streaming
6. **Métricas**: latencia entre fin-de-pregunta y aparición de respuesta sugerida

Stack: Python, proveedores STT/LLM cloud con opción local (pluggable).

## Roadmap (Milestones)

- **M1**: Captura de audio loopback (PoC) — Windows
- **M2**: STT en tiempo real — transcripción en consola
- **M3**: VAD / segmentación de preguntas
- **M4**: LLM + respuesta sugerida
- **M5**: Medición y tuning de latencia
- **M6**: Pulido de UI de consola
- **Futuro**: soporte español/bilingüe, captura de mic del humano, automatización 100%, portabilidad cloud

## Entornos de prueba

1. **Contra la plataforma**: sesión real contra `ai-voice-interview-platform` en local (Clara, 8 preguntas fijas)
2. **Llamada con colega**: sesión Zoom/Meet con ingeniero haciendo de entrevistador (más variedad, entorno controlado)

## Prerequisitos

- Python 3.10+
- `ai-voice-interview-platform` corriendo en local (`docker-compose up`)
- API keys para proveedor STT y LLM elegido
- Dispositivo de audio con salida accesible por loopback (headset/parlantes)

## Desarrollo

```bash
# Clonar / entrar al proyecto
cd "C:\Users\ASUS\CodingProjects\interviewer helper"

# (A implementar) Crear env virtual y instalar deps
python -m venv venv
source venv/Scripts/activate  # Windows
pip install -r requirements.txt

# (A implementar) Correr
python main.py
```

## Estado

🟡 **En planeación** — lista de tareas inicial a definir. Milestone 1 (captura de audio) en construcción.
