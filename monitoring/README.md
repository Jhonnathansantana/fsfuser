# Pila de Monitoreo con Prometheus, Grafana y Loki

Este proyecto despliega una pila de monitoreo completa utilizando Docker y Docker Compose. Incluye Prometheus para la recolección de métricas, Grafana para la visualización, Loki para la agregación de logs, y otros exportadores para obtener datos del host y de los contenedores.

## Índice

- [Componentes](#componentes)
- [Arquitectura](#arquitectura)
- [Prerrequisitos](#prerrequisitos)
- [Estructura de Archivos](#estructura-de-archivos)
- [Instalación y Configuración](#instalación-y-configuración)
- [Uso](#uso)
- [Extender el Monitoreo](#extender-el-monitoreo)

## Componentes

La pila está compuesta por los siguientes servicios:

-   **Prometheus**: Una base de datos de series temporales que recolecta y almacena métricas.
-   **Grafana**: Una plataforma de visualización para crear dashboards a partir de diversas fuentes de datos, incluyendo Prometheus y Loki.
-   **Loki**: Un sistema de agregación de logs, inspirado en Prometheus, que no indexa el contenido de los logs, sino un conjunto de etiquetas para cada flujo de logs.
-   **Promtail**: El agente que recolecta logs y los envía a Loki. Está configurado para descubrir y recolectar logs de contenedores Docker automáticamente.
-   **Node Exporter**: Un exportador de métricas del hardware y del sistema operativo del host.
-   **cAdvisor (Container Advisor)**: Proporciona información sobre el uso de recursos y características de rendimiento de los contenedores en ejecución.

## Arquitectura

El flujo de datos en esta pila de monitoreo es el siguiente:

1.  **Métricas**:
    -   `Node Exporter` expone las métricas del sistema anfitrión (CPU, memoria, disco, red).
    -   `cAdvisor` expone las métricas de los contenedores Docker (uso de CPU, memoria, red por contenedor).
    -   `Prometheus` recolecta (hace "scrape") estas métricas periódicamente y las almacena.
2.  **Logs**:
    -   `Promtail` descubre todos los contenedores Docker en ejecución, lee sus logs (`stdout`/`stderr`).
    -   Envía los flujos de logs a `Loki`, añadiendo etiquetas como el nombre del contenedor.
3.  **Visualización**:
    -   `Grafana` se conecta a `Prometheus` como fuente de datos para visualizar las métricas.
    -   `Grafana` se conecta a `Loki` como fuente de datos para explorar y visualizar los logs.
    -   Los usuarios pueden crear dashboards en Grafana para correlacionar métricas y logs en un solo lugar.

## Prerrequisitos

-   Docker
-   Docker Compose

## Estructura de Archivos

```
monitoring/
├── docker-compose.yml      # Define todos los servicios de la pila.
├── prometheus.yml          # Configuración de Prometheus (targets a monitorear).
├── loki-config.yml         # Configuración de Loki.
└── promtail-config.yml     # Configuración de Promtail (recolección de logs).
```

## Instalación y Configuración

1.  **Clona o descarga este repositorio.**

2.  **Revisa los archivos de configuración**:
    -   `prometheus.yml`: Ya está configurado para monitorear `node-exporter` y `cadvisor`. Puedes añadir más `jobs` para monitorear tus propias aplicaciones.
    -   `loki-config.yml`: Configuración básica para que Loki se ejecute en modo "single-binary" y almacene datos en un volumen.
    -   `promtail-config.yml`: Configurado para descubrir automáticamente los logs de todos los contenedores Docker que se ejecutan en el mismo host.

3.  **Inicia la pila de monitoreo**:
    Desde el directorio `monitoring`, ejecuta el siguiente comando:
    ```bash
    docker-compose up -d
    ```
    Esto descargará las imágenes y creará e iniciará todos los contenedores en segundo plano.

4.  **Verifica que los servicios estén en ejecución**:
    ```bash
    docker-compose ps
    ```
    Deberías ver todos los servicios (`prometheus`, `grafana`, `loki`, etc.) con el estado `Up`.

## Uso

Una vez que la pila esté en funcionamiento, puedes acceder a las diferentes interfaces de usuario:

-   **Prometheus**: `http://localhost:9090`
    -   Puedes explorar las métricas recolectadas y el estado de los `targets` (en `Status > Targets`).
-   **Grafana**: `http://localhost:3000`
    -   **Login inicial**: `admin` / `admin`. Se te pedirá que cambies la contraseña en el primer inicio de sesión.
    -   **Añadir fuentes de datos**:
        1.  Ve a `Configuration (engranaje) > Data Sources`.
        2.  **Añadir Prometheus**:
            -   Selecciona `Prometheus`.
            -   En el campo `URL`, introduce `http://prometheus:9090`.
            -   Haz clic en `Save & Test`.
        3.  **Añadir Loki**:
            -   Selecciona `Loki`.
            -   En el campo `URL`, introduce `http://loki:3100`.
            -   Haz clic en `Save & Test`.
    -   **Explorar datos**:
        -   Usa la sección `Explore` para consultar métricas de Prometheus (con PromQL) y logs de Loki (con LogQL).
        -   Importa dashboards pre-construidos desde la comunidad de Grafana (por ejemplo, para Node Exporter o Docker) o crea los tuyos propios.

## Extender el Monitoreo

La pila de monitoreo está diseñada para ser extensible. A continuación, se muestra un ejemplo práctico de cómo puedes monitorear los contenedores del servicio `rustdesk-server`, que se encuentra en un archivo `docker-compose.yml` separado.

El principio es simple: **ambas pilas de servicios deben compartir una red Docker común para que Prometheus pueda recolectar métricas.**

### Paso 1: Conectar los Servicios a la Red de Monitoreo

Modifica el archivo `/home/fsfuser/docker/rustdesk-server/docker-compose.yml` para que reconozca y utilice la red de la pila de monitoreo.

```yaml
# /home/fsfuser/docker/rustdesk-server/docker-compose.yml

version: '3.8'

networks:
  rustdesk-net:
    driver: bridge
  # Añade la red de monitoreo como una red externa
  monitoring_net:
    external:
      name: monitoring_monitoring-net # El nombre es <directorio>_<nombre_red>

volumes:
  rustdesk-data:

services:
  hbbs:
    # ... (configuración existente sin cambios) ...
    networks:
      - rustdesk-net
      - monitoring_net # Conecta el servicio a la red de monitoreo

  hbbr:
    # ... (configuraciones existente sin cambios) ...
    networks:
      - rustdesk-net
      - monitoring_net # Conecta el servicio a la red de monitoreo
```
**Nota:** El nombre de la red externa por defecto es `nombre-del-directorio_nombre-de-la-red`. Como la pila de monitoreo está en el directorio `monitoring`, la red se llama `monitoring_monitoring-net`.

### Paso 2: Recolectar Logs (¡No se necesita hacer nada!)

Gracias a la configuración de `Promtail`, que utiliza el socket de Docker para descubrir contenedores en el host, los logs de los contenedores `hbbs` y `hbbr` serán recolectados **automáticamente** tan pronto como se inicien. No se requiere ninguna configuración adicional.

Puedes encontrarlos en Grafana (Explore -> Loki) con consultas como:
-   `{container_name="hbbs"}`
-   `{container_name="hbbr"}`

### Paso 3: Recolectar Métricas (Si estuvieran disponibles)

El servidor RustDesk no expone métricas de Prometheus por defecto. Sin embargo, si lo hiciera (por ejemplo, en el puerto `8000`), así es como lo añadirías a Prometheus editando `/home/fsfuser/docker/monitoring/prometheus.yml`:

```yaml
# ... (configuraciones existentes en prometheus.yml) ...

- job_name: 'rustdesk-server'
  static_configs:
    # Prometheus puede resolver los nombres de los contenedores
    # porque ahora están en la misma red.
    - targets: ['hbbs:8000', 'hbbr:8000']
```

Después de añadir el `job`, reinicia Prometheus para que cargue la nueva configuración:
```bash
# Desde el directorio /home/fsfuser/docker/monitoring
docker-compose restart prometheus
```

Con estos pasos, has integrado completamente una aplicación externa en tu sistema de monitoreo centralizado.