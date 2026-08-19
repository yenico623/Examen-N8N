# ☕ DeliveryChat - Bot de Automatización de Pedidos para Cafetería

¡Bienvenido/a a **DeliveryChat**! Este proyecto es un sistema conversacional interactivo para la gestión de pedidos de cafetería en tiempo real, desarrollado con **n8n**, **Telegram Bot API** y **Google Sheets** como base de datos de sesiones y transacciones.

---

## 🚀 Características Principales

* 🤖 **Flujo Conversacional Interactivo:** Navegación por catálogo de productos y categorías mediante teclados interactivos (`inline_keyboard`) en Telegram.
* ⏰ **Control Dinámico de Horarios de Atención (Nueva Funcionalidad):** Bloqueo automático del flujo de compra fuera de la jornada laboral (Lunes a Viernes, de 8:00 AM a 5:00 PM), manteniendo disponible la consulta del menú y la navegación.
* 🛒 **Gestión de Carrito y Totales:** Cálculo automático de Subtotal, **IVA (19%)**, costo de domicilio y **Total a Pagar**, además de contabilizar puntos de fidelización.
* 📦 **Captura Dinámica de Datos:** Máquina de estados para recolección progresiva de Método de Pago, Tipo de Entrega, Dirección, Teléfono y Correo Electrónico.
* 📊 **Persistencia de Datos:** Integración con Google Sheets dividida en pestañas relacionales para `SESSIONS` y `PEDIDOS`.
* 🔔 **Notificaciones Push en Tiempo Real:** Alertas automáticas al cliente cuando el administrador actualiza el estado de la orden (`EN PREPARACION`, `EN CAMINO`, `ENTREGADO`).

---

## 🛠️ Tecnologías Utilizadas

* **n8n:** Motor y orquestador de flujos de trabajo orientados a eventos (*Event-Driven Architecture*).
* **Telegram Bot API:** Interfaz conversacional y canal de atención al cliente.
* **Google Sheets API:** Base de datos para persistencia de sesiones y registro histórico de ventas.
* **JavaScript (Node.js):** Lógica personalizada en nodos `Code` para validación de horarios, cálculo de impuesto/precios y manipulación de estructuras JSON.

---

## 📊 Estructura de la Base de Datos (Google Sheets)

### 1. Pestaña `SESSIONS`
Mantiene el estado de la máquina de estados y el carrito de compras en tiempo real.

| Columna | Descripción |
| :--- | :--- |
| `telegram_id` | Identificador único del usuario en Telegram |
| `nombre` | Nombre registrado del cliente |
| `pantalla_actual` | Estado actual de la sesión (ej: `INICIO`, `ESPERANDO_DIRECCION`) |
| `carrito_temporal` | Estructura JSON con los productos seleccionados y cantidades |
| `ultimo_cambio` | Timestamp del último evento |

### 2. Pestaña `PEDIDOS`
Almacena el historial consolidado de las órdenes confirmadas.

| Columna | Descripción |
| :--- | :--- |
| `id_pedido` | Código de orden único (ej: `PED-8492`) |
| `id_usuario` | Telegram ID del cliente |
| `detalles_pedido` | Objeto JSON completo con los productos y datos de contacto |
| `subtotal` / `impuesto` / `total_pago` | Liquidación económica detallada de la compra |
| `estado` | Estado de la orden (`RECIBIDO`, `EN PREPARACION`, `EN CAMINO`, `ENTREGADO`) |

---

## ⏰ Regla de Negocio: Restricción de Horario - EXAMEN #1

El sistema implementa una validación dinámica en el momento en que el usuario intenta **Confirmar un Pedido**:

```text
[ Confirmar Pedido ] ──► [ Code: VALIDAR_HORARIO ] ──► [ IF: esta_abierto ]
                                                             │
                                   ┌─────────────────────────┴─────────────────────────┐
                                   ▼ (TRUE)                                            ▼ (FALSE)
                      [ Continúa a Datos de Envío ]                         [ 🌙 Enviar Mensaje de Cierre ]
