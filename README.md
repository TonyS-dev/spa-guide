# 🚀 Guía: SPA para Principiantes con Seguridad, JsonServer Vite

## Plantilla Reutilizable para Cualquier Proyecto

> **Esta guía te permitirá crear cualquier SPA**: Academia online, E-commerce, Administrador de personal, Sistema de inventario, etc.

---

## 🎯 Estructura Básica del Proyecto

### Estructura de Carpetas Sugerida

```
mi-proyecto-spa/
├── index.html                    # Página principal
├── package.json                  # Configuración de dependencias
├── db.json                       # Base de datos simulada
├── .gitignore                    # Archivos a ignorar
├── public/
│   └── css/
│       └── main.css             # Estilos generales
└── src/
    ├── controllers/             # Lógica de paginas y herramientas (autenticación, API, CRUD)
    ├── pages/                   # Páginas HTML (login, registro, dashboard, etc.)
    ├── routers/                 # Sistema de rutas
    └── utils/                   # Utilidades (autenticación, validaciones, notificaciones)
```

---

## ⚡ Instalación y Estructura con Vite

### PASO 0: Crear el Proyecto con Vite y Configuración Concurrente

1. Crea el proyecto con Vite:

```bash
npm create vite@latest mi-proyecto-spa -- --template vanilla
cd mi-proyecto-spa
npm install
```

2. Instala json-server y concurrently:

```bash
npm install -D json-server concurrently
```

3. Agrega los scripts en tu `package.json`:

```json
"scripts": {
        "dev": "vite",
        "build": "vite build",
        "preview": "vite preview",
        "json-server": "json-server --watch db.json --port 3001",
        "start": "concurrently \"npm run json-server\" \"npm run dev\"",
        "start:dev": "concurrently --names \"API,DEV\" --prefix-colors \"blue,green\" \"npm run json-server\" \"npm run dev\""
    }

```

4. Inicia ambos servidores:

```bash
npm run start:dev
```

---

## 📋 PASO A PASO: CONSTRUYENDO LA BASE

Cada paso incluye explicaciones y comentarios para que entiendas qué hace cada parte del código.

### PASO 1: Crea el HTML Principal

**`index.html`** es la página principal donde se mostrará todo el contenido de tu app. Aquí solo necesitas un contenedor y enlaces a tus archivos de estilos y JavaScript.

```html
<!-- index.html: Estructura básica de la app -->
<!DOCTYPE html>
<html lang="es">
    <head>
        <meta charset="UTF-8" />
        <meta name="viewport" content="width=device-width, initial-scale=1.0" />
        <title>Mi SPA</title>
        <link rel="stylesheet" href="public/css/main.css" />
    </head>
    <body>
        <div id="main-content">
            <!-- Header dinámico -->
            <header id="header"></header>

            <!-- Contenido principal -->
            <main id="app">
                <!-- Aquí se carga el contenido dinámico -->
            </main>
        </div>
        <!-- Aquí se carga el JavaScript principal -->
        <script type="module" src="src/utils/main.js"></script>
    </body>
</html>
```

### PASO 2: Configurar la Base de Datos Simulada (con contraseñas hasheadas)

**`db.json` - Ejemplo con SHA-256:**

```json
{
    "users": [
        {
            "id": 1,
            "username": "admin",
            "email": "admin@ejemplo.com",
            "password": "6c3b6e6e4b6c2c1b9b419911cfef95c0d6c6e3c7b2e3e8e2e3e3e8e2e3e3e8e2", // hash SHA-256 de 'admin123' (ejemplo)
            "role": "admin"
        },
        {
            "id": 2,
            "username": "user1",
            "email": "user1@ejemplo.com",
            "password": "bcb1b1b1b1b1b1b1b1b1b1b1b1b1b1b1b1b1b1b1b1b1b1b1b1b1b1b1b1b1b1b1b1", // hash SHA-256 de 'user123' (ejemplo)
            "role": "user"
        }
    ]
}
```

**Comandos para inicializar:**

```bash
npm init -y
npm install -g json-server
json-server --watch db.json --port 3001
> Puedes obtener el hash de una contraseña usando la función hashText del paso 3 y pegarlo en el JSON.
```

---

## 🔗 Configuración de la API

Agrega en tu archivo de utilidades o controlador:

