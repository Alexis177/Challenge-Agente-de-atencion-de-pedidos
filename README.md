# 🛍️ Agente de atención a pedidos por WhatsApp

Asistente conversacional para una tienda en línea, desarrollado en Python con Llama 3.1, Ollama, Llama Stack y WhatsApp Cloud API.

Proyecto del challenge **Integración entre Llama y WhatsApp**.

## Objetivo

Consultar el estado de pedidos y responder preguntas frecuentes mediante herramientas de function calling, manteniendo el contexto de la conversación por número de teléfono.

## Funcionalidades

- Consulta del estado de pedidos.
- Respuestas sobre envíos, devoluciones, pagos y facturas.
- Memoria de conversación mediante Llama Stack.
- Respuesta fija de prioridad ante problemas con cargos.
- Respuesta fija para solicitudes de atención humana.
- Saludo automático.
- Procesamiento en segundo plano.
- Lock por número para evitar llamadas simultáneas sobre una conversación.
- Detección de mensajes duplicados mediante `wamid`.
- Registro de errores de Llama Stack, Ollama y WhatsApp.

## Tecnologías

| Tecnología | Función |
|---|---|
| Python | Lógica del asistente |
| Google Colab | Entorno de ejecución |
| Ollama | Ejecución del modelo |
| Llama 3.1 8B | Generación de respuestas y solicitudes de herramientas |
| Llama Stack | Inferencia y almacenamiento de conversaciones |
| FastAPI | Recepción de solicitudes del webhook |
| Uvicorn | Servidor HTTP |
| WhatsApp Cloud API | Comunicación con WhatsApp |
| ngrok | Exposición pública del servidor |

## Requisitos

- Google Colab, preferentemente con GPU.
- Una aplicación de Meta configurada para WhatsApp Cloud API.
- Un número destinatario autorizado para las pruebas.
- Una cuenta de ngrok.
- Una versión de Llama Stack compatible con las APIs utilizadas en el notebook.

## Credenciales

Configurar los siguientes valores en **Colab Secrets** y permitir su acceso desde el notebook:

| Secret | Descripción |
|---|---|
| `WHATSAPP_ACCESS_TOKEN` | Token de acceso a WhatsApp |
| `PHONE_NUMBER_ID` | Identificador del número de WhatsApp |
| `WABA_TOKEN_ID` | Identificador de la cuenta de WhatsApp Business |
| `NGROK_AUTHTOKEN` | Token de autenticación de ngrok |

El notebook carga `WABA_TOKEN_ID` en la variable `WABA_ID`, aunque el flujo mostrado no utiliza esta variable.

Definir también `VERIFY_TOKEN` en el código y utilizar el mismo valor al configurar el webhook en Meta.

No publicar credenciales en el repositorio.

## Ejecución

1. Abrir el notebook en Google Colab.
2. Instalar Ollama e iniciar su servidor.
3. Descargar el modelo `llama3.1:8b`.
4. Instalar Llama Stack y registrar el modelo de Ollama.
5. Iniciar Llama Stack y cargar las credenciales.
6. Ejecutar las celdas de memoria y herramientas.
7. Ejecutar la generación de respuestas, los locks y las reglas fijas.
8. Definir los endpoints y la función de envío.
9. Iniciar Uvicorn y ngrok.
10. Configurar en Meta la URL pública terminada en `/webhook`.
11. Verificar el webhook y suscribir la aplicación a los mensajes.
12. Enviar los mensajes de prueba desde WhatsApp.

Evitar iniciar dos veces los servidores del ejemplo y del challenge en los mismos puertos. Se recomienda ejecutar el challenge desde una sesión limpia.

## Herramientas de function calling

### Consultar pedido

La función `consultar_pedido(numero_pedido)` busca el pedido en el diccionario `PEDIDOS`.

| Pedido de ejemplo | Estado |
|---|---|
| `1001` | En preparación |
| `1002` | Enviado y en camino |
| `1003` | Entregado |

Si falta el número, la función lo solicita. Si no existe, informa que no se encontró.

### Buscar pregunta frecuente

La función `buscar_pregunta_frecuente(pregunta)` busca coincidencias de texto en `PREGUNTAS_FRECUENTES`.

Incluye respuestas sobre:

- Envíos.
- Devoluciones.
- Formas de pago.
- Facturas.

Los pedidos y las políticas son datos de ejemplo definidos en el código. No existe una conexión con una tienda o base de datos externa.

## Flujo de procesamiento

```text
Mensaje de WhatsApp
        ↓
Webhook de FastAPI
        ↓
Comprobación de wamid duplicado
        ↓
Procesamiento en segundo plano
        ↓
Lock por número de teléfono
        ↓
Evaluación de reglas fijas
        ├── Problema con un cargo → Mensaje de prioridad
        ├── Atención humana → Mensaje de contacto
        ├── Solo saludo → Presentación del asistente
        └── Otra consulta → Llama Stack
                                  ↓
                         Solicitud de herramienta
                                  ↓
                         Ejecución de la función
                                  ↓
                         Respuesta con el resultado
        ↓
Envío por WhatsApp
```

El código permite hasta tres rondas de ejecución de herramientas antes de registrar un error si el modelo sigue solicitándolas.
