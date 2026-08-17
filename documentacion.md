# Árbol familiar

## 📑 Especificación de Arquitectura e Infraestructura de Software
1. **Resumen Ejecutivo**
El presente documento describe la arquitectura de software, stack tecnológico e infraestructura cloud elegida para el desarrollo del **Starter Kit / Boilerplate Web**. El objetivo principal es construir una plataforma moderna, escalable, segura y desacoplada, utilizando una arquitectura de microservicios/decoupled de capa gratuita (Free Tier Operations), garantizando costo $0.00 USD de mantenimiento operativo sin comprometer los estándares de la industria.

2. **Visión General de la Infraestructura y Componentes**
La solución adopta una **Arquitectura Desacoplada (Decoupled Frontend / Backend Architecture)**. El sistema se divide en cuatro pilares independientes:

    ```
    [ Capa de Presentación ]       [ Capa de Aplicación / API ]      [ Capa de Persistencia y Archivos ]
    ┌─────────────────────────┐     ┌───────────────────────────┐     ┌─────────────────────────────────┐
    │     Frontend (SPA)      │     │      Backend (REST API)   │     │    Relational DB & S3 Storage   │
    │  Vue 3 + Vite + Pinia   │ ──> │   Node.js + Express +     │ ──> │  PostgreSQL + Cloud Storage     │
    │                         │     │        Prisma ORM         │     │            (Supabase)           │
    └─────────────────────────┘     └───────────────────────────┘     └─────────────────────────────────┘
                │                                │                                    │
                ▼                                ▼                                    ▼
        Hosted en Vercel                Hosted en Render.com               Hosted en Supabase Cloud
    ```

3. **Matriz de Componentes Tecnológicos y Justificación**
    1. **Capa de Presentación (Frontend)**
        + Tecnología Core: Vue.js 3 (Composition API / `<script setup>`) + Vite.
        + Gestión de Estado: Pinia.
        + Enrutamiento: Vue Router (con Navigation Guards para RBAC).
        + Infraestructura de Despliegue: Vercel / Netlify.
        + ¿Por qué esta elección?:
            + Rendimiento y Ligereza: Vite ofrece tiempos de compilación e instanciación (HMR) en milisegundos.
            + SPA Pura: Al ser un renderizado del lado del cliente (CSR), la web se puede hospedar de forma estática en CDNs globales de Vercel/Netlify de forma totalmente gratuita y ultrarrápida.
            + Seguridad y Mantenibilidad: La aplicación frontend no maneja lógica sensible de negocio ni almacenamiento directo de credenciales de BD.
    2. **Capa de Negocio / API (Backend)**
        + Tecnología Core: Node.js + Express.js.
        + Capa de Abstracción de Datos (ORM): Prisma ORM.
        + Mecanismo de Autenticación: JSON Web Tokens (JWT) + Hashing seguro de contraseñas (bcrypt / argon2).
        + Infraestructura de Despliegue: Render.com (Web Service).
        + ¿Por qué esta elección?:
            + Desacoplamiento Operativo: Al exponer una API REST pura (/api/v1/...), el backend es agnóstico del cliente. En el futuro, aplicaciones móviles (iOS/Android) o clientes de terceros podrán consumir la misma API sin cambios.
            + Seguridad Absoluta: Las claves de entorno (DATABASE_URL, JWT_SECRET) residen exclusivamente dentro de los contenedores aislados de Render.
            + Prisma ORM: Aporta un tipado estricto, previene ataques de Inyección SQL (SQLi) y permite gestionar migraciones de esquemas de base de datos de manera automatizada y declarativa.
    3. **Capa de Persistencia de Datos y Archivos**
        + Motor de Base de Datos: PostgreSQL.
        + Gestión de Archivos (Object Storage): Supabase Storage (Compatible con AWS S3).
        + Proveedor de Infraestructura: Supabase Cloud.
        + ¿Por qué esta elección?:
            + Integridad Referencial (SQL vs NoSQL): Para la gestión de Usuarios, Roles (RBAC) y Permisos, el modelo relacional SQL es superior a NoSQL. Permite aplicar restricciones de clave foránea (Foreign Keys), transacciones ACID y relaciones N:M (Muchos a Muchos) nativas.
            + Consolidación de Servicios: Supabase resuelve en una sola plataforma gratuita la base de datos relacional (PostgreSQL) y el almacenamiento masivo de imágenes/PDFs (Bucket S3), simplificando la gestión de tokens de API y facturación.

