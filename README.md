# Tarea 2. Convertir un monolitico en microservicios

# Sistema de Compras Online - Arquitectura de Microservicios

Este documento proporciona las instrucciones detalladas para desplegar el sistema de compras online basado en microservicios utilizando Docker y Docker Compose.

## Pre-requisitos

Para poder desplegar y ejecutar el sistema, necesitas tener instaladas las siguientes herramientas en tu máquina:

* **Docker Desktop:** Incluye Docker Engine, Docker CLI y Docker Compose. Puedes descargarlo e instalarlo desde el sitio oficial de Docker:
    * **Windows:** [Descargar Docker Desktop para Windows](https://docs.docker.com/desktop/install/windows-install/)
    * **macOS:** [Descargar Docker Desktop para macOS](https://docs.docker.com/desktop/install/mac-install/)
    * **Linux:** [Instalar Docker Engine en Linux](https://docs.docker.com/engine/install/) (luego instala Docker Compose V2 si no viene incluido: `sudo apt-get update && sudo apt-get install docker-compose-plugin`)

* **Git** (opcional, pero recomendado para clonar el repositorio):
    * [Descargar Git](https://git-scm.com/downloads)

## Estructura de Directorios Requerida

Para que el `docker-compose.yaml` funcione correctamente, tu proyecto debe tener la siguiente estructura de directorios. Cada subdirectorio para un servicio (ej. `products-service`) debe contener el código fuente de ese microservicio y su `Dockerfile` correspondiente.

├── docker-compose.yaml
├── api-gateway/
│   ├── Dockerfile_Nginx    # Dockerfile para la imagen de Nginx
│   └── nginx.conf          # Archivo de configuración de Nginx
├── products-service/
│   └── Dockerfile          # Dockerfile para el servicio de productos
│   └── ... (código fuente del servicio de productos)
├── orders-service/
│   └── Dockerfile          # Dockerfile para el servicio de pedidos
│   └── ... (código fuente del servicio de pedidos)
├── payment-service/
│   └── Dockerfile          # Dockerfile para el servicio de pagos
│   └── ... (código fuente del servicio de pagos)
├── returns-service/
│   └── Dockerfile          # Dockerfile para el servicio de devoluciones
│   └── ...
├── user-service/
│   └── Dockerfile          # Dockerfile para el servicio de usuarios
│   └── ...
├── reviews-service/
│   └── Dockerfile          # Dockerfile para el servicio de revisiones
│   └── ...
├── offers-service/
│   └── Dockerfile          # Dockerfile para el servicio de ofertas
│   └── ...
├── inventory-service/
│   └── Dockerfile          # Dockerfile para el servicio de inventario
│   └── ...
├── notification-service/
│   └── Dockerfile          # Dockerfile para el servicio de notificaciones
│   └── ...
├── reports-service/
│   └── Dockerfile          # Dockerfile para el servicio de reportes
│   └── ...
└── customer-support-service/
└── Dockerfile          # Dockerfile para el servicio de atención al cliente
└──

Asegúrate de que cada `Dockerfile` dentro de los directorios de servicio esté correctamente configurado para construir la imagen de tu aplicación (ej., para una aplicación Node.js, Java, Python, etc.).

## Pasos para el Despliegue

1.  **Clonar el Repositorio (si aplica):**
    Si este `README.md` es parte de un repositorio Git, primero clona el repositorio a tu máquina local y navega hasta el directorio raíz del proyecto:

    ```bash
    git clone <URL_del_repositorio>
    cd <nombre_del_repositorio>
    ```

2.  **Configurar el API Gateway (Nginx):**
    El archivo `api-gateway/nginx.conf` es crucial para el enrutamiento de las solicitudes externas a los microservicios correctos. Abre este archivo y asegúrate de que las directivas `proxy_pass` apunten a los nombres de servicio y puertos internos correctos dentro de la red de Docker Compose (ej., `http://products-service:3000;`).

    **Ejemplo de configuración básica en `api-gateway/nginx.conf`:**
    ```nginx
    events { worker_connections 1024; }

    http {
        upstream products_service {
            server products-service:3000; # Asume que tu servicio de productos escucha en el puerto 3000
        }
        upstream orders_service {
            server orders-service:3001; # Asume que tu servicio de pedidos escucha en el puerto 3001
        }
        # ... Define más upstreams para otros servicios si es necesario

        server {
            listen 80; # El puerto que expone Nginx al exterior

            location /products/ {
                proxy_pass http://products_service/;
            }

            location /orders/ {
                proxy_pass http://orders_service/;
            }

            # ... Agrega más bloques location para otras rutas de tus microservicios
        }
    }
    ```

3.  **Construir las Imágenes y Levantar los Servicios:**
    Desde el directorio raíz de tu proyecto (donde se encuentra el archivo `docker-compose.yaml`), ejecuta el siguiente comando en tu terminal:

    ```bash
    docker-compose up --build -d
    ```
    * `up`: Este comando crea y arranca todos los contenedores definidos en `docker-compose.yaml`.
    * `--build`: Fuerza a Docker Compose a reconstruir las imágenes Docker para los servicios que tienen un `Dockerfile`. Esto es esencial la primera vez que despliegas o cuando realizas cambios en el código de tus microservicios.
    * `-d`: Ejecuta los contenedores en modo "detached" (en segundo plano), lo que significa que el terminal quedará libre para que puedas seguir trabajando.

4.  **Verificar el Estado de los Servicios:**
    Para comprobar que todos los contenedores se están ejecutando correctamente, puedes usar el siguiente comando:

    ```bash
    docker-compose ps
    ```
    Deberías ver una lista de todos los servicios definidos en tu `docker-compose.yaml` (ej., `products-service`, `orders-service`, `kafka`, `zookeeper`, etc.) con su estado `Up`.

5.  **Acceder al Sistema:**
    Una vez que todos los servicios estén en ejecución y el API Gateway esté funcionando, puedes acceder al sistema de compras online a través de tu navegador web o cliente API (como Postman/Insomnia) utilizando la dirección del API Gateway. Por defecto, el API Gateway estará accesible en:

    ```
    http://localhost/
    ```
    Asegúrate de que el puerto 80 en tu máquina host no esté siendo utilizado por otra aplicación, ya que el API Gateway está configurado para escuchar en ese puerto.

    Puedes probar las rutas que hayas configurado en tu `nginx.conf` para interactuar con los microservicios (ej., `http://localhost/products/`, `http://localhost/orders/`).

## Comandos Útiles de Docker Compose

* **Ver los Logs de Todos los Servicios:**
    ```bash
    docker-compose logs -f
    ```
    Para ver los logs de un servicio específico (ej., `products-service`):
    ```bash
    docker-compose logs -f products-service
    ```

* **Detener los Servicios (sin eliminar datos):**
    ```bash
    docker-compose stop
    ```

* **Detener y Eliminar Contenedores y Redes (manteniendo los datos de los volúmenes):**
    ```bash
    docker-compose down
    ```

* **Detener y Eliminar TODO (Contenedores, Redes y Volúmenes de Datos):**
    ```bash
    docker-compose down -v
    ```
    **¡Precaución!** Este comando eliminará permanentemente todos los datos almacenados en los volúmenes de Docker, incluyendo tus bases de datos. Úsalo solo si deseas iniciar el sistema completamente desde cero y perder todos los datos existentes.