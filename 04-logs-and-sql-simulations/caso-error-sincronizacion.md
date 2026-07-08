# 📊 Caso de Estudio: Diagnóstico mediante Logs del Servidor y Consultas SQL
Este documento simula un escenario de soporte técnico de nivel intermedio (L1.5 / L2), donde se investiga una falla reportada por un usuario analizando directamente los registros del backend y ejecutando consultas en la base de datos relacional.

---
## 🛑 1. El Reporte del Cliente
*   **Usuario:** `corporativo_asociado@empresa.com`
*   **Mensaje del Ticket:** *"Desde esta mañana, al entrar al módulo de historial de transacciones, la pantalla se queda en blanco o muestra un mensaje de 'Error de conexión 502'. Necesitamos exportar el reporte urgente para contabilidad."*
---
## 📑 2. Análisis del Log del Servidor (Backend Error Log)
Al ingresar al servidor a través de la terminal y revisar el archivo de registros del backend (`/var/log/node-app/error.log`), se identifica el siguiente bloque repetido en la hora exacta del reporte del cliente:

```text
[2026-07-07T14:22:11.402Z] ERROR (API-Gateway): Failed to fetch transaction data for user_id: 8942
[2026-07-07T14:22:11.405Z] ERROR (Database-Connector): Connection timeout after 5000ms
    at Pool.getConnection (/app/node_modules/mariadb/lib/pool.js:42:15)
    at Error: ENOTFOUND mariadb-cluster.internal.net
    Cause: getaddrinfo ENOTFOUND mariadb-cluster.internal.net
```
### 🧠 Deducción Técnica del Log: ###
El backend intentó buscar los datos del usuario con ID 8942.
El error ENOTFOUND mariadb-cluster.internal.net indica que el servidor de la aplicación no pudo resolver la dirección (DNS) de la base de datos. No es un problema del código de la app, es un problema de red interna o de que el clúster de MariaDB está caído.

---
## 💾 3. Investigación en la Base de Datos (SQL)
Para asegurar que la cuenta del usuario no tenga ninguna anomalía o bloqueo manual previo a la caída de la red, se ejecuta la siguiente consulta en el servidor de respaldo:
```SQL
SELECT 
    u.id AS user_id, 
    u.email, 
    u.status AS account_status, 
    s.subscription_plan, 
    COUNT(t.id) AS total_transactions
FROM users u
LEFT JOIN subscriptions s ON u.id = s.user_id
LEFT JOIN transactions t ON u.id = t.user_id
WHERE u.email = 'corporativo_asociado@empresa.com'
GROUP BY u.id, s.subscription_plan;
```
### 📊 Resultado de la consulta (Mockup de Consola): ###
```
user_id: 8942

email: corporativo_asociado@empresa.com

account_status: ACTIVE

subscription_plan: ENTERPRISE

total_transactions: 14,205
````
### 🧠 Deducción de los Datos: ###
La cuenta del usuario está activa y pertenece al plan Enterprise (alta prioridad). Tiene más de 14,000 transacciones, lo que descarta que sea un usuario nuevo sin datos. El problema es 100% la caída de la conexión reflejada en el log.

---
## ✅ 4. Resolución y Escalamiento ##
*   **Acción Interna:** Se escala el caso con prioridad ALTA al equipo de DevOps/SysAdmins adjuntando el error ENOTFOUND mariadb-cluster.internal.net, indicando que el clúster interno de MariaDB en AWS no está respondiendo.
*   **Solución:** DevOps reinicia el servicio DNS interno de la VPC. El sistema vuelve a la normalidad.

---
## ✉️ 5. Respuesta Final Enviada al Cliente ##

> **Asunto:** Solución a su inconveniente con el historial de transacciones - Ticket #[Número]
>
> Hola [Nombre del Cliente],
>
>Te escribo para informarte que el acceso a tu historial de transacciones ha sido completamente restablecido y ya puedes exportar tus reportes contables con normalidad.
> 
> Nuestro equipo técnico identificó una interrupción temporal en la comunicación de nuestra red interna que conectaba el panel con el servidor de almacenamiento. No se trató de un fallo en tu cuenta ni hubo pérdida de datos; tu información (las 14,205 transacciones registradas) se encuentra intacta y segura. Lamentamos profundamente los inconvenientes que este retraso haya causado en tu jornada de hoy.
> 
> Quedo a tu disposición si necesitas asistencia con cualquier otro módulo de la plataforma.
> 
> Saludos cordiales,
> 
> **Eward Fuentes**  
> Especialista de Soporte Técnico - SaaS Team
