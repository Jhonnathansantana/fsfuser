﻿# 🖥️ Servidor RustDesk Autoalojado con Monitoreo y Métricas

Este proyecto proporciona una configuración completa de Docker Compose para desplegar un servidor de escritorio remoto **RustDesk** autoalojado, junto con una potente **pila de monitoreo** basada en Prometheus, Grafana y Loki.

## ✨ Características Clave

- **Servidor RustDesk Completo**: Incluye los servicios `hbbs` (Servidor de ID/Rendezvous) y `hbbr` (Servidor de Relay) para una solución de escritorio remoto totalmente funcional.
- **Monitoreo Integrado**:
  - **Logs centralizados con Loki**: Captura y visualiza los logs de los servicios de RustDesk en tiempo real.
  - **Métricas con Prometheus**: Recolecta métricas del host y de los contenedores.
  - **Dashboards con Grafana**: Visualiza logs y métricas en dashboards intuitivos.
- **Configuración Sencilla y Segura**: Utiliza un archivo `.env` para una configuración rápida sin modificar los archivos de Compose, con instrucciones claras para asegurar tu servidor.
- **Persistencia de Datos**: Todos los datos importantes (RustDesk, Grafana, Prometheus, Loki) se guardan en volúmenes de Docker.

## 🏗️ Arquitectura

La solución se divide en dos pilas de Docker Compose que se comunican a través de una red externa:

1.  **Pila de Monitoreo (`monitoring`)**: Contiene Grafana, Loki, Promtail, Prometheus, etc. Crea una red llamada `monitoring_monitoring-net`.
2.  **Pila de RustDesk (`rustdesk-server`)**: Contiene los servicios `hbbs` y `hbbr`. Se conecta a la red `monitoring_monitoring-net` para que Promtail pueda descubrir y recolectar sus logs automáticamente.

## 📂 Estructura del Repositorio

```
.
├── docker/
│   ├── monitoring/
│   │   └── loki-config.yml   # Configuración de Loki para la recolección de logs
│   └── rustdesk-server/
│       └── docker-compose.yml # Define los servicios de RustDesk (hbbs y hbbr)
└── README.md
```

## Requisitos Previos

- Docker
- Docker Compose
- Una IP pública o un nombre de dominio que apunte a la máquina donde se desplegará el servidor.
- Puertos `21115-21119` abiertos en tu firewall.

## Configuración

Antes de iniciar los servicios, es **esencial** que configures tu dirección IP pública o dominio.

1.  **Abre el archivo**: `docker/rustdesk-server/docker-compose.yml`
2.  **Localiza el servicio `hbbs`**: Dentro de este servicio, encontrarás la sección `command`.
3.  **Actualiza la IP**: Reemplaza la IP de ejemplo `192.168.100.253` con tu IP pública o tu nombre de dominio.

    ```yaml
    # ...
    services:
      hbbs:
        # ...
        # Reemplaza YOUR_PUBLIC_IP_OR_DOMAIN con tu IP pública o dominio.
        command: hbbs -r YOUR_PUBLIC_IP_OR_DOMAIN:21117
    # ...
    ```

## 🚀 Despliegue

El despliegue se realiza en dos pasos, ya que RustDesk depende de la red creada por la pila de monitoreo.

1.  **Iniciar la Pila de Monitoreo**:
    *Asumiendo que tienes un `docker-compose.yml` en el directorio `docker/monitoring`.*
    ```bash
    cd docker/monitoring
    docker-compose up -d
    ```

2.  **Iniciar el Servidor RustDesk**:
    ```bash
    cd ../rustdesk-server
    docker-compose up -d
    ```

## 💻 Configuración del Cliente RustDesk

Para que tus clientes de RustDesk se conecten a tu servidor autoalojado:

1.  Abre la configuración del cliente (haz clic en el menú `...` junto a tu ID).
2.  **ID Server**: Introduce la IP pública o el dominio de tu servidor (ej: `mi.servidor.com`).
3.  **Relay Server**: Introduce la IP pública o el dominio de tu servidor, seguido del puerto `21117` (ej: `mi.servidor.com:21117`).
4.  **Key**: Deja este campo en blanco si no has configurado una clave pública en el servidor.

¡Y listo! Tu cliente ahora se comunicará a través de tu infraestructura.

🔒 Seguridad: Generar y Usar una Clave (Recomendado) + +Para evitar que cualquiera use tu servidor, es fundamental generar una clave y configurarla en tus clientes. + 

1. Genera las claves: Desde el directorio docker/rustdesk-server, ejecuta el siguiente comando. Creará los archivos de clave en el volumen persistente y luego se detendrá.

bash
docker-compose run --rm hbbs

2. Obtén tu clave pública: Ahora, visualiza la clave pública con uno de los siguientes métodos: +

### Método 1: Leer desde el host (Recomendado)

   ```bash

   sudo cat /var/lib/docker/volumes/rustdesk-server_rustdesk-data/_data/id_ed25519.pub

   ```
### Método 2: Usar docker exec (si los contenedores ya están corriendo)

   ```bash

   docker exec hbbs cat /root/id_ed25519.pub

   ```
3. Copia la clave: Copia la cadena de caracteres que aparece. La necesitarás para configurar tus clientes. + +4. Inicia los servicios: Si aún no lo has hecho, inicia el servidor con docker-compose up -d. +