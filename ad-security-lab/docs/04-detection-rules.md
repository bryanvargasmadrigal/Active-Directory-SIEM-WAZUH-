# Reglas de detección

## Cómo probé cada cosa

Para cada evento seguí siempre el mismo circuito: generar la acción en el DC, confirmar que Windows la registró en su propio log, y después ir a buscarla en Wazuh.

## Creación de cuenta de usuario

Generación:

```powershell
New-ADUser -Name "TestUser01" -Enabled $false
```

Confirmación en Windows:

```powershell
Get-WinEvent -LogName Security -MaxEvents 10 | Where-Object {$_.Id -eq 4720}
```

En Wazuh (Threat Hunting → Events):

```
rule.groups:windows_security
```

El usuario también quedó visible directamente en Active Directory Users and Computers, lo cual sirve como segunda confirmación fuera del SIEM.

## Fallo de autenticación

Generación: intento de login con usuario o contraseña incorrectos contra el dominio.

Búsqueda en Wazuh:

```
data.win.system.eventID:4625
```

o filtrando directamente por la descripción de la regla:

```
rule.description: "Logon Failure - Unknown user or bad password"
```

**Resultado real:** 2 hits capturados, ambos clasificados bajo la regla **60122** ("Logon Failure - Unknown user or bad password"), nivel 5. Se ven claramente separados en el tiempo en el histograma del dashboard, lo cual confirma que Wazuh está procesando cada intento como un evento independiente, no agrupándolos de forma rara.

## Algo que aprendí sobre el dashboard

El selector de rango de fecha/hora de Wazuh se queda fijo en el último rango que usaste. Si generás un evento después de ese rango y buscás, te va a decir "No results match your search criteria" aunque el evento sí llegó y está indexado. Antes de asumir que algo falló, hay que revisar primero si el rango de tiempo es el correcto.

## Pendiente

- Evento 4728 (agregar a un grupo de seguridad)
- Evento 4724 (reset de contraseña)
- Simular varios logins fallidos seguidos para ver si dispara una regla de correlación (fuerza bruta)
