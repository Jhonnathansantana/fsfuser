# Servidor RustDesk Autoalojado con Monitoreo

Este repositorio contiene la configuración de Docker para desplegar un servidor de RustDesk autoalojado y una pila de monitoreo para la recolección de logs con Loki.

## Características

- **Servidor RustDesk Completo**: Incluye los servicios `hbbs` (Servidor de ID/Rendezvous) y `hbbr` (Servidor de Relay).
- **Persistencia de Datos**: Utiliza volúmenes de Docker para mantener los datos de RustDesk.
- **Integración con Monitoreo**: Los contenedores de RustDesk están preparados para conectarse a una red de monitoreo externa (basada en Loki, Grafana, etc.).
- **Fácil Despliegue**: Orquestado con Docker Compose para un despliegue sencillo.

## Estructura del Repositorio

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