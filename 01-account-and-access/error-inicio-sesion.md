# [KB-0101] Falla en el inicio de sesión mediante Terceros (OAuth de Google / GitHub)

## 🔍 1. Síntoma (Reporte del Usuario)
El cliente reporta que al hacer clic en el botón "Iniciar sesión con Google", la pantalla parpadea, regresa al formulario de login original y muestra un banner rojo con el mensaje: *"Error de autenticación. Por favor, intente más tarde o contacte a soporte"*. El usuario afirma que sus credenciales de Google son correctas.

---

## 🛠️ 2. Diagnóstico Técnico (Área de Soporte L1/L2)
Tras analizar el flujo y solicitar al usuario una captura de la consola del navegador (F12 -> Console), se detecta lo siguiente:
1. **Error en Consola:** `AuthError: redirect_uri_mismatch`.
2. **Origen del problema:** La URL desde la que el usuario intenta ingresar (ej. un subdominio nuevo de la empresa o una versión de desarrollo regional) no está registrada en la lista de "Orígenes autorizados" dentro de la consola de credenciales de la API de Google (Google Cloud Console). El token JWT no puede ser emitido debido a esta discrepancia de seguridad.
3. **Verificación de cuenta:** El estado del usuario en la base de datos interna (`SELECT status, login_method FROM users WHERE email = 'usuario@email.com';`) muestra que la cuenta está activa, pero su método original de registro fue mediante correo/contraseña clásica, no por OAuth.

---

## ✅ 3. Pasos para la Solución (Workaround)
1. **Acción Inmediata para el Cliente:** Indicar al usuario que intente ingresar utilizando el método tradicional de correo electrónico y contraseña. Si no recuerda su clave, pedirle que use el flujo de "Olvidé mi contraseña" para unificar sus accesos.
2. **Solución Definitiva (Escalamiento a DevOps/SysAdmin):** Reportar de inmediato al equipo de Infraestructura el subdominio exacto que causó el `redirect_uri_mismatch` para que sea añadido a la lista blanca de la consola de Google Cloud de la empresa.

---

## ✉️ 4. Plantilla de Respuesta Sugerida al Cliente (Macro)

> **Asunto:** Actualización sobre su inconveniente para iniciar sesión - Ticket #[Número]
>
> Hola [Nombre del Cliente],
>
> Espero que estés teniendo un buen día. Te escribo para darte el diagnóstico sobre el inconveniente que experimentaste al intentar ingresar con el botón de Google.
>
> Hemos detectado que el sistema de seguridad bloqueó temporalmente el acceso debido a una discrepancia en la URL de redirección desde la que intentabas conectar. Nuestro equipo de ingeniería ya está trabajando en actualizar los permisos para que puedas usar este botón sin problemas en las próximas horas.
>
> **Para que puedas ingresar a trabajar en la plataforma de inmediato, te sugiero seguir estos pasos:**
> 1. Regresa a la página de inicio de sesión estándar.
> 2. Introduce tu correo electrónico y la contraseña que configuraste al crear tu cuenta.
> 3. En caso de que no recuerdes tu contraseña clásica, haz clic en el enlace **"¿Olvidó su contraseña?"** para recibir un enlace de acceso directo en tu correo.
>
> Lamento el inconveniente que esto te causa. Por favor, confírmame si lograste ingresar con este método alternativo.
>
> Atentamente,
> **Eward Fuentes**  
> Especialista de Soporte Técnico - SaaS Team
