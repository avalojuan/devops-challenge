# Prueba técnica DevOps

El siguiente proyecto describe una posible solución al desafío técnico propuesto por [Craftech](https://craftech.io). El mismo abarca herramientas y conceptos de DevOps, como ser Docker, CI/CD y servicios distribuidos como AWS.

## Prueba 1 - Diagrama de Red

### Consigna:

Produzca un diagrama de red (puede utilizar
lucidchart) de una aplicación web en GCP o AWS y escriba una descripción de
texto de 1/2 a 1 página de sus elecciones y arquitectura.
El diseño debe soportar:

- Cargas variables
- Contar con HA (alta disponibilidad)
- Frontend en Js
- Backend con una base de datos relacional y una no relacional
- La aplicación backend consume 2 microservicios externos
  El diagrama debe hacer un mejor uso de las soluciones distribuidas.

### Resolución:

![diagrama de red con servicios de AWS](https://res.cloudinary.com/avalojuan/image/upload/v1684725312/Diagrama_Red_Avalo_Juan_Craftech_qacyzg.jpg)

A continuación se detallan los servicios y componentes utilizados en la arquitectura:

### Administración de Dominio con Route53

- Route53 nos permite configurar y gestionar los registros DNS necesarios para dirigir el tráfico hacia nuestros servicios.

### Balanceo de Cargas con ALB (Application Load Balancer)

- ALB nos permite crear reglas y distribuir el tráfico entre las instancias de EC2 que ejecutan nuestro backend en las zonas de disponibilidad.
- Con ALB, podemos definir reglas basadas en el contenido de las solicitudes y direccionar el tráfico de manera eficiente a las instancias adecuadas.

### Almacenamiento y Distribución de Archivos Estáticos

- Para el frontend, consideramos que los archivos son estáticos y optamos por utilizar Amazon S3 Standard ya que este redunda los datos en un mínimo de 3 zonas de disponibilidad.
- Utilizamos CloudFront para la distribución de contenido, lo que nos permite hacer uso de HTTPS y nos brinda alta disponibilidad.

### Backend en EC2

- Para el backend, hemos definido dos zonas de disponibilidad en las cuales se ejecutan nuestras instancias EC2.
- Cada zona de disponibilidad cuenta con tres subredes: una subred pública y dos subredes privadas.
- Utilizamos un NAT Gateway en la subred pública para permitir que el backend realice consultas a los microservicios externos desde la subred privada.
- La subred privada backend es donde se ejecuta nuestra API en una instancia EC2.

### Auto Scaling Group

- Agrupamos las EC2 de ambas zonas de disponibilidad en un Auto Scaling Group, lo que nos permite definir reglas de escalado en función de la carga y la capacidad deseada.

### Bases de Datos Relacional y No Relacional

- En la tercera subred, hemos configurado dos servicios de bases de datos:
  - RDS para la base de datos relacional.
  - DynamoDB para la base de datos no relacional.
- Se realiza de forma automática la replicación síncrona de datos entre las zonas de disponibilidad A y B como respaldo. Lo que garantiza la disponibilidad y la recuperación ante posibles fallas.

## Prueba 2 - Despliegue de una aplicación Django y React.js:

### Consigna:

Elaborar el deployment dockerizado de una aplicación en django (backend) con frontend en React.js contenida en el repositorio.

- Es necesario desplegar todos los servicios en un solo docker-compose.
- Se deben entregar los Dockerfiles pertinentes para elaborar el despliegue y justificar la forma en la que elabora el deployment (supervisor, scripts, dockercompose, kubernetes, etc)

### Resolución:

Se utilizó Visual Studio Code (VSCode) como entorno de desarrollo de los `Dockerfile` y el archivo `docker-compose.yml`.

Los mismos se ejecutaron con Docker engine en ubuntu 22.04.

### Descripción del proceso de resolución:

1. Frontend:

   - Se creó el `Dockerfile` para el frontend, que incluye la instalación de las dependencias y la compilación del código de React.js.
   - Se decidió utilizar la versión 14 de Node, considerando la versión de React (16) utilizada en el proyecto y para evitar posibles incompatibilidades de dependencias con versiones más recientes.

2. Backend:

   - Se comenzó utilizando Python 3.8 y se agregaron las dependencias necesarias, como `graphviz-dev` para `pygraphviz`.
   - Luego, se tuvo que cambiar la versión de Python a 3.7 para que sea compatible con la versión de `psycopg2=2.7.6`, tal como se especifica en la [documentación de PyPI](https://pypi.org/project/psycopg2/2.7.6/).
   - Al hacer las solicitudes desde React hacia la API del backend, se encontró un error de CORS en el navegador. Para solucionarlo, se editó el archivo `settings.py` del backend y se agregó `CORS_ALLOW_CREDENTIALS = True` para permitir las solicitudes con credenciales.

3. Docker compose:

   - Se definieron tres servicios: `frontend`, `backend` y `db`.
   - Para los servicios `frontend` y `backend`, se realizó el build a partir de sus respectivos Dockerfiles.
   - Para el servicio `db`, se utilizó la imagen pública de Postgres y se definió un volumen para persistir los datos.
   - Se asignaron los puertos correspondientes para los servicios de frontend y backend, y se utilizó el puerto estándar de Postgres según la documentación oficial.
   - Se crearon archivos `.env` para cada servicio con sus respectivas variables de entorno.

4. Para ejecutar las migraciones de la db desde la api:
   - Se optó por ejecutar las migraciones desde el mismo contenedor de Docker utilizando el comando `docker compose exec`, utilizando el nombre del servicio definido en el `Dockerfile`.

### Despliegue de la aplicación con Docker Compose en una PC

**Requisitos**

- [Docker engine](https://docs.docker.com/engine/install/)
- [Docker Compose](https://docs.docker.com/compose/install/)

**Pasos para el despliegue**

1. Clonar el repositorio y acceder a los archivos de la prueba 2:

   ```bash
   git clone https://github.com/avalojuan/devops-challenge.git
   cd devops-challenge/prueba-2
   ```

2. Ejecutar el siguiente comando para iniciar el despliegue de los contenedores en segundo plano:

   ```bash
   docker-compose up -d

   ```

3. Ejecutar las migraciones de la base de datos:

   ```bash
   docker compose exec backend python manage.py makemigrations
   docker compose exec backend python manage.py migrate
   ```

   Estos comandos ejecutan las migraciones necesarias para configurar la base de datos del backend.

Puedes acceder a la aplicación frontend a través de tu navegador web en la siguiente dirección:

- URL: http://localhost:3000

**_Detener el despliegue_**

Si deseas eliminar los contenedores, puedes ejecutar el siguiente comando en el directorio donde se encuentra el `docker-compose.yml`:

```bash
docker compose down
```

### Despliegue de la aplicación en AWS

Una posible solución es utilizar el servicio ECS de AWS, siguiendo los pasos descritos en la [guía oficial de docker](https://docs.docker.com/cloud/ecs-integration/).

**Resumen de la guia:**

1.  Asignarle los permisos requeridos a tus credenciales de AWS.
2.  Crear un AWS context utilizando access key ID y secret.
3.  Ejecutar docker compose para desplegar la aplicación.
    \
    _Ya que nuestras variables de entorno se obtienen de archivos .env, debemos subir los mismos a un bucket S3 y configurarlo en nuestro ECS como muestra la siguiente [guía oficial de AWS](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/taskdef-envfiles.html)_.

## Prueba 3 - CI/CD

### Consigna:

Dockerizar un nginx con el index.html default.
Elaborar un pipeline que ante cada cambio realizado sobre el index.html buildee
la nueva imagen y la actualize en la plataforma elegida. (docker-compose,
swarm, kuberenetes, etc.) Para la creacion del CI/CD se puede utilizar cualquier
plataforma (CircleCI, Gitlab, Github, Bitbucket.)

### Resolución:

Elegí GitHub con GitHub Actions para crear el flujo CI/CD ya que en otra ocasión había utilizado GitLab y me pareció oportuno probar una nueva plataforma.
A su vez realicé el despliegue en un droplet de Digital Ocean con Ubuntu, ingresando en primera instancia por SSH para instalar docker, con el cual luego ejecutaría la imagen resultante del build en el workflow.
Esto lo hice creando keys SSH nuevas de forma local, y configurando los DNS de un dominio propio con el droplet para reemplazar la url aleatoria de DO.

Se puede acceder al droplet en el [siguiente link](http://craftech-challenge.avalojuan.com/)

### Descripcion del workflow

**on:**
Define condiciones de ejecucion del job, en este caso para la rama main, y en el path donde se encuentra nuestro `index.html`.

**env:**
Definimos variables de entorno. Algunas declaradas previamente como secrets en la configuración del repositorio, como ser credenciales de Dockerhub, o los datos y keys necesarios para acceder al droplet por SSH. Otra variable definida es IMAGE_TAG que extrae el SHA del commit, para poder identificar el origen y cambios de la imagen que luego desplegaremos.

**jobs:**

- **_build_and_push_**: con el runner ubuntu-latest y actions/checkout@v3 para acceder al commit, y ejecutar los posteriores comandos para: autenticarnos en DockerHub, hacer el build de la imagen con su respectivo tag, y hacer el push de esta imagen al registry de DockerHub. _Se pueden ver las imagenes creadas en [este link](https://hub.docker.com/repository/docker/avalojuan/craftech-challenge-task-3/general)._
- **_deploy_**: utilizamos el action de appleboy/ssh-action para ejecutar comandos de manera remota mediante SSH, en este caso, en nuestro droplet con docker. Los comandos que utilizamos, detienen el contenedor a reemplazar y ejecutan uno nuevo utilizando la imagen creada en el step previo, asignandole un name y port, para que pueda ser accedido desde internet.
