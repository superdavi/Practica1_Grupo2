## INTEGRANTES.
| Nombre | Cargo | URL GitHub |
|---|:---:|---:|
| Daniel Alquinga | 🐛 Desarrollador | https://github.com/superdavi/Practica1_Grupo2.git |
| Daniel Baldeon | 🐛 Desarrollador |  |
| Bryan Miño | 🐛 Desarrollador | |
| Wilson Segovia | 🐛 Desarrollador | |
| Leonardo Tuguminago | 🐛 Desarrollador | |

# Sistema de Gestión de Vehículos con Docker

Este proyecto implementa un sistema de gestión de vehículos utilizando Docker, MySQL y phpMyAdmin. El sistema permite administrar propietarios y sus vehículos mediante una base de datos relacional.

## Arquitectura del Sistema

- **MySQL 8.3**: Base de datos para almacenar información de propietarios y vehículos
- **phpMyAdmin 5.2.2**: Interfaz web para administración de la base de datos
- **Docker Network**: Red personalizada para comunicación entre contenedores

## Configuración e Instalación

### PASO 1: Configuración de Contenedores Docker

```bash
# MySQL
docker run -d \
--name db-mysql-vehiculos \
--network netw-vehiculos \
--env-file .env \
-v mysql_data:/var/lib/mysql \
-v "$PWD"/init.sql:/docker-entrypoint-initdb.d/init.sql \
-p 3306:3306 \
mysql:8.3

# phpMyAdmin
docker run -d \
--name web_phpmyadmin_vehiculos \
--network netw-vehiculos \
--env-file .env \
-e PMA_HOST=db-mysql-vehiculos \
-p 8080:80 \
phpmyadmin:5.2.2
```

**Explicación:**
- **MySQL Container**: Crea un contenedor con MySQL 8.3, monta un volumen persistente para los datos y ejecuta un script de inicialización
- **phpMyAdmin Container**: Proporciona interfaz web conectada al contenedor MySQL
- **Network**: Ambos contenedores utilizan la red personalizada `netw-vehiculos`
- **Volumes**: Persistencia de datos MySQL y script de inicialización
- **Ports**: MySQL en puerto 3306, phpMyAdmin en puerto 8080
**Captura de la Ejecución**
<img width="886" height="609" alt="image" src="https://github.com/user-attachments/assets/ee9c91e9-a51f-4880-9fc3-50803a45073c" />

### PASO 2: Estructura de Base de Datos (init.sql)

```sql
-- Crear tabla de propietario
CREATE TABLE propietario (
    id INT AUTO_INCREMENT PRIMARY KEY,
    cedula VARCHAR(15) NOT NULL,
    nombre VARCHAR(100) NOT NULL,
    telefono VARCHAR(20),
    direccion VARCHAR(150)
);

-- Crear tabla de vehiculo
CREATE TABLE vehiculo (
    id INT AUTO_INCREMENT PRIMARY KEY,
    placa VARCHAR(8) NOT NULL,
    marca VARCHAR(50) NOT NULL,
    modelo VARCHAR(50) NOT NULL,
    anio INT NOT NULL,
    propietario_id INT,
    FOREIGN KEY (propietario_id) REFERENCES propietario(id)
);

-- Insertar registros en propietario
INSERT INTO propietario (cedula, nombre, telefono, direccion) VALUES
('1111111111', 'Daniel Alquinga', '0992222222', 'ABC'),
('1222222222', 'Leonardo Tuguminago', '0993333333', 'DEF'),
('1333333333', 'Bryan Miño', '0994444444', 'GHI'),
('1444444444', 'Wilson Segovia', '0995555555', 'JKL'),
('1555555555', 'Daniel Baldeon', '0996666666', 'MNO');

-- Insertar registros en vehiculo
INSERT INTO vehiculo (placa, marca, modelo, anio, propietario_id) VALUES
('PBB-5555', 'Hyundai', 'i10', 2018, 1),
('PBB-2222', 'Mazda', 'Cx3', 2012, 2),
('PBB-3333', 'Chevrolet', 'Aveo', 2017, 3),
('PBB-9999', 'Nissan', 'Sentra', 2019, 4),
('PCC-4444', 'Toyota', '4Runner', 2022, 5);
```

**Explicación:**
- **Tabla propietario**: Almacena información personal de los dueños de vehículos
- **Tabla vehiculo**: Contiene datos de los vehículos con referencia al propietario
- **Relación**: Clave foránea entre vehículo y propietario (1:N)
- **Datos de prueba**: 5 propietarios y 5 vehículos para testing
**Captura de la Ejecución**
<img width="886" height="886" alt="image" src="https://github.com/user-attachments/assets/0d18e38a-4db4-466e-b965-7ddac3243d8c" />