```javascript
const API_URL = "http://localhost:3001"; // Cambia el endpoint por el recurso que necesites ej: http://localhost:3001/users
```

---

## 🔐 AUTENTICACIÓN BÁSICA Y SEGURA

### PASO 3: Controlador de Autenticación con Hash SHA-256 y comparación con JSON

**`src/utils/auth.js`**

```javascript
import { getData } from "../controllers/apiController.js";

// auth.js: Login y registro con hash SHA-256

// Función para hashear texto con SHA-256
async function hashText(text) {
    const encoder = new TextEncoder();
    const data = encoder.encode(text);
    const hashBuffer = await crypto.subtle.digest('SHA-256', data);
    const hashArray = Array.from(new Uint8Array(hashBuffer));
    const hashHex = hashArray.map(b => b.toString(16).padStart(2, '0')).join('');
    return hashHex;
}

// Iniciar sesión
async function login(credentials) {
    const hashedPassword = await hashText(credentials.password);
    const users = await getData();
    const user = users.find(u => u.username === credentials.username && u.password === hashedPassword);
    if (user) {
        localStorage.setItem('currentUser', JSON.stringify(user));
        // Redirección automática según el rol
        if (user.role === 'admin') {
            window.location.href = '/dashboard';
        } else if (user.role === 'user') {
            window.location.href = '/dashboard-user';
        } else if (user.role === 'guest') {
            window.location.href = '/dashboard-guest';
        } else {
            window.location.href = '/';
        }
        return { success: true, user };
    } else {
        return { success: false, message: "Credenciales inválidas" };
    }
}

Ejemplo: cómo construir userData desde los inputs del formulario de registro
const form = document.getElementById('registerForm');
form.addEventListener('submit', async (e) => {
    e.preventDefault();
    const formData = new FormData(form);
    const userData = {
        username: formData.get('username'),
        email: formData.get('email'),
        password: formData.get('password'),
        role: 'user' // o puedes obtenerlo de otro input si lo tienes
    };
    await register(userData);
});

// Registrar usuario
async function register(userData) {
    userData.password = await hashText(userData.password);
    // Guarda el usuario en la base de datos (JSON Server)
    await httpRequest('POST', API_URL, userData);
    return { success: true, user: userData };
}

export { login, register, hashText };
```

---

## 🌐 Funciones de Petición HTTP

Agrega estas funciones en tu archivo de utilidades:

```javascript
/**
 * Realiza una petición HTTP al servidor
 * @param {string} method - Método HTTP (GET, POST, PUT, PATCH, DELETE)
 * @param {string} url - URL a la que se envía la petición
 * @param {Object|null} body - Cuerpo de la petición (para POST/PUT/PATCH)
 * @returns {Promise<Object>} Respuesta como JSON
 * @throws {Error} Si la petición falla
 */
async function httpRequest(method, url, body = null) {
    try {
        const options = {
            method: method,
            headers: {
                "Content-Type": "application/json",
            },
            body: body ? JSON.stringify(body) : null,
        };
        const response = await fetch(url, options);
        if (!response.ok) {
            throw new Error(`HTTP error! status: ${response.status}`);
        }
        return await response.json();
    } catch (error) {
        console.error("❌ HTTP request error:", error);
        throw error;
    }
}

// Obtener datos (GET)
async function getData() {
    try {
        const response = await fetch(API_URL);
        if (!response.ok) {
            throw new Error(`HTTP error! status: ${response.status}`);
        }
        const Database = await response.json();
        return Database;
    } catch (error) {
        console.error("❌ Error fetching data:", error);
        return [];
    }
}
```

---

## 🔑 ALMACENAMIENTO SEGURO

### PASO 4: Almacenamiento Seguro con Session Storage

**`src/utils/storage.js`**

```javascript
// Almacenar usuario en sessionStorage
function setUser(user) {
    sessionStorage.setItem('currentUser', JSON.stringify(user));
}

// Obtener usuario de sessionStorage
function getUser() {
    const user = sessionStorage.getItem('currentUser');
    return user ? JSON.parse(user) : null;
}

// Eliminar usuario de sessionStorage
function removeUser() {
    sessionStorage.removeItem('currentUser');
}

export { storeUser, getUser, clearUser };
```

---

