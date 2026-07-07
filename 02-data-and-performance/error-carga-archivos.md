# [KB-0201] Error al cargar archivos pesados en la plataforma (Módulo de Almacenamiento)

## 🔍 1. Síntoma (Reporte del Usuario)
El cliente abre un ticket indicando que, al intentar arrastrar y soltar (Drag & Drop) un archivo en la zona de carga, la interfaz se queda congelada con un spinner de carga infinito o muestra un mensaje genérico de "Error en el servidor".

---

## 🛠️ 2. Diagnóstico Técnico (Área de Soporte L1/L2)
Al replicar el incidente o revisar las herramientas de desarrollador en el navegador (F12 -> pestaña Network), se identifican los siguientes puntos:
1. **Código de Estado HTTP:** El servidor backend responde con un error `400 Bad Request` o `413 Payload Too Large`.
2. **Origen del problema:** El middleware del servidor (ej. Multer) está configurado con un límite estricto de tamaño de archivo (ej. 10MB), y el usuario está intentando subir un archivo que supera esta capacidad.
3. **Validación en Base de Datos:** Al ejecutar `SELECT user_tier FROM users WHERE email = 'cliente@email.com';`, se confirma que el usuario se encuentra en el "Plan Gratuito", el cual restringe el tamaño de almacenamiento.

---

## ✅ 3. Pasos para la Solución (Workaround)
Para resolver la solicitud del cliente de inmediato, sigue estos pasos:
1. **Validación del archivo:** Solicita al cliente verificar el peso del archivo que intenta subir.
2. **Solución alternativa:** Si el archivo supera los 10MB, sugiera comprimir el archivo (ZIP/PDF optimizado) o, si corresponde, guiarlo al módulo de facturación para actualizar su plan a "Premium" (que permite hasta 100MB).
3. **Escalamiento (Opcional):** Si el archivo mide menos de 10MB y sigue fallando, escala el ticket al equipo de Backend (L3) adjuntando el ID del usuario y el log exacto del error.

---

## ✉️ 4. Plantilla de Respuesta Sugerida al Cliente (Macro)

> **Asunto:** Actualización sobre su inconveniente con la carga de archivos - Ticket #[Número]
>
> Hola [Nombre del Cliente],
>
> Espero que te encuentres muy bien. Te escribo en relación al inconveniente que reportaste al intentar subir tus archivos a la plataforma.
>
> Tras revisar el estado de tu cuenta y los registros del sistema, identificamos que el archivo que intentas cargar supera el límite de 10 megabytes (MB) permitido para tu plan actual. Es por esto que la plataforma interrumpe la carga de forma automática por motivos de seguridad y rendimiento.
>
> **Para solucionarlo de inmediato, tienes dos opciones muy sencillas:**
> 1. Reducir el tamaño del archivo (comprimiéndolo en un formato .zip o reduciendo su resolución).
> 2. Si manejas archivos pesados de forma habitual, puedes expandir tu capacidad hasta 100MB por archivo actualizando tu suscripción en la sección *Ajustes > Facturación*.
>
> Quedo a tu total disposición si necesitas ayuda adicional durante el proceso. ¡Que tengas un excelente día!
>
> Saludos cordiales,
> **Eward Fuentes**  
> Especialista de Soporte Técnico - SaaS Team
