# Semana 4: Base de Datos RutaCapital (SQL Server y Oracle)

**Autor:** Harold Segundo Gonzalez  
**Docente:** Jaider J. Quintero Mendoza  
**Curso:** Base de Datos 2  
**Fecha:** 2026

---

## 📋 Tabla de Contenidos

1. [Descripción del Proyecto](#1-descripción-del-proyecto)
2. [Reglas de Nomenclatura](#2-reglas-de-nomenclatura)
3. [Modelo Entidad-Relación](#3-modelo-entidad-relación)
   - 3.1 [Descripción de las relaciones](#31-descripción-de-las-relaciones)
   - 3.2 [Diccionario de datos](#32-diccionario-de-datos)
   - 3.3 [Decisiones de diseño](#33-decisiones-de-diseño)
   - 3.4 [Orden de creación](#34-orden-de-creación)
4. [SQL Server](#4-sql-server)
   - 4.1 [Terminal DBeaver](#41-terminal-dbeaver)
   - 4.2 [Crear la base de datos y las tablas](#42-crear-la-base-de-datos-y-las-tablas)
5. [Oracle](#5-oracle)
   - 5.1 [Terminal DBeaver](#51-terminal-dbeaver)
   - 5.2 [SQL Developer](#52-sql-developer)
6. [Conclusiones](#6-conclusiones)

---

## 1. Descripción del Proyecto

RutaCapital administra rutas urbanas, paradas ordenadas, horarios, buses y conductores. La solución debe planear asignaciones sin cruces de turno, registrar salidas reales, incidentes y mantenimientos que afecten la disponibilidad de la flota.

---

## 2. Reglas de Nomenclatura

- **Tablas:** minúsculas, en plural, en inglés
- **Llaves foráneas:** `nombre_tabla_singular + _id`

---

## 3. Modelo Entidad-Relación

| Entidad | Tabla | Atributos |
|---------|-------|-----------|
| Ruta | `routes` | id, name, description, is_active, created_at, updated_at |
| Parada | `stops` | id, name, description, is_active, created_at, updated_at |
| Horario | `schedules` | id, name, description, is_active, created_at, updated_at |
| Bus | `buses` | id, name, description, is_active, created_at, updated_at |
| Conductor | `conductors` | id, name, description, is_active, created_at, updated_at |
| RutaParada | `route_stops` | id, route_id, stop_id, stop_order, is_active |
| Asignacion | `assignments` | id, schedule_id, bus_id, conductor_id, is_active |
| Incidente | `incidents` | id, assignment_id, type, date, description, is_active |
| Mantenimiento | `maintenance` | id, bus_id, type, scheduled_date, close_date, cost, status |

### 3.1 Descripción de las relaciones

El modelo usa las siguientes relaciones para representar la operación de la empresa:

| Relación | Tipo | Explicación |
|----------|------|-------------|
| `routes` - `route_stops` | 1 a muchos | Una ruta puede tener varias paradas. |
| `stops` - `route_stops` | 1 a muchos | Una parada puede aparecer en varias rutas. La tabla intermedia también guarda el orden de la parada. |
| `schedules` - `assignments` | 1 a muchos | Un horario puede utilizarse en varias asignaciones. |
| `buses` - `assignments` | 1 a muchos | Un bus puede participar en varias asignaciones a lo largo del tiempo. |
| `conductors` - `assignments` | 1 a muchos | Un conductor puede tener varias asignaciones a lo largo del tiempo. |
| `assignments` - `incidents` | 1 a muchos | Una asignación puede registrar varios incidentes. |
| `buses` - `maintenance` | 1 a muchos | Un bus puede tener varios registros de mantenimiento. |

`route_stops` es una tabla intermedia porque la relación entre rutas y paradas es de muchos a muchos: una ruta contiene varias paradas y una parada puede pertenecer a varias rutas.

### 3.2 Diccionario de datos

El diccionario de datos describe la finalidad de cada tabla y de sus columnas. En las tablas principales, `id` es la clave primaria y `is_active` permite desactivar registros sin eliminarlos físicamente.

#### Tabla `routes`

Almacena las rutas disponibles para el servicio de transporte.

| Columna | Tipo | Descripción |
|---------|------|-------------|
| `id` | `INT IDENTITY` | Identificador único y autogenerado de la ruta. |
| `name` | `VARCHAR(100)` | Nombre de la ruta. Es obligatorio. |
| `description` | `TEXT` | Información adicional de la ruta. |
| `is_active` | `VARCHAR(20)` | Estado lógico del registro; por defecto `ACTIVE`. |
| `created_at` | `DATETIME` | Fecha y hora de creación; usa `GETDATE()` por defecto. |
| `updated_at` | `DATETIME` | Fecha y hora de actualización inicial; usa `GETDATE()` por defecto. |

#### Tabla `stops`

Registra las paradas donde los usuarios pueden abordar o bajar del bus. Sus columnas tienen la misma estructura general de `routes`: `id` identifica la parada, `name` guarda su nombre, `description` sus detalles, `is_active` su estado y `created_at`/`updated_at` las fechas de control.

#### Tabla `schedules`

Define los horarios o turnos que pueden asignarse a un bus y a un conductor. Contiene `id`, `name`, `description`, `is_active`, `created_at` y `updated_at`, con la misma función descrita para las tablas principales.

#### Tabla `buses`

Contiene el inventario de buses de la empresa. `id` identifica cada bus, `name` permite reconocerlo, `description` registra sus características, `is_active` indica si está disponible para nuevas operaciones y las columnas de fecha controlan su registro.

#### Tabla `conductors`

Contiene el registro de conductores. `id` identifica al conductor, `name` almacena su nombre, `description` permite agregar información adicional, `is_active` indica si puede recibir asignaciones y `created_at`/`updated_at` mantienen la trazabilidad básica.

#### Tabla `route_stops`

Relaciona cada ruta con sus paradas y conserva el orden en que deben recorrerse.

| Columna | Tipo | Descripción |
|---------|------|-------------|
| `id` | `INT IDENTITY` | Identificador único de la relación. |
| `route_id` | `INT` | Clave foránea que apunta a `routes.id`. |
| `stop_id` | `INT` | Clave foránea que apunta a `stops.id`. |
| `stop_order` | `INT` | Posición de la parada dentro de la ruta. |
| `is_active` | `VARCHAR(20)` | Estado lógico de la relación. |

La restricción `route_stops_unique` evita registrar dos veces la misma combinación de ruta y parada.

#### Tabla `assignments`

Representa la asignación operativa de un horario, un bus y un conductor.

| Columna | Tipo | Descripción |
|---------|------|-------------|
| `id` | `INT IDENTITY` | Identificador único de la asignación. |
| `schedule_id` | `INT` | Clave foránea hacia `schedules.id`. |
| `bus_id` | `INT` | Clave foránea hacia `buses.id`. |
| `conductor_id` | `INT` | Clave foránea hacia `conductors.id`. |
| `is_active` | `VARCHAR(20)` | Indica si la asignación está vigente. |

Las tres claves foráneas garantizan que no se pueda asignar un horario, bus o conductor inexistente.

#### Tabla `incidents`

Registra novedades ocurridas durante una asignación.

| Columna | Tipo | Descripción |
|---------|------|-------------|
| `id` | `INT IDENTITY` | Identificador único del incidente. |
| `assignment_id` | `INT` | Clave foránea hacia la asignación relacionada. |
| `type` | `VARCHAR(100)` | Tipo o categoría del incidente. Es obligatorio. |
| `date` | `DATE` | Día en que ocurrió el incidente. Es obligatorio. |
| `description` | `TEXT` | Detalle de lo sucedido. |
| `is_active` | `VARCHAR(20)` | Estado lógico del registro. |

#### Tabla `maintenance`

Planifica y registra los mantenimientos realizados a los buses.

| Columna | Tipo | Descripción |
|---------|------|-------------|
| `id` | `INT IDENTITY` | Identificador único del mantenimiento. |
| `bus_id` | `INT` | Clave foránea hacia `buses.id`. |
| `type` | `VARCHAR(100)` | Tipo de mantenimiento. Es obligatorio. |
| `scheduled_date` | `DATE` | Fecha programada para el mantenimiento. Es obligatoria. |
| `close_date` | `DATE` | Fecha de finalización; puede quedar vacía mientras esté pendiente. |
| `cost` | `DECIMAL(10,2)` | Costo del mantenimiento con dos decimales. |
| `status` | `VARCHAR(50)` | Estado del proceso; por defecto `PENDING`. |

### 3.3 Decisiones de diseño

- Se utilizan claves primarias autogeneradas (`IDENTITY`) para evitar que el usuario tenga que asignar manualmente los identificadores.
- Las claves foráneas protegen la integridad referencial: no se pueden crear relaciones con registros que no existen.
- `NOT NULL` se aplica a los datos indispensables para identificar o utilizar una entidad.
- Los estados (`is_active` y `status`) permiten conservar el historial y cambiar la disponibilidad de un registro sin borrarlo.
- La restricción `UNIQUE` de `route_stops` evita duplicar una misma parada dentro de una ruta.

### 3.4 Orden de creación

Las tablas deben crearse en este orden para que las claves foráneas encuentren primero las tablas referenciadas:

1. Crear la base de datos y seleccionarla con `USE`.
2. Crear las tablas principales: `routes`, `stops`, `schedules`, `buses` y `conductors`.
3. Crear `route_stops`, que depende de `routes` y `stops`.
4. Crear `assignments`, que depende de `schedules`, `buses` y `conductors`.
5. Crear `incidents`, que depende de `assignments`.
6. Crear `maintenance`, que depende de `buses`.

---

## 4. SQL Server

### 4.1  Abrir DBeaver

1. Abre DBeaver desde Windows
2. Haz clic en Nueva Conexión (enchufe con "+")
3. Selecciona SQL Server
4. Llenar los datos

![Conexión a SQL Server en DBeaver](images/semana4-sqlserver-dbeaver-conexion.png)
![Visualización gráfica de la conexión a SQL Server en DBeaver](images/semana4-sqlserver-dbeaver-conexion2.png)


### 4.2 Crear la base de datos desde la terminal de DBeaver

Abre el editor SQL de DBeaver y ejecuta los siguientes comandos en el orden indicado.

#### 4.2.1 Crear la base de datos

```sql
CREATE DATABASE BasededatosSqlServer1;
```

![Base de datos BasededatosSqlServer1](images/semana4-sqlserver-dbeaver-base-de-datos.png)

#### 4.2.2 Seleccionar la base de datos

```sql
USE BasededatosSqlServer1;
```

![Selección de la base de datos BasededatosSqlServer1](images/semana4-sqlserver-dbeaver-use.png)

#### 4.2.3 Crear las tablas principales

Las tablas principales no tienen dependencias entre ellas y pueden crearse en cualquier orden.

**Tabla: `routes`**

```sql
CREATE TABLE routes (
   id INT IDENTITY(1,1) PRIMARY KEY,
   name VARCHAR(100) NOT NULL,
   description TEXT,
   is_active VARCHAR(20) DEFAULT 'ACTIVE',
   created_at DATETIME DEFAULT GETDATE(),
   updated_at DATETIME DEFAULT GETDATE()
);
```

![Tabla routes](images/semana4-sqlserver-dbeaver-toutes.png)
![Visualización gráfica de la tabla routes](images/semana4-sqlserver-dbeaver-routes2.png)

**Tabla: `stops`**

```sql
CREATE TABLE stops (
   id INT IDENTITY(1,1) PRIMARY KEY,
   name VARCHAR(100) NOT NULL,
   description TEXT,
   is_active VARCHAR(20) DEFAULT 'ACTIVE',
   created_at DATETIME DEFAULT GETDATE(),
   updated_at DATETIME DEFAULT GETDATE()
);
```

![Tabla stops](images/semana4-sqlserver-dbeaver-stops.png)
![Visualización gráfica de la tabla stops](images/semana4-sqlserver-dbeaver-stops2.png)

**Tabla: `schedules`**

```sql
CREATE TABLE schedules (
   id INT IDENTITY(1,1) PRIMARY KEY,
   name VARCHAR(100) NOT NULL,
   description TEXT,
   is_active VARCHAR(20) DEFAULT 'ACTIVE',
   created_at DATETIME DEFAULT GETDATE(),
   updated_at DATETIME DEFAULT GETDATE()
);
```

![Tabla schedules](images/semana4-sqlserver-dbeaver-schedules.png)
![Visualización gráfica de la tabla schedules](images/semana4-sqlserver-dbeaver-schedules2.png)

**Tabla: `buses`**

```sql
CREATE TABLE buses (
   id INT IDENTITY(1,1) PRIMARY KEY,
   name VARCHAR(100) NOT NULL,
   description TEXT,
   is_active VARCHAR(20) DEFAULT 'ACTIVE',
   created_at DATETIME DEFAULT GETDATE(),
   updated_at DATETIME DEFAULT GETDATE()
);
```

![Tabla buses](images/semana4-sqlserver-dbeaver-buses.png)
![Visualización gráfica de la tabla buses](images/semana4-sqlserver-dbeaver-buses2.png)

**Tabla: `conductors`**

```sql
CREATE TABLE conductors (
   id INT IDENTITY(1,1) PRIMARY KEY,
   name VARCHAR(100) NOT NULL,
   description TEXT,
   is_active VARCHAR(20) DEFAULT 'ACTIVE',
   created_at DATETIME DEFAULT GETDATE(),
   updated_at DATETIME DEFAULT GETDATE()
);
```

![Tabla conductors](images/semana4-sqlserver-dbeaver-conductors.png)
![Visualización gráfica de la tabla conductors](images/semana4-sqlserver-dbeaver-conductors2.png)

#### 4.2.4 Crear las tablas relacionadas

`route_stops` depende de `routes` y `stops`, por lo que debe crearse después de ellas.

**Tabla: `route_stops`**

```sql
CREATE TABLE route_stops (
   id INT IDENTITY(1,1) PRIMARY KEY,
   route_id INT NOT NULL,
   stop_id INT NOT NULL,
   stop_order INT NOT NULL,
   is_active VARCHAR(20) DEFAULT 'ACTIVE',
   CONSTRAINT route_stops_routes_fk FOREIGN KEY (route_id) REFERENCES routes(id),
   CONSTRAINT route_stops_stops_fk FOREIGN KEY (stop_id) REFERENCES stops(id),
   CONSTRAINT route_stops_unique UNIQUE (route_id, stop_id)
);
```

![Tabla route_stops](images/semana4-sqlserver-dbeaver-route_stops.png)
![Visualización gráfica de la tabla route_stops](images/semana4-sqlserver-dbeaver-route_stops2.png)

`assignments` depende de `schedules`, `buses` y `conductors`.

**Tabla: `assignments`**

```sql
CREATE TABLE assignments (
   id INT IDENTITY(1,1) PRIMARY KEY,
   schedule_id INT NOT NULL,
   bus_id INT NOT NULL,
   conductor_id INT NOT NULL,
   is_active VARCHAR(20) DEFAULT 'ACTIVE',
   CONSTRAINT assignments_schedules_fk FOREIGN KEY (schedule_id) REFERENCES schedules(id),
   CONSTRAINT assignments_buses_fk FOREIGN KEY (bus_id) REFERENCES buses(id),
   CONSTRAINT assignments_conductors_fk FOREIGN KEY (conductor_id) REFERENCES conductors(id)
);
```

![Tabla assignments](images/semana4-sqlserver-dbeaver-assignments.png)
![Visualización gráfica de la tabla assignments](images/semana4-sqlserver-dbeaver-assignments2.png)

`incidents` depende de `assignments`.

**Tabla: `incidents`**

```sql
CREATE TABLE incidents (
   id INT IDENTITY(1,1) PRIMARY KEY,
   assignment_id INT NOT NULL,
   type VARCHAR(100) NOT NULL,
   date DATE NOT NULL,
   description TEXT,
   is_active VARCHAR(20) DEFAULT 'ACTIVE',
   CONSTRAINT incidents_assignments_fk FOREIGN KEY (assignment_id) REFERENCES assignments(id)
);
```

![Tabla incidents](images/semana4-sqlserver-dbeaver-incidents.png)
![Visualización gráfica de la tabla incidents](images/semana4-sqlserver-dbeaver-incidents2.png)

`maintenance` depende de `buses`.

**Tabla: `maintenance`**

```sql
CREATE TABLE maintenance (
   id INT IDENTITY(1,1) PRIMARY KEY,
   bus_id INT NOT NULL,
   type VARCHAR(100) NOT NULL,
   scheduled_date DATE NOT NULL,
   close_date DATE,
   cost DECIMAL(10,2),
   status VARCHAR(50) DEFAULT 'PENDING',
   CONSTRAINT maintenance_buses_fk FOREIGN KEY (bus_id) REFERENCES buses(id)
);
```

![Tabla maintenance](images/semana4-sqlserver-dbeaver-maintenance.png)
![Visualización gráfica de la tabla maintenance](images/semana4-sqlserver-dbeaver-maintenance2.png)

### Creación de la base de datos mediante la interfaz gráfica de SQL Server Management Studio

**Conexión a SQL Server Management Studio**

![Conexión a SQL Server Management Studio](images/semana4-ssms-conectado.png)

**Creación de la base de datos**

![Creación de la base de datos](images/semana4-ssms-crear-base.png)

**Tabla: `routes`**

![Tabla routes](images/semana4-ssms-routes.png)

**Tabla: `stops`**

![Tabla stops](images/semana4-ssms-stops.png)

**Tabla: `schedules`**

![Tabla schedules](images/semana4-ssms-schedules.png)

**Tabla: `buses`**

![Tabla buses](images/semana4-ssms-buses.png)

**Tabla: `conductors`**

![Tabla conductors](images/semana4-ssms-conductors.png)

**Tabla: `route_stops`**

![Tabla route_stops](images/semana4-ssms-route_stops.png)

**Tabla: `assignments`**

![Tabla assignments](images/semana4-ssms-assignments.png)

**Tabla: `incidents`**

![Tabla incidents](images/semana4-ssms-incidents.png)

---

## 5. Oracle

### 5.1 Terminal DBeaver

La conexión a Oracle se realizó desde DBeaver y los objetos se crearon mediante el editor SQL.

![Conexión a Oracle en DBeaver](images/semana4-oracle-dbeaver-conexion.png)

#### 5.1.1 Crear el tablespace

```sql
CREATE TABLESPACE BasededatosOracle1_ts
DATAFILE '/opt/oracle/oradata/XE/XEPDB1/BasededatosOracle1_ts.dbf'
SIZE 100M
AUTOEXTEND ON;
```

![Creación del tablespace de Oracle](images/semana4-oracle-dbeaver-crearbase.png)

#### 5.1.2 Crear las tablas principales

**Tabla: `routes`**

```sql
CREATE TABLE routes (
   id NUMBER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
   "name" VARCHAR2(100) NOT NULL,
   description CLOB,
   is_active VARCHAR2(20) DEFAULT 'ACTIVE',
   created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
   updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

![Tabla routes en Oracle](images/semana4-oracle-dbeaver-routes.png)

**Tabla: `stops`**

```sql
CREATE TABLE stops (
   id NUMBER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
   "name" VARCHAR2(100) NOT NULL,
   description CLOB,
   is_active VARCHAR2(20) DEFAULT 'ACTIVE',
   created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
   updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

![Tabla stops en Oracle](images/semana4-oracle-dbeaver-stop.png)

**Tabla: `schedules`**

```sql
CREATE TABLE schedules (
   id NUMBER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
   "name" VARCHAR2(100) NOT NULL,
   description CLOB,
   is_active VARCHAR2(20) DEFAULT 'ACTIVE',
   created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
   updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

![Tabla schedules en Oracle](images/semana4-oracle-dbeaver-schedules.png)

**Tabla: `buses`**

```sql
CREATE TABLE buses (
   id NUMBER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
   "name" VARCHAR2(100) NOT NULL,
   description CLOB,
   is_active VARCHAR2(20) DEFAULT 'ACTIVE',
   created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
   updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

![Tabla buses en Oracle](images/semana4-oracle-dbeaver-buses.png)

**Tabla: `conductors`**

```sql
CREATE TABLE conductors (
   id NUMBER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
   "name" VARCHAR2(100) NOT NULL,
   description CLOB,
   is_active VARCHAR2(20) DEFAULT 'ACTIVE',
   created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
   updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

![Tabla conductors en Oracle](images/semana4-oracle-dbeaver-conductors.png)

#### 5.1.3 Crear las tablas relacionadas

**Tabla: `route_stops`**

```sql
CREATE TABLE route_stops (
   id NUMBER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
   route_id NUMBER NOT NULL,
   stop_id NUMBER NOT NULL,
   stop_order NUMBER NOT NULL,
   is_active VARCHAR2(20) DEFAULT 'ACTIVE',
   CONSTRAINT route_stops_routes_fk FOREIGN KEY (route_id) REFERENCES routes(id),
   CONSTRAINT route_stops_stops_fk FOREIGN KEY (stop_id) REFERENCES stops(id),
   CONSTRAINT route_stops_unique UNIQUE (route_id, stop_id)
);
```

![Tabla route_stops en Oracle](images/semana4-oracle-dbeaver-route_stops.png)

**Tabla: `assignments`**

```sql
CREATE TABLE assignments (
   id NUMBER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
   schedule_id NUMBER NOT NULL,
   bus_id NUMBER NOT NULL,
   conductor_id NUMBER NOT NULL,
   is_active VARCHAR2(20) DEFAULT 'ACTIVE',
   CONSTRAINT assignments_schedules_fk FOREIGN KEY (schedule_id) REFERENCES schedules(id),
   CONSTRAINT assignments_buses_fk FOREIGN KEY (bus_id) REFERENCES buses(id),
   CONSTRAINT assignments_conductors_fk FOREIGN KEY (conductor_id) REFERENCES conductors(id)
);
```

![Tabla assignments en Oracle](images/semana4-oracle-dbeaver-assignments.png)

**Tabla: `incidents`**

```sql
CREATE TABLE incidents (
   id NUMBER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
   assignment_id NUMBER NOT NULL,
   "type" VARCHAR2(100) NOT NULL,
   "date" DATE NOT NULL,
   description CLOB,
   is_active VARCHAR2(20) DEFAULT 'ACTIVE',
   CONSTRAINT incidents_assignments_fk FOREIGN KEY (assignment_id) REFERENCES assignments(id)
);
```

![Tabla incidents en Oracle](images/semana4-oracle-dbeaver-incidents.png)

**Tabla: `maintenance`**

```sql
CREATE TABLE maintenance (
   id NUMBER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
   bus_id NUMBER NOT NULL,
   "type" VARCHAR2(100) NOT NULL,
   scheduled_date DATE NOT NULL,
   close_date DATE,
   cost NUMBER(10,2),
   status VARCHAR2(50) DEFAULT 'PENDING',
   CONSTRAINT maintenance_buses_fk FOREIGN KEY (bus_id) REFERENCES buses(id)
);
```

![Tabla maintenance en Oracle](images/semana4-oracle-dbeaver-maintenance.png)

#### 5.1.4 Verificar las tablas

```sql
SELECT table_name FROM user_tables;
```

Las tablas creadas fueron `routes`, `stops`, `schedules`, `buses`, `conductors`, `route_stops`, `assignments`, `incidents` y `maintenance`.

### 5.2 SQL Developer

> **Pendiente**

---


**Autor:** Harold Segundo Gonzalez  
**Docente:** Jaider J. Quintero Mendoza  
**Curso:** Base de Datos 2  
**Fecha:** 2026