## 🗺️ SISTEMA DE RUTAS BÁSICO Y PROTEGIDO

### PASO 5: Router Simple con Guardian y Acceso por Rol

**`src/routers/router.js`**

```javascript
// Router simple para SPA con guardian y roles
import { getUserFromStorage } from "../utils/guardian.js";

const routes = {
    "/": { page: "dashboard.html", protected: true },
    "/login": { page: "login.html", protected: false },
    "/register": { page: "register.html", protected: false },
    "/dashboard": { page: "dashboard.html", protected: true, role: ["admin"] },
    "/dashboard-user": { page: "dashboard-user.html", protected: true, role: ["user"] },
    "/dashboard-guest": { page: "dashboard-guest.html", protected: true, role: ["guest"] },
    "/forbidden": { page: "forbidden.html", protected: true },
    "/not-found": { page: "404.html", protected: false },
};

async function navigateTo(path) {
    const route = routes[path] || routes["/not-found"];
    // Guardian: solo permite si está autenticado
    if (route.protected) {
        const user = getUserFromStorage();
        if (!user) {
            window.history.pushState({}, '', '/login');
            return navigateTo('/login');
        }
        // Acceso por rol
        if (route.role && !route.role.includes(user.role)) {
            window.history.pushState({}, '', '/forbidden');
            return navigateTo('/forbidden');
        } 
    }
    const response = await fetch(`src/pages/${route.page}`);
    const content = await response.text();
    document.getElementById("app").innerHTML = content;
}

function initRouter() {
    document.addEventListener("click", function (e) {
        if (e.target.matches("[data-link]")) {
            e.preventDefault();
            const href = e.target.getAttribute("href");
            navigateTo(href);
        }
    });
    window.addEventListener("popstate", function () {
        navigateTo(window.location.pathname);
    });
    navigateTo(window.location.pathname);
}

export { initRouter, navigateTo };
```


---

## 🛡️ GUARDIAN DE SESIÓN Y ROL

### PASO 6: Guardian Simple

**`src/utils/guardian.js`**

```javascript
// Guardián: obtiene usuario autenticado desde localStorage
function getUserFromStorage() {
    const user = localStorage.getItem('currentUser');
    return user ? JSON.parse(user) : null;
}

export { getUserFromStorage };
```

---

## 🔧 VALIDACIONES BÁSICAS

### PASO 6: Validaciones Simples

**`src/utils/validation.js`**

```javascript
function validateLogin(credentials) {
    const errors = [];
    if (!credentials.username) errors.push("Usuario requerido");
    if (!credentials.password) errors.push("Contraseña requerida");
    return errors;
}

function validateRegister(userData, confirmPassword) {
    const errors = [];
    if (!userData.username) errors.push("Usuario requerido");
    if (!userData.email) errors.push("Email requerido");
    if (!userData.password) errors.push("Contraseña requerida");
    if (userData.password !== confirmPassword) errors.push("Las contraseñas no coinciden");
    return errors;
}

export { validateLogin, validateRegister };
```

---

## 🚀 INICIALIZADOR SIMPLE

### PASO 7: Punto de Entrada

**`src/utils/main.js`**

```javascript
import { initRouter } from "../routers/router.js";

function initApp() {
    initRouter();
    console.log("SPA inicializada");
}

document.addEventListener("DOMContentLoaded", initApp);
```

---

## 📄 PÁGINAS HTML BÁSICAS

### PASO 8: Páginas Base

**`src/pages/login.html`:**

```html
<div class="auth-container">
    <div class="auth-card">
        <h2>Iniciar Sesión</h2>
        <form id="loginForm">
            <div class="form-group">
                <label for="username">Usuario:</label>
                <input type="text" id="username" name="username" required />
            </div>
            <div class="form-group">
                <label for="password">Contraseña:</label>
                <input type="password" id="password" name="password" required />
            </div>
            <button type="submit" class="btn btn-primary">
                Iniciar Sesión
            </button>
        </form>
        <p class="auth-link">
            ¿No tienes cuenta? <a href="/register" data-link>Regístrate aquí</a>
        </p>
    </div>
</div>
```

**`src/pages/register.html`:** 

