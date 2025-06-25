# **Aplicación 1 (Antes de la creación de la BD)**
<div align="justify">

**Objetivo dentro del proyecto:** 

La aplicación app1 en el proyecto es una instancia del servicio que provee funcionalidades específicas de un CRUD, como gestionar y mostrar los productos almacenados en la base de datos. Su rol principal es procesar las solicitudes que recibe, acceder a la base de datos para leer o modificar información, y devolver respuestas al usuario o al balanceador. Forma parte del backend que, junto con otras instancias como app2, garantiza que la aplicación sea accesible, confiable y escalable cuando se usa en conjunto con el balanceador.

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
      addresses: [192.168.100.20/24]
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

## **Instalación de node.js con nvm**

* **Pasos para la instalación**
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
node -v
```
debería responder: v22.16.0

```bash
nvm current
```
debería responder: v22.16.0

```bash
npm -v
```
debería responder: 10.9.2

## **Instalación de express js**

* **Creación de la carpeta que contendrá la aplicación:**

```bash
mkdir myapp1
cd myapp1
```
* **Creación de la subcarpeta que contendrá el código de la API:**
Se la crea dentor de la carpeta myapp1
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
description: API Backend SIS313
entry point: (index.js)
test command:
git repository:
keywords: API, SIS313
author: SIS313
license: (ISC)
About to write to /home/app1/myapp1/api/package.json:

{
  "name": "api",
  "version": "1.0.0",
  "description": "API Backend SIS313",
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


Is this OK? (yes) y
```

* **Verificación:**
Dentro de /myapp1/api se ejecuta:
```bash
ls
```
Se debería ver: package.json, que es donde se guarda los archivos que permiten trabajar con nodejs y expressjs de la aplicación, usados en index.js que guarda el código que permite a la aplicación funcionar
* **Añadir extensiones dentro del proyecto:**
```bash
npm install -g npm@11.4.2
```

## **Archivo de prueba:**
El archivo de prueba con el código de express se encuentra e¿dentro de index.js, por lo que allí es donde se carga el código, entrando en:
```bash
nano index.js
```
Dentro se haya este código:
```bash
const express = require('express');
const app = express();
const PORT = 3001;

app.use(express.json()); // Para interpretar JSON

// Ruta de prueba
app.get('/', (req, res) => {
  res.send('¡Hola desde la API SIS313 - APP1!');
});

app.listen(PORT, '0.0.0.0', () => {
  console.log(Servidor API corriendo en http://localhost:${PORT});
});
```
Para hacer correr la API, se ejecuta:
```bash
node index.js
```
Si esta corriendo correctamente, devuelve: Servidor API corriendo en http://localhost:3001

Para verificar su funcionamiento (sin el blanceador) se entra a:
```bash
192.168.100.20:3001 #ip de la máquina virtual y el puerto donde ejecuta index.js
```

Para verificar su funcionamiento (con el blanceador) se entra a:
```bash
sis313.final #dns del balanceador
```
# **Aplicación 1 (Después de la creación de la BD)**
# **Cliente de mariadb:**
* **Añadir extensiones de cliente para mariadb:**
```bash
npm install mysql2
```

