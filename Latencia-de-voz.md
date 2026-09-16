# Latencia en sistemas de voz con IA

La latencia en la IA conversacional es el tiempo total que transcurre desde que el usuario termina de hablar hasta que el sistema comienza a ofrecer una respuesta.

En un agente de voz, la latencia es un factor fundamental para la experiencia del usuario. Una latencia baja permite que las respuestas lleguen rápidamente y hace que la conversación resulte más natural, fluida y similar a una conversación humana.

## Escenarios de latencia

De forma orientativa, podemos distinguir los siguientes escenarios:

- **Menos de 500 ms:** el agente se percibe como muy ágil y eficiente. En muchos casos, el usuario apenas percibe el tiempo transcurrido entre terminar de hablar y recibir la respuesta.
- **Entre 500 y 1000 ms:** comienza a existir un retraso perceptible. La conversación sigue siendo funcional, pero pueden aparecer pequeñas pausas entre los turnos.
- **Más de 1000 ms:** la respuesta empieza a percibirse como lenta. Las pausas pueden romper el ritmo de la conversación y hacer que la interacción resulte menos natural.

Como objetivo general, en un agente de voz es recomendable intentar mantener la latencia por debajo de 500-700 ms, especialmente en interacciones que requieren respuestas rápidas.

## ¿De dónde procede la latencia?

La latencia total no depende de un único elemento, sino de la suma de diferentes procesos que intervienen durante una conversación:

- **STT (Speech-to-Text):** convierte la voz del usuario en texto.
- **LLM (Large Language Model):** procesa la petición, interpreta el contexto y genera la respuesta.
- **TTS (Text-to-Speech):** convierte la respuesta generada por el modelo de texto a audio.
- **VAD (Voice Activity Detection):** detecta cuándo el usuario empieza y termina de hablar, permitiendo determinar cuándo debe comenzar el procesamiento de la petición.
- **Red y overhead:** incluye el tiempo necesario para transmitir los datos entre los diferentes servicios y componentes del sistema.

De forma simplificada:

> **Latencia total ≈ VAD + STT + LLM + TTS + Red**

Por este motivo, conseguir una latencia muy baja requiere optimizar cada componente del sistema, evitando esperas y procesos innecesarios.

## ¿Cómo reducir la latencia?

Existen diferentes estrategias para reducir la latencia de un agente de voz:

### 1. Utilizar un TTS optimizado para velocidad

Elegir un sistema Text-to-Speech que genere audio rápidamente reduce el tiempo que el usuario tiene que esperar para escuchar la respuesta.

### 2. Elegir un LLM rápido y eficiente

No siempre es necesario utilizar el modelo más potente. Para un agente de voz, puede ser más importante utilizar un modelo con baja latencia, especialmente cuando las respuestas no requieren un razonamiento complejo. También es recomendable evitar generar más tokens de los necesarios.

### 3. Utilizar un VAD rápido

Un sistema de Voice Activity Detection eficiente permite detectar rápidamente cuándo el usuario ha terminado de hablar y comenzar el procesamiento de la petición.

### 4. Utilizar streaming en el TTS

El TTS mediante streaming permite comenzar a reproducir el audio antes de que se haya generado completamente la respuesta.

Por ejemplo:
`LLM genera → TTS comienza a sintetizar → se reproduce el primer fragmento de audio → continúa la generación`

De esta forma, el usuario puede empezar a escuchar las primeras palabras mientras el sistema continúa procesando el resto de la respuesta.

### 5. Reducir el tamaño del prompt

Un prompt excesivamente grande puede aumentar el tiempo de procesamiento y el número de tokens utilizados. Es recomendable mantener el contexto y las instrucciones lo más compactos posible, enviando únicamente la información necesaria.

### 6. Optimizar la BBDD y el RAG

Las consultas a bases de datos y sistemas RAG (Retrieval-Augmented Generation) deben realizarse únicamente cuando sean necesarias. Además, es recomendable recuperar la cantidad mínima de información relevante, evitando enviar al LLM grandes cantidades de datos que no necesita.

### 7. Optimizar la infraestructura

La ubicación de los servidores también influye en la latencia. Siempre que sea posible, los diferentes componentes del sistema deben estar geográficamente próximos, reduciendo así el tiempo necesario para transmitir los datos entre ellos.

## Conclusión

La clave para conseguir un agente de voz con latencia ultrabaja no consiste únicamente en elegir un modelo rápido. Es necesario optimizar todo el flujo:

`Usuario → VAD → STT → LLM → TTS → Usuario`

Cada milisegundo añadido por uno de estos componentes aumenta la latencia total. Por ello, la estrategia más eficaz consiste en optimizar cada etapa, utilizar procesamiento en streaming y eliminar todas las esperas innecesarias.
