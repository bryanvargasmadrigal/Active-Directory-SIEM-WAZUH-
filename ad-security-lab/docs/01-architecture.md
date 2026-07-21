# Arquitectura

## Topología real

Dos laptops físicas, VirtualBox como hipervisor, ambas colgadas del mismo router doméstico.

**Lab-1** corre dos VMs:
- Windows Server 2022 (Domain Controller, dominio `lab.local`)
  - Adaptador 1: red interna
  - Adaptador 2: Bridge
  - Adaptador 3: NAT
- Windows 10 (unido al dominio)
  - Adaptador 1: red interna (comparte segmento con el DC)
  - Adaptador 2: NAT

**Lab-2** corre una VM:
- Ubuntu Desktop con el stack completo de Wazuh
  - Adaptador 1: NAT
  - Adaptador 2: Bridge

El Bridge en ambos laptops es lo que permite que el DC y la VM Wazuh se vean entre sí en la red local, aunque estén en máquinas físicas distintas. El NAT es solo para que cada VM tenga salida a internet cuando hace falta (descargar el instalador del agente, por ejemplo) — el flujo principal agente→manager no depende de internet.

## Comunicación agente-manager

- Puerto `1514/tcp`: envío de eventos del agente al manager
- Puerto `1515/tcp`: registro inicial del agente

## Por qué la red interna en Lab-1

Windows Server y Windows 10 comparten una red interna para simular el segmento de dominio típico de una oficina, separado del resto del tráfico. El DC resuelve DNS y autentica ahí mismo, sin depender del router para esa parte.

## Decisiones de diseño

- **Desktop Experience en el Server**: prioricé tener GUI para administrar AD mientras aprendía, aunque signifique más consumo de recursos que un Server Core.
- **IP estática en el DC**: un Domain Controller no debería moverse de IP nunca, así que se configuró manualmente en vez de dejarlo en DHCP.
- **Separación física en dos laptops**: esto obligó a resolver problemas de conectividad real entre dos hosts distintos (no solo VMs en el mismo hipervisor), que es más parecido a cómo se ve una red real con varios servidores.
