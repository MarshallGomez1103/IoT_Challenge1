# Conversación: wiki de WMS - Water Monitoring System

> Exportación ordenada de la conversación de trabajo. Se preserva el contenido sustantivo; se omiten muletillas y repeticiones propias del reconocimiento de voz.

## Inicio

**Usuario:** Quiero hacer una asignación de Internet de las Cosas. Tengo un prototipo y debo crear una wiki. Primero quiero compartir lo que pide la asignación y ejemplos de wikis anteriores para construir una estructura; después quiero responder preguntas sobre lo que hemos hecho.

**Asistente:** Se revisaron el enunciado, la rúbrica, el artículo de referencia y el código del ESP32. El prototipo se ajusta al reto por usar ESP32, sensores ambientales, medición de distancia para nivel de agua, LCD y alertas físicas sin comunicaciones convencionales.

**Usuario:** El proyecto está en `IoT Project` en el Escritorio. No usamos Bluetooth. La IA se utilizó como apoyo iterativo para el código y para organizar la documentación, pero el equipo valida el prototipo.

**Asistente:** El enunciado exige una wiki con diseño, implementación, validación, autoevaluación, conclusiones, retos, trabajo futuro, anexos, video y transparencia de IA. La rúbrica valora restricciones, arquitectura, diagramas, pruebas, comunicación y contribuciones.

**Usuario:** El ultrasónico está sobre un recipiente. Si el agua baja, aumenta la distancia al agua; al superar el umbral debe aparecer alerta en LCD, alternar rojo y azul en el LED RGB y sonar el buzzer. El umbral y las dimensiones se definirán después.

**Asistente:** Se identificó que la lógica final de escasez debe activarse cuando la distancia aumente. El código vigente todavía usa una condición de proximidad, por lo que esa última iteración debe realizarse antes de las pruebas finales.

## Referencias de estructura revisadas

- https://github.com/MarshallGomez1103/Anita/wiki
- https://github.com/MarshallGomez1103/Proyecto2/wiki
- https://github.com/MarshallGomez1103/ProyectoFinal/wiki

**Asistente:** Las referencias usan Home, índice, páginas separadas de desarrollo, resultados, retos, roles y referencias. Se acordó conservar ese estilo y cubrir todos los requisitos de la entrega.

## Nombre y objetivo acordados

**Usuario:** El sistema se llamará WMS, Water Monitoring System. El objetivo debe incluir que no usa Bluetooth, LoRa ni redes convencionales.

**Asistente:**

> Diseñar y validar un prototipo funcional de bajo costo, denominado WMS - Water Monitoring System, que monitoree el nivel de agua y variables ambientales para generar alertas visuales y sonoras in situ ante posibles escenarios de escasez, sin utilizar redes de comunicación convencionales.

## Estructura acordada de la wiki

1. Home
2. Contexto del reto y requisitos
3. Equipo, roles y contribuciones
4. Diseño de la solución
5. Diseño e implementación de hardware
6. Diseño e implementación de software
7. Configuración experimental y validación
8. Resultados y análisis
9. Autoevaluación del protocolo de pruebas
10. Retos presentados y conclusiones
11. Uso de inteligencia artificial
12. Referencias y anexos

**Usuario:** La wiki ya se creó manualmente. El usuario construye las páginas en GitHub y el asistente ayuda a redactar cada sección a partir de información real del prototipo. Los datos pendientes deben destacarse con bloques rojos `CAUTION`.

## Contenido elaborado

### Home

```md
# WMS - Water Monitoring System

> Sistema IoT de bajo costo para el monitoreo local del nivel de agua y de variables ambientales.

## Resumen general

WMS - Water Monitoring System es un prototipo funcional orientado al monitoreo de posibles condiciones de escasez de agua. El sistema adquiere información del nivel de agua y de variables ambientales mediante sensores conectados a un ESP32. A partir de estas mediciones, presenta la información en una pantalla LCD y genera alertas visuales y sonoras in situ cuando se identifica una condición crítica.

La solución no utiliza redes de comunicación convencionales. Por ello, la notificación se realiza directamente en el lugar de monitoreo mediante una pantalla LCD, un LED RGB y un buzzer.

## Motivación

La disponibilidad de agua es una preocupación relevante en la región Sabana Centro de Cundinamarca, especialmente ante escenarios de sequía y desabastecimiento. Contar con una solución de bajo costo que permita observar localmente el nivel de agua y las condiciones ambientales puede apoyar la identificación temprana de situaciones que requieren atención.

## Objetivo principal

Diseñar y validar un prototipo funcional de bajo costo, denominado WMS - Water Monitoring System, que monitoree el nivel de agua y variables ambientales para generar alertas visuales y sonoras in situ ante posibles escenarios de escasez, sin utilizar redes de comunicación convencionales.
```

