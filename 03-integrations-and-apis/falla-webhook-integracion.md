# [KB-0301] Falla en la sincronización de datos mediante Webhooks (Integración de Terceros)

## 🔍 1. Síntoma (Reporte del Usuario)
Un cliente corporativo reporta que tras procesar un pago exitoso en su pasarela de pagos externa (Stripe), el sistema de nuestra plataforma SaaS no actualiza automáticamente el estado de su factura a "Pagada", obligándolos a realizar la activación de créditos de forma manual.

---

## 🛠️ 2. Diagnóstico Técnico (Área de Soporte L2)
Al inspeccionar el historial de eventos en el panel de integraciones y revisar las herramientas de logs del servidor, se identifica lo siguiente:
1. **Código de Respuesta HTTP:** El servidor de la plataforma SaaS está retornando un error `500 Internal Server Error` ante las peticiones de Stripe.
2. **Análisis de la Carga Útil (Payload JSON):** Se observa que el JSON enviado por Stripe incluye un nuevo campo en su API de producción que nuestro servidor no esperaba procesar, causando una excepción no controlada en el backend.
3. **Validación:** El webhook receptor falló en la verificación de la firma de seguridad (`Stripe-Signature`), interpretando la petición válida como un acceso no autorizado debido a un cambio en la clave secreta (*Webhook Secret*) por parte del cliente.

---

## ✅ 3. Pasos para la Solución (Workaround)
1. **Acción Manual de Contingencia:** El agente de soporte valida el ID del pago en Stripe y, tras comprobar el éxito de la transacción, actualiza manualmente el estado de la cuenta en nuestra base de datos interna usando:
   `UPDATE invoices SET status = 'paid' WHERE stripe_charge_id = 'ch_3MxsK2L...';`
2. **Acción para el Cliente:** Solicitar al administrador de sistemas del cliente que acceda a su panel de Stripe y verifique si la *Signing Secret* coincide con la configurada en nuestra plataforma.
3. **Escalamiento:** Enviar el fragmento del Payload JSON que causó la ruptura al equipo de Desarrollo/API para que modifiquen el validador del backend y acepte los nuevos campos de forma segura sin arrojar un error general.

---

## ✉️ 4. Plantilla de Respuesta Sugerida al Cliente (Macro)

> **Asunto:** Solución e investigación de incidencia con integración de pagos - Ticket #[Número]
>
> Hola [Nombre del Administrador],
>
> Espero que te encuentres muy bien. Te escribo para notificarte que hemos solucionado el estado de las facturas pendientes de forma manual en tu cuenta para que tu equipo pueda seguir operando sin interrupciones.
>
> Respecto a la falla de sincronización automática, nuestro equipo de soporte de segundo nivel (L2) identificó que el servidor está rechazando las alertas automáticas debido a un desajuste en la clave de firma de seguridad (*Signing Secret*) del webhook en tu panel de Stripe. Esto causa que el sistema rechace el aviso de pago por motivos de protección de datos.
>
> **Para reactivar la sincronización automática, agradecemos que tu equipo de sistemas verifique lo siguiente:**
> 1. Ingresen a su cuenta de Stripe > *Developers* > *Webhooks*.
> 2. Revisen que la URL del endpoint apunte correctamente a nuestra plataforma actual.
> 3. Asegúrense de que la clave *Signing Secret* coincida exactamente con la que tienen guardada en los ajustes de integración dentro de nuestra plataforma.
>
> Adjunto la documentación técnica de nuestra API por si tus desarrolladores necesitan revisar la estructura requerida. Quedamos atentos a tu confirmación una vez validado este paso.
>
> Cordialmente,
> **Eward Fuentes**  
> Technical Support Engineer - L2 Team