4. **Control de Acceso Basado en Roles (RBAC - Security Model)**
+ La arquitectura implementa un modelo de seguridad en dos niveles:
    1. Nivel Backend (Autorización Firme): Middlewares interceptores de peticiones (authorize(['ADMIN', 'SUPER_ADMIN'])). Si el token JWT presentado no contiene el privilegio requerido en el backend, la API responderá con un estado HTTP 403 Forbidden inmediatamente, protegiendo los datos.
    2. Nivel Frontend (UX / Navegación): El router de Vue.js utiliza Guards para evaluar el estado del usuario en Pinia. Si un usuario sin privilegios intenta acceder a una ruta administrativa (ej. /admin), es redirigido automáticamente a la vista de acceso denegado o login.

5. **Estrategia de Costos e Infraestructura ($0 USD)**
    Servicio            | Proveedor         | Recurso Gratis Asignado               | Límite de Seguridad
    --------------------|-------------------|---------------------------------------|---------------------------------------------------------------
    Frontend            | Vercel / Netlify  | 100 GB / mes de Ancho de Banda        | Sin cobros automáticos (se suspende el servicio si se excede).
    Backend             | APIRender.com     | 750 horas / mes de ejecución Linux    | Se duerme tras inactividad. Sin cobros automáticos.
    Base de Datos       | Supabase          | 500 MB PostgreSQL dedicado            | No requiere tarjeta de crédito.
    Archivos (PDF/Img)  | Supabase Storage  | 1 GB de Almacenamiento S3             | Totalmente aislado de la BD principal.


## 📖 Guía de Configuración de Base de Datos y Almacenamiento
Esta sección detalla el proceso paso a paso para desplegar la capa de datos en la nube (Supabase) y preparar el entorno de desarrollo local.

### 🛠️ PARTE 1: Creación del Proyecto en Supabase (Producción / Cloud)

