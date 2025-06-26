# **Configuración del Proxy (Balanceador)**
<div align="justify">
  
El proxy o balanceador reparte las solicitudes de los usuarios entre las dos aplicaciones (App1 y App2) que están en diferentes servidores y diferentes máquinas.  Esto permite que el sistema funcione de manera más estable y eficiente, ya que mejora la disponibilidad del servicio y reparte la carga entre ambos servidores. Actúa como un punto de acceso único para usuarios, de modo que los usuarios no necesitan conocer las direcciones IP individuales de cada aplicación.  En caso de que una instancia deje de funcionar, la otra puede continuar atendiendo las solicitudes, lo que garantiza la continuidad del servicio frente a fallos. 

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
      addresses: [192.168.235.100/24]
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

## Instalación y configuración de Nginx:
* **Instalación de Nginx:**

   Ejecutamos el comando para instalar el servidor web Nginx:
  
   ```bash
   sudo apt update && sudo apt install nginx
   ```
   
* **Verificación del Estado del Servicio Nginx:**

   Verificamos que el servicio Nginx esté en ejecución usando el comando:
   
   ```bash
   sudo systemctl status nginx
   ```

   Si el estado no es: `active (running)`, iniciarlo con:
   ```bash
   sudo systemctl start nginx
   ```
   
   Y habilitarlo para el inicio automático con:
   ```bash
   sudo systemctl enable nginx
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

  Si cambiamos el puerto SSH en el archivo `/etc/ssh/sshd_config`, especificamos el nuevo puerto, en este caso 2222, usamos el comando:

  ```bash
  sudo ufw allow 2222/tcp
  ```

   Habilitamos el tráfico HTTP (puerto 80) y HTTPS (puerto 443) para Nginx con el comando:
   ```bash
   sudo ufw allow 'Nginx Full'
   ```
   Verifica el estado del firewall con:
   ```bash
   sudo ufw status
   ```

* **Instalación de módulos:**
```bash
 sudo apt install php8.3-fpm php8.3-mbstring php8.3-curl php8.3-xml php8.3-mysql php8.3-zip
```

## **Configuración del archivo del proxy (balanceador)**
* **Creamos el sitio habilitado**
  Ingresamos al archivo: `/etc/nginx/sites-available/proxy.usfx.bo`

* **Configuramos el archivo**
```bash
upstream appservers {
   server 192.168.235.101:3001 max_fails=2 fail_timeout=10s;
   server 192.168.235.102:3002 max_fails=2 fail_timeout=10s;
}

server {
   listen 80;
   server_name sis313.usfx.bo;

   location / {
       proxy_pass http://appservers;
       proxy_http_version 1.1;

       proxy_set_header Upgrade $http_upgrade;
       proxy_set_header Connection 'upgrade';

       proxy_set_header Host $host;
       proxy_set_header X-Real-IP $remote_addr;
       proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
       proxy_set_header X-Forwarded-Proto $scheme;

       proxy_cache_bypass $http_upgrade;

       proxy_redirect off;

       add_header X-Frame-Options "DENY" always;
       add_header X-Content-Type-Options "nosniff" always;
       add_header X-XSS-Protection "1; mode=block" always;
       add_header Referrer-Policy "no-referrer" always;
       add_header Permissions-Policy "geolocation=(), microphone=(), camera>

       gzip on;
       gzip_types
        text/plain
        text/css
        application/json
        application/javascript
        text/xml
        application/xml
        application/xml+rss
        text/javascript;

   }
}

```
Se incluyeron las IP's de las aplicaciones con las que se realizará el balanceo, el puerto al que escuchará que es el de HTTP, cabeceras de seguridad, entre otras configuraciones

* **Establecemos un enlace simbólico**
```bash
 sudo ln -s /etc/nginx/sites-available/proxy.usfx.bo /etc/nginx/sites-enabled/
   ```
* **Revisamos de sintaxis del archivo Nginx**
```bash
sudo nginx -t
   ```
Tras colocar el comando anterior, debe devolverse:
```bash
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```
* **Eliminamos el enlace default**
```bash
 sudo rm /etc/nginx/sites-enabled/default
   ```
* **Verificamos la creación exitosa del enlace**
```bash
ls /etc/nginx/sites-enabled/
   ```
Devuelve el nombre del archivo creado: `proxy.usfx.bo`
* **Reiniciamos el servicio aplicando los cambios**
```bash
sudo systemctl restart nginx
   ```

# **En el Host de la máquina Cliente**
## **Añadimos la dirección IP del Balanceador y su nombre de dominio**
```bash
192.168.235.100		sis313.usfx.bo
   ```
## **Prueba de conectividad con las apps**
```bash
curl http://192.168.100.20:3001/
curl http://192.168.100.30:3002/
   ```
Se devuelve el código de las apps
# **Comprobación final**
En el navegador buscamos: `sis313.usfx.bo`
Aparece la app1 y al refrescar la página aparece la app2

</div>
