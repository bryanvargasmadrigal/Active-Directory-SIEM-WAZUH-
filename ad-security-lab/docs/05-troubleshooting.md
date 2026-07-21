# Troubleshooting real

Todo lo de acá pasó de verdad mientras armaba el lab. Lo dejo con los comandos exactos porque el proceso de diagnóstico importa tanto como la solución.

## Perdí la contraseña de admin del dashboard

La cambié desde la terminal en algún momento y se me olvidó. El archivo `wazuh-install-files.tar` solo tiene la contraseña original de instalación, no las que cambiás después, así que ese atajo no sirvió.

Reset manual del hash:

```bash
cd /usr/share/wazuh-indexer/plugins/opensearch-security/tools/
sudo bash hash.sh
```

Eso pide la nueva contraseña y devuelve un hash. Ese hash va en:

```bash
sudo nano /etc/wazuh-indexer/opensearch-security/internal_users.yml
```

reemplazando el valor de `hash:` bajo la sección `admin:`. Después aplicar:

```bash
cd /usr/share/wazuh-indexer/plugins/opensearch-security/tools/
sudo bash securityadmin.sh -cd /etc/wazuh-indexer/opensearch-security/ \
  -icl -key /etc/wazuh-indexer/certs/admin-key.pem \
  -cert /etc/wazuh-indexer/certs/admin.pem \
  -cacert /etc/wazuh-indexer/certs/root-ca.pem -nhnv
```

Detalle que no es obvio: `wazuh-indexer` tiene que estar corriendo antes de este último comando, o tira `ERR: Seems there is no OpenSearch running on localhost:9200`.

## El dashboard entró en crash loop

Después de tocar el `internal_users.yml` a mano, el dashboard quedó en un estado raro: `systemctl status` decía `active`, pero el puerto 443 nunca aparecía escuchando y el navegador no cargaba nada.

La pista real estaba en:

```bash
sudo journalctl -u wazuh-dashboard -n 100 --no-pager
```

Ahí se veía `Scheduled restart job, restart counter is at 61` — o sea, systemd lo estaba reiniciando constantemente y el "active" era engañoso, porque el proceso se caía apenas arrancaba.

En vez de seguir peleando con un stack trace de Node.js sin saber si el problema era de certificados o de sincronización con el indexer, decidí reinstalar el stack completo. El cálculo fue simple: reinstalar tomaba 1-2 horas con resultado predecible, seguir debuggeando algo que no tenía identificado del todo podía tomar más y sin garantía de arreglarlo.

```bash
sudo systemctl stop wazuh-dashboard wazuh-indexer wazuh-manager
sudo apt-get remove --purge wazuh-dashboard wazuh-indexer wazuh-manager -y
sudo rm -rf /var/ossec /etc/wazuh-indexer /etc/wazuh-dashboard /usr/share/wazuh-indexer /usr/share/wazuh-dashboard
```

Y de nuevo el instalador todo-en-uno.

## El manager decía "failed" pero en realidad estaba funcionando

Después de un restart, `systemctl status wazuh-manager` mostraba:

```
Active: failed (Result: timeout)
```

Antes de asumir que algo estaba roto, revisé si los daemons internos seguían corriendo:

```bash
sudo /var/ossec/bin/wazuh-control status
```

Y sí — `wazuh-analysisd`, `wazuh-remoted`, `wazuh-db`, `wazuh-authd`, todos en `running`. Los únicos "not running" eran módulos opcionales que ni siquiera uso (`clusterd`, `maild`, `agentlessd`, `csyslogd`). O sea, el timeout fue solo de systemd esperando una confirmación que tardó demasiado en llegar, no una falla real del servicio.

Solución, sin tocar nada más:

```bash
sudo systemctl reset-failed wazuh-manager
```

Esto confirmó algo importante: en Wazuh, el estado que reporta `systemctl` no siempre refleja lo que está pasando realmente con los procesos internos. Antes de reinstalar por un "failed", vale la pena chequear `wazuh-control status` primero.

## El agente quedó huérfano después de reinstalar el manager

Cada vez que reinstalé el stack de Wazuh desde cero, el dashboard mostraba "No agents were added to the manager" — pero el log del agente en el DC seguía diciendo `Agent is now online`. El agente viejo tenía credenciales de registro que el manager nuevo no reconocía.

Solución: desinstalar el agente en el DC (`msiexec /x`) y volver a registrarlo con el comando que genera el wizard de `Deploy new agent` en el dashboard nuevo.

## Reconstruí el Domain Controller desde cero

Fallas de hardware/VM obligaron a borrar la VM del DC completa en un punto. Tuve que rehacer todo: nueva VM, Windows Server 2022, `Install-ADDSForest` para levantar el dominio de nuevo, y reinstalar el agente Wazuh apuntando al mismo manager (que no tuvo que tocarse para nada).

Lo que confirmó esto: mantener el manager de Wazuh completamente independiente del ciclo de vida del DC permitió reconstruir el endpoint sin perder ni un poco de la configuración del SIEM.
