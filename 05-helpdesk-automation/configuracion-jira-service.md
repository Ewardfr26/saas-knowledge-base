# 🛠️ Configuración y Automatización de Entorno Help Desk (Jira Service Management)

Este documento detalla la arquitectura, el diseño de flujos de trabajo (Workflows) y las reglas de automatización implementadas en un entorno de pruebas (Sandbox) de Jira Service Management, optimizado para la atención de incidentes en una plataforma SaaS.

---

## ⚙️ 1. Flujo de Estados del Ticket (Workflow)
Para evitar retrasos en las respuestas, se diseñó un flujo de trabajo simplificado de 5 estados por los que transita cada solicitud de soporte:

* **Abierto (Open):** El ticket ha sido creado por el usuario desde el portal o vía correo electrónico.
* **En Investigación (In Progress):** Un agente de soporte ha tomado el caso y está diagnosticando la causa raíz.
* **Pendiente del Cliente (Pending Customer):** Se solicitó más información al usuario (ej. capturas de pantalla, logs). El tiempo de SLA de resolución se pausa automáticamente.
* **Escalado (Escalated):** El caso requiere intervención de Ingeniería o DevOps (Soporte L3).
* **Resuelto (Resolved):** El problema fue solucionado y se envió la confirmación de cierre al usuario.

---

## 🤖 2. Reglas de Automatización Implementadas (Triggers & Actions)
Se configuraron tres reglas críticas mediante el motor de automatización nativo de Jira para reducir el trabajo manual del equipo:

### Regla A: Priorización automática por palabras clave (SLA Triage)
* **Disparador (Trigger):** Al crearse un nuevo ticket.
* **Condición:** Si el *Asunto* o la *Descripción* contiene términos críticos como: `"error de pago"`, `"no puedo ingresar"`, `"caído"`, o `"seguridad"`.
* **Acción:** Cambiar la prioridad del ticket automáticamente a **Alta / Crítica** y asignar el componente `Urgente - L2`.

### Regla B: Alerta de inactividad del cliente (Auto-Close)
* **Disparador:** Transcurridos 3 días hábiles en estado *Pendiente del Cliente*.
* **Acción:** Enviar un recordatorio por correo electrónico automático solicitando la información. Si pasan 2 días más sin respuesta, el sistema cambia el estado a **Resuelto** bajo la categoría *"Cerrado por inactividad"*.

### Regla C: Respuestas Preestablecidas (Macros de un Clic)
Se estructuró una librería de respuestas rápidas para incidentes comunes (Caché del navegador, reseteo de contraseñas y webhooks desactualizados) que los agentes pueden inyectar al ticket con un solo atajo de teclado, reduciendo el tiempo de respuesta inicial (FRT) en un 40%.

---

## 📊 3. Métricas de Éxito Definidas (SLAs)
Se establecieron dos acuerdos de nivel de servicio (SLAs) estrictos basados en las mejores prácticas de la industria de software:

1. **Tiempo de Primera Respuesta (Time to First Response):**
   * Tickets Críticos/Altos: Menos de 30 minutos.
   * Tickets Medios/Bajos: Menos de 2 horas.
2. **Tiempo de Resolución Completa (Time to Resolution):**
   * Tickets Críticos: Menos de 4 horas.
   * Tickets Generales: Menos de 24 horas.

---

## 📸 4. Evidencia del Panel (Ecosistema Operativo)
*(Nota: Para complementar este portafolio en una entrevista real, se configuran las vistas y colas de tickets personalizadas en Jira, organizadas por: "Mis asignados", "Urgentes por vencer" y "Esperando Terceros").*
