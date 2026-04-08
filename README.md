# GuardianZMG

## Descripción

`GuardianZMG` es un flujo de trabajo de n8n diseñado como asistente de movilidad inteligente para la Zona Metropolitana de Guadalajara (ZMG). El flujo recibe mensajes desde Telegram, clasifica la intención del usuario y, cuando se solicita una ruta, genera un plan de viaje personalizado usando:

- Modelo de lenguaje Google Gemini
- API de Google Maps para rutas de transporte público
- Datos de clima de OpenWeatherMap
- Información de tráfico en tiempo real via RSS
- Datos de estaciones de MiBici
- Cálculo de tarifa oficial del transporte público de la ZMG

El resultado es un reporte de viaje estructurado y orientado a usuarios de Guadalajara, Zapopan, Tlaquepaque y Tonalá.
<img width="1722" height="876" alt="image" src="https://github.com/user-attachments/assets/0b5184f4-24a5-4074-bdf5-01d8c64da2da" />


## Arquitectura del flujo

1. `Telegram Trigger`
   - Dispara el flujo cuando se recibe un mensaje en el bot de Telegram.
<img width="1866" height="514" alt="image" src="https://github.com/user-attachments/assets/6444e2f3-ad66-4bd4-b540-0ecb86e76982" />

2. `Basic LLM Chain`
   - Clasifica la intención del mensaje en `RUTA` o `CHARLA`.
3. `If`
   - Si la intención es `RUTA`, procesa la solicitud con el agente de IA.
   - Si es `CHARLA`, responde con un mensaje de saludo y guía.
<img width="1154" height="516" alt="image" src="https://github.com/user-attachments/assets/c414a78b-ad7e-43f4-a82b-2be880534bb3" />

4. `AI Agent`
   - Orquesta las herramientas y genera la respuesta final en texto.
5. Herramientas de apoyo:
   - `Google Gemini Chat Model`: modelo de lenguaje.
   - `HTTP Request`: consulta Google Maps Directions API.
   - `obtener_clima`: consulta OpenWeatherMap.
   - `TraficoZMG`: consulta RSS de incidentes y tráfico.
   - `MiBici`: consulta estado de estaciones de bicicletas públicas.
   - `calculadora_tarifas`: calcula el costo total de la ruta.
6. `Send a text message`
   - Envía la respuesta final al usuario de Telegram.

## Nodos principales

- **Telegram Trigger**: recibe mensajes entrantes desde Telegram.
- **Basic LLM Chain**: clasifica si el usuario quiere una ruta o simplemente chatear.
- **If**: dirige el flujo entre generación de ruta y mensaje de bienvenida.
- **AI Agent**: circuito principal que integra múltiples herramientas y aplica reglas de negocio.
- **Google Gemini Chat Model**: motor de lenguaje para interpretar y generar texto.
- **HTTP Request**: realiza la consulta a Google Maps Directions API en modo `transit`.
- **obtener_clima**: obtiene condiciones meteorológicas de Guadalajara.
- **calculadora_tarifas**: calcula el costo usando la tarifa oficial de $11.00 MXN.
- **TraficoZMG**: obtiene información de tráfico desde un feed RSS.
- **MiBici**: obtiene datos de la red de bicicletas públicas.
- **Send a text message**: entrega resultado con teclado inline en Telegram.

## Requisitos

- n8n instalado y funcionando.
- Cuenta y credenciales válidas para:
  - Telegram Bot API
  - Google Gemini / PaLM API
  - OpenWeatherMap API
  - Google Maps Directions API
- Acceso a Internet para servicios externos.

## Configuración de credenciales

Asegúrate de configurar las siguientes credenciales en n8n:

- `Telegram account`
- `Google Gemini(PaLM) Api account 2`
- `OpenWeatherMap account`

También es recomendable mover cualquier clave sensible de `HTTP Request` a credenciales seguras o variables de entorno. En el flujo actual se usa una clave de Google Maps embebida en el nodo `HTTP Request`.

## Cómo importar el flujo

1. Abre n8n.
2. Ve a `Workflows` > `Import`.
3. Selecciona el archivo `GuardianZMG.json`.
4. Guarda el workflow importado.
5. Asocia las credenciales necesarias a cada nodo.

## Uso

- El bot se activa con cualquier mensaje entrante de Telegram.
- Si el mensaje contiene una solicitud de ruta, el bot solicitará origen y destino.
- El flujo responde con una guía de transporte público que incluye:
  - ruta estimada
  - tiempo
  - costo total
  - recomendaciones de MiBici si aplica
  - alerta de seguridad y condiciones del clima

## Buenas prácticas

- Revisa periódicamente las credenciales y actualiza las claves expiradas.
- Comprueba que el feed RSS de `TraficoZMG` siga activo.
- Valida que la API de Google Maps use facturación activa y cuota suficiente.
- Mantén el bot de Telegram en modo `active` sólo si se requiere producción.

## Notas técnicas

- El cálculo de tarifas está definido en `calculadora_tarifas` con tarifa fija de `11.00 MXN` por unidad.
- El agente de IA usa un prompt estricto para protegerse contra alucinaciones y garantizar formato de salida controlado.
- La respuesta enviada al usuario usa `replyMarkup` con un botón inline que enlaza a rutas de Moovit para Guadalajara.

## Contacto

Para mantenimiento o mejoras de este flujo, revisa la configuración de los nodos `AI Agent` y `Basic LLM Chain` en n8n, ya que contienen la lógica de intención y el prompt principal del asistente.
