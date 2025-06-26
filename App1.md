# **Configuración en App 1**
<div align="justify">

App1 es una de las dos instancias backend que forman parte de la infraestructura requerida para el proyecto. Esta aplicación proporciona funcionalidades CRUD (Create, Read, Update, Delete) conectadas a una base de datos MySQL/MariaDB. Se ejecuta en una máquina virtual con IP estática y es balanceada por un servidor proxy NGINX. En conjunto con App2, garantiza alta disponibilidad y distribución de carga.

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
      addresses: [192.168.235.101/24]
      routes:
         - to: default
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

## **Instalación de node.js con nvm**

En la página de instalación de node.js se brindan los siguientes comandos para la instalación de node.js v22.17.0 (LTS) para Linux usando nvm con npm:

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash
```
```bash
\. "$HOME/.nvm/nvm.sh"
```
```bash
nvm install 22
```
```bash
node -v #imprime "v22.16.0"
```
```bash
nvm current #imprime "v22.16.0"
```
```bash
npm -v #imprime "10.9.2"
```

## **Instalación de express js**

* **Creación de la carpeta que contendrá la aplicación:**

```bash
mkdir app2
```
ingresamos a la carpeta recién creada:
```bash
cd app2
```
* **Creación de la subcarpeta que contendrá el código de la API:**
```bash
mkdir api
cd api/
```
* **Instalación de express js:**
```bash
npm init
```
* **Configuraciones:**

```bash
package name: (api)
version: (1.0.0)
description: API SIS313
entry point: (index.js)
test command:
git repository:
keywords: API, SIS313
author: SIS313
license: (ISC)
About to write to /home/appserver1/app1/api/package.json:

{
  "name": "api",
  "version": "1.0.0",
  "description": "API SIS313",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [
    "API",
    "SIS313"
  ],
  "author": "SIS313",
  "license": "ISC"
}


Is this OK? (yes) yes
```

* **Verificación:**
En /myapp1/api se ejecuta el comando:
```bash
ls
```
Este devuleve package.json, que es donde se guarda los archivos que permiten trabajar con node.js y express.js de la aplicación, usados en index.js que guarda el código que hace que la aplicación funcione.

* **Añadimos extensiones dentro del proyecto:**
```bash
npm install -g npm@11.4.2
```
* **Añadimos extensiones de para usar express:**
```bash
npm install express
```

## **Configuramos un archivo de prueba:**
El archivo index.js al que ingresamos con el siguiente comando dentro de app1/api:
```bash
nano index.js
```
Escribimos el siguiente código:
```bash
const express = require('express')
const app = express()
const port = 3001

app.get('/', (req, res) => {
  res.send('App 1')
})

app.listen(port, () => {
  console.log(`Servidor corriendo en el puerto ${port}`)
})

```
Habilitamos el tráfico por el puerto por el que corre nuestra app con el comando:
```bash
sudo ufw allow 3001/tcp
```

Corremos la API, con el comando:
```bash
node index.js
```
Si corre correctamente, devuelve el mensaje: Servidor corriendo en el puerto 3001

Verificamos su funcionamiento (sin el blanceador) ingresando en el navegador la ip de la VM y el puerto en el que corre la app node.js, de la siguiente forma:
```bash
192.168.235.101:3001
```

# **Después de la implementación del Proxy (Balanceador)**
Se verifica el funcionamiento en el navegador, ingresando a:
```bash
sis313.usfx.bo #dns del proxy declarada en el host de la máquina cliente 
```
# **Después de la creación de la BD**
# **Cliente de mariadb:**
* **Añadimos extensiones de cliente para mariadb:**
```bash
npm install mysql2
```

# **Archivo CRUD para la app:**
```bash
const express = require('express');
const mysql = require('mysql2');
const app = express();
const PORT = 3001;

// Middlewares
app.use(express.urlencoded({ extended: true }));
app.use(express.json());

// Conexión a MySQL/MariaDB
const db = mysql.createConnection({
  host: '192.168.235.98',
  user: 'appuser',
  password: 'App1@',
  database: 'supermercado'
});

db.connect((err) => {
  if (err) {
    console.error('Error de conexión a la base de datos:', err.stack);
    return;
  }
  console.log('Conectado a la base de datos con ID:', db.threadId);
});

