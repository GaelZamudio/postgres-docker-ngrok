# Postgres en Docker con acceso remoto (ngrok)

## Índice

- [¿Qué hace este proyecto?](#qué-hace-este-proyecto)
- [¿Qué es un contenedor Docker?](#qué-es-un-contenedor-docker)
- [¿Qué es una imagen de Docker?](#qué-es-una-imagen-de-docker)
- [Imágenes utilizadas en este proyecto](#imágenes-utilizadas-en-este-proyecto)
- [¿Qué es un volumen de Docker?](#qué-es-un-volumen-de-docker)
- [Requisitos](#requisitos)
- [Configuración](#configuración)
- [Levantar el proyecto](#levantar-el-proyecto)
- [Obtener tu dirección pública](#obtener-tu-dirección-pública)
- [Conectarse con pgAdmin](#conectarse-con-pgadmin)
- [Detener el proyecto](#detener-el-proyecto)
- [Notas importantes](#notas-importantes)
- [Referencias](#referencias)

## ¿Qué hace este proyecto?

Este proyecto levanta dos piezas usando Docker:

- **PostgreSQL**: un sistema gestor de bases de datos relacional. 
  Aquí corre dentro de un [contenedor](#qué-es-un-contenedor-docker), es decir, aislado del sistema 
  operativo de tu máquina, con todas sus dependencias ya incluidas.
- **ngrok**: un servicio que crea un túnel entre tu máquina e internet, 
  exponiendo un puerto local (en este caso, el de Postgres) mediante 
  una dirección pública temporal. Esto permite que alguien fuera de tu 
  red local se conecte a tu base de datos sin necesidad de configurar 
  routers, IPs públicas fijas, ni redirección de puertos manual.

En pocas palabras, este es un proyecto para levantar un servidor de PostgreSQL en Docker y exponerlo 
a internet mediante un túnel de ngrok, para poder conectarse desde 
pgAdmin (u otro cliente) sin importar la red en la que estés.
En este caso para interactuar con la base de datos de forma visual, se usa 
**pgAdmin**, un cliente gráfico para PostgreSQL que permite conectarse 
a servidores remotos o locales, ejecutar consultas SQL y administrar 
bases de datos sin necesidad de usar la terminal.

## ¿Qué es un contenedor Docker?

Un contenedor es un entorno aislado y ligero que empaqueta una 
aplicación junto con todo lo que necesita para funcionar (librerías, 
configuración, dependencias), sin necesidad de instalar nada de eso 
directamente en el sistema operativo anfitrión. Esto permite que 
cualquier persona, en cualquier máquina con Docker instalado, pueda 
levantar exactamente el mismo entorno sin importar su sistema 
operativo o configuración previa; que es justamente lo que hace posible que 
cualquiera pueda clonar este repositorio y tener su propio servidor 
de Postgres funcionando.

## ¿Qué es un volumen de Docker?

Los contenedores son efímeros: si se elimina un contenedor, todo lo 
que se haya guardado dentro de él se pierde. Un volumen es un mecanismo administrado por Docker para conservar los datos 
generados por un contenedor, de forma independiente al ciclo de vida 
del propio contenedor. Así, aunque el contenedor se detenga, se elimine, 
o se vuelva a crear desde cero, los datos guardados en el volumen 
permanecen intactos.

En este proyecto, el volumen se usa para que la información guardada 
en la base de datos (tablas, registros, etc.) no se pierda cada vez 
que el contenedor de Postgres se reinicia o se vuelve a levantar:

```yaml
volumes:
  - ./data:/var/lib/postgresql/data
```

Esto conecta la carpeta `data/` de tu máquina con la carpeta interna 
del contenedor donde Postgres guarda sus archivos¿.

## ¿Qué es una imagen de Docker?

Una imagen es una plantilla de solo lectura que contiene todo lo 
necesario para ejecutar una aplicación: el código, las librerías, 
las herramientas del sistema y la configuración necesaria. Cuando 
una imagen se ejecuta, se convierte en un contenedor, es decir, la 
imagen es la "plantilla" y el contenedor es la instancia funcionando en 
ese momento. Una misma imagen puede usarse para crear múltiples 
contenedores, todos partiendo exactamente del mismo punto de inicio.

## Imágenes utilizadas en este proyecto

- **`postgres:17`**: imagen oficial de PostgreSQL, publicada por el 
  equipo de PostgreSQL en [Docker Hub](https://hub.docker.com/_/postgres).
- **`ngrok/ngrok:latest`**: imagen oficial de ngrok, publicada por 
  ngrok Inc. en [Docker Hub](https://hub.docker.com/r/ngrok/ngrok).
  
## Requisitos

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) instalado
  - En Windows debes tener la virtualización activada en el BIOS 
    y WSL2 instalado.
- Cuenta gratuita en [ngrok](https://ngrok.com/signup)
- [pgAdmin](https://www.pgadmin.org/download/) (u otro cliente de Postgres) 
  para conectarte a la base de datos

## Configuración

1. Clona este repositorio:
```bash
   git clone https://github.com/GaelZamudio/postgres-docker-ngrok.git
   cd postgres-docker-ngrok
```

2. Copia el archivo de ejemplo de variables de entorno:
```bash
   cp .env.example .env
```

3. Consigue tu propio authtoken de ngrok en 
   https://dashboard.ngrok.com/get-started/your-authtoken 
   y pégalo en tu archivo `.env`, en la variable `NGROK_AUTHTOKEN`.

## Levantar el proyecto
Abre la terminal en la ruta donde clonaste el repositorio y ejecuta:
```bash
docker compose up -d
```

Esto levanta dos contenedores:
- **postgres_server**: el servidor de PostgreSQL
- **ngrok_tunel**: el túnel que expone Postgres a internet

Verifica que ambos estén corriendo con:
```bash
docker ps
```

## Obtener tu dirección pública

Abre en tu navegador:
http://localhost:4040

Ahí verás una línea con una estructura como `tcp://X.tcp.ngrok.io:XXXXX` esos son el 
host (`X.tcp.ngrok.io`) y puerto (`XXXXX`) que necesitas para conectarte desde fuera.

## Conectarse con pgAdmin

Primero necesitas abrir pgAdmin, dar clic derecho en servers y posteriormente dar clic en registrar.
Ahora, en la pestaña **Connection**:

| Campo | Valor |
|---|---|
| Host | el dominio que te dio ngrok (ej. `X.tcp.ngrok.io`) |
| Port | el puerto que te dio ngrok |
| Username | `postgres` (o el que hayas puesto en `.env`) |
| Password | el que hayas puesto en `.env` |
| Database | el que hayas puesto en `.env` (por default `postgres`) |

## Detener el proyecto

```bash
docker compose down
```

Esto detiene y elimina los contenedores (no borra tus datos, que 
quedan guardados en la carpeta `data/` gracias al volumen configurado).

## Notas importantes

Cada persona que levanta este proyecto tiene su **propia** base de 
datos, aislada de las demás. Nadie se conecta automáticamente a la 
de otro, a menos que decida compartir su URL de ngrok con alguien más.

Mientras `docker compose up -d` esté corriendo el servidor seguirá activo; si apagas tu pc o cierras Docker la conexión se caerá.

La URL de ngrok **cambia cada vez que se reinicia el contenedor** 
(en el plan gratuito), así que hay que volver a consultarla en 
`localhost:4040` después de cada reinicio.

## Referencias

- Docker Docs. *What is a Container?* https://www.docker.com/resources/what-container/
- Docker Docs. *Docker overview.* https://docs.docker.com/get-started/docker-overview/
- Docker Docs. *What is an image?* https://docs.docker.com/get-started/docker-concepts/the-basics/what-is-an-image/
- Docker Docs. *Volumes.* https://docs.docker.com/engine/storage/volumes/
- PostgreSQL Global Development Group. *What is PostgreSQL?* https://www.postgresql.org/docs/current/intro-whatis.html
- pgAdmin Development Team. https://www.pgadmin.org/
- ngrok Inc. *What is ngrok?* https://ngrok.com/docs/what-is-ngrok/
- Docker Hub. *postgres.* https://hub.docker.com/_/postgres
- Docker Hub. *ngrok/ngrok.* https://hub.docker.com/r/ngrok/ngrok
