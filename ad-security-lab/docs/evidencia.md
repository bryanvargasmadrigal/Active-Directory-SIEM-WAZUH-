# Evidencia

Índice de todas las capturas del proyecto, organizadas por carpeta. Los archivos deben colocarse en `screenshots/` respetando esta estructura para que los links del resto de la documentación funcionen.

## `screenshots/architecture/`

**`topology.png`**
Diagrama de red completo: dos laptops físicas (Lab-1 y Lab-2) conectadas al mismo router. Lab-1 corre Windows Server (AD) y Windows 10 sobre red interna compartida; Lab-2 corre el Ubuntu con Wazuh. Se detallan los adaptadores de cada VM (interna, Bridge, NAT).

## `screenshots/setup/`

**`manager-timeout-reset.png`**
Secuencia completa: `wazuh-apid is running`, seguido de `sudo systemctl reset-failed wazuh-manager` y el status quedando en `inactive (dead)` tras el reset — el punto de partida antes de confirmar que el servicio volvía a levantar limpio.

**`dashboard-active.png`**
`systemctl status wazuh-dashboard` mostrando `active (running)`, con el log de `opensearch-dashboards` corriendo sin errores tras el incidente del crash loop.

**`indexer-active.png`**
`systemctl status wazuh-indexer` en `active (running)`, confirmando que el tercer componente del stack (manager, dashboard, indexer) está operativo.

**`agent-control-list.png`**
`sudo /var/ossec/bin/agent_control -l` mostrando dos agentes: el `000` local del propio manager, y el `002` correspondiente al DC (`wazuh`), en estado `Active`.

**`overview-agents-summary.png`**
Vista general del dashboard: "Agents Summary" con 1 activo y 0 desconectados, más el resumen de alertas de las últimas 24 horas (296 de severidad media, 591 de severidad baja).

## `screenshots/detections/`

**`auth-failure-first-hit.png`**
Primera detección de fallo de autenticación en Threat Hunting: 1 hit bajo "Authentication failure", con el desglose por grupo de reglas (`authentication_failed`, `windows`, `windows_security`).

**`auth-failure-two-hits.png`**
Segunda ronda de pruebas: 2 hits confirmados de la regla **60122** ("Logon Failure - Unknown user or bad password"), nivel 5, separados en el tiempo dentro del histograma — evidencia de que cada intento de login fallido se procesó como evento independiente.

**`ad-users-testuser01.png`**
Active Directory Users and Computers mostrando `TestUser01` creado dentro de la OU `Users` del dominio `lab.local`, como confirmación visual directa en el DC del evento que después se buscó en Wazuh.

**`server-manager-addsdns.png`**
Panel del Administrador del servidor mostrando los roles AD DS y DNS ya instalados y con estado activo en el Domain Controller.

## Cómo se relacionan con la documentación

| Captura | Documento donde se referencia |
|---|---|
| `topology.png` | `01-architecture.md` |
| `dashboard-active.png`, `indexer-active.png` | `02-installation.md` |
| `agent-control-list.png`, `server-manager-addsdns.png` | `03-agent-deployment.md` |
| `auth-failure-first-hit.png`, `auth-failure-two-hits.png`, `ad-users-testuser01.png` | `04-detection-rules.md` |
| `manager-timeout-reset.png` | `05-troubleshooting.md` |
