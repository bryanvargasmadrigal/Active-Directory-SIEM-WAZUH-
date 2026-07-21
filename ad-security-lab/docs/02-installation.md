# Instalación de Wazuh

## Requisitos

- VM Ubuntu con al menos 4 GB de RAM (mejor con 8)
- Adaptador Bridge para que hable con el DC

## Instalación todo-en-uno

```bash
cd ~
curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh
curl -sO https://packages.wazuh.com/4.14/config.yml
sudo bash wazuh-install.sh -a
```

Tarda entre 15 y 25 minutos. Al final muestra la contraseña de `admin` para el dashboard — hay que copiarla en el momento, porque no se vuelve a mostrar por consola después.

## Verificación

```bash
sudo systemctl status wazuh-manager
sudo systemctl status wazuh-indexer
sudo systemctl status wazuh-dashboard
```

Los tres deberían decir `active (running)`.

## Acceso

```
https://<IP-de-la-VM-Wazuh>
```

Usuario `admin`, la contraseña generada en la instalación. El certificado es autofirmado, así que el navegador va a marcar advertencia — es normal en un lab.

## Una regla que aprendí a las malas

Nunca cambiar la contraseña de `admin` editando `internal_users.yml` a mano sin regenerar bien los certificados de seguridad después. Eso me rompió el dashboard en un crash loop (documentado en el troubleshooting). Los cambios de contraseña deberían hacerse desde `Security → Internal users` en el propio dashboard, no tocando el YAML directamente.

También conviene guardar el archivo `wazuh-install-files.tar` (tiene certificados y contraseñas originales) fuera de la VM, por si el disco se corrompe o hay que reinstalar.