* **Añadir extensiones de para usar express:**
```bash
npm install express
```
# **Archivo CRUD para la app:**
```bash
const express = require('express');
const mysql = require('mysql2');
const app = express();
const PORT = 3001;

app.use(express.urlencoded({ extended: true })); // Para leer datos de formularios POST
app.use(express.json());

// Conexión a la base de datos
const db = mysql.createConnection({
  host: '192.168.100.40',
  user: 'appuser',
  password: 'App123@',
  database: 'tienda'
});

db.connect((err) => {
  if (err) {
    console.error('Error conectando a la base de datos:', err);
  } else {
    console.log('Conectado a la base de datos');
  }
});

// Página principal: muestra productos + formulario y tabla con acciones
app.get('/', (req, res) => {
  db.query('SELECT * FROM productos', (err, productos) => {
    if (err) return res.status(500).send('Error consultando la base de datos');

    let html = `
      <!DOCTYPE html>
      <html lang="es">
      <head>
        <meta charset="UTF-8" />
        <title>Productos - SIS313 - APP1</title>
        <style>
          body { font-family: Arial, sans-serif; margin: 40px; }
          table { border-collapse: collapse; width: 100%; margin-top: 20px; }
          th, td { border: 1px solid #ddd; padding: 8px; text-align: center; }
          th { background-color: #f2f2f2; }
          form input { padding: 6px; margin-right: 10px; }
          button { padding: 6px 12px; }
          .btn { cursor: pointer; }
          .btn-edit { color: blue; }
          .btn-delete { color: red; }
        </style>
      </head>
      <body>
        <h1>Productos - SIS313</h1>

        <form id="formAgregar">
          <input name="nombre" placeholder="Nombre del producto" required />
          <input name="precio" placeholder="Precio" type="number" step="0.01" required />
          <button type="submit">Agregar Producto</button>
        </form>

        <table id="tablaProductos">
          <thead>
            <tr>
              <th>ID</th><th>Nombre</th><th>Precio</th><th>Acciones</th>
            </tr>
          </thead>
          <tbody>
            ${productos.map(p => `
              <tr data-id="${p.id}">
                <td>${p.id}</td>
                <td class="nombre">${p.nombre}</td>
                <td class="precio">$${p.precio != null ? Number(p.precio).toFixed(2) : '0.00'}</td>
                <td>
                  <button class="btn btn-edit" onclick="editar(${p.id})">Editar</button>
                  <button class="btn btn-delete" onclick="borrar(${p.id})">Borrar</button>
                </td>
              </tr>
            `).join('')}
          </tbody>
        </table>

        <script>
          const form = document.getElementById('formAgregar');
          form.addEventListener('submit', async e => {
            e.preventDefault();
            const formData = new FormData(form);
            const data = {
              nombre: formData.get('nombre'),
              precio: parseFloat(formData.get('precio'))
            };
            const res = await fetch('/agregar', {
              method: 'POST',
              headers: {'Content-Type': 'application/json'},
              body: JSON.stringify(data)
            });
            if (res.ok) {
              form.reset();
              cargarProductos();
            } else {
              alert('Error al agregar producto');
            }
          });

          async function cargarProductos() {
            const res = await fetch('/productos');
            const productos = await res.json();
            const tbody = document.querySelector('#tablaProductos tbody');
            tbody.innerHTML = productos.map(p => \`
              <tr data-id="\${p.id}">
                <td>\${p.id}</td>
                <td class="nombre">\${p.nombre}</td>
                <td class="precio">$\${p.precio != null ? Number(p.precio).toFixed(2) : '0.00'}</td>
                <td>
                  <button class="btn btn-edit" onclick="editar(\${p.id})">Editar</button>
                  <button class="btn btn-delete" onclick="borrar(\${p.id})">Borrar</button>
                </td>
              </tr>\`).join('');
          }

          async function borrar(id) {
            if (!confirm('¿Seguro que quieres borrar este producto?')) return;
            const res = await fetch('/borrar/' + id, { method: 'DELETE' });
            if (res.ok) {
              cargarProductos();
            } else {
              alert('Error al borrar producto');
            }
          }

          async function editar(id) {
            const tr = document.querySelector(\tr[data-id="\${id}"]\);
            const nombreTd = tr.querySelector('.nombre');
            const precioTd = tr.querySelector('.precio');

            const nombreOld = nombreTd.textContent;
            const precioOld = precioTd.textContent.replace('$','');

            nombreTd.innerHTML = \<input type="text" id="editNombre" value="\${nombreOld}">\;
            precioTd.innerHTML = \<input type="number" step="0.01" id="editPrecio" value="\${precioOld}">\;

            const btnEdit = tr.querySelector('.btn-edit');
            btnEdit.textContent = 'Guardar';
            btnEdit.onclick = async () => {
              const nuevoNombre = document.getElementById('editNombre').value;
              const nuevoPrecio = parseFloat(document.getElementById('editPrecio').value);
              const res = await fetch('/editar/' + id, {
                method: 'PUT',
                headers: {'Content-Type': 'application/json'},
                body: JSON.stringify({ nombre: nuevoNombre, precio: nuevoPrecio })
              });
              if (res.ok) {
                cargarProductos();
              } else {
                alert('Error al editar producto');
              }
            };
          }

          // Carga inicial
          cargarProductos();
        </script>
      </body>
      </html>
    `;
    res.send(html);
  });
});

// API para obtener productos (JSON)
app.get('/productos', (req, res) => {
  db.query('SELECT * FROM productos', (err, results) => {
    if (err) return res.status(500).json({ error: 'Error en la base de datos' });
    res.json(results);
  });
});

// Agregar producto
app.post('/agregar', (req, res) => {
  const { nombre, precio } = req.body;
  db.query('INSERT INTO productos (nombre, precio) VALUES (?, ?)', [nombre, precio], (err) => {
    if (err) return res.status(500).json({ error: 'Error al agregar producto' });
    res.status(200).json({ mensaje: 'Producto agregado' });
  });
});

// Editar producto
app.put('/editar/:id', (req, res) => {
  const id = req.params.id;
  const { nombre, precio } = req.body;
  db.query('UPDATE productos SET nombre = ?, precio = ? WHERE id = ?', [nombre, precio, id], (err) => {
    if (err) return res.status(500).json({ error: 'Error al editar producto' });
    res.status(200).json({ mensaje: 'Producto actualizado' });
  });
});

// Borrar producto
app.delete('/borrar/:id', (req, res) => {
  const id = req.params.id;
  db.query('DELETE FROM productos WHERE id = ?', [id], (err) => {
    if (err) return res.status(500).json({ error: 'Error al borrar producto' });
    res.status(200).json({ mensaje: 'Producto borrado' });
  });
});

app.listen(PORT, '0.0.0.0', () => {
  console.log(Servidor API corriendo en http://localhost:${PORT});
});
```

Si esta corriendo correctamente, devuelve: 
- Servidor API corriendo en http://localhost:3001
Conectado a la base de datos

## **Verificación**
Para verificar su funcionamiento (con el blanceador) se entra a:
```bash
sis313.final #ip de la máquina virtual del balanceador que conecta a la app 1 donde ejecuta index.js
```

</div>
