# Active-Directory-SIEM-WAZUH

Este es mi homelab de detección: un Domain Controller de Windows Server siendo monitorizado en tiempo real por Wazuh, configurado para detectar logs diarios en una compañia — alguien creando cuentas, intentos de login fallidos, cambios raros en grupos de seguridad.

No es tan solo un tutorial copiado y pegado. Lo armé, lo rompí varias veces, y volví a levantarlo. Esa parte de "se rompió y así lo arreglé" está documentada abajo tal cual pasó, porque creo que eso dice más de mí que una captura de pantalla perfecta a la primera y sobre todo estos errores son los que construyen el conocimiento real.

## Por qué existe esto

Estoy construyendo mi perfil hacia roles de NOC/SOC L1 y necesitaba algo más que certificados en PDF. Quería un entorno donde pudiera:

- Levantar un Domain Controller desde cero y entender cómo se comporta
- Conectar un SIEM real (no una demo) y ver tráfico de verdad llegando
- Configurar auditoría de Windows a mano, sin wizards mágicos
- Generar un evento de seguridad y perseguirlo hasta verlo aparecer como alerta

## Cómo está armado

Dos laptops físicas corriendo VirtualBox como hipervisor, conectadas a un router doméstico:

- **Lab-1**: Windows Server (AD DS) + Windows 10, ambos con un adaptador de red interna compartida entre sí, más Bridge/NAT para salir
- **Lab-2**: Ubuntu Desktop corriendo el stack completo de Wazuh (manager, indexer, dashboard)

Dominio: `lab.local`. Comunicación agente↔manager por los puertos estándar de Wazuh (1514 eventos, 1515 registro).

## Stack técnico

- Wazuh 4.14.6 (instalación todo-en-uno: manager + indexer + dashboard)
- Windows Server 2022 Standard con Desktop Experience, promovido a Domain Controller
- Windows 10 unido al dominio
- Wazuh Agent en el DC, leyendo los canales Security, System, Application, Directory Service y DNS Server
- Auditoría avanzada de Windows habilitada manualmente vía `auditpol` (no viene por defecto)

## Qué detecta hoy

| Evento | Qué significa | Estado |
|---|---|---|
| Creación de cuenta de usuario (4720) | Alguien creó un usuario nuevo en el dominio | Probado y confirmado |
| Fallo de autenticación (Logon Failure, regla 60122) | Intento de login con usuario o contraseña incorrectos | Probado, 2 hits confirmados |

## Documentación

- [Arquitectura](docs/01-architecture.md) — cómo está conectado todo y por qué
- [Instalación de Wazuh](docs/02-installation.md) — el stack completo, paso a paso
- [Despliegue del agente](docs/03-agent-deployment.md) — cómo conecté el DC al SIEM
- [Reglas de detección](docs/04-detection-rules.md) — qué generé y cómo lo cacé
- [Troubleshooting real](docs/05-troubleshooting.md) — todo lo que se rompió y cómo lo arreglé

## Evidencia

Todas las capturas están en `screenshots/`, organizadas por fase. Índice completo con descripción de cada una en [`docs/evidencia.md`](docs/evidencia.md).

Los puntos más fuertes:
- Diagrama de red completo de las dos laptops y sus adaptadores
- Manager con 3 servicios activos, incluyendo el reset de un falso "failed" por timeout
- Agente reportando Active desde el dashboard
- Dos alertas reales de "Logon Failure" cazadas en Threat Hunting
- El usuario de prueba visible directamente en Active Directory Users and Computers

## Sobre las fallas

Esto tuvo su cuota de dolor de cabeza real: perdí la contraseña de admin del dashboard, el dashboard entró en un crash loop que aparentaba estar "active" cuando en realidad se reiniciaba cada pocos segundos, tuve que reinstalar el stack completo una vez, y reconstruí el Domain Controller desde cero después de que la VM fallara. Todo eso está en el troubleshooting con los comandos exactos que usé para diagnosticar y resolver. Prefiero mostrar eso a fingir que todo salió perfecto al primer intento.

---

Brayan Vargas Madrigal — Soporte IT / Estudiante de Ciencias de la Computación (UCR)
[GitHub](https://github.com/bryanvargasmadrigal)
