# Semana 3: Base de Datos RutaCapital (MySQL y PostgreSQL)

**Autor:** Harold Segundo Gonzalez  
**Docente:** Jaider J. Quintero Mendoza  
**Curso:** Base de Datos 2  
**Fecha:** 2026

---

## 📋 Tabla de Contenidos

1. [Descripción del Proyecto](#1-descripción-del-proyecto)
2. [Reglas de Nomenclatura](#2-reglas-de-nomenclatura)
3. [Modelo Entidad-Relación](#3-modelo-entidad-relación)
4. [MySQL](#4-mysql)
5. [PostgreSQL](#5-postgresql)
6. [Conclusiones](#6-conclusiones)

---

## 1. Descripción del Proyecto

RutaCapital administra rutas urbanas, paradas ordenadas, horarios, buses y conductores. La solución debe planear asignaciones sin cruces de turno, registrar salidas reales, incidentes y mantenimientos que afecten la disponibilidad de la flota.

---

## 2. Reglas de Nomenclatura

- **Tablas:** minúsculas, en plural, en inglés
- **Llaves foráneas:** nombre_tabla_singular + `_id`

---

## 3. Modelo Entidad-Relación

| Entidad | Tabla | Atributos |
|---------|-------|-----------|
| Ruta | `routes` | id, name, description, is_active, created_at, updated_at |
| Parada | `stops` | id, name, description, is_active, created_at, updated_at |
| RutaParada | `route_stops` | id, route_id, stop_id, stop_order, is_active |
| Horario | `schedules` | id, name, description, is_active, created_at, updated_at |
| Bus | `buses` | id, name, description, is_active, created_at, updated_at |
| Conductor | `conductors` | id, name, description, is_active, created_at, updated_at |
| Asignacion | `assignments` | id, schedule_id, bus_id, conductor_id, is_active |
| Incidente | `incidents` | id, assignment_id, type, date, description, is_active |
| Mantenimiento | `maintenance` | id, bus_id, type, scheduled_date, close_date, cost, status |

---

# 4. MySQ

**EMPEZAMOS CON MySQL POR TERMINAL UBUNTU**

## 1. Conectarse a MySQL

```bash
sudo docker exec -it mysql-server mysql -u harold -p
# Password: 123456
```
![conectarse](images/semana3-mysql-conexion.png)
## 2. Crear la base de datos

```bash
SHOW DATABASES;
CREATE DATABASE BasededatosMysql1;
SHOW DATABASES;
```
![Crear base de datos](images/semana3-mysql1-database.png.png)

![ver las bases de datos](images/semana3-mysql1-database2.png)

## 3. Usar la base de datos

```bash
USE BasededatosMysql1;
```

![usar base de datos](images/semana3-mysql1-use.png)

## 4. Crear las tablas

```bash
CREATE TABLE routes (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    description TEXT,
    is_active ENUM('ACTIVE', 'INACTIVE') DEFAULT 'ACTIVE',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```
![table routes](images/semana3-mysql1-table-routes.png)
![table routes visualiacion](images/semana3-mysql1-table-routes-tes.png)

```bash
CREATE TABLE stops (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    description TEXT,
    is_active ENUM('ACTIVE', 'INACTIVE') DEFAULT 'ACTIVE',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```
![table stop](images/semana3-mysql1-table-stops.png)
![table stop visualizacion](images/semana3-mysql1-table-stops2.png)
```bash
CREATE TABLE route_stops (
    id INT AUTO_INCREMENT PRIMARY KEY,
    route_id INT NOT NULL,
    stop_id INT NOT NULL,
    stop_order INT NOT NULL,
    is_active ENUM('ACTIVE', 'INACTIVE') DEFAULT 'ACTIVE',
    FOREIGN KEY (route_id) REFERENCES routes(id),
    FOREIGN KEY (stop_id) REFERENCES stops(id),
    UNIQUE KEY unique_route_stop (route_id, stop_id)
);
```
![table routes-stop](images/semana3-mysql1-table-route-stop.png)
![table routes-stop visualizacion](images/semana3-mysql1-table-route-stop2.png)

```bash
CREATE TABLE schedules (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    description TEXT,
    is_active ENUM('ACTIVE', 'INACTIVE') DEFAULT 'ACTIVE',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```
![table schedules](images/semana3-mysql1-table-schedules.png)
![table schedules visualizacion](images/semana3-mysql1-table-schedules1.png)

```bash
CREATE TABLE buses (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    description TEXT,
    is_active ENUM('ACTIVE', 'INACTIVE') DEFAULT 'ACTIVE',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```
![table buses](images/semana3-mysql1-table-buses.png)
![table buses visualizacion](images/semana3-mysql1-table-buses2.png)

```bash
CREATE TABLE conductors (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    description TEXT,
    is_active ENUM('ACTIVE', 'INACTIVE') DEFAULT 'ACTIVE',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```
![table conductors](images/semana3-mysql1-table-conductors.png)
![table conductors visualizacion](images/semana3-mysql1-table-incidents2.png)
```bash
CREATE TABLE assignments (
    id INT AUTO_INCREMENT PRIMARY KEY,
    schedule_id INT NOT NULL,
    bus_id INT NOT NULL,
    conductor_id INT NOT NULL,
    is_active ENUM('ACTIVE', 'INACTIVE') DEFAULT 'ACTIVE',
    FOREIGN KEY (schedule_id) REFERENCES schedules(id),
    FOREIGN KEY (bus_id) REFERENCES buses(id),
    FOREIGN KEY (conductor_id) REFERENCES conductors(id)
);
```
![table assignments](images/semana3-mysql1-table-assignments.png)
![table assignments visualizacion](images/semana3-mysql1-table-assignments2.png)
```bash
CREATE TABLE incidents (
    id INT AUTO_INCREMENT PRIMARY KEY,
    assignment_id INT NOT NULL,
    type VARCHAR(100) NOT NULL,
    date DATE NOT NULL,
    description TEXT,
    is_active ENUM('ACTIVE', 'INACTIVE') DEFAULT 'ACTIVE',
    FOREIGN KEY (assignment_id) REFERENCES assignments(id)
);
```
![table incidents](images/semana3-mysql1-table-incidents.png)
![table incidents visualizacion](images/semana3-mysql1-table-incidents2.png)
```bash
CREATE TABLE maintenance (
    id INT AUTO_INCREMENT PRIMARY KEY,
    bus_id INT NOT NULL,
    type VARCHAR(100) NOT NULL,
    scheduled_date DATE NOT NULL,
    close_date DATE,
    cost DECIMAL(10,2),
    status VARCHAR(50) DEFAULT 'PENDING',
    FOREIGN KEY (bus_id) REFERENCES buses(id)
);
```

![table maintenance](images/semana3-mysql1-table-maintenance.png)
![table maintenance visualizacion](images/semana3-mysql1-table-maintenance2.png)

**CREAR LA BASE DE DATOS EN WORKBENCH**

## 1: Abrir MySQL Workbench

1. Abre MySQL Workbench desde Windows
2. Conéctate a tu servidor MySQL (Host: 172.24.20.10, Port: 3306, User: harold, Password: 123456)

![conectar](images/semana3-workbench-conexion.png)

## 2: Crear la base de datos BasededatosMysql2

![crear base de datos](images/semana3-workbench-create-database.png)

## 3: Crear la tabla **routes**
![crear tbala](images/semana3-workbench-tables-routes.png)

## 4: Crear la tabla **stops**
![crear tbala](images/semana3-workbench-table-stop.png)

## 5: Crear la tabla **route_stops**
![crear tbala](images/semana3-workbench-table-route-stops-fk.png)
![crear tbala](images/semana3-workbench-table-route-stops-fk2.png)
![crear tbala](images/semana3-workbench-table-route_stops.png)

## 6: Crear la tabla **schedules**
![crear tbala](images/semana3-workbench-table-schedules.png)

## 7: Crear la tabla **buses**
![crear tbala](images/semana3-workbench-table-buses.png)

## 8: Crear la tabla **conductors**
![crear tbala](images/semana3-workbench-table-conductors.png)

## 9: Crear la tabla **assignments**
![crear tbala](images/semana3-workbench-table-assignmentsfk.png)
![crear tbala](images/semana3-workbench-table-assignments.png)

## 10: Crear la tabla **incidents**
![crear tbala](images/semana3-workbench-table-incidentsfk.png)
![crear tbala](images/semana3-workbench-table-incidents.png)


## 11: Crear la tabla **maintenance**
![crear tbala](images/semana3-workbench-table-maintenancefk.png)
![crear tbala](images/semana3-workbench-table-maintenance.png)


# *PostgreSQL DBeaver*

Se creó la base de datos BasededatosPostgres1 utilizando la interfaz gráfica de DBeaver.

## 1. Conectar al conetenedor:
![conexion](images/semana3-postgres-dbeaver-conexion.png)

## 2. Creación de la base de datos BasededatosPostgres1.
![conexion](images/semana3-postgres1-database.png)

## 4. Tabla routes:
![conexion](images/semana3-postgres1-routes.png)

## 5. Tabla stops:
![conexion](images/semana3-postgres1-stops.png)

## 6. Tabla schedules:
![conexion](images/semana3-postgres1-schedules.png)

## 7. Tabla buses:
![conexion](images/semana3-postgres1-buses.png)

## 8. Tabla conductors:
![conexion](images/semana3-postgres1-conductors.png)

## 9. Tabla route_stops:
![conexion](images/semana3-postgres1-route_stops.png)

## 10. Tabla assignments:
![conexion](images/semana3-postgres1-assignments.png)

## 11. Tabla incidents :
![conexion](images/semana3-postgres1-incidents.png)

## 12. Tabla maintenance:
![conexion](images/semana3-postgres1-maintenance.png)

## PostgreSQL pgAdmin

Se creó la base de datos `BasededatosPostgres2` utilizando la interfaz gráfica de pgAdmin.

## 1. Conectar al sevidor:
![conexion](images/semana3-pgadmin-conexion.png)

## 2. Creación de la base de datos BasededatosPostgres1.
```bash
CREATE DATABASE BasededatosPostgres2;
```
![conexion](images/semana3-pgadmin2-database.png)

## 4. Tabla routes:
```bash
CREATE TABLE public.routes (
    id serial NOT NULL,
    name varchar(100) NOT NULL,
    description text NULL,
    is_active varchar(20) DEFAULT 'ACTIVE' NULL,
    created_at timestamp DEFAULT CURRENT_TIMESTAMP NULL,
    updated_at timestamp DEFAULT CURRENT_TIMESTAMP NULL,
    CONSTRAINT routes_pk PRIMARY KEY (id)
);
```
![conexion](images/semana3-pgadmin-routes.png)
![conexion](images/semana3-pgadmin-routes2.png)

## 5. Tabla stops:
```bash
CREATE TABLE public.stops (
    id serial NOT NULL,
    name varchar(100) NOT NULL,
    description text NULL,
    is_active varchar(20) DEFAULT 'ACTIVE' NULL,
    created_at timestamp DEFAULT CURRENT_TIMESTAMP NULL,
    updated_at timestamp DEFAULT CURRENT_TIMESTAMP NULL,
    CONSTRAINT stops_pk PRIMARY KEY (id)
);
```
![conexion](images/semana3-pgadmin-stops.png)
![conexion](images/semana3-pgadmin-stops2.png)

## 6. Tabla schedules:
```bash
CREATE TABLE public.schedules (
    id serial NOT NULL,
    name varchar(100) NOT NULL,
    description text NULL,
    is_active varchar(20) DEFAULT 'ACTIVE' NULL,
    created_at timestamp DEFAULT CURRENT_TIMESTAMP NULL,
    updated_at timestamp DEFAULT CURRENT_TIMESTAMP NULL,
    CONSTRAINT schedules_pk PRIMARY KEY (id)
);
```
![conexion](images/semana3-pgadmin-schedules.png)
![conexion](images/semana3-pgadmin-schedules2.png)

## 7. Tabla buses:
```bash
CREATE TABLE public.buses (
    id serial NOT NULL,
    name varchar(100) NOT NULL,
    description text NULL,
    is_active varchar(20) DEFAULT 'ACTIVE' NULL,
    created_at timestamp DEFAULT CURRENT_TIMESTAMP NULL,
    updated_at timestamp DEFAULT CURRENT_TIMESTAMP NULL,
    CONSTRAINT buses_pk PRIMARY KEY (id)
);
```
![conexion](images/semana3-pgadmin-buses.png)
![conexion](images/semana3-pgadmin-buses2.png)

## 8. Tabla conductors:
```bash
CREATE TABLE public.conductors (
    id serial NOT NULL,
    name varchar(100) NOT NULL,
    description text NULL,
    is_active varchar(20) DEFAULT 'ACTIVE' NULL,
    created_at timestamp DEFAULT CURRENT_TIMESTAMP NULL,
    updated_at timestamp DEFAULT CURRENT_TIMESTAMP NULL,
    CONSTRAINT conductors_pk PRIMARY KEY (id)
);
```
![conexion](images/semana3-pgadmin-conductors.png)
![conexion](images/semana3-pgadmin-conductors2.png)

## 9. Tabla route_stops:
```bash
CREATE TABLE public.route_stops (
    id serial NOT NULL,
    route_id int NOT NULL,
    stop_id int NOT NULL,
    stop_order int NOT NULL,
    is_active varchar(20) DEFAULT 'ACTIVE' NULL,
    CONSTRAINT route_stops_pk PRIMARY KEY (id),
    CONSTRAINT route_stops_routes_fk FOREIGN KEY (route_id) REFERENCES public.routes(id),
    CONSTRAINT route_stops_stops_fk FOREIGN KEY (stop_id) REFERENCES public.stops(id),
    CONSTRAINT route_stops_unique UNIQUE (route_id, stop_id)
);
```
![conexion](images/semana3-pgadmin-route_stops.png)
![conexion](images/semana3-pgadmin-route_stops2.png)

## 10. Tabla assignments:
```bash
CREATE TABLE public.assignments (
    id serial NOT NULL,
    schedule_id int NOT NULL,
    bus_id int NOT NULL,
    conductor_id int NOT NULL,
    is_active varchar(20) DEFAULT 'ACTIVE' NULL,
    CONSTRAINT assignments_pk PRIMARY KEY (id),
    CONSTRAINT assignments_schedules_fk FOREIGN KEY (schedule_id) REFERENCES public.schedules(id),
    CONSTRAINT assignments_buses_fk FOREIGN KEY (bus_id) REFERENCES public.buses(id),
    CONSTRAINT assignments_conductors_fk FOREIGN KEY (conductor_id) REFERENCES public.conductors(id)
);
```
![conexion](images/semana3-pgadmin-assignments.png)
![conexion](images/semana3-pgadmin-assignments2.png)

## 11. Tabla incidents :
```bash
CREATE TABLE public.incidents (
    id serial NOT NULL,
    assignment_id int NOT NULL,
    type varchar(100) NOT NULL,
    date date NOT NULL,
    description text NULL,
    is_active varchar(20) DEFAULT 'ACTIVE' NULL,
    CONSTRAINT incidents_pk PRIMARY KEY (id),
    CONSTRAINT incidents_assignments_fk FOREIGN KEY (assignment_id) REFERENCES public.assignments(id)
);
```
![conexion](images/semana3-pgadmin-incidents.png)
![conexion](images/semana3-pgadmin-incidents2.png)

## 12. Tabla maintenance:
```bash
CREATE TABLE public.maintenance (
    id serial NOT NULL,
    bus_id int NOT NULL,
    type varchar(100) NOT NULL,
    scheduled_date date NOT NULL,
    close_date date NULL,
    cost decimal(10,2) NULL,
    status varchar(50) DEFAULT 'PENDING' NULL,
    CONSTRAINT maintenance_pk PRIMARY KEY (id),
    CONSTRAINT maintenance_buses_fk FOREIGN KEY (bus_id) REFERENCES public.buses(id)
);
```
![Tabla maintenance](images/semana3-pgadmin-maintenance.png)

![Estructura maintenance](images/semana3-pgadmin-maintenance2.png)

---