```html
<div class="auth-container">
    <div class="auth-card">
        <h2>Registrarse</h2>
        <form id="registerForm">
            <div class="form-group">
                <label for="username">Usuario:</label>
                <input type="text" id="username" name="username" required />
            </div>
            <div class="form-group">
                <label for="email">Email:</label>
                <input type="email" id="email" name="email" required />
            </div>
            <div class="form-group">
                <label for="password">Contraseña:</label>
                <input type="password" id="password" name="password" required />
            </div>
            <div class="form-group">
                <label for="confirmPassword">Confirmar Contraseña:</label>
                <input
                    type="password"
                    id="confirmPassword"
                    name="confirmPassword"
                    required
                />
            </div>
            <button type="submit" class="btn btn-primary">Registrarse</button>
        </form>
        <p class="auth-link">
            ¿Ya tienes cuenta? <a href="/login" data-link>Inicia sesión aquí</a>
        </p>
    </div>
</div>
```

**`src/pages/dashboard.html`:**

```html
<div class="dashboard-container">
    <div class="dashboard-header">
        <h1>Dashboard</h1>
    </div>

    <div class="dashboard-content">
        <!-- Contenido del dashboard -->
        <p>Bienvenido al panel de administración.</p>
    </div>
</div>
```

**`src/pages/dashboard-user.html`:**

```html
<div class="dashboard-container">
    <h1>Dashboard Usuario</h1>
    <p>Bienvenido, usuario.</p>
</div>
```

**`src/pages/dashboard-guest.html`:**

```html
<div class="dashboard-container">
    <h1>Dashboard Invitado</h1>
    <p>Bienvenido, invitado.</p>
</div>
```

**`src/pages/404.html`:**

```html
<div class="error-container">
    <div class="error-content">
        <h1>404</h1>
        <h2>Página No Encontrada</h2>
        <p>
            La página que buscas no existe o no tienes permisos para acceder a
            ella.
        </p>
        <a href="/dashboard" data-link class="btn btn-primary"
            >Volver al Dashboard</a
        >
    </div>
</div>
```

**`src/pages/forbidden.html`:**

```html
<div class="error-container">
    <div class="error-content">
        <h1>403</h1>
        <h2>Acceso Prohibido</h2>
        <p>No tienes permisos para acceder a esta página.</p>
        <a href="/" data-link class="btn btn-primary">Ir al inicio</a>
    </div>
</div>
```

---

## 🖥️ Dashboards por Rol

**`src/pages/dashboard.html`** (admin):
```html
<div class="dashboard-container">
    <h1>Dashboard Admin</h1>
    <p>Bienvenido, administrador.</p>
</div>
```

**`src/pages/dashboard-user.html`** (user):
```html
<div class="dashboard-container">
    <h1>Dashboard Usuario</h1>
    <p>Bienvenido, usuario.</p>
</div>
```

**`src/pages/dashboard-guest.html`** (guest):
```html
<div class="dashboard-container">
    <h1>Dashboard Invitado</h1>
    <p>Bienvenido, invitado.</p>
</div>
```

---

## 🎨 ESTILOS CSS BÁSICOS

si prefieren más simpleza añadir [tailwindcss](https://tailwindcss.com/) o [bootstrap](https://getbootstrap.com/) para los estilos.

### PASO 9: Estilos Base

**`public/css/main.css`:**

```css
/* ====== RESET Y BASE ====== */
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    font-family: "Segoe UI", Tahoma, Geneva, Verdana, sans-serif;
    line-height: 1.6;
    color: #333;
    background-color: #f5f5f5;
}

#main-content {
    min-height: 100vh;
    display: flex;
    flex-direction: column;
}

/* ====== NAVEGACIÓN ====== */
.navbar {
    background: #2c3e50;
    color: white;
    padding: 1rem 2rem;
    display: flex;
    justify-content: space-between;
    align-items: center;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.nav-brand h1 {
    font-size: 1.5rem;
    font-weight: bold;
}

.nav-menu {
    display: flex;
    align-items: center;
    gap: 1rem;
}

.nav-menu a {
    color: white;
    text-decoration: none;
    padding: 0.5rem 1rem;
    border-radius: 4px;
    transition: background-color 0.3s;
}

.nav-menu a:hover {
    background-color: rgba(255, 255, 255, 0.1);
}

.user-info {
    font-size: 0.9rem;
    color: #ecf0f1;
    margin-right: 1rem;
}

.logout-btn {
    background-color: #e74c3c;
    color: white;
    border: none;
    padding: 0.5rem 1rem;
    border-radius: 4px;
    cursor: pointer;
    font-weight: bold;
    transition: background-color 0.3s;
}

.logout-btn:hover {
    background-color: #c0392b;
}
/* ====== CONTENIDO PRINCIPAL ====== */
#app {
    flex: 1;
    padding: 2rem;
}
/* ====== CONTENEDORES GENÉRICOS ====== */
.container {
    max-width: 1200px;
    margin: 0 auto;
    padding: 0 1rem;
}

.auth-container,
.error-container {
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100%;
}

/* ====== FORMULARIOS ====== */
form {
    display: flex;
    flex-direction: column;
    gap: 1rem;
}

.form-group {
    display: flex;
    flex-direction: column;
}

.form-group label {
    margin-bottom: 0.5rem;
    font-weight: bold;
    color: #555;
}

.form-group input,
.form-group select,
.form-group textarea {
    padding: 0.75rem;
    border: 1px solid #ccc;
    border-radius: 4px;
    font-size: 1rem;
}

.form-group input:focus,
.form-group select:focus,
.form-group textarea:focus {
    outline: none;
    border-color: #3498db;
    box-shadow: 0 0 5px rgba(52, 152, 219, 0.5);
}

.form-actions {
    display: flex;
    gap: 1rem;
    justify-content: flex-end;
    margin-top: 1rem;
}
/* ====== BOTONES ====== */
.btn {
    padding: 0.75rem 1.5rem;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    font-size: 1rem;
    font-weight: bold;
    transition: all 0.3s;
    text-decoration: none;
    display: inline-block;
    text-align: center;
}

.btn-primary {
    background-color: #3498db;
    color: white;
}

.btn-primary:hover {
    background-color: #2980b9;
}

.btn-secondary {
    background-color: #95a5a6;
    color: white;
}

.btn-secondary:hover {
    background-color: #7f8c8d;
}

.btn-danger {
    background-color: #e74c3c;
    color: white;
}
.btn-danger:hover {
    background-color: #c0392b;
}

.btn-sm {
    padding: 0.4rem 0.8rem;
    font-size: 0.8rem;
}
/* ====== MODAL ====== */
.modal {
    display: none;
    position: fixed;
    z-index: 1000;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
    overflow: auto;
    background-color: rgba(0, 0, 0, 0.5);
}

.modal.active {
    display: flex;
    justify-content: center;
    align-items: center;
}

.modal-content {
    background-color: #fff;
    padding: 2rem;
    border-radius: 8px;
    box-shadow: 0 5px 15px rgba(0, 0, 0, 0.3);
    width: 90%;
    max-width: 600px;
    position: relative;
    animation: slideIn 0.3s ease-out;
}

@keyframes slideIn {
    from {
        transform: translateY(-30px);
        opacity: 0;
    }
    to {
        transform: translateY(0);
        opacity: 1;
    }
}

.modal-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 1.5rem;
    border-bottom: 1px solid #eee;
    padding-bottom: 1rem;
}

.modal-close {
    font-size: 2rem;
    font-weight: bold;
    cursor: pointer;
    background: none;
    border: none;
    color: #aaa;
}

.modal-close:hover {
    color: #333;
}
```

### PASO 15: Estilos de Componentes

**`public/css/components.css`:**

```css
/* ====== NOTIFICACIONES ====== */
#notifications {
    position: fixed;
    top: 20px;
    right: 20px;
    z-index: 2000;
    display: flex;
    flex-direction: column;
    gap: 10px;
}

.notification {
    padding: 1rem;
    border-radius: 6px;
    color: white;
    box-shadow: 0 3px 10px rgba(0, 0, 0, 0.2);
    min-width: 300px;
    opacity: 0.95;
    animation: fadeInRight 0.5s ease-out;
}

.notification-content {
    display: flex;
    justify-content: space-between;
    align-items: center;
}

.notification-close {
    background: none;
    border: none;
    color: white;
    font-size: 1.2rem;
    cursor: pointer;
    margin-left: 1rem;
    opacity: 0.7;
}
.notification-close:hover {
    opacity: 1;
}

.notification-success {
    background-color: #27ae60;
}
.notification-error {
    background-color: #c0392b;
}
.notification-warning {
    background-color: #f39c12;
}
.notification-info {
    background-color: #2980b9;
}

@keyframes fadeInRight {
    from {
        transform: translateX(100%);
        opacity: 0;
    }
    to {
        transform: translateX(0);
        opacity: 1;
    }
}

/* ====== TARJETAS DE AUTENTICACIÓN ====== */
.auth-card {
    background: white;
    padding: 2rem 3rem;
    border-radius: 8px;
    box-shadow: 0 4px 20px rgba(0, 0, 0, 0.1);
    width: 100%;
    max-width: 450px;
}

.auth-card h2 {
    text-align: center;
    margin-bottom: 2rem;
    color: #2c3e50;
}

.auth-link {
    text-align: center;
    margin-top: 1.5rem;
    font-size: 0.9rem;
}

.auth-link a {
    color: #3498db;
    text-decoration: none;
    font-weight: bold;
}

/* ====== DASHBOARD ====== */
.dashboard-container {
    padding: 1rem;
}
.dashboard-header,
.welcome-section {
    margin-bottom: 2rem;
}
.dashboard-header h1,
.welcome-section h2 {
    font-size: 2rem;
    color: #2c3e50;
}

.stats-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
    gap: 1.5rem;
    margin-bottom: 2rem;
}

.stat-card {
    background: white;
    padding: 1.5rem;
    border-radius: 8px;
    box-shadow: 0 2px 5px rgba(0, 0, 0, 0.05);
    text-align: center;
}
.stat-card h3 {
    font-size: 1rem;
    color: #7f8c8d;
    margin-bottom: 0.5rem;
}
.stat-number {
    font-size: 2.5rem;
    font-weight: bold;
    color: #2c3e50;
}

.dashboard-sections {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
    gap: 2rem;
}

.section {
    background: white;
    padding: 1.5rem;
    border-radius: 8px;
    box-shadow: 0 2px 5px rgba(0, 0, 0, 0.05);
}
.section h3 {
    margin-bottom: 1rem;
    border-bottom: 2px solid #ecf0f1;
    padding-bottom: 0.5rem;
}

/* User Dashboard specific */
.entities-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
    gap: 1rem;
}
.entity-card,
.available-entity {
    border: 1px solid #ecf0f1;
    padding: 1rem;
    border-radius: 6px;
}
.entity-card h4 {
    color: #34495e;
}
.entity-meta {
    font-size: 0.8rem;
    color: #7f8c8d;
    margin: 0.5rem 0;
}
.entity-actions {
    margin-top: 1rem;
}
.no-data {
    color: #95a5a6;
    padding: 1rem;
    text-align: center;
}

/* ====== PÁGINA CRUD ====== */
.crud-container {
    background: white;
    padding: 2rem;
    border-radius: 8px;
    box-shadow: 0 4px 20px rgba(0, 0, 0, 0.05);
}
.crud-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 1.5rem;
}
.crud-filters {
    margin-bottom: 1.5rem;
}
.filter-group {
    display: flex;
    gap: 1rem;
}
.filter-group input,
.filter-group select {
    padding: 0.5rem;
    border-radius: 4px;
    border: 1px solid #ccc;
}
.crud-table {
    overflow-x: auto;
}
#dataTable {
    width: 100%;
    border-collapse: collapse;
}
#dataTable th,
#dataTable td {
    padding: 0.75rem 1rem;
    text-align: left;
    border-bottom: 1px solid #ecf0f1;
}
#dataTable th {
    background-color: #f9f9f9;
    font-weight: bold;
    color: #555;
}
#dataTable tr:hover {
    background-color: #f5f5f5;
}
td.actions {
    display: flex;
    gap: 0.5rem;
}

/* ====== PÁGINA DE ERROR 404 ====== */
.error-content {
    text-align: center;
    background: white;
    padding: 3rem;
    border-radius: 8px;
}
.error-content h1 {
    font-size: 6rem;
    color: #e74c3c;
    margin-bottom: 0;
}
.error-content h2 {
    font-size: 2rem;
    color: #34495e;
    margin-bottom: 1rem;
}
.error-content p {
    color: #7f8c8d;
    margin-bottom: 2rem;
}
```

---

by Antonio Santiago
