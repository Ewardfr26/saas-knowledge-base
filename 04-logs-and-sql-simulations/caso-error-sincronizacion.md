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
---
🧠 Deducción Técnica del Log:
El backend intentó buscar los datos del usuario con ID 8942.

El error ENOTFOUND mariadb-cluster.internal.net indica que el servidor de la aplicación no pudo resolver la dirección (DNS) de la base de datos. No es un problema del código de la app, es un problema de red interna o de que el clúster de MariaDB está caído.

## 💾 3. Investigación en la Base de Datos (SQL)
Para asegurar que la cuenta del usuario no tenga ninguna anomalía o bloqueo manual previo a la caída de la red, se ejecuta la siguiente consulta en el servidor de respaldo:
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
