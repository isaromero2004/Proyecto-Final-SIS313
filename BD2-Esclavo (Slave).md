# **Base de datos 2 - Esclava (Slave)**
<div align="justify">

**Objetivo dentro del proyecto:** 

La BD 2 como esclava, está configurada para conectarse al maestro y replicar en tiempo real todos los cambios realizados sobre la base de datos tienda. Funciona solo en modo lectura, asegurando la disponibilidad de datos sin afectar la carga del maestro. Configurada con su propio server-id, se importó el respaldo inicial desde el maestro conectándose mediante el usuario replicador. Una vez iniciada, la esclava mantiene la sincronización automáticamente, actuando como respaldo o punto de lectura para balanceo de carga.

---

## Configuración de Red

### Configuración de la IP estática configurada con Netplan:

En el documento: `/etc/netplan/50-cloud-init.yaml`

```bash
network:
  version: 2
  ethernets:
    ens33:
      dhcp4: false
      addresses: [192.168.235.98/24]
      routes:
         - to:  default
           via: 192.168.235.163
      nameservers:
           addresses: [8.8.8.8, 8.8.4.4]
```

Aplicamos la nueva configuración con: 

```bash
sudo netplan apply
```

## **Instalación y configuración de MariaDB**
* **Instalamos MariaDB Server:**
```bash
sudo apt update
sudo apt install mariadb-server -y
   ```
* **Configuramos MariaDB**
```bash
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
   ```
Dentro añadir las configuraciones:
```bash
bind-address = 0.0.0.0
server-id = 2
relay-log = /var/log/mysql/mariadb-relay-bin
```
Asegúrurase de que el archivo de log exista o que MariaDB lo pueda crear, en caso de que no, utilizar los siguientes comandos:
* Creamos el directorio
```bash
sudo mkdir -p /var/log/mysql
 ```
* Asignamos la propiedad del directorio al usuario y grupo: `mysql`
```bash
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

* **Reiniciar el servicio para aplicar los cambios:**
```bash
sudo systemctl restart mariadb
   ```

* **Configuración Básica del Firewall (UFW):**

Si UFW no está habilitado, lo habilitamos con el comando:
   ```bash
   sudo ufw enable
   ```

  Configuramos el firewall para permitir las conexiones entrantes al puerto SSH con el            siguiente comando: 

  ```bash
  sudo ufw allow ssh
  ```

  Si cambiamos el puerto SSH en el archivo `/etc/ssh/sshd_config`, especificamos el nuevo puerto, en este caso 77, usamos el comando:

  ```bash
  sudo ufw allow 77/tcp
  ```

   Habilitamos el puerto de mariadb:
   ```bash
   sudo ufw allow 3306
   ```
   Verifica el estado del firewall con:
   ```bash
   sudo ufw status
   ```

* **Configurar replicación (conectar al maestro):**
```bash
sudo mysql < /home/bdesclavo/supermercado.sql
   ```
* **En caso de que haya un esclavo corriendo**
```bash
sudo mariadb
STOP SLAVE;
RESET SLAVE ALL;
```

* **Configurar replicación (conectamos al maestro):**
Entrar a MariaDB
```bash
sudo mariadb
   ```

Y dentro de MariaDB:
```bash
  CHANGE MASTER TO
  MASTER_HOST='192.168.100.40',
  MASTER_USER='replicador',
  MASTER_PASSWORD='TuPasswordFuerte',
  MASTER_LOG_FILE='mariadb-bin.000001',
  MASTER_LOG_POS=330;
   ```
* **Iniciamos al esclavo:**
```bash
START SLAVE;
   ```

* **Verificamos estado de replicación:**
```bash
SHOW SLAVE STATUS\G
   ```
Se devolverá:
```bash
 Slave_IO_State: Waiting for master to send event
                   Master_Host: 192.168.235.98
                   Master_User: replicador
                   Master_Port: 3306
                 Connect_Retry: 60
               Master_Log_File: mariadb-bin.000002
           Read_Master_Log_Pos: 976
                Relay_Log_File: mariadb-relay-bin.000003
                 Relay_Log_Pos: 1277
         Relay_Master_Log_File: mariadb-bin.000002
              Slave_IO_Running: Yes
             Slave_SQL_Running: Yes
               Replicate_Do_DB:
           Replicate_Ignore_DB:
            Replicate_Do_Table:
        Replicate_Ignore_Table:
       Replicate_Wild_Do_Table:
   Replicate_Wild_Ignore_Table:
                    Last_Errno: 0
                    Last_Error:
                  Skip_Counter: 0
           Exec_Master_Log_Pos: 976
               Relay_Log_Space: 1889
               Until_Condition: None
                Until_Log_File:
                 Until_Log_Pos: 0
            Master_SSL_Allowed: No
            Master_SSL_CA_File:
            Master_SSL_CA_Path:
               Master_SSL_Cert:
             Master_SSL_Cipher:
                Master_SSL_Key:
         Seconds_Behind_Master: 0
 Master_SSL_Verify_Server_Cert: No
                 Last_IO_Errno: 0
                 Last_IO_Error:
                Last_SQL_Errno: 0
                Last_SQL_Error:
   Replicate_Ignore_Server_Ids:
              Master_Server_Id: 1
                Master_SSL_Crl:
            Master_SSL_Crlpath:
                    Using_Gtid: No
                   Gtid_IO_Pos:
       Replicate_Do_Domain_Ids:
   Replicate_Ignore_Domain_Ids:
                 Parallel_Mode: optimistic
                     SQL_Delay: 0
           SQL_Remaining_Delay: NULL
       Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
              Slave_DDL_Groups: 0
Slave_Non_Transactional_Groups: 0
    Slave_Transactional_Groups: 3
          Replicate_Rewrite_DB:
   ```
Donde podemos observar estas líneas que demuestran que la conexión fue exitosa:
```bash
Slave_IO_Running: Yes
Slave_SQL_Running: Yes
Seconds_Behind_Master: 0
   ```
### **Para observar los datos replicados de bd1, usar los comandos:**
Ingresar a MariaDB:
```bash
sudo mariadb
   ```
Ejecutar los comandos:
```bash
USE supermercado;
SELECT * FROM productos;
   ```
</div>
