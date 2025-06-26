# **Base de datos 1**

<div align="justify">
   
La base de datos bd1 se encarga de gestionar toda la información del sistema, específicamente los datos relacionados con los productos del supermercado. Ofrece respaldo estructurado a las aplicaciones (App1 y App2), permitiendo realizar operaciones CRUD a través de consultas SQLmediante consultas SQL, y se ejecuta en un servidor MariaDB configurado para aceptar conexiones remotas, lo que facilita la comunicación con los servidores de aplicación distribuidos en la red.
La base de datos 1 como maestra debe gestionar y almacenar la información principal de la aplicación, permitiendo tanto operaciones de lectura como de escritura. Está configurada con un server-id, tiene habilitado el registro binario (log_bin) y permite conexiones remotas para que los esclavos puedan replicar sus cambios. Además, se creó un usuario con privilegios de replicación (replicador) y se generó un respaldo (mysqldump) con metadatos de replicación que se usó para sincronizar inicialmente al esclavo.

## **Instalación y configuración de MariaDB**
* **Instalamos MariaDB Server:**
```bash
sudo apt update
sudo apt install mariadb-server -y
   ```
* **Ejecutamos el script de seguridad inicial:**
```bash
sudo mysql_secure_installation
   ```
Este desactivará el usuario anónimo, deshabilitará el acceso remoto de root, eliminará la base de datos de prueba, mantendrá la autenticación por contraseña y recargará los privilegios.

## **Base de datos en MariaDB:**
* **Ingresamos al cliente de MariaDB:**
```bash
sudo mysql -u root -p
   ```
* **Creamos la base de datos: Supermercado**
En el caso de proyectos es una BD simple que simula un catálogo de productos y sus precios, alterables por medio de CRUD en las Apps
* **Ingresamos al cliente de MariaDB:**
```bash
sudo mariadb
   ```
* **Creamos BD:**
```bash
CREATE DATABASE supermercado;
   ```
* **Creamos BD usuario para la aplicación:**

 ```bash
CREATE USER 'appuser'@'%' IDENTIFIED BY 'App1@';
GRANT ALL PRIVILEGES ON supermercado.* TO 'appuser'@'%';
FLUSH PRIVILEGES;
  ```

* **Usamos la base de datos y creamos la tabla "productos":**
 ```bash
USE supermercado;

CREATE TABLE productos (
  id INT AUTO_INCREMENT PRIMARY KEY,
  nombre VARCHAR(100),
  precio DECIMAL(10,2)
);

  ```

* **Insertamos productos de ejemplo:**
 ```bash
INSERT INTO productos (nombre, precio) VALUES
('Leche', 2.50),
('Pan', 1.80),
('Arroz', 3.20);
  ```

## **Habilitar conexión remota**

* **Editar archivo de configuración:**
 ```bash
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
  ```

Dentro cambiar:
 ```bash
bind-address = 127.0.0.1
  ```
Por:
 ```bash
bind-address = 0.0.0.0
  ```

* **Reiniciar el servicio MariaDB:**
 ```bash
sudo systemctl restart mariadb
  ```

* **Verificar que el puerto 3306 esté escuchando:**
 ```bash
ss -tlnp | grep 3306
  ```

# **Configuración de la Base de Datos como maestra:**
* **Editar el archivo de configuración:**
 ```bash
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
  ```
Dentro se añade:
 ```bash
bind-address = 0.0.0.0
server-id = 1
log_bin = /var/log/mysql/mariadb-bin
binlog_do_db = tienda       # (opcional si solo replicas esa base)
  ```
Asegúrurase de que el archivo de log exista o que MariaDB lo pueda crear, en caso de que no, utilizar los siguientes comandos:
* Creamos el directorio
```bash
sudo mkdir -p /var/log/mysql
 ```
* Asignamos la propiedad del directorio al usuario y grupo: `mysql`
* ```bash
sudo chown -R mysql:mysql /var/log/mysql
 ```
* Damos permisos al directorio:
```bash
sudo chmod -R 750 /var/log/mysql
 ```
Los permisos son de la siguiente forma:
  * 7 = lectura + escritura + ejecución para el propietario (mysql)
  * 5 = lectura + ejecución para el grupo
  * 0 = ningún acceso para otros usuarios

* **Reiniciamos el servicio para aplicar las modificaciones:**
 ```bash
sudo systemctl restart mariadb
  ```

## **Creamos el usuario de replicación**
* **Conectar a MariaDB:**
 ```bash
sudo mariadb
  ```
Ejecutamos los siguientes comandos:
 ```bash
CREATE USER 'replicador'@'%' IDENTIFIED BY 'TuPasswordFuerte';
GRANT REPLICATION SLAVE ON *.* TO 'replicador'@'%';
FLUSH PRIVILEGES;
  ```
* **Ver estado del log binario:**
 ```bash
SHOW MASTER STATUS;
  ```
Que devuelve la siguiente tabla:
 ```bash
+--------------------+----------+--------------+------------------+
| File               | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+--------------------+----------+--------------+------------------+
| mariadb-bin.000001 |      330 | supermercado |                  |
+--------------------+----------+--------------+------------------+
  ```

* **Crear dump de la base supermercado con metadatos para replicación:**
 ```bash
sudo mysqldump --databases tienda --master-data > tienda.sql
  ```

* **Copiar el dump a la esclava:**
 ```bash
scp -P 77 supermercado.sql ubu2@192.168.235.102:/home/bdesclavo/
  ```
</div>
