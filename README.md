# Documentación: Configuración de Motores de Base de Datos con Docker Compose

**Autor:** Harold Segundo Gonzalez  
**Docente:** Jaider J. Quintero Mendoza  
**Curso:** Base de Datos 2  
**Fecha:** 2026

---

## 📋 Tabla de Contenidos

1. [Requisitos Previos](#1-requisitos-previos)
2. [Creación de Carpetas](#2-creación-de-carpetas)
3. [Creación de la Red Docker](#3-creación-de-la-red-docker)
4. [Configuración de MySQL](#4-configuración-de-mysql)
5. [Configuración de PostgreSQL](#5-configuración-de-postgresql)
6. [Configuración de SQL Server](#6-configuración-de-sql-server)
7. [Conexiones desde DBeaver](#7-conexiones-desde-dbeaver)
8. [Resumen de Credenciales](#8-resumen-de-credenciales)
9. [Conclusiones](#9-conclusiones)

---

## 1. Requisitos Previos

Antes de comenzar con la configuración de los motores de bases de datos, es necesario verificar que Docker esté correctamente instalado en el sistema. Docker es la plataforma que nos permite crear y gestionar contenedores para cada motor de base de datos.

Se ejecutaron los siguientes comandos para confirmar la instalación:

```bash
sudo docker --version
sudo docker compose version
```
![Verificación de Docker](images/docker-version.png)

## 2. Creación de Carpetas
Para mantener un orden en el proyecto, se creó una estructura de carpetas que organiza los archivos de configuración y los datos de cada motor de base de datos. Esta estructura permite tener separados los servicios y facilita la administración de cada motor.

Se crearon las siguientes carpetas:

```bash
mkdir -p ~/ia-lab/services/motores-bd/{mysql,postgres,mssql,oracle}
mkdir -p ~/ia-lab/data/{mysql,postgres,mssql,oracle}
```

![Creacion de carpetas](images/folder-structure(2).png)
![Estructura de carpetas](images/folder-structure(3).png)

## 3. Creación de la Red Docker

Red compartida para contenedores
Para que todos los contenedores de bases de datos puedan comunicarse entre sí, se creó una red Docker compartida llamada ia-lab-network. Esta red permite que los contenedores se encuentren y se comuniquen utilizando sus nombres de contenedor como dirección.

```bash
docker network create ia-lab-network
```

![Creación de la Red Docker](images/network-create.png)

## 4. Configuración de MySQL

## 4.1 Archivos de configuración
Se crearon los archivos necesarios para configurar el contenedor de MySQL. El archivo docker-compose.yml define el servicio, y el archivo .env contiene las variables de entorno como la contraseña del usuario root y la base de datos inicial.

docker-compose.yml:

```bash
services:
  mysql:
    image: mysql:8.0
    container_name: mysql-server
    restart: unless-stopped
    env_file:
      - .env
    ports:
      - "3306:3306"
    volumes:
      - ../../../data/mysql:/var/lib/mysql
    command: >
      --character-set-server=utf8mb4
      --collation-server=utf8mb4_unicode_ci
      --bind-address=0.0.0.0
    networks:
      - ia-lab-network
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 30s

networks:
  ia-lab-network:
    external: true
```

![Archivos de configuración](images/mysql-env.png)

.env:

```bash
TZ=America/Bogota
MYSQL_ROOT_PASSWORD=123456
MYSQL_DATABASE=tecnogua
```
![.env](images/mysql-compose2.png)

## 4.2 Levantar el contenedor

Se ejecutó el comando para levantar el contenedor de MySQL en segundo plano. El contenedor se descargó desde Docker Hub y se inició con la configuración definida.

```bash
cd ~/ia-lab/services/motores-bd/mysql
sudo docker compose up -d
```
![Levantar el contenedor](images/mysql-running.png)

## 4.3 Creación de usuario propio con acceso remoto

Siguiendo la guía, se conectó al servidor MySQL como usuario root para crear un usuario personalizado con acceso desde cualquier equipo (%). Esto permite conectarse remotamente a la base de datos desde otros equipos de la red.

```bash
sudo docker exec -it mysql-server mysql -u root -p
# Password: 123456
```
```bash
CREATE DATABASE harold_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'harold'@'%' IDENTIFIED BY '123456';
GRANT ALL PRIVILEGES ON *.* TO 'harold'@'%';
GRANT ALL PRIVILEGES ON harold_db.* TO 'harold'@'%';
FLUSH PRIVILEGES;
```
![creacion de usuario propio](images/mysql-create-user.png)

## 4.4 Verificación del usuario creado

Se verificó que el usuario harold haya sido creado correctamente con acceso desde cualquier equipo (%).

```bash
SELECT user, host FROM mysql.user WHERE user = 'harold';
```

![Verificación del usuario creado](images/mysql-verify-user.png)
![Verificación del usuario creado](images/mysql-verify-user(2).png)

## 4.5 Prueba del nuevo usuario

Se probó la conexión al servidor MySQL utilizando el nuevo usuario harold para verificar que funciona correctamente.

```bash
sudo docker exec -it mysql-server mysql -u harold -p
# Password: 123456
```
![verificacion](images/verificacion.png)



## 4.6 Prueba de conexión desde DBeaver

Una vez creado el usuario harold, se procedió a probar la conexión remota desde DBeaver.

Obtener la IP de WSL:

```bash
hostname -I
# IP obtenida: 172.24.20.10
```
Campo	        Valor

Host	        172.24.20.10

Port	        3306

Database	    harold_db

Username	    harold

Password	    123456

![Bbeaver](images/mysql-dbeaver.png)

## 4.7 Backup de la base de datos

Se realizó una copia de seguridad de la base de datos harold_db utilizando el usuario harold. El backup se guardó en la carpeta /mnt/d/academia/bd/ con la fecha actual en el nombre del archivo.

```bash
sudo docker exec mysql-server mysqldump -u harold -p123456 harold_db > /mnt/d/academia/bd/backup_harold_db_$(date +%Y%m%d).sql
```
![Bakuup](images/mysql-backup-verify.png)

## Configuración de PostgreSQL

## 5.1 Archivos de configuración
Se crearon los archivos de configuración para PostgreSQL. El archivo .env define el usuario administrador (ialab), la contraseña y la base de datos inicial.

docker-compose.yml:
