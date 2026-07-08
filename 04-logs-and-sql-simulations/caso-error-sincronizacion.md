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
