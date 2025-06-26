# **Configuración del Proxy (Balanceador)**
<div align="justify">
  
El balanceador en tu proyecto tiene como objetivo repartir las solicitudes de los usuarios entre las dos aplicaciones que tienes desplegadas en diferentes servidores, mejorando así la disponibilidad, escalabilidad y rendimiento del sistema. Actúa como un punto de entrada único, de modo que los usuarios no necesitan conocer las direcciones específicas de cada backend, y además ayuda a que si una instancia falla, la otra pueda seguir atendiendo las peticiones, asegurando así la continuidad del servicio.  

**Pasos realizados:**

## **Configuración de IP en la máquina virtual**

* **Cambio de IP dinámica a estatica:**
```bash
 sudo nano /etc/netplan/50-cloud-init.yaml
   ```
Dentro del archivo /etc/...
```bash
 network:
  version: 2
  ethernets:
    enp0s3:
      dhcp4: false
      addresses: [192.168.100.10/24]
      routes:
      - to: default
        via: 192.168.100.1
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]

   ```

## **Instalación y configuración de Nginx:**
* **Instalación de Nginx:**
```bash
   sudo apt update && sudo apt install nginx
   ```

* **Verificación del Estado del Servicio Nginx:**
```bash
   sudo systemctl status nginx
   ```

En caso de que este no este en estado de running:
 ```bash
   sudo systemctl start nginx
   ```

En caso de que este no este en estado de enable:
 ```bash
   sudo systemctl enable nginx
   ```

* **Instalación de módulos:**
```bash
 sudo apt install php8.3-fpm php8.3-mbstring php8.3-curl php8.3-xml php8.3-mysql php8.3-zip
   ```
* **Restablecimiento del servicio:**
```bash
   sudo systemctl restart nginx
   ```

## **Configuración del archivo de balanceador**
* **Configuración del sitio habilitado**
```bash
 sudo nano /etc/nginx/sites-available/balanceo_SIS313
   ```
Detro del archivo /etc/...
```bash
upstream mis_apps {
    server 192.168.100.20:3001;
    server 192.168.100.30:3002;
}

server {
    listen 80;
    server_name sis313.final;

    location / {
        proxy_pass http://mis_apps;
        proxy_http_version 1.1;

        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        proxy_cache_bypass $http_upgrade;

        # Evita que Nginx cambie las redirecciones de la app
        proxy_redirect off;
    }
}
```
* **Enlace simbólico**
```bash
 sudo ln -s /etc/nginx/sites-available/balanceo_SIS313 /etc/nginx/sites-enabled/
   ```
* **Revisión de sintaxis del archivo Nginx**
```bash
sudo nginx -t
   ```
* **Eliminar el enlace default**
```bash
 sudo rm /etc/nginx/sites-enabled/default
   ```
* **Si el enlace fue exitoso se ve el nombre del archivo de Nginx creado**
```bash
ls /etc/nginx/sites-enabled/
   ```

* **Reinicio del servicio con los cambios**
```bash
sudo systemctl restart nginx
   ```
# **Añadir a hosts ip y dns del balanceador**
```bash
192.168.100.10 sis313.final
   ```

## **Prueba de conectividad con las apps**
```bash
curl http://192.168.100.20:3001/
curl http://192.168.100.30:3002/
   ```
## **Hosts - Dns**
Verificar que la ip de la máquina virtual del balanceador y su dns se encuentre añadido en la carpeta de hosts en la máquina cliente:
```bash
192.168.100.10 sis313.final
   ```
</div>