// Ruta principal - Interfaz CRUD
app.get('/', (req, res) => {
  db.query('SELECT * FROM productos', (err, resultados) => {
    if (err) {
      console.error(err);
      return res.status(500).send('Error al cargar productos');
    }

    const productos = resultados.map(p => ({
      id: p.id,
      nombre: p.nombre,
      precio: parseFloat(p.precio).toFixed(2)
    }));

    const html = `
    <!DOCTYPE html>
    <html lang="es">
    <head>
      <meta charset="UTF-8">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <title>Gestión de Supermercado</title>
      <style>
        body { font-family: Arial, sans-serif; margin: 20px; }
        table { width: 100%; border-collapse: collapse; margin-top: 20px; }
        th, td { border: 1px solid #ddd; padding: 8px; text-align: left; }
        th { background-color: #f2f2f2; }
        form { margin-bottom: 20px; background: #f9f9f9; padding: 15px; border-radius: 5px; }
        input, button { padding: 8px; margin-right: 10px; }
        button { cursor: pointer; }
        .btn-edit { background: #4CAF50; color: white; border: none; }
        .btn-delete { background: #f44336; color: white; border: none; }
      </style>
    </head>
    <body>
      <h1>Gestión de Productos</h1>

      <form id="formProducto">
        <input type="text" name="nombre" placeholder="Nombre" required>
        <input type="number" name="precio" placeholder="Precio" step="0.01" required>
        <button type="submit">Agregar</button>
      </form>

      <table>
        <thead>
          <tr>
            <th>ID</th>
            <th>Nombre</th>
            <th>Precio</th>
            <th>Acciones</th>
          </tr>
        </thead>
        <tbody>
          ${productos.map(p => `
            <tr data-id="${p.id}">
              <td>${p.id}</td>
              <td>${p.nombre}</td>
              <td>$${p.precio}</td>
              <td>
                <button class="btn-edit" onclick="editarProducto(${p.id})">Editar</button>
                <button class="btn-delete" onclick="eliminarProducto(${p.id})">Eliminar</button>
              </td>
            </tr>
          `).join('')}
        </tbody>
      </table>

      <script>
        const form = document.getElementById('formProducto');

        form.addEventListener('submit', async (e) => {
          e.preventDefault();
          const formData = new FormData(form);
          const producto = {
            nombre: formData.get('nombre'),
            precio: parseFloat(formData.get('precio'))
          };

          try {
            const response = await fetch('/productos', {
              method: 'POST',
              headers: { 'Content-Type': 'application/json' },
              body: JSON.stringify(producto)
            });

            if (response.ok) {
              location.reload();
            } else {
              alert('Error al agregar producto');
            }
          } catch (error) {
            console.error('Error:', error);
          }
        });

        async function eliminarProducto(id) {
          if (confirm('¿Eliminar este producto?')) {
            try {
              const response = await fetch('/productos/' + id, { method: 'DELETE' });
              if (response.ok) location.reload();
            } catch (error) {
              console.error('Error:', error);
            }
          }
        }

        async function editarProducto(id) {
          const nombre = prompt('Nuevo nombre:');
          const precio = parseFloat(prompt('Nuevo precio:'));

          if (nombre && !isNaN(precio)) {
            try {
              const response = await fetch('/productos/' + id, {
                method: 'PUT',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({ nombre, precio })
              });

              if (response.ok) location.reload();
            } catch (error) {
              console.error('Error:', error);
            }
          }
        }
      </script>
    </body>
    </html>
    `;

    res.send(html);
  });
});

// API REST
app.get('/productos', (req, res) => {
  db.query('SELECT * FROM productos', (err, resultados) => {
    if (err) return res.status(500).json({ error: err.message });
    res.json(resultados);
  });
});

app.post('/productos', (req, res) => {
  const { nombre, precio } = req.body;
  db.query(
    'INSERT INTO productos (nombre, precio) VALUES (?, ?)',
    [nombre, precio],
    (err) => {
      if (err) return res.status(500).json({ error: err.message });
      res.status(201).json({ mensaje: 'Producto agregado' });
    }
  );
});

app.put('/productos/:id', (req, res) => {
  const { id } = req.params;
  const { nombre, precio } = req.body;
  db.query(
    'UPDATE productos SET nombre = ?, precio = ? WHERE id = ?',
    [nombre, precio, id],
    (err) => {
      if (err) return res.status(500).json({ error: err.message });
      res.json({ mensaje: 'Producto actualizado' });
    }
  );
});

app.delete('/productos/:id', (req, res) => {
  const { id } = req.params;
  db.query(
    'DELETE FROM productos WHERE id = ?',
    [id],
    (err) => {
      if (err) return res.status(500).json({ error: err.message });
      res.json({ mensaje: 'Producto eliminado' });
    }
  );
});

// Iniciar servidor
app.listen(PORT, '0.0.0.0', () => {
  console.log(`Servidor ejecutándose en http://localhost:${PORT}`);
});
```

Si corre correctamente, devuelve el mensaje: 
- Servidor ejecutándose en http://localhost:3001
Conectado a la base de datos con ID:

## **Verificación**
Verificamos su funcionamiento igual que anteriormente, con el proxy:
```bash
sis313.usfx.bo
```

</div>
