# Despliegue del agente en el DC

## Generar el comando de instalación

Desde el dashboard: `Endpoints` → `Deploy new agent` → Windows → poner la IP de la VM Wazuh como `Server address`. El wizard arma el comando con todo incluido.

## Instalar en el DC

PowerShell como Administrador:

```powershell
Invoke-WebRequest -Uri https://packages.wazuh.com/4.x/windows/wazuh-agent-4.14.6-1.msi -OutFile $env:tmp\wazuh-agent.msi
msiexec.exe /i $env:tmp\wazuh-agent.msi /q WAZUH_MANAGER="<IP-VM-Wazuh>"
```

```powershell
NET START WazuhSvc
```

## Configurar los canales que realmente importan

Por defecto el agente no lee los canales de seguridad de Windows. Hay que editar `C:\Program Files (x86)\ossec-agent\ossec.conf` y agregar esto antes de `</ossec_config>`:

```xml
<localfile>
  <location>Security</location>
  <log_format>eventchannel</log_format>
</localfile>

<localfile>
  <location>System</location>
  <log_format>eventchannel</log_format>
</localfile>

<localfile>
  <location>Application</location>
  <log_format>eventchannel</log_format>
</localfile>

<localfile>
  <location>Directory Service</location>
  <log_format>eventchannel</log_format>
</localfile>

<localfile>
  <location>DNS Server</location>
  <log_format>eventchannel</log_format>
</localfile>
```

Reiniciar:

```powershell
NET STOP WazuhSvc
NET START WazuhSvc
```

## Habilitar auditoría avanzada

Sin esto, el canal Security queda casi vacío de eventos útiles. En un sistema en español:

```powershell
auditpol /set /subcategory:"Inicio de sesión" /success:enable /failure:enable
auditpol /set /subcategory:"Administración de cuentas de usuario" /success:enable /failure:enable
auditpol /set /subcategory:"Administración de grupos de seguridad" /success:enable /failure:enable
```

Confirmar con:

```powershell
auditpol /get /category:*
```

## Verificar que quedó conectado

Desde la VM Wazuh:

```bash
sudo /var/ossec/bin/agent_control -l
```

Tiene que listar el agente como `Active`.

Desde el DC:

```powershell
Get-Content "C:\Program Files (x86)\ossec-agent\ossec.log" -Tail 30
```

Buscando la línea `Connected to the server` y `Agent is now online`.

## Nota sobre reinstalaciones

Cada vez que reinstalé el manager Wazuh desde cero, el agente viejo quedó "huérfano" — el dashboard mostraba "No agents were added to the manager" aunque el agente en el DC seguía diciendo que estaba online. Hubo que desinstalar el agente en el DC y volver a registrarlo con el comando nuevo del wizard. Reinstalar el manager no es transparente para los agentes que ya estaban conectados.