#### Paso 1: Registro e Inicio de Sesión
1. Dirígete a [supabase.com](https://supabase.com).
2. Haz clic en "Start your project" / "Sign In".
3. Selecciona la opción "Continue with GitHub" para autenticarte usando tu cuenta de GitHub (esto evita crear contraseñas adicionales y facilitará futuras integraciones).

#### Paso 2: Crear una Nueva Organización y Proyecto
1. En el panel principal (Dashboard), haz clic en el botón "New Project".
2. Si es tu primera vez, te pedirá seleccionar una Organization (puedes crear una con tu nombre o el nombre del proyecto).
3. Completa el formulario de creación con los siguientes datos:
    + Name: starter-kit-db (o el nombre de tu proyecto).
    + Database Password: ⚠️ ¡Mucha atención aquí! Genera una contraseña segura y guárdala en un lugar seguro (un gestor de contraseñas). La necesitarás para construir la URL de conexión.
    + Region: Selecciona la región geográfica más cercana a ti o a tus usuarios principales (por ejemplo, West Europe / Frankfurt o US East / N. Virginia).
    + Pricing Plan: Asegúrate de seleccionar Free - $0/month.
4. Haz clic en "Create new project".
    + ⏳ Nota: Supabase tardará entre 1 y 2 minutos en aprovisionar la base de datos PostgreSQL en la nube.

#### Paso 3: Obtener las Credenciales de la Base de Datos (PostgreSQL)
Para que nuestro Backend en Node.js (vía Prisma ORM) se conecte a la base de datos, necesitamos el Connection String (Cadena de conexión).
1. En el menú lateral izquierdo de Supabase, ve a Project Settings (el icono de engranaje ⚙️ en la parte inferior).
2. Selecciona la sección Database.
3. Desplázate hasta la sección Connection string.
4. Selecciona la pestaña URI o Transaction pooler (para Prisma se recomienda la pestaña URI o utilizar el puerto 6543 / 5432 según el modo).
5. Copia la cadena que tiene una estructura similar a esta:
    ```
    postgresql://postgres:[YOUR-PASSWORD]@db.xxxxxx.supabase.co:5432/postgres
    ```
6. Reemplaza [YOUR-PASSWORD] en la cadena por la contraseña real que creaste en el Paso 2

#### Paso 4: Crear el Bucket de Almacenamiento para Archivos (Storage)
Para guardar las imágenes de perfil, PDFs y documentos sin recargar la base de datos:
1. En el menú lateral izquierdo de Supabase, haz clic en el icono de Storage (🗂️).
2. Haz clic en el botón "Create a new bucket".
3. Configura el bucket con los siguientes parámetros:
    + Bucket Name: app-uploads
    + Public bucket: ACTIVADO 🟢 (esto permitirá que las imágenes y PDFs sean accesibles mediante una URL pública https://... desde el navegador).
4. Haz clic en "Save".
5. Ve a Project Settings ⚙️ -> API y copia las siguientes claves necesarias para subir archivos desde la API:
    + Project URL: [https://xxxxxx.supabase.co](https://xxxxxx.supabase.co)
    + anon / public key: eyJhbGciOiJKV1QiLC... (Clave pública)
    + service_role key: eyJhbGciOiJKV1QiLC... (Clave privada para el backend - ¡mantener secreta!).

### 💻 PARTE 2: Configuración del Entorno de Base de Datos Local (Desarrollo)
Para trabajar de manera fluida en tu máquina local sin depender de internet ni afectar la base de datos en Supabase, configuraremos un PostgreSQL local.

#### Opción A (Recomendada): Usando Docker Compose
Si tienes Docker instalado en tu equipo, esta es la forma más limpia:
1. Creamos un archivo docker-compose.yml en la raíz de nuestro proyecto backend (lo crearemos formalmente más adelante):
```yml
services:
  postgres:
    image: postgres:15-alpine
    container_name: local_postgres
    environment:
      POSTGRES_USER: dev_user
      POSTGRES_PASSWORD: dev_password
      POSTGRES_DB: local_starter_db
    ports:
      - "5432:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
```

#### Opción B: PostgreSQL Instalado Nativo
Si no usas Docker, puedes instalar PostgreSQL directamente en tu sistema operativo (PostgresApp en macOS, Installer en Windows/Linux) y crear una base de datos local llamada `local_starter_db`.

### 🔑 PARTE 3: Matriz de Variables de Entorno (.env)
A continuación se documenta el esquema del archivo .env que utilizará nuestro Backend para alternar fácilmente entre la BD Local y Supabase mediante simples comentarios:
```env
# ==========================================
# PUERTO DEL SERVIDOR BACKEND
# ==========================================
PORT=4000
NODE_ENV=development
# NODE_ENV=production

# ==========================================
# CADENA DE CONEXIÓN A POSTGRESQL (PRISMA)
# ==========================================
# 🔹 OPCIÓN LOCAL (Activa para desarrollo local):
DATABASE_URL="postgresql://dev_user:dev_password@localhost:5432/local_starter_db?schema=public"

# 🔸 OPCIÓN SUPABASE (Descomentar para staging/producción):
# DATABASE_URL="postgresql://postgres:TU_CONTRASEÑA_AQUI@db.xxxxxx.supabase.co:5432/postgres?schema=public"

# ==========================================
# CONFIGURACIÓN DE AUTENTICACIÓN (JWT)
# ==========================================
JWT_SECRET="clave_secreta_super_segura_para_desarrollo_local_123"
JWT_EXPIRES_IN="7d"

# ==========================================
# CONFIGURACIÓN SUPABASE STORAGE (ARCHIVOS)
# ==========================================
SUPABASE_URL="https://xxxxxx.supabase.co"
SUPABASE_SERVICE_ROLE_KEY="eyJhbGciOiJKV... (tu service_role key)"
SUPABASE_BUCKET_NAME="app-uploads"
```


## 📄 Guía de Estructuración Local y Control de Versiones (Git & GitHub)
Esta sección documenta el procedimiento estándar para organizar el espacio de trabajo local, levantar la infraestructura de desarrollo mediante Docker y vincular los proyectos con repositorios remotos en GitHub.

### 🏗️ PARTE 1: Organización del Espacio de Trabajo (Multi-Repositorio)
A diferencia de un monolito, la arquitectura desacoplada requiere separar el código fuente de la infraestructura y de los diferentes clientes.

+ **Estructura de Directorios**
    ```
    mi-proyecto-starter/          <─── Directory Raíz (Contenedor de Infraestructura)
    │
    ├── .gitignore                 <─── Ignora variables de entorno y datos de Docker
    ├── docker-compose.yml         <─── Definición del servicio PostgreSQL Local
    │
    ├── starter-backend/           <─── [Repo 1] API REST (Node.js + Express + Prisma)
    └── starter-frontend/          <─── [Repo 2] Cliente Web (Vue 3 + Vite + Pinia)
    ```

### 🚀 PARTE 2: Creación de la Estructura e Infraestructura Local

#### Paso 1: Crear la Jerarquía de Directorios
Ejecuta los siguientes comandos en tu terminal para inicializar el espacio de trabajo:
```bash
# 1. Crear la carpeta raíz del proyecto y acceder a ella
mkdir mi-proyecto-starter
cd mi-proyecto-starter

# 2. Crear los directorios independientes para Backend y Frontend
mkdir starter-backend
mkdir starter-frontend
```

#### Paso 2: Crear el Orquestador de Infraestructura Local (docker-compose.yml)
+ En la raíz del proyecto (`mi-proyecto-starter/`), crea el archivo `docker-compose.yml`:
```yml
services:
  postgres_dev:
    image: postgres:15-alpine
    container_name: local_starter_postgres
    restart: always
    environment:
      POSTGRES_USER: dev_user
      POSTGRES_PASSWORD: dev_password
      POSTGRES_DB: local_starter_db
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

+ Crea también un archivo `.gitignore` en la raíz para evitar subir archivos temporales o volúmenes accidentales:
```gitignore
# Docker / Datos locales
.postgres_data/
*.log

# Archivos de entorno
.env
.env.local
```

#### Paso 3: Inicializar la Base de Datos Local
+ Ejecuta el siguiente comando en la raíz para levantar el contenedor de PostgreSQL en segundo plano:
```bash
docker compose up -d
```

+ Verificación del Servicio:
    + Para confirmar que la base de datos está activa y escuchando en el puerto 5432:
        ```bash
        docker ps
        ```
    + Deberías ver el contenedor local_starter_postgres con estado Up.

### 🐙 PARTE 3: Inicialización y Publicación en GitHub
Para mantener un despliegue independiente (CI/CD) en la nube, crearemos dos repositorios separados en GitHub: uno para el backend y otro para el frontend.

#### Paso 1: Crear los Repositorios en GitHub
1. Inicia sesión en tu cuenta de GitHub.
2. Crea el primer repositorio:
    + Repository name: `starter-backend`
    + Description: API REST de Autenticación y RBAC (Node.js, Express, Prisma)
    + Visibility: Public o Private (a tu preferencia).
    + Initialize this repository with: Deja todas las casillas desmarcadas.
    + Haz clic en "Create repository".
3. Crea el segundo repositorio:
    + Repository name: starter-frontend
    + Description: Cliente SPA de Autenticación y Panel de Control (Vue 3, Vite, Pinia)
    + Visibility: Public o Private.
    + Initialize this repository with: Deja todas las casillas desmarcadas.
    + Haz clic en "Create repository".

#### Paso 2: Inicializar Git en el Backend (starter-backend)
Entra en la carpeta del backend, inicializa el repositorio local, crea un archivo inicial y vincúlalo a GitHub:
```bash
# 1. Navegar al directorio del backend
cd starter-backend

# 2. Inicializar Git
git init
git branch -M main

# 3. Crear un archivo README inicial
echo "# Starter Backend API" > README.md

# 4. Registrar los cambios locales
git add .
git commit -m "chore: initial commit - backend workspace setup"

# 5. Vincular y subir a GitHub (reemplaza 'TU_USUARIO' por tu nombre de usuario en GitHub)
# git remote add origin https://github.com/TU_USUARIO/starter-backend.git
git remote add origin git@github.com:TU_USUARIO/starter-backend.git
git push -u origin main
```



```bash
```



```bash
```


familytree2026-backend
familytree2026-frontend
familytree2026-documentacion


git remote add origin https://github.com/TU_USUARIO/starter-backend.git
git remote add origin https://github.com/petrix12/familytree2026-backend.git

git remote add origin git@github.com:petrix12/familytree2026-backend.git


git remote add origin https://github.com/TU_USUARIO/starter-backend.git


git remote add origin git@github.com:TU_USUARIO/starter-backend.git