### PASO 3: Creación de Red Docker

```bash
docker network create --driver bridge netw-vehiculos
docker network ls
```
**Captura de la Ejecución**

<img width="886" height="213" alt="image" src="https://github.com/user-attachments/assets/08eda610-37bd-415f-aed4-d355f445a46b" />

**Salida esperada:**
```
NETWORK ID     NAME             DRIVER    SCOPE
c768b266264f   bridge           bridge    local
e21e631237e9   host             host      local
ef173c5efe1b   netw-vehiculos   bridge    local
dd26efa30c17   none             null      local
```
**Captura de la Ejecución**
<img width="886" height="213" alt="image" src="https://github.com/user-attachments/assets/91472fb3-f8d7-495c-b728-c12773b8fc36" />

**Explicación:**
- Crea una red personalizada tipo `bridge` llamada `netw-vehiculos`
- Permite comunicación entre contenedores por nombre
- Aislamiento de red del resto del sistema

## PASO 4 :: Descarga y Ejecución de MySql
# Ejecutar el contenedor MySQL
docker run -d --name db-mysql-vehiculos --network netw-vehiculos --env-file .env -v mysql_data:/var/lib/mysql -p 3306:3306 mysql:8.3

# Verificar estado de contenedores
docker ps -a
CONTAINER ID   IMAGE       COMMAND                  CREATED         STATUS         PORTS                                                  NAMES
39bc102b715e   mysql:8.3   "docker-entrypoint.s…"  10 seconds ago  Up 9 seconds   0.0.0.0:3306->3306/tcp, :::3306->3306/tcp, 33060/tcp  db-mysql-vehiculos
Explicación:

CONTAINER ID: Identificador único del contenedor (39bc102b715e)
IMAGE: Imagen utilizada (mysql:8.3)
COMMAND: Comando de entrada ejecutado
CREATED/STATUS: Tiempo de creación y estado actual
PORTS: Mapeo de puertos - MySQL accesible en puerto 3306 tanto en IPv4 como IPv6
NAMES: Nombre asignado al contenedor (db-mysql-vehiculos)
Estado: "Up 9 seconds" indica que el contenedor está ejecutándose correctamente

### PASO 5: Descarga y Ejecución de phpMyAdmin

```bash
docker run -d \
--name web_phpmyadmin_vehiculos \
--network netw-vehiculos \
--env-file .env \
-e PMA_HOST=db-mysql-vehiculos \
-p 8080:80 \
phpmyadmin:5.2.2
```

**Salida:**
```
Unable to find image 'phpmyadmin:5.2.2' locally
5.2.2: Pulling from library/phpmyadmin
[Capas descargadas...]
Pull complete
```

**Explicación:**
- Docker descarga la imagen phpMyAdmin si no existe localmente
- Configura la conexión al contenedor MySQL mediante `PMA_HOST`
- Expone el servicio web en el puerto 8080

## Configuración Adicional

### Archivo .env
```env
MYSQL_ROOT_PASSWORD=tu_password_seguro
MYSQL_DATABASE=vehiculos_db
MYSQL_USER=vehiculos_user
MYSQL_PASSWORD=vehiculos_pass
```

### Comandos Útiles

```bash
# Ver contenedores en ejecución
docker ps

# Ver logs de un contenedor
docker logs db-mysql-vehiculos

# Conectar a MySQL directamente
docker exec -it db-mysql-vehiculos mysql -u root -p

# Detener todos los contenedores
docker stop db-mysql-vehiculos web_phpmyadmin_vehiculos

# Eliminar contenedores
docker rm db-mysql-vehiculos web_phpmyadmin_vehiculos

# Eliminar red
docker network rm netw-vehiculos
```

## Acceso al Sistema

- **phpMyAdmin**: http://localhost:8080
- **MySQL directo**: localhost:3306
- **Credenciales**: Configuradas en el archivo `.env`

<img width="886" height="419" alt="image" src="https://github.com/user-attachments/assets/b08f4e44-b7a5-4059-a62f-17b2b74a4ff4" />
<img width="886" height="469" alt="image" src="https://github.com/user-attachments/assets/621bd885-4111-48cf-994e-f85104e800b4" />


## Estructura del Proyecto

```
PracticaGrupo2/
├── .env
├── init.sql
├── despliegue.txt
└── README.md
```

## Características Técnicas

- **Persistencia**: Los datos se mantienen entre reinicios del contenedor
- **Escalabilidad**: Fácil agregar más contenedores o servicios
- **Seguridad**: Red aislada y variables de entorno para credenciales
- **Mantenibilidad**: Configuración declarativa y reproducible