### Contexto del reto y requisitos

**Asistente:** Se propuso separar requisitos funcionales de restricciones de diseño para evidenciar trazabilidad ante la rúbrica: los requisitos indican qué debe hacer WMS y las restricciones bajo qué límites debe hacerlo.

Requisitos funcionales acordados:

- Medir distancia sensor-agua para estimar nivel.
- Adquirir temperatura, humedad, presión y estimación de radiación.
- Procesar lecturas en ESP32.
- Mostrar variables en LCD y permitir navegación con botón.
- Activar LCD, LED RGB y buzzer si la distancia supera el umbral crítico.

Restricciones acordadas:

- ESP32, bajo costo y alertas in situ.
- Sin redes de comunicación convencionales.
- Alimentación estable y conexiones seguras.
- Umbral definido mediante dimensiones reales y validación experimental.

### Equipo, roles y contribuciones

**Usuario:** Integrantes: Thomas Gómez, David Sastre y Andrés Suárez. Thomas participó en planteamiento, ESP32, código, montaje, pruebas y verificación. David y Andrés deben describir sus aportes con sus propias palabras.

**Mensaje acordado para David y Andrés:**

> Para documentar la wiki de WMS, envíenme un párrafo corto explicando qué actividades realizaron en el proyecto: diseño, búsqueda de materiales, montaje, alambrado, código, pruebas, ajustes u otras. Escriban solo lo que realmente hicieron.

### Diseño de la solución

**Asistente:** WMS se diseñó como sistema embebido de monitoreo local: sensores -> ESP32 -> procesamiento -> LCD, LED RGB y buzzer. Los módulos acordados son adquisición de nivel, adquisición ambiental, procesamiento local, lógica de alerta, interfaz local y alertas físicas.

Criterios de diseño:

- Monitoreo local.
- Medición de nivel por ultrasónico.
- Contexto ambiental.
- Alerta comprensible.
- Interacción por botón.
- Modularidad.
- Bajo costo.
- Seguridad eléctrica.

Regla de alerta final acordada:

`Si distancia_sensor_agua > umbral_critico, entonces activar alerta.`

Pendiente: definir usuario objetivo, contexto de uso y forma de implementación del modelo de negocio.

### Diseño e implementación de hardware

Componentes y conexiones documentadas:

| Componente | Conexión |
|---|---|
| HC-SR04 TRIG / ECHO | GPIO 18 / GPIO 19 con divisor de voltaje en ECHO |
| DHT22 DATA | GPIO 27 |
| BMP180 y LCD | I2C GPIO 21 / GPIO 22; LCD `0x27` |
| Fotoresistor | GPIO 34 |
| LED RGB | GPIO 25, 26 y 32; ánodo común |
| Buzzer | GPIO 33; activación `LOW` |
| Botón | GPIO 14 con `INPUT_PULLUP` |

Pendientes acordados:

> [!CAUTION]
> Completar altura interna del recipiente, distancia sensor-fondo, fotografías del montaje, esquemático eléctrico final y fuente de alimentación usada.

Fórmula documentada:

`nivel_estimado = D_sensor_fondo - distancia_sensor_agua`

### Diseño e implementación de software

Funciones reales revisadas: `medirDistancia()`, `actualizarAlerta()`, `apagarAlerta()`, `revisarBoton()`, `mostrarPantalla()`, `setup()` y `loop()`.

Pantallas LCD:

1. Distancia y presión.
2. Temperatura y humedad.
3. Irradiancia estimada.

Pendientes acordados:

> [!CAUTION]
> Ajustar `actualizarAlerta()` para activar la alerta cuando la distancia aumente y supere el umbral crítico. Agregar enlace al código final documentado y validar la estimación de irradiancia.

### Configuración experimental y validación

El protocolo acordado incluye:

- CP-01: encendido inicial.
- CP-02: lectura de variables.
- CP-03: cambio de pantallas.
- CP-04: nivel sin condición crítica.
- CP-05: condición de escasez.
- CP-06: recuperación.

La evidencia final debe incluir fotos, tabla de datos, video demostrativo de máximo cinco minutos y participación de todos los integrantes.

### Resultados y análisis

**Asistente:** La tabla debe registrarse con datos reales de las pruebas: distancia, nivel estimado, variables ambientales, LCD, LED RGB, buzzer y resultado para CP-01 a CP-06.

> [!CAUTION]
> Completar resultados, valor de `umbral_critico`, análisis de cumplimiento de requisitos y enlace al video antes de entregar.

Límite documentado: la irradiancia actual es una estimación configurada con fotoresistor, no una medición certificada.

### Autoevaluación del protocolo de pruebas

La autoevaluación cubre medición de distancia, variables ambientales, navegación LCD, alerta, recuperación y funcionamiento integrado. Debe completarse con las evidencias obtenidas y registrar limitaciones reales observadas.

### Retos presentados y conclusiones

Retos reales comunicados por el usuario:

- Identificación inicial de los sensores.
- Estado físico y disponibilidad de materiales de la universidad.
- Longitud limitada de cables.
- Definición de alimentación del ESP32.
- Coordinación de horarios del equipo.
- Diseño de una maqueta funcional y ordenada.

Alcance logrado: integración de ESP32 con medición de distancia, temperatura, humedad, presión y estimación de radiación; LCD, LED RGB y buzzer para operación local sin redes convencionales.

> [!CAUTION]
> Antes de entregar, complementar conclusiones con los resultados definitivos de validación y limitaciones observadas.

### Uso de inteligencia artificial

Uso acordado:

- Apoyo iterativo para el código a partir de los componentes disponibles y del comportamiento esperado.
- Apoyo de OpenAI Codex para organizar la wiki, requisitos, diagramas y coherencia técnica.
- El equipo conserva las decisiones de diseño, montaje, pruebas, validación y conclusiones.

> [!CAUTION]
> Registrar el nombre exacto de la herramienta usada para el código, pegar enlace compartible de la conversación o transcribir las instrucciones exactas enviadas a la IA. Revisar que el enlace no incluya datos privados.

Validación acordada de IA: revisar coherencia con componentes, niveles eléctricos, cargar código en ESP32, observar el prototipo y ajustar el programa según pruebas.

### Referencias y anexos

Referencias base acordadas:

1. Ministerio de Ambiente y Desarrollo Sostenible, reporte de El Niño, 2026.
2. Caracol Radio, riesgo de desabastecimiento en Cundinamarca, 2026.
3. Infobae, escenarios de riesgo de Bogotá, 2026.
4. Gobernación de Cundinamarca, plan de contingencia del fenómeno de El Niño, 2026.
5. IEEE Author Center Journals, *IEEE Editorial Style Manual*.
6. Wiki Anita.
7. Wiki Proyecto2.
8. Wiki ProyectoFinal.

Anexos acordados:

- Código fuente final documentado.
- Esquemático y diagrama de conexiones.
- Fotografías del montaje y maqueta.
- Registro de pruebas y datos.
- Actas y evidencia de contribuciones.
- Video demostrativo.
- Registro de uso de IA.

> [!CAUTION]
> Añadir las hojas de datos o documentación técnica realmente consultada para ESP32, HC-SR04, DHT22, BMP180, LCD I2C y fotoresistor. Enlazar los anexos finales y comprobar que el video se reproduzca en Teams sin descarga.

## Cierre de la conversación

**Usuario:** La wiki quedó con una estructura clara, con tablas, diagramas Mermaid y pendientes destacados. Las pruebas, el código final, fotografías, video, aportes de los compañeros y enlaces a anexos se completarán antes de la entrega.
