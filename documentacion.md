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


## 📖 Configuración de Base de Datos y Almacenamiento
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


## 📄 Estructuración Local y Control de Versiones (Git & GitHub)
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

#### Paso 3: Inicializar Git en el Frontend (starter-frontend)
Regresa a la raíz, entra en la carpeta del frontend e inicializa su propio control de versiones:
```bash
# 1. Volver a la raíz y entrar al directorio del frontend
cd ../starter-frontend

# 2. Inicializar Git
git init
git branch -M main

# 3. Crear un archivo README inicial
echo "# Starter Frontend App" > README.md

# 4. Registrar los cambios locales
git add .
git commit -m "chore: initial commit - frontend workspace setup"

# 5. Vincular y subir a GitHub (reemplaza 'TU_USUARIO' por tu nombre de usuario en GitHub)
# git remote add origin https://github.com/TU_USUARIO/starter-frontend.git
git remote add origin git@github.com:TU_USUARIO/starter-frontend.git
git push -u origin main
```


## 📑 Instalación de Node.js mediante NVM en WSL (Ubuntu)
Esta sección registra la preparación del entorno de ejecución de Node.js en el subsistema Linux (WSL) utilizando NVM para gestionar versiones de forma aislada y sin permisos de superusuario (root).

### 🛠️ Paso 1: Instalar NVM (Node Version Manager)
+ Ejecuta el script oficial de instalación de NVM en tu terminal de WSL:
    ```bash
    curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
    ```
+ Una vez finalizada la descarga, recarga la configuración de tu terminal para activar nvm:
    ```bash
    source ~/.bashrc
    ```

### 🛠️ Paso 2: Instalar la versión estable de Node.js (LTS)
+ Con NVM activo, instala la última versión con soporte extendido (LTS) de `Node.js` y `npm`:
    ```bash
    nvm install --lts
    ```
+ NVM la configurará automáticamente como la versión por defecto de tu sistema.

### 🛠️ Paso 3: Verificar la Instalación
+ Comprueba que tanto `node` como `npm` están disponibles en tu terminal:
    ```bash
    node -v
    npm -v
    ```

## ⚙️ Inicialización del Backend y Modelo de Datos (Prisma ORM)
Esta sección detalla el proceso para estructurar la API REST en Node.js, configurar las dependencias de seguridad y desplegar el modelo de datos relacional (Usuarios y Roles - RBAC) utilizando Prisma ORM.

1. Inicialización del Proyecto e Instalación de Dependencias:
    + Ubicados en la carpeta familytree2026-backend, inicializamos el paquete de Node.js e instalamos el conjunto de librerías necesarias:
        ```bash
        # 1. Navegar al directorio del backend
        cd ~/projects/family_tree2026/familytree2026-backend

        # 2. Inicializar package.json con valores por defecto
        npm init -y

        # 3. Instalar dependencias de producción
        npm install express @prisma/client bcryptjs jsonwebtoken dotenv cors @supabase/supabase-js

        # 4. Instalar dependencias de desarrollo
        npm install -D prisma nodemon        
        ```
    + Desglose de Paquetes Instalados
        Paquete                 | Tipo          | Propósito
        ------------------------|---------------|---------------------------------------------------------------------------------------
        express                 | Producción    | Framework HTTP para crear endpoints de la API REST.
        @prisma/client          | Producción    | Cliente autocompletado y tipado para consultar la base de datos.
        bcryptjs                | Producción    | Hashing unidireccional de contraseñas de usuarios.
        jsonwebtoken            | Producción    | Generación y verificación de tokens de autenticación (JWT).
        cors                    | Producción    | Middleware para permitir peticiones del cliente Vue 3 (Cross-Origin Resource Sharing).
        @supabase/supabase-js   | Producción    | SDK para interactuar con el bucket de archivos S3 en Supabase.
        prisma                  | Desarrollo    | CLI para gestionar esquemas, migraciones y el cliente ORM.
        nodemon                 | Desarrollo    | Reinicio automático del servidor Node.js ante cambios de código.

2. Configuración de Prisma ORM:
    + Inicializamos la estructura base de Prisma en el proyecto ejecutando:
        ```bash
        npx prisma init
        ```
    + Este comando genera:
        + El directorio `prisma/` con el archivo principal `schema.prisma`.
        + Un archivo `.env` en la raíz de `familytree2026-backend`.

3. Definición del Esquema Relacional (RBAC: Users & Roles)
    + Abre el archivo `prisma/schema.prisma` para definir la estructura relacional para el control de acceso basado en roles (N:M - Muchos a Muchos) y reemplaza su código por este:
        ```json
        generator client {
            provider = "prisma-client-js"
        }

        datasource db {
            provider = "postgresql"
        }

        // Modelo de Usuario
        model User {
            id        String     @id @default(uuid())
            name      String
            email     String     @unique
            password  String
            isActive  Boolean    @default(true)
            avatarUrl String?
            createdAt DateTime   @default(now())
            updatedAt DateTime   @updatedAt
            roles     UserRole[]

            @@map("users")
        }

        // Modelo de Rol
        model Role {
            id          String     @id @default(uuid())
            name        String     @unique // Ej: "SUPER_ADMIN", "ADMIN", "USER"
            description String?
            users       UserRole[]

            @@map("roles")
        }

        // Tabla Intermedia para Relación N:M
        model UserRole {
            userId String
            roleId String
            user   User   @relation(fields: [userId], references: [id], onDelete: Cascade)
            role   Role   @relation(fields: [roleId], references: [id], onDelete: Cascade)

            @@id([userId, roleId])
            @@map("user_roles")
        }       
        ```
4. Configurar `prisma.config.ts`:
    + Abre el archivo `prisma.config.ts` que se creó automáticamente en la raíz de familytree2026-backend y asegúrate de que contenga lo siguiente:
        ```ts
        import { defineConfig } from '@prisma/config';

        export default defineConfig({
            earlyAccess: true,
            schema: 'prisma/schema.prisma',
            datasourceUrl: process.env.DATABASE_URL,
        });        
        ```
5. Configuración del Archivo `.env` Local
    + Edita el archivo `.env` en la raíz de `familytree2026-backend` para apuntar al contenedor de PostgreSQL creado con Docker:
        ```env
        # ==========================================
        # CONFIGURACIÓN DEL SERVIDOR
        # ==========================================
        PORT=4000
        NODE_ENV=development

        # ==========================================
        # CONEXIÓN A POSTGRESQL LOCAL (DOCKER)
        # ==========================================
        DATABASE_URL="postgresql://dev_user:dev_password@localhost:5432/local_starter_db?schema=public"

        # ==========================================
        # AUTENTICACIÓN (JWT)
        # ==========================================
        JWT_SECRET="familytree_dev_jwt_secret_key_2026_super_secure"
        JWT_EXPIRES_IN="7d"

        # ==========================================
        # ALMACENAMIENTO NUBE (SUPABASE STORAGE)
        # ==========================================
        SUPABASE_URL="https://xxxxxx.supabase.co"
        SUPABASE_SERVICE_ROLE_KEY="tu_service_role_key_aqui"
        SUPABASE_BUCKET_NAME="app-uploads"
        ```        

6. Ejecución de la Migración Inicial:
    + Con el contenedor de Docker activo (docker-compose up -d), ejecuta el siguiente comando para generar las tablas físicas en PostgreSQL:
        ```bash
        npx prisma migrate dev --name init_users_and_roles
        ```
    + Verificación de la Base de Datos:
        + Para verificar que las tablas users, roles y user_roles se crearon correctamente con sus relaciones, puedes abrir la interfaz gráfica de Prisma:
            ```bash
            npx prisma studio
            ```
        + (Abre en el navegador una consola web interactiva en http://localhost:5555).

## 📑 Backend Base, Carga Inicial de Datos (Seed Script) y Servidor Express
Esta sección consolida la inicialización del backend en Node.js, la seguridad de repositorio, la carga de datos iniciales (Seed con Prisma 7) y la API REST con Express.

1. Protección de Archivos Sensibles (`.gitignore`): Antes de realizar cualquier commit, crea el archivo `.gitignore` en la raíz de familytree2026-backend:
    ```gitignore
    # Dependencias
    node_modules/

    # Variables de entorno
    .env
    .env.local
    .env.*

    # Archivos de registro y sistema
    *.log
    .DS_Store
    dist/    
    ```
2. Carga Inicial de Datos (Seed Script):
    + Paso A: Instalar Adaptador de PostgreSQL para Prisma 7
        ```bash
        npm install @prisma/adapter-pg pg
        ```
    + Paso B: Crear el Script `prisma/seed.js`:
        ```js
        const { PrismaClient } = require('@prisma/client');
        const { PrismaPg } = require('@prisma/adapter-pg');
        const { Pool } = require('pg');
        require('dotenv').config();

        // Inicializar el pool de conexiones con la URL de la base de datos
        const pool = new Pool({ connectionString: process.env.DATABASE_URL });
        const adapter = new PrismaPg(pool);
        const prisma = new PrismaClient({ adapter });

        async function main() {
            console.log('🌱 Iniciando la carga de datos iniciales (Seed)...');

            const roles = [
                { name: 'SUPER_ADMIN', description: 'Acceso total y gestión del sistema' },
                { name: 'ADMIN', description: 'Administrador de contenido y usuarios' },
                { name: 'USER', description: 'Usuario estándar registrado' },
            ];

            for (const role of roles) {
                await prisma.role.upsert({
                    where: { name: role.name },
                    update: {},
                    create: role,
                });
            }

            console.log('✅ Roles creados/verificados correctamente en la base de datos.');
        }

        main()
        .catch((e) => {
            console.error('❌ Error ejecutando el seed:', e);
            process.exit(1);
        })
        .finally(async () => {
            await prisma.$disconnect();
            await pool.end();
        });       
        ```
    + Paso C: Configurar el Comando de Seed en `prisma.config.ts`:
        + Abre `familytree2026-backend/prisma.config.ts` y añade la propiedad seed: `"node prisma/seed.js"` dentro de migrations::
            ```ts
            // ...
            export default defineConfig({
                // ...
                migrations: {
                    path: "prisma/migrations",
                    seed: "node prisma/seed.js",    // <-- Agregar esta línea
                },
                // ...
            })        
            ```
    + Paso D: Generar el Prisma Client: Ejecuta el siguiente comando en la terminal de familytree2026-backend:
        ```bash
        npx prisma generate
        ```
        + Este comando leerá tu esquema `prisma/schema.prisma` y compilará la librería `@prisma/client` dentro de `node_modules/`.
    + Paso E: Ejecutar el Seed: Ejecuta el siguiente comando en la terminal para registrar los roles:
        ```bash
        npx prisma db seed
        ```
3. Cliente Reutilizable de Prisma (`src/config/prisma.js`)
    + Para evitar abrir múltiples conexiones a la base de datos en los controladores, centralizamos la instancia:
        ```js
        const { PrismaClient } = require('@prisma/client');
        const { PrismaPg } = require('@prisma/adapter-pg');
        const { Pool } = require('pg');
        require('dotenv').config();

        const pool = new Pool({ connectionString: process.env.DATABASE_URL });
        const adapter = new PrismaPg(pool);
        const prisma = new PrismaClient({ adapter });

        module.exports = prisma;        
        ```

4. Crear el Punto de Entrada de la API Express (`src/app.js`):
    + Crea la carpeta `src/` dentro de `familytree2026-backend` y dentro crea el archivo `app.js`:
        ```js
        const express = require('express');
        const cors = require('cors');
        require('dotenv').config();

        // Rutas
        const authRoutes = require('./routes/auth.routes');
        const adminRoutes = require('./routes/admin.routes');

        const app = express();
        const PORT = process.env.PORT || 4000;

        // Middlewares Globales
        app.use(cors());
        app.use(express.json());

        // Ruta raíz informativa
        app.get('/', (req, res) => {
            res.send('API REST de FamilyTree2026 ejecutándose. Visita /api/v1/health para estado.');
        });

        // Ruta de comprobación de estado (Health Check)
        app.get('/api/v1/health', (req, res) => {
            res.status(200).json({
                status: 'success',
                message: 'API FamilyTree2026 operativa',
                environment: process.env.NODE_ENV,
                timestamp: new Date().toISOString(),
            });
        });

        // Registrar Rutas de la API
        app.use('/api/v1/auth', authRoutes);
        app.use('/api/v1/admin', adminRoutes);

        // Inicialización del Servidor
        app.listen(PORT, () => {
            console.log(`🚀 Servidor ejecutándose en http://localhost:${PORT}`);
            console.log(`📌 Entorno: ${process.env.NODE_ENV || 'development'}`);
        });      
        ```

5. Configurar Script de Arranque en `package.json`:
    + Añade los scripts de ejecución dentro de la sección "scripts" de tu `package.json`:
        ```json
        "scripts": {
            "start": "node src/app.js",
            "dev": "nodemon src/app.js"
        }
        ```
    + Prueba de Vuelo:
        ```bash
        npm run dev
        ```
    + Abre tu navegador o cliente HTTP y accede a `http://localhost:4000/api/v1/health`. Si ves el JSON de respuesta con el mensaje de éxito, ¡tu servidor Backend está 100% configurado y funcionando!

6. Subir a GitHub:
    + Asegúrate de crear el archivo `.gitignore` ahora mismo en `familytree2026-backend`, y luego ejecuta:
        ```bash
        # 1. Verificar qué archivos detecta Git (NO deben aparecer node_modules ni .env)
        git status

        # 2. Agregar cambios y subir a GitHub
        git add .
        git commit -m "feat: setup Express, Prisma 7 adapter, seed script and gitignore"
        git push origin main
        ```

## 📑 Módulo de Autenticación (Auth)
Esta fase implementa el registro de usuarios, el inicio de sesión y la emisión de tokens JWT (JSON Web Tokens) que incluirán los roles del usuario para proteger las rutas de la API.

1. Instalar Dependencias de Seguridad: Ejecuta el siguiente comando en la terminal de `familytree2026-backend`:
    ```bash
    npm install bcryptjs jsonwebtoken express-validator
    ```
    + bcryptjs: Para encriptar las contraseñas de forma segura antes de guardarlas en PostgreSQL.
    + jsonwebtoken: Para generar y verificar tokens de sesión firmados.
    + express-validator: Para validar los datos de entrada (email válido, contraseña fuerte, campos requeridos) antes de procesarlos.

2. Definir Variables de Entorno para JWT: Abre tu archivo `.env` y añade las claves de configuración para los tokens JWT:
    ```env
    # JWT Settings
    JWT_SECRET=super_secret_key_familytree_2026_change_in_production
    JWT_EXPIRES_IN=24h
    ```

3. Estructura de Capas para la Autenticación: Crearemos los archivos necesarios siguiendo la arquitectura limpia del proyecto:
    ```
    src/
    ├── config/
    │   └── prisma.js               <-- Cliente centralizado de Prisma
    ├── controllers/
    │   └── auth.controller.js      <-- Lógica de Registro y Login
    ├── middlewares/
    │   └── validate.middleware.js  <-- Validaciones de entrada con express-validator
    ├── routes/
    │   └── auth.routes.js          <-- Endpoints de /api/v1/auth
    └── app.js                      <-- Registrar las nuevas rutas    
    ```

4. Middleware de Validación (`src/middlewares/validate.middleware.js`): Crea la carpeta `src/middlewares/` y dentro el archivo `validate.middleware.js`:
    ```js
    const { validationResult } = require('express-validator');

    const validate = (req, res, next) => {
        const errors = validationResult(req);
        if (!errors.isEmpty()) {
            return res.status(400).json({
                status: 'fail',
                errors: errors.array().map((err) => ({
                    field: err.path,
                    message: err.msg,
                })),
            });
        }
        next();
    };

    module.exports = validate;    
    ```

5. Controlador de Autenticación (`src/controllers/auth.controller.js`): Crea la carpeta `src/controllers/` y dentro el archivo `auth.controller.js`:
    ```js
    const bcrypt = require('bcryptjs');
    const jwt = require('jsonwebtoken');
    const prisma = require('../config/prisma');

    // Auxiliar para generar Tokens JWT
    const generateToken = (user, roles) => {
        return jwt.sign(
            {
                id: user.id,
                email: user.email,
                roles: roles,
            },
            process.env.JWT_SECRET,
            { expiresIn: process.env.JWT_EXPIRES_IN || '24h' }
        );
    };

    // 1. REGISTRO DE USUARIO
    const register = async (req, res) => {
        try {
            const { email, password, firstName, lastName } = req.body;
            const fullName = `${firstName} ${lastName}`;

            // Verificar si el usuario ya existe
            const existingUser = await prisma.user.findUnique({ where: { email } });
            if (existingUser) {
                return res.status(400).json({
                    status: 'fail',
                    message: 'El correo electrónico ya está registrado',
                });
            }

            // Buscar el rol por defecto (USER)
            const defaultRole = await prisma.role.findUnique({ where: { name: 'USER' } });
            if (!defaultRole) {
                return res.status(500).json({
                    status: 'error',
                    message: 'El rol por defecto (USER) no existe en la base de datos',
                });
            }

            // Encriptar la contraseña
            const salt = await bcrypt.genSalt(10);
            const passwordHash = await bcrypt.hash(password, salt);

            // Crear el usuario y asignarle el rol USER en una transacción implícita
            const newUser = await prisma.user.create({
                data: {
                    email,
                    password: passwordHash, // Usamos la columna 'password' del schema
                    name: fullName,         // Usamos la columna 'name' del schema
                    roles: {
                        create: {
                            roleId: defaultRole.id,
                        },
                    },
                },
                select: {
                    id: true,
                    email: true,
                    name: true,
                    createdAt: true,
                },
            });

            // Generar Token
            const token = generateToken(newUser, ['USER']);

            return res.status(201).json({
                status: 'success',
                message: 'Usuario registrado correctamente',
                data: {
                    user: newUser,
                    token,
                },
            });
        } catch (error) {
            console.error('Error en registro:', error);
            return res.status(500).json({ status: 'error', message: 'Error interno del servidor' });
        }
    };

    // 2. INICIO DE SESIÓN (LOGIN)
    const login = async (req, res) => {
        try {
            const { email, password } = req.body;

            // Buscar usuario con sus roles asociados
            const user = await prisma.user.findUnique({
                where: { email },
                include: {
                    roles: {
                        include: {
                            role: true,
                        },
                    },
                },
            });

            if (!user || !user.isActive) {
                return res.status(401).json({
                    status: 'fail',
                    message: 'Credenciales inválidas o cuenta desactivada',
                });
            }

            // Comprobar contraseña (usando user.password)
            const isPasswordValid = await bcrypt.compare(password, user.password);
            if (!isPasswordValid) {
                return res.status(401).json({
                    status: 'fail',
                    message: 'Credenciales inválidas',
                });
            }

            // Extraer nombres de roles
            const userRoles = user.roles.map((ur) => ur.role.name);

            // Generar Token JWT
            const token = generateToken(user, userRoles);

            return res.status(200).json({
                status: 'success',
                message: 'Inicio de sesión exitoso',
                data: {
                    user: {
                        id: user.id,
                        email: user.email,
                        name: user.name, // ✅ Corregido (en lugar de firstName / lastName)
                        roles: userRoles,
                    },
                    token,
                },
            });
        } catch (error) {
            console.error('Error en login:', error);
            return res.status(500).json({ status: 'error', message: 'Error interno del servidor' });
        }
    };

    module.exports = { register, login };  
    ```

6. Rutas de Autenticación (`src/routes/auth.routes.js`): Crea la carpeta `src/routes/` y dentro el archivo `auth.routes.js`:
    ```js
    const express = require('express');
    const { body } = require('express-validator');
    const { register, login } = require('../controllers/auth.controller');
    const validate = require('../middlewares/validate.middleware');

    const router = express.Router();

    // Reglas de validación para Registro
    const registerValidation = [
        body('email').isEmail().withMessage('Debe proporcionar un correo electrónico válido'),
        body('password').isLength({ min: 6 }).withMessage('La contraseña debe tener al menos 6 caracteres'),
        body('firstName').notEmpty().withMessage('El nombre es obligatorio'),
        body('lastName').notEmpty().withMessage('El apellido es obligatorio'),
        validate,
    ];

    // Reglas de validación para Login
    const loginValidation = [
        body('email').isEmail().withMessage('Debe proporcionar un correo electrónico válido'),
        body('password').notEmpty().withMessage('La contraseña es obligatoria'),
        validate,
    ];

    // Definición de Endpoints
    router.post('/register', registerValidation, register);
    router.post('/login', loginValidation, login);

    module.exports = router;    
    ```

7. Conectar Rutas en `src/app.js`: Actualiza `src/app.js` para registrar el enrutador de autenticación:
    ```js
    // ...
    require('dotenv').config();

    const authRoutes = require('./routes/auth.routes'); // <- Línea nueva

    const app = express();
    // ...

    // Ruta de comprobación de estado (Health Check)
    app.get('/api/v1/health', (req, res) => {
        // ...
    });

    // Registrar Rutas de la API
    app.use('/api/v1/auth', authRoutes);                // <- Línea nueva

    // Inicialización del Servidor
    // ...
    ```

## 📑 Verificación de Autenticación y Control de Acceso (RBAC)
1. Probar los Endpoints con un Cliente HTTP (Postman, Bruno o Insomnia): Prueba directamente los endpoints `/register` y `/login` para confirmar la emisión correcta de tokens JWT y el hash de contraseñas:
    + 🧪 Prueba 1: Registrar un nuevo usuario
        + Método: POST
        + URL: http://localhost:4000/api/v1/auth/register
        + Body (JSON):
            ```json
            {
                "firstName": "Juan",
                "lastName": "Pérez",
                "email": "juan@example.com",
                "password": "Password123!"
            }
            ```
        + Respuesta Esperada (201 Created): Recibirás el objeto user (sin el campo password) junto con el token JWT generado.
    + 🧪 Prueba 2: Iniciar Sesión
        + Método: POST
        + URL: http://localhost:4000/api/v1/auth/login
        + Body (JSON):
            ```json
            {
                "email": "juan@example.com",
                "password": "Password123!"
            }
            ```
        + Respuesta Esperada (200 OK): Devolverá los datos del usuario, el listado de sus roles ["USER"] y un nuevo token firmado.

2. Construir los Middlewares de Seguridad (JWT & RBAC): Una vez verificados los endpoints de autenticación, debemos crear los middlewares que protegerán las rutas privadas del backend.
    + Crea el archivo `src/middlewares/auth.middleware.js`:
        ```js
        const jwt = require('jsonwebtoken');

        // 1. Verificar si la petición incluye un Token JWT válido
        const authenticateJWT = (req, res, next) => {
            const authHeader = req.headers.authorization;

            if (!authHeader || !authHeader.startsWith('Bearer ')) {
                return res.status(401).json({
                    status: 'fail',
                    message: 'Acceso no autorizado. Debe proporcionar un Token Bearer',
                });
            }

            const token = authHeader.split(' ')[1];

            try {
                const decoded = jwt.verify(token, process.env.JWT_SECRET);
                req.user = decoded; // Adjunta el usuario (id, email, roles) al objeto request
                next();
            } catch (error) {
                return res.status(403).json({
                    status: 'fail',
                    message: 'Token inválido o expirado',
                });
            }
        };

        // 2. Control de Acceso Basado en Roles (RBAC)
        const authorizeRoles = (...allowedRoles) => {
            return (req, res, next) => {
                if (!req.user || !req.user.roles) {
                    return res.status(403).json({
                        status: 'fail',
                        message: 'Acceso denegado. Sin roles asignados',
                    });
                }

                const hasRole = req.user.roles.some((role) => allowedRoles.includes(role));

                if (!hasRole) {
                    return res.status(403).json({
                        status: 'fail',
                        message: 'No tienes los permisos requeridos para ejecutar esta acción',
                    });
                }

                next();
            };
        };

        module.exports = { authenticateJWT, authorizeRoles };
        ```


## 🔐 Endpoint de Verificación de Sesión (`/me`)
Para completar la base de autenticación reutilizable (Starter Kit) y permitir que el cliente Frontend pueda consultar el perfil del usuario autenticado o verificar la validez de un token guardado al recargar la página, se agrega el endpoint `GET /api/v1/auth/me`.

1. Actualizar el Controlador de Autenticación (`src/controllers/auth.controller.js`):
    Añade la función `getMe` al archivo de controladores:
    ```js
    // 3. OBTENER USUARIO ACTUAL (VERIFICAR SESIÓN)
    const getMe = async (req, res) => {
        try {
            const user = await prisma.user.findUnique({
                where: { id: req.user.id },
                select: {
                    id: true,
                    email: true,
                    name: true,
                    createdAt: true,
                    roles: {
                        select: {
                            role: {
                                select: { name: true },
                            },
                        },
                    },
                },
            });

            if (!user) {
                return res.status(404).json({
                    status: 'fail',
                    message: 'Usuario no encontrado',
                });
            }

            const userRoles = user.roles.map((ur) => ur.role.name);

            return res.status(200).json({
                status: 'success',
                data: {
                    user: {
                        id: user.id,
                        email: user.email,
                        name: user.name,
                        roles: userRoles,
                        createdAt: user.createdAt,
                    },
                },
            });
        } catch (error) {
            console.error('Error en getMe:', error);
            return res.status(500).json({ status: 'error', message: 'Error interno del servidor' });
        }
    };

    // 4. CIERRE DE SESIÓN (LOGOUT)
    const logout = async (req, res) => {
    try {
        // En arquitecturas stateless (JWT en Authorization Header), el servidor confirma
        // el cierre de sesión para que el Frontend proceda a destruir el token almacenado.
        return res.status(200).json({
        status: 'success',
        message: 'Sesión cerrada correctamente',
        });
    } catch (error) {
        console.error('Error en logout:', error);
        return res.status(500).json({ status: 'error', message: 'Error interno del servidor' });
    }
    };

    module.exports = { register, login, getMe, logout };
    ```

2. Proteger la Ruta en `src/routes/auth.routes.js`:
    Abre `src/routes/auth.routes.js`, importa el middleware `authenticateJWT` y la función `getMe`, e integra la ruta protegida:
    ```js
    const express = require('express');
    const { body } = require('express-validator');
    const { register, login, getMe, logout } = require('../controllers/auth.controller');
    const { authenticateJWT } = require('../middlewares/auth.middleware');
    const validate = require('../middlewares/validate.middleware');

    const router = express.Router();

    // ... (validaciones de register y login) ...

    router.post('/register', registerValidation, register);
    router.post('/login', loginValidation, login);

    // Endpoint protegido para verificar estado de sesión de usuario logueado
    router.get('/me', authenticateJWT, getMe);
    router.post('/logout', authenticateJWT, logout);

    module.exports = router;
    ```

3. Verification en Cliente HTTP (Postman / Insomnia):
    + 🧪 Prueba 3: Consultar Perfil con JWT
        + Método: GET
        + URL: http://localhost:4000/api/v1/auth/me
        + Headers: 
            + `Authorization`: `Bearer <TOKEN_OBTENIDO_EN_LOGIN>`
        + Respuesta Esperada (200 OK):
            ```json
            {
                "status": "success",
                "data": {
                    "user": {
                        "id": "30c24045-757e-483e-8dc0-77f21feb4630",
                        "email": "juan@example.com",
                        "name": "Juan Pérez",
                        "roles": [
                            "USER"
                        ],
                        "createdAt": "2026-08-18T18:15:26.475Z"
                    }
                }
            }
            ```
    + 🧪 Prueba 3: Logout
        + Método: POST
        + URL: http://localhost:4000/api/v1/auth/logout
        + Headers: Authorization: Bearer <TU_TOKEN_JWT>
        + Respuesta Esperada (200 OK):
            ```json
            {
                "status": "success",
                "message": "Sesión cerrada correctamente"
            }
            ```


## 🎨 Inicialización de la Capa de Presentación (Frontend SPA)
Esta fase cubre la construcción del cliente SPA dentro de la carpeta `familytree2026-frontend` (o `starter-frontend`) utilizando Vue 3 (Composition API / `<script setup>`), Vite, Pinia, Vue Router y Axios, con Tailwind CSS v4 para los estilos.

### Paso 1: Creación del Proyecto Vue 3 con Vite
+ Antes de instalar paquetes de terceros, generamos la estructura oficial de Vue 3 dentro de la carpeta del frontend:
    ```bash
    # Ubicarse en la raíz del proyecto
    cd ~/projects/family_tree2026

    # Crear el proyecto Vue 3 (o dentro de familytree2026-frontend)
    npm create vue@latest familytree2026-frontend
    ```
    + Opciones durante la creación del CLI de Vue:
        + Project name: familytree2026-frontend
        + Add TypeScript? No
        + Add JSX Support? No
        + Add Vue Router for Single Page Application development? Yes (Opción recomendada)
        + Add Pinia for state management? Yes (Opción recomendada)
        + Add Vitest for Unit Testing? No
        + Add an End-to-End Testing Solution? No
        + Add ESLint for code quality? Yes
+ Una vez creado, navegamos al directorio e instalamos las dependencias base generadas por Vue:
    ```bash
    cd familytree2026-frontend
    npm install
    ```
    + En caso de error, ejecutar:
    ```bash
    npm install --legacy-peer-deps
    ```

### Paso 2: Instalación de Dependencias Adicionales y Tailwind CSS v4
+ Instalamos Axios para las peticiones HTTP y el plugin oficial de Tailwind CSS v4 para Vite:
    ```bash
    # Instalar cliente HTTP
    npm install axios
    # En caso de error
    npm install axios --legacy-peer-deps

    # Instalar Tailwind CSS v4 y su integración con Vite
    npm install -D tailwindcss @tailwindcss/vite
    # En caso de error
    npm install -D tailwindcss @tailwindcss/vite --legacy-peer-deps
    ```

### ⚙️ Paso 3: Configuración de Vite y Tailwind v4
1. Abre el archivo `vite.config.js` en la raíz de `familytree2026-frontend` y déjalo exactamente así:
    ```js
    import { fileURLToPath, URL } from 'node:url'
    import { defineConfig } from 'vite'
    import vue from '@vitejs/plugin-vue'
    import tailwindcss from '@tailwindcss/vite'

    export default defineConfig({
        plugins: [
            vue(),
            tailwindcss(),
        ],
        resolve: {
            alias: {
                '@': fileURLToPath(new URL('./src', import.meta.url))
            }
        }
    })
    ```
2. Abre el archivo `src/assets/main.css` (o `src/style.css`), borra todo lo que tenga dentro y deja únicamente esta línea:
    ```css
    @import "tailwindcss";
    ```
    + Si existe el archivo `src/assets/base.css`, puedes borrarlo o vaciarlo para que no interfiera con las clases de Tailwind.

### 🌐 Paso 4: Cliente HTTP Centralizado (`src/api/axios.js`)
1. Crea el archivo `.env` en la raíz de `familytree2026-frontend`:
    ```env
    # URL Base de la API REST para el Backend local
    VITE_API_BASE_URL=http://localhost:4000/api/v1
    ```
2. Crea la `carpeta src/api/` y el archivo `src/api/axios.js`:
    ```js
    import axios from 'axios';

    const api = axios.create({
        baseURL: import.meta.env.VITE_API_BASE_URL || 'http://localhost:4000/api/v1',
        headers: {
            'Content-Type': 'application/json',
        },
    });

    // Interceptor para inyectar automáticamente el Token Bearer si existe en localStorage
    api.interceptors.request.use((config) => {
        const token = localStorage.getItem('token');
        if (token) {
            config.headers.Authorization = `Bearer ${token}`;
        }
        return config;
    });

    export default api;
    ```

### 🍍 Paso 5: Store de Autenticación con Pinia (`src/stores/auth.store.js`)
+ Crea o reemplaza el archivo en `src/stores/auth.store.js`:
    ```js
    import { defineStore } from 'pinia';
    import api from '../api/axios';

    export const useAuthStore = defineStore('auth', {
        state: () => ({
            user: null,
            token: localStorage.getItem('token') || null,
            loading: false,
            error: null,
        }),

        getters: {
            isAuthenticated: (state) => !!state.token && !!state.user,
            userRoles: (state) => state.user?.roles || [],
        },

        actions: {
            // 1. Iniciar Sesión
            async login(credentials) {
                this.loading = true;
                this.error = null;
                try {
                    const response = await api.post('/auth/login', credentials);
                    // Verificación defensiva de la estructura
                    const data = response.data?.data || response.data;
                    
                    this.token = data.token;
                    this.user = data.user;
                    localStorage.setItem('token', data.token);

                    return response.data;
                } catch (err) {
                    this.error = err.response?.data?.message || 'Error al iniciar sesión';
                    throw err;
                } finally {
                    this.loading = false;
                }
            },

            // 2. Registrar Usuario
            async register(userData) {
                this.loading = true;
                this.error = null;
                try {
                    const response = await api.post('/auth/register', userData);
                    const { user, token } = response.data.data;

                    this.token = token;
                    this.user = user;
                    localStorage.setItem('token', token);

                    return response.data;
                } catch (err) {
                    this.error = err.response?.data?.message || 'Error al registrar usuario';
                    throw err;
                } finally {
                    this.loading = false;
                }
            },

            // 3. Verificar Sesión al recargar la página
            async fetchUser() {
                if (!this.token) return;

                this.loading = true;
                try {
                    const response = await api.get('/auth/me');
                    this.user = response.data.data.user;
                } catch (err) {
                    console.error('Sesión expirada o token inválido:', err);
                    this.logout();
                } finally {
                    this.loading = false;
                }
            },

            // 4. Cerrar Sesión
            async logout() {
                try {
                    if (this.token) {
                    await api.post('/auth/logout');
                    }
                } catch (err) {
                    console.warn('Error respondiendo al servidor en logout:', err);
                } finally {
                    this.user = null;
                    this.token = null;
                    localStorage.removeItem('token');
                }
            },
        },
    });
    ```

### 🚦 Paso 6: Configuración de Vue Router con Guards (`src/router/index.js`)
+ Abre o crea el archivo `src/router/index.js` y reemplaza su contenido:
    ```js
    import { createRouter, createWebHistory } from 'vue-router';
    import { useAuthStore } from '../stores/auth.store';

    // Vistas
    import HomeView from '../views/HomeView.vue';
    import LoginView from '../views/LoginView.vue';
    import RegisterView from '../views/RegisterView.vue';
    import DashboardView from '../views/DashboardView.vue';

    const router = createRouter({
        history: createWebHistory(import.meta.env.BASE_URL),
        routes: [
            {
                path: '/',
                name: 'home',
                component: HomeView,
            },
            {
                path: '/login',
                name: 'login',
                component: LoginView,
                meta: { requiresGuest: true },
            },
            {
                path: '/register',
                name: 'register',
                component: RegisterView,
                meta: { requiresGuest: true },
            },
            {
                path: '/dashboard',
                name: 'dashboard',
                component: DashboardView,
                meta: { requiresAuth: true },
            },
        ],
    });

    // Navigation Guard Global
    router.beforeEach(async (to) => {
        const authStore = useAuthStore();

        // Rehidratar sesión si hay token pero no datos de usuario en memoria
        if (authStore.token && !authStore.user) {
            await authStore.fetchUser();
        }

        const isAuthenticated = authStore.isAuthenticated;

        // 1. Ruta requiere autenticación y el usuario NO está logueado
        if (to.meta.requiresAuth && !isAuthenticated) {
            return { name: 'login' };
        }

        // 2. Ruta requiere ser invitado (Guest) y el usuario SÍ está logueado
        if (to.meta.requiresGuest && isAuthenticated) {
            return { name: 'dashboard' };
        }

        return true;
    });

    export default router;
    ```

### 🎨 Paso 7: Vistas de Autenticación y Dashboard (`src/views/`)
1. Formulario de Inicio de Sesión (`src/views/LoginView.vue`)
    + Crea el archivo `src/views/LoginView.vue`:
        ```vue
        <script setup>
            import { ref } from 'vue';
            import { useRouter } from 'vue-router';
            import { useAuthStore } from '../stores/auth.store';

            const authStore = useAuthStore();
            const router = useRouter();

            const form = ref({
                email: '',
                password: '',
            });

            const handleSubmit = async () => {
                try {
                    await authStore.login(form.value);
                    router.push({ name: 'dashboard' });
                } catch (err) {
                    console.error('Error al iniciar sesión:', err);
                }
            };
        </script>

        <template>
            <div class="min-h-screen flex items-center justify-center bg-slate-900 text-slate-100 p-4">
                <div class="w-full max-w-md bg-slate-800 rounded-2xl shadow-xl p-8 border border-slate-700">
                    <h2 class="text-2xl font-bold text-center text-emerald-400 mb-6">Iniciar Sesión</h2>

                    <div v-if="authStore.error" class="mb-4 p-3 bg-red-500/20 border border-red-500/50 rounded-lg text-red-300 text-sm">
                        {{ authStore.error }}
                    </div>

                    <form @submit.prevent="handleSubmit" class="space-y-4">
                        <div>
                            <label class="block text-sm font-medium mb-1">Correo Electrónico</label>
                            <input
                                v-model="form.email"
                                type="email"
                                required
                                class="w-full px-4 py-2 bg-slate-900 border border-slate-700 rounded-lg focus:outline-none focus:border-emerald-500 text-slate-200"
                                placeholder="correo@ejemplo.com"
                            />
                        </div>

                        <div>
                            <label class="block text-sm font-medium mb-1">Contraseña</label>
                            <input
                                v-model="form.password"
                                type="password"
                                required
                                class="w-full px-4 py-2 bg-slate-900 border border-slate-700 rounded-lg focus:outline-none focus:border-emerald-500 text-slate-200"
                                placeholder="••••••••"
                            />
                        </div>

                        <button
                            type="submit"
                            :disabled="authStore.loading"
                            class="w-full py-2.5 px-4 bg-emerald-600 hover:bg-emerald-500 font-semibold rounded-lg shadow-md transition-colors disabled:opacity-50 cursor-pointer"
                        >
                            {{ authStore.loading ? 'Cargando...' : 'Entrar' }}
                        </button>
                    </form>

                    <p class="mt-6 text-center text-sm text-slate-400">
                        ¿No tienes cuenta?
                        <router-link to="/register" class="text-emerald-400 hover:underline">Regístrate aquí</router-link>
                    </p>
                </div>
            </div>
        </template>        
        ```

2. Formulario de Registro (`src/views/RegisterView.vue`)
    + Crea el archivo `src/views/RegisterView.vue`:
        ```vue
        <script setup>
            import { ref } from 'vue';
            import { useRouter } from 'vue-router';
            import { useAuthStore } from '../stores/auth.store';

            const authStore = useAuthStore();
            const router = useRouter();

            const form = ref({
                firstName: '',
                lastName: '',
                email: '',
                password: '',
            });

            const handleSubmit = async () => {
                try {
                    await authStore.register(form.value);
                    router.push({ name: 'dashboard' });
                } catch (err) {
                    console.error('Error en registro:', err);
                }
            };
        </script>

        <template>
            <div class="min-h-screen flex items-center justify-center bg-slate-900 text-slate-100 p-4">
                <div class="w-full max-w-md bg-slate-800 rounded-2xl shadow-xl p-8 border border-slate-700">
                    <h2 class="text-2xl font-bold text-center text-emerald-400 mb-6">Crear Cuenta</h2>

                    <div v-if="authStore.error" class="mb-4 p-3 bg-red-500/20 border border-red-500/50 rounded-lg text-red-300 text-sm">
                        {{ authStore.error }}
                    </div>

                    <form @submit.prevent="handleSubmit" class="space-y-4">
                        <div class="grid grid-cols-2 gap-4">
                            <div>
                                <label class="block text-sm font-medium mb-1">Nombre</label>
                                <input
                                    v-model="form.firstName"
                                    type="text"
                                    required
                                    class="w-full px-4 py-2 bg-slate-900 border border-slate-700 rounded-lg focus:outline-none focus:border-emerald-500 text-slate-200"
                                    placeholder="Juan"
                                />
                            </div>
                            <div>
                                <label class="block text-sm font-medium mb-1">Apellido</label>
                                <input
                                    v-model="form.lastName"
                                    type="text"
                                    required
                                    class="w-full px-4 py-2 bg-slate-900 border border-slate-700 rounded-lg focus:outline-none focus:border-emerald-500 text-slate-200"
                                    placeholder="Pérez"
                                />
                            </div>
                        </div>

                        <div>
                            <label class="block text-sm font-medium mb-1">Correo Electrónico</label>
                            <input
                                v-model="form.email"
                                type="email"
                                required
                                class="w-full px-4 py-2 bg-slate-900 border border-slate-700 rounded-lg focus:outline-none focus:border-emerald-500 text-slate-200"
                                placeholder="correo@ejemplo.com"
                            />
                        </div>

                        <div>
                            <label class="block text-sm font-medium mb-1">Contraseña</label>
                            <input
                                v-model="form.password"
                                type="password"
                                required
                                class="w-full px-4 py-2 bg-slate-900 border border-slate-700 rounded-lg focus:outline-none focus:border-emerald-500 text-slate-200"
                                placeholder="Mínimo 6 caracteres"
                            />
                        </div>

                        <button
                            type="submit"
                            :disabled="authStore.loading"
                            class="w-full py-2.5 px-4 bg-emerald-600 hover:bg-emerald-500 font-semibold rounded-lg shadow-md transition-colors disabled:opacity-50 cursor-pointer"
                        >
                            {{ authStore.loading ? 'Registrando...' : 'Registrarse' }}
                        </button>
                    </form>

                    <p class="mt-6 text-center text-sm text-slate-400">
                        ¿Ya tienes cuenta?
                        <router-link to="/login" class="text-emerald-400 hover:underline">Inicia sesión</router-link>
                    </p>
                </div>
            </div>
        </template>
        ```

3. Vista Protegida del Dashboard (`src/views/DashboardView.vue`)
    + Crea el archivo `src/views/DashboardView.vue`:
        ```vue
        <script setup>
            import { useRouter } from 'vue-router';
            import { useAuthStore } from '../stores/auth.store';

            const authStore = useAuthStore();
            const router = useRouter();

            const handleLogout = async () => {
                await authStore.logout();
                router.push({ name: 'login' });
            };
        </script>

        <template>
            <div class="min-h-screen bg-slate-900 text-slate-100 flex flex-col">
                <!-- Navbar -->
                <header class="bg-slate-800 border-b border-slate-700 py-4 px-6 flex justify-between items-center">
                    <h1 class="text-xl font-bold text-emerald-400">Starter App Dashboard</h1>
                    <div class="flex items-center gap-4">
                        <span class="text-sm text-slate-300">{{ authStore.user?.name }}</span>
                        <button
                            @click="handleLogout"
                            class="px-3 py-1.5 bg-red-600/80 hover:bg-red-500 text-sm font-medium rounded-lg transition-colors cursor-pointer"
                        >
                            Cerrar Sesión
                        </button>
                    </div>
                </header>

                <!-- Main Content -->
                <main class="flex-1 p-6 max-w-4xl mx-auto w-full">
                    <div class="bg-slate-800 border border-slate-700 rounded-2xl p-6 shadow-lg">
                        <h2 class="text-lg font-semibold text-emerald-400 mb-4">Perfil de Usuario Autenticado</h2>
                        
                        <div class="space-y-3 text-slate-300">
                            <p><strong class="text-slate-100">ID:</strong> {{ authStore.user?.id }}</p>
                            <p><strong class="text-slate-100">Nombre:</strong> {{ authStore.user?.name }}</p>
                            <p><strong class="text-slate-100">Correo:</strong> {{ authStore.user?.email }}</p>
                            <p>
                                <strong class="text-slate-100">Roles:</strong>
                                    <span
                                        v-for="role in authStore.userRoles"
                                        :key="role"
                                        class="ml-2 inline-block px-2 py-0.5 bg-emerald-500/20 border border-emerald-500/40 text-emerald-300 text-xs font-semibold rounded"
                                    >
                                        {{ role }}
                                </span>
                            </p>
                        </div>
                    </div>
                </main>
            </div>
        </template>
        ```

4. Limpiar `src/App.vue`:
    + Abre `src/App.vue` y reemplaza todo su contenido con esto:
        ```vue
        <script setup>
            import { RouterView } from 'vue-router'
        </script>

        <template>
            <RouterView />
        </template>
        ```

5. Rediseñar la Landing Page (`src/views/HomeView.vue`)
    + Reemplaza el contenido de `src/views/HomeView.vue` para que la raíz / muestre una bienvenida profesional:
        ```vue
        <script setup>
            import { useAuthStore } from '../stores/auth.store';

            const authStore = useAuthStore();
        </script>

        <template>
            <div class="min-h-screen bg-slate-900 text-slate-100 flex flex-col justify-between">
                <!-- Navbar simple -->
                <header class="py-6 px-8 flex justify-between items-center border-b border-slate-800">
                    <div class="flex items-center gap-2">
                        <span class="text-2xl">🌳</span>
                        <span class="font-bold text-xl text-emerald-400">FamilyTree 2026</span>
                    </div>
                    <div>
                        <router-link
                            v-if="authStore.isAuthenticated"
                            to="/dashboard"
                            class="px-4 py-2 bg-emerald-600 hover:bg-emerald-500 rounded-lg text-sm font-semibold transition-colors"
                        >
                            Ir al Dashboard
                        </router-link>
                        <div v-else class="flex gap-4">
                            <router-link to="/login" class="px-4 py-2 text-slate-300 hover:text-white text-sm font-medium">
                                Iniciar Sesión
                            </router-link>
                            <router-link to="/register" class="px-4 py-2 bg-emerald-600 hover:bg-emerald-500 rounded-lg text-sm font-semibold transition-colors">
                                Registrarse
                            </router-link>
                        </div>
                    </div>
                </header>

                <!-- Hero Section -->
                <main class="flex-1 flex flex-col items-center justify-center text-center px-4 max-w-3xl mx-auto">
                    <span class="px-3 py-1 bg-emerald-500/10 border border-emerald-500/30 text-emerald-400 text-xs font-semibold rounded-full mb-6">
                        Starter Kit 2026
                    </span>
                    <h1 class="text-4xl sm:text-6xl font-extrabold tracking-tight mb-6">
                        Gestiona la historia de tu familia de forma <span class="text-emerald-400">segura y moderna</span>.
                    </h1>
                    <p class="text-slate-400 text-lg mb-8 max-w-xl">
                        Plataforma construida sobre Node.js, Express, PostgreSQL y Vue 3 con autenticación basada en tokens JWT.
                    </p>
                    <div class="flex gap-4">
                        <router-link
                            :to="authStore.isAuthenticated ? '/dashboard' : '/register'"
                            class="px-6 py-3 bg-emerald-600 hover:bg-emerald-500 font-semibold rounded-xl shadow-lg transition-colors"
                        >
                            {{ authStore.isAuthenticated ? 'Ir a mi Panel' : 'Comenzar Ahora' }}
                        </router-link>
                    </div>
                </main>

                <!-- Footer -->
                <footer class="py-6 text-center text-slate-500 text-sm border-t border-slate-800">
                    &copy; 2026 FamilyTree. Todos los derechos reservados.
                </footer>
            </div>
        </template>
        ```

6. Crear Vista 404 (`src/views/NotFoundView.vue`)
    + Crea el archivo `src/views/NotFoundView.vue`:
```vue
<template>
    <div class="min-h-screen bg-slate-900 text-slate-100 flex flex-col items-center justify-center p-4 text-center">
        <h1 class="text-8xl font-black text-emerald-500 mb-2">404</h1>
        <h2 class="text-2xl font-bold mb-4">Página no encontrada</h2>
        <p class="text-slate-400 mb-6 max-w-md">
            La ruta a la que intentas acceder no existe o ha sido movida a otro lugar.
        </p>
        <router-link
            to="/"
            class="px-5 py-2.5 bg-slate-800 hover:bg-slate-700 border border-slate-700 rounded-lg text-emerald-400 font-medium transition-colors"
        >
            Volver al Inicio
        </router-link>
    </div>
</template>
```

7. Actualizar las rutas en `src/router/index.js`
    + Añade la ruta comodín al final del arreglo routes en `src/router/index.js`:
```js
import NotFoundView from '../views/NotFoundView.vue';

// En la lista de rutas:
routes: [
    { path: '/', name: 'home', component: HomeView },
    { path: '/login', name: 'login', component: LoginView, meta: { requiresGuest: true } },
    { path: '/register', name: 'register', component: RegisterView, meta: { requiresGuest: true } },
    { path: '/dashboard', name: 'dashboard', component: DashboardView, meta: { requiresAuth: true } },
    // Comodín para capturar cualquier ruta inexistente
    { path: '/:pathMatch(.*)*', name: 'not-found', component: NotFoundView },
]
```

## 🔒 Estrategia de Roles y Control de Acceso (RBAC)

### ⚙️ Paso 1: Ajuste en Backend — Usuario Sin Rol Por Defecto
+ Primero, garantizamos que el registro asigna una lista vacía de roles.
+ Abre `familytree2026-backend/src/controllers/auth.controller.js` y verifica que en el método register no estés asignando ningún rol por defecto (o asegúrate de pasarlo vacío []):
    ```js
    const bcrypt = require('bcryptjs');
    const jwt = require('jsonwebtoken');
    const prisma = require('../config/prisma');

    const generateToken = (user, roles = []) => {
        return jwt.sign(
            {
                id: user.id,
                email: user.email,
                roles: roles,
            },
            process.env.JWT_SECRET,
            { expiresIn: process.env.JWT_EXPIRES_IN || '24h' }
        );
    };

    // 1. REGISTRO DE USUARIO (Sin roles por defecto)
    const register = async (req, res) => {
        try {
            const { email, password, firstName, lastName } = req.body;
            const fullName = `${firstName} ${lastName}`;

            const existingUser = await prisma.user.findUnique({ where: { email } });
            if (existingUser) {
                return res.status(400).json({
                    status: 'fail',
                    message: 'El correo electrónico ya está registrado',
                });
            }

            const salt = await bcrypt.genSalt(10);
            const passwordHash = await bcrypt.hash(password, salt);

            // Se crea el usuario SIN incluir la relación de roles
            const newUser = await prisma.user.create({
                data: {
                    email,
                    password: passwordHash,
                    name: fullName,
                },
                select: {
                    id: true,
                    email: true,
                    name: true,
                    createdAt: true,
                },
            });

            // Se genera token con un arreglo de roles vacío []
            const token = generateToken(newUser, []);

            return res.status(201).json({
                status: 'success',
                message: 'Usuario registrado correctamente (sin permisos asignados)',
                data: {
                    user: {
                        ...newUser,
                        roles: [],
                    },
                    token,
                },
            });
        } catch (error) {
            console.error('Error en registro:', error);
            return res.status(500).json({ status: 'error', message: 'Error interno del servidor' });
        }
    };
    ```

### 🌱 Paso 2: Crear Seeder del SUPER_ADMIN (`src/seeders/superadmin.seeder.js`)
Crearemos un script reutilizable e independiente que inserta las tablas iniciales de roles y crea la cuenta del Administrador Principal si no existe.
1. Crea la carpeta `src/seeders/` en `familytree2026-backend`.
2. Crea el archivo `src/seeders/superadmin.seeder.js`:
    ```js
    const bcrypt = require('bcryptjs');
    const prisma = require('../config/prisma');
    require('dotenv').config();

    const seedSuperAdmin = async () => {
        try {
            console.log('🌱 Iniciando Seeder de SuperAdmin...');

            const adminEmail = process.env.SUPER_ADMIN_EMAIL || 'admin@familytree.com';
            const adminPassword = process.env.SUPER_ADMIN_PASSWORD;

            if (!adminPassword) {
                throw new Error('❌ Error: Debes definir SUPER_ADMIN_PASSWORD en tu archivo .env');
            }

            // 1. Asegurar los 3 roles base en la BD
            const roles = [
                { name: 'SUPER_ADMIN', description: 'Acceso total y gestión del sistema' },
                { name: 'ADMIN', description: 'Administrador de contenido y usuarios' },
                { name: 'USER', description: 'Usuario estándar' },
            ];

            for (const r of roles) {
                await prisma.role.upsert({
                    where: { name: r.name },
                    update: {},
                    create: r,
                });
            }

            // 2. Obtener el ID del rol SUPER_ADMIN
            const superAdminRole = await prisma.role.findUnique({
                where: { name: 'SUPER_ADMIN' },
            });

            // 3. Crear o actualizar el Usuario SUPER_ADMIN
            const salt = await bcrypt.genSalt(10);
            const hashedPassword = await bcrypt.hash(adminPassword, salt);

            const adminUser = await prisma.user.upsert({
                where: { email: adminEmail },
                update: {},
                create: {
                    name: 'Super Admin',
                    email: adminEmail,
                    password: hashedPassword,
                    roles: {
                        create: {
                            roleId: superAdminRole.id,
                        },
                    },
                },
            });

            console.log('✅ Seeder ejecutado con éxito.');
            console.log(`👤 SuperAdmin verificado: ${adminUser.email}`);
        } catch (error) {
            console.error('❌ Error ejecutando el Seeder:', error.message);
        } finally {
            await prisma.$disconnect();
        }
    };

    seedSuperAdmin();
    ```
3. Configuración del Archivo `.env` Local
    + Edita el archivo `.env` en la raíz de `familytree2026-backend` y agrega las siguiente variables de entorno:
        ```env
        SUPER_ADMIN_EMAIL = admin@familytree.com
        SUPER_ADMIN_PASSWORD = 12345678
        ```
4. Agrega el comando para correr el seeder en el `package.json` de tu Backend:
    ```json
    "scripts": {
        "dev": "nodemon src/app.js",
        "seed": "node src/seeders/superadmin.seeder.js"
    }
    ```
5. Ejecuta en la terminal del backend:
    ```bas
    node src/seeders/superadmin.seeder.js
    ```

## Controlador para administración de usuarios (Backend)
1. Seeder de Usuarios Falsos (Estilo Laravel)
    + Instala la librería de generación de datos falsos en el backend:
        ```bash
        npm install -D @faker-js/faker
        ```
2. Crea el archivo `src/seeders/users.seeder.js`:
    ```js
    const { fakerES: faker } = require('@faker-js/faker');
    const bcrypt = require('bcryptjs');
    const prisma = require('../config/prisma');

    const seedUsers = async (quantity = 25) => {
        try {
            console.log(`🌱 Generando ${quantity} usuarios falsos...`);

            const defaultPassword = await bcrypt.hash('Password123!', 10);
            const usersData = [];

            for (let i = 0; i < quantity; i++) {
                const firstName = faker.person.firstName();
                const lastName = faker.person.lastName();
                
                usersData.push({
                    name: `${firstName} ${lastName}`,
                    email: faker.internet.email({ firstName, lastName }).toLowerCase(),
                    password: defaultPassword,
                });
            }

            // Insertar masivamente
            await prisma.user.createMany({
                data: usersData,
                skipDuplicates: true,
            });

            console.log(`✅ ${quantity} usuarios creados exitosamente.`);
        } catch (error) {
            console.error('❌ Error seeding usuarios:', error.message);
        } finally {
            await prisma.$disconnect();
        }
    };

    seedUsers();
    ```
3. Ejecútalo con: 
    ```bash
    node src/seeders/users.seeder.js
    ```

4. Backend: Middleware de Protección RBAC y Controlador Admin
    + Middleware de Verificación de Roles (`src/middlewares/role.middleware.js`): Este middleware valida que el usuario logueado tenga alguno de los roles requeridos:
        ```js
        const requireRoles = (...allowedRoles) => {
            return (req, res, next) => {
                if (!req.user || !req.user.roles) {
                    return res.status(403).json({
                        status: 'fail',
                        message: 'Acceso denegado: Usuario sin información de roles',
                    });
                }

                const hasRole = req.user.roles.some((role) => allowedRoles.includes(role));

                if (!hasRole) {
                    return res.status(403).json({
                        status: 'fail',
                        message: 'No tienes los permisos necesarios para realizar esta acción',
                    });
                }

                next();
            };
        };

        module.exports = { requireRoles };
        ```
5. Controlador de Administración de Usuarios (`src/controllers/admin.controller.js`): Incluye búsqueda parcial por coincidencia (search), paginación y actualización de roles:
    ```js
    const prisma = require('../config/prisma');

    // Listar usuarios con búsqueda y paginación
    const getUsers = async (req, res) => {
        try {
            const { search = '', page = 1, limit = 10 } = req.query;
            const skip = (parseInt(page) - 1) * parseInt(limit);

            const where = search
            ? {
                OR: [
                    { name: { contains: search, mode: 'insensitive' } },
                    { email: { contains: search, mode: 'insensitive' } },
                ],
            }
            : {};

            const [total, users] = await prisma.$transaction([
                prisma.user.count({ where }),
                prisma.user.findMany({
                    where,
                    skip,
                    take: parseInt(limit),
                    orderBy: { createdAt: 'desc' },
                    select: {
                        id: true,
                        name: true,
                        email: true,
                        isActive: true,
                        createdAt: true,
                        roles: {
                            select: {
                                role: { select: { name: true } },
                            },
                        },
                    },
                }),
            ]);

            // Formatear la estructura de respuesta de roles
            const formattedUsers = users.map((u) => ({
                ...u,
                roles: u.roles.map((r) => r.role.name),
            }));

            return res.status(200).json({
                status: 'success',
                data: {
                    users: formattedUsers,
                    pagination: {
                        total,
                        page: parseInt(page),
                        totalPages: Math.ceil(total / parseInt(limit)),
                    },
                },
            });
        } catch (error) {
            console.error('Error al obtener usuarios:', error);
            return res.status(500).json({ status: 'error', message: 'Error interno del servidor' });
        }
    };

    // Asignar / Cambiar Roles de un usuario
    const updateUserRoles = async (req, res) => {
        try {
            const { id } = req.params;
            const { roles } = req.body; // Ejemplo: ["ADMIN", "USER"] o []

            // 1. Eliminar asignaciones de roles actuales
            await prisma.userRole.deleteMany({ where: { userId: id } });

            // 2. Obtener IDs de los nuevos roles solicitados
            if (roles && roles.length > 0) {
                const dbRoles = await prisma.role.findMany({
                    where: { name: { in: roles } },
                });

                // 3. Crear nuevas relaciones
                const userRolesData = dbRoles.map((role) => ({
                    userId: id,
                    roleId: role.id,
                }));

                await prisma.userRole.createMany({ data: userRolesData });
            }

            return res.status(200).json({
                status: 'success',
                message: 'Roles actualizados correctamente',
            });
        } catch (error) {
            console.error('Error al actualizar roles:', error);
            return res.status(500).json({ status: 'error', message: 'Error interno del servidor' });
        }
    };

    // Actualizar información básica del usuario (Nombre y Email)
    const updateUser = async (req, res) => {
        try {
            const { id } = req.params;
            const { name, email } = req.body;

            // Validar que el usuario exista
            const existingUser = await prisma.user.findUnique({ where: { id } });
            if (!existingUser) {
                return res.status(404).json({
                    status: 'fail',
                    message: 'Usuario no encontrado',
                });
            }

            // Si se intenta cambiar el email, verificar que no esté registrado por otro usuario
            if (email && email !== existingUser.email) {
                const emailTaken = await prisma.user.findUnique({ where: { email } });
                if (emailTaken) {
                    return res.status(400).json({
                        status: 'fail',
                        message: 'El correo electrónico ya está en uso por otro usuario',
                    });
                }
            }

            // Actualizar solo nombre e email (los campos omitidos se mantienen intactos)
            const updatedUser = await prisma.user.update({
                where: { id },
                data: {
                    name: name || existingUser.name,
                    email: email || existingUser.email,
                },
                select: {
                    id: true,
                    name: true,
                    email: true,
                    isActive: true,
                    createdAt: true,
                    updatedAt: true,
                    roles: {
                        select: {
                            role: { select: { name: true } },
                        },
                    },
                },
            });

            // Formatear salida de roles
            const formattedUser = {
                ...updatedUser,
                roles: updatedUser.roles.map((r) => r.role.name),
            };

            return res.status(200).json({
                status: 'success',
                message: 'Perfil de usuario actualizado correctamente',
                data: { user: formattedUser },
            });
        } catch (error) {
            console.error('Error al actualizar usuario:', error);
            return res.status(500).json({ status: 'error', message: 'Error interno del servidor' });
        }
    };

    module.exports = { getUsers, updateUserRoles, updateUser };
    ```
6. Definir Rutas en Express (`src/routes/admin.routes.js`)
    ```js
    const express = require('express');
    const router = express.Router();

    // Importar los middlewares exportados desde auth.middleware.js
    const { authenticateJWT, authorizeRoles } = require('../middlewares/auth.middleware');

    // Importar controladores de administración
    const { getUsers, updateUserRoles, updateUser } = require('../controllers/admin.controller');

    // Proteger todas las rutas de este router
    router.use(authenticateJWT);
    router.use(authorizeRoles('SUPER_ADMIN'));

    // Rutas
    router.get('/users', getUsers);
    router.put('/users/:id', updateUser);
    router.put('/users/:id/roles', updateUserRoles);

    module.exports = router;
    ```

## CRUD usuarios Frontend
1. 📡 Crear el servicio de API (`src/services/admin.service.js`)
    + Crea este archivo para encapsular las peticiones HTTP de administración:
        ```js
        import api from '@/api/axios';

        export const adminService = {
            async getUsers(params = {}) {
                const response = await api.get('/admin/users', { params });
                return response.data;
            },

            // Actualizar datos del perfil (nombre y correo)
            async updateUser(userId, userData) {
                const response = await api.put(`/admin/users/${userId}`, userData);
                return response.data;
            },

            async updateUserRoles(userId, roles) {
                const response = await api.put(`/admin/users/${userId}/roles`, { roles });
                return response.data;
            },
        };  
        ```
2. 🎨 Crear la Vista UsersAdminView.vue (`src/views/admin/UsersAdminView.vue`)
+ Crea la carpeta src/views/admin/ si no existe y añade la vista:
```vue
<template>
    <!-- 
    <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-8"> -->
    <div class="min-h-screen bg-slate-900 max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-8">
        <!-- Encabezado -->
        <div class="md:flex md:items-center md:justify-between mb-8">
            <div class="flex-1 min-w-0">
                <h1 class="text-2xl font-bold text-slate-900 dark:text-white">Gestión de Usuarios</h1>
                <p class="mt-1 text-sm text-slate-500 dark:text-slate-400">Administra los permisos y accesos de la plataforma en tiempo real.</p>                
            </div>
        </div>

        <!-- Barra de Búsqueda y Filtros -->
        <div class="bg-white dark:bg-slate-800 border border-slate-200 dark:border-slate-700 shadow-sm rounded-2xl p-4 mb-6">
            <div class="relative">
                <input
                    v-model="searchQuery"
                    @input="handleSearch"
                    type="text"
                    placeholder="Buscar por nombre o correo electrónico..."
                    class="w-full bg-slate-50 dark:bg-slate-900/50 text-slate-900 dark:text-white border border-slate-200 dark:border-slate-600 rounded-lg px-10 py-2.5 placeholder-slate-400 focus:outline-none focus:ring-2 focus:ring-emerald-500 transition-shadow"
                />
                <svg
                    class="w-5 h-5 text-slate-400 absolute left-3 top-3"
                    fill="none"
                    stroke="currentColor"
                    viewBox="0 0 24 24"
                >
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z" />
                </svg>
            </div>
        </div>

        <!-- Tabla de Usuarios -->
        <div class="bg-white dark:bg-slate-800 border border-slate-200 dark:border-slate-700 rounded-2xl shadow-sm overflow-hidden">
            <div v-if="loading" class="p-12 text-center text-slate-500 dark:text-slate-400">
                <span class="animate-spin inline-block w-8 h-8 border-4 border-emerald-500 border-t-transparent rounded-full mb-2"></span>
                <p>Cargando usuarios...</p>
            </div>

            <div v-else-if="users.length === 0" class="p-12 text-center text-slate-500 dark:text-slate-400">
                No se encontraron usuarios que coincidan con la búsqueda.
            </div>

            <table v-else class="min-w-full divide-y divide-slate-200 dark:divide-slate-700">
                <thead class="bg-slate-50 dark:bg-slate-900/50">
                    <tr>
                        <th class="px-6 py-4 text-left text-xs font-semibold text-slate-500 dark:text-slate-400 uppercase tracking-wider">Usuario</th>
                        <th class="px-6 py-4 text-left text-xs font-semibold text-slate-500 dark:text-slate-400 uppercase tracking-wider">Roles Asignados</th>
                        <th class="px-6 py-4 text-left text-xs font-semibold text-slate-500 dark:text-slate-400 uppercase tracking-wider">Fecha Registro</th>
                        <th class="px-6 py-4 text-right text-xs font-semibold text-slate-500 dark:text-slate-400 uppercase tracking-wider">Acciones</th>
                    </tr>
                </thead>
                <tbody class="divide-y divide-slate-200 dark:divide-slate-700">
                    <tr v-for="user in users" :key="user.id" class="hover:bg-slate-50 dark:hover:bg-slate-700/30 transition-colors">
                        <!-- Info Usuario -->
                        <td class="px-6 py-4 whitespace-nowrap">
                            <div class="flex items-center">
                                <div class="w-10 h-10 rounded-full bg-emerald-100 dark:bg-slate-700 flex items-center justify-center font-bold text-emerald-600 dark:text-emerald-400 uppercase border border-emerald-200 dark:border-slate-600">
                                    {{ user.name ? user.name.charAt(0) : 'U' }}
                                </div>
                                <div class="ml-4">
                                    <div class="text-sm font-medium text-slate-900 dark:text-slate-200">{{ user.name }}</div>
                                    <div class="text-sm text-slate-500 dark:text-slate-400">{{ user.email }}</div>
                                </div>
                            </div>
                        </td>

                        <!-- Badges de Roles -->
                        <td class="px-6 py-4 whitespace-nowrap">
                            <div class="flex flex-wrap gap-1.5">
                                <span
                                    v-for="role in user.roles"
                                    :key="role"
                                    :class="getRoleBadgeClass(role)"
                                    class="px-2.5 py-0.5 rounded-full text-xs font-semibold border"
                                >
                                    {{ role }}
                                </span>
                                <span v-if="user.roles.length === 0" class="px-2.5 py-0.5 rounded-full text-xs font-medium bg-slate-100 dark:bg-slate-700 text-slate-500 dark:text-slate-400 border border-slate-200 dark:border-slate-600">
                                    Sin permisos (Guest)
                                </span>
                            </div>
                        </td>

                        <!-- Fecha -->
                        <td class="px-6 py-4 whitespace-nowrap text-sm text-slate-500 dark:text-slate-400">
                            {{ formatDate(user.createdAt) }}
                        </td>

                        <!-- Acciones -->
                        <td class="px-6 py-4 whitespace-nowrap text-right text-sm font-medium">
                            <button
                                @click="openRoleModal(user)"
                                class="px-3 py-1.5 bg-emerald-50 dark:bg-emerald-500/10 text-emerald-600 dark:text-emerald-400 border border-emerald-200 dark:border-emerald-500/30 hover:bg-emerald-600 hover:text-white dark:hover:bg-emerald-500 dark:hover:text-white rounded-lg transition-all"
                            >
                                Editar Roles
                            </button>
                        </td>
                    </tr>
                </tbody>
            </table>

            <!-- Paginación -->
            <div v-if="pagination.totalPages > 1" class="px-6 py-4 bg-slate-50 dark:bg-slate-900/40 border-t border-slate-200 dark:border-slate-700 flex items-center justify-between">
                <span class="text-sm text-slate-500 dark:text-slate-400">
                    Página {{ pagination.page }} de {{ pagination.totalPages }}
                </span>
                <div class="flex gap-2">
                    <button
                        :disabled="pagination.page === 1"
                        @click="changePage(pagination.page - 1)"
                        class="px-3 py-1 bg-white dark:bg-slate-800 border border-slate-300 dark:border-slate-600 hover:bg-slate-50 dark:hover:bg-slate-700 text-slate-700 dark:text-slate-300 rounded-md disabled:opacity-50 disabled:cursor-not-allowed transition-colors"
                    >
                        Anterior
                    </button>
                    <button
                        :disabled="pagination.page === pagination.totalPages"
                        @click="changePage(pagination.page + 1)"
                        class="px-3 py-1 bg-white dark:bg-slate-800 border border-slate-300 dark:border-slate-600 hover:bg-slate-50 dark:hover:bg-slate-700 text-slate-700 dark:text-slate-300 rounded-md disabled:opacity-50 disabled:cursor-not-allowed transition-colors"
                    >
                        Siguiente
                    </button>
                </div>
            </div>
        </div>

        <!-- Modal de Asignación de Roles -->
        <div v-if="selectedUser" class="fixed inset-0 z-50 flex items-center justify-center bg-slate-900/50 dark:bg-black/70 backdrop-blur-sm">
            <div class="bg-white dark:bg-slate-800 border border-slate-200 dark:border-slate-700 rounded-2xl w-full max-w-md p-6 shadow-2xl">
                <h3 class="text-xl font-bold text-slate-900 dark:text-slate-100 mb-1">Gestionar Roles</h3>
                <p class="text-sm text-slate-500 dark:text-slate-400 mb-4">
                    Modificando permisos para <span class="text-emerald-600 dark:text-emerald-400 font-semibold">{{ selectedUser.name }}</span>
                </p>

                <div class="space-y-3 mb-6">
                    <label v-for="role in availableRoles" :key="role" class="flex items-center space-x-3 p-3 rounded-xl bg-slate-50 dark:bg-slate-900/50 border border-slate-200 dark:border-slate-700 cursor-pointer hover:border-emerald-300 dark:hover:border-slate-500 transition-colors">
                        <input
                            type="checkbox"
                            :value="role"
                            v-model="modalRoles"
                            class="w-4 h-4 text-emerald-600 bg-white dark:bg-slate-800 border-slate-300 dark:border-slate-600 rounded focus:ring-emerald-500"
                        />
                        <span class="text-sm font-medium text-slate-700 dark:text-slate-200">{{ role }}</span>
                    </label>
                </div>

                <div class="flex justify-end gap-3">
                    <button
                        @click="selectedUser = null"
                        class="px-4 py-2 bg-slate-100 dark:bg-slate-700 hover:bg-slate-200 dark:hover:bg-slate-600 text-slate-700 dark:text-slate-200 font-medium rounded-xl transition-colors"
                    >
                        Cancelar
                    </button>
                    <button
                        @click="saveUserRoles"
                        :disabled="saving"
                        class="px-4 py-2 bg-emerald-600 hover:bg-emerald-500 text-white font-medium rounded-xl disabled:opacity-50 transition-colors"
                    >
                        {{ saving ? 'Guardando...' : 'Guardar Cambios' }}
                    </button>
                </div>
            </div>
        </div>
    </div>
</template>

<script setup>
    import { ref, onMounted } from 'vue';
    import { adminService } from '../../services/admin.service';

    // --- ESTADOS GENERALES Y TABLA ---
    const users = ref([]);
    const loading = ref(true);
    const saving = ref(false);
    const searchQuery = ref('');
    const pagination = ref({ page: 1, totalPages: 1, total: 0 });
    let searchTimeout = null;

    // --- ESTADOS PARA EDICIÓN DE ROLES ---
    const selectedUser = ref(null);
    const modalRoles = ref([]);
    const availableRoles = ['SUPER_ADMIN', 'ADMIN', 'USER'];

    // --- ESTADOS PARA EDICIÓN DE DATOS PERSONALES (NUEVO) ---
    const editingUser = ref(null);
    const userForm = ref({ name: '', email: '' });

    // --- LÓGICA DE CARGA Y BÚSQUEDA ---
    const fetchUsers = async (page = 1) => {
        loading.value = true;
        try {
            const res = await adminService.getUsers({
                search: searchQuery.value,
                page,
                limit: 10,
            });
            users.value = res.data.users;
            pagination.value = res.data.pagination;
        } catch (err) {
            console.error('Error al cargar usuarios:', err);
        } finally {
            loading.value = false;
        }
    };

    const handleSearch = () => {
        clearTimeout(searchTimeout);
        searchTimeout = setTimeout(() => {
            fetchUsers(1);
        }, 300);
    };

    const changePage = (newPage) => {
        fetchUsers(newPage);
    };

    // --- LÓGICA DE ROLES ---
    const openRoleModal = (user) => {
        selectedUser.value = user;
        modalRoles.value = [...user.roles];
    };

    const saveUserRoles = async () => {
        if (!selectedUser.value) return;
        saving.value = true;
        try {
            await adminService.updateUserRoles(selectedUser.value.id, modalRoles.value);
            selectedUser.value.roles = [...modalRoles.value];
            selectedUser.value = null;
        } catch (err) {
            alert('Error al guardar los roles');
        } finally {
            saving.value = false;
        }
    };

    // --- LÓGICA DE DATOS PERSONALES (NUEVO) ---
    const openEditModal = (user) => {
        editingUser.value = user;
        userForm.value = { name: user.name, email: user.email };
    };

    const saveUserData = async () => {
        if (!editingUser.value) return;
        saving.value = true;
        try {
            const res = await adminService.updateUser(editingUser.value.id, userForm.value);
            
            // Actualiza de inmediato la fila en la reactividad local de Vue
            editingUser.value.name = res.data.user.name;
            editingUser.value.email = res.data.user.email;
            
            editingUser.value = null;
        } catch (err) {
            alert(err.response?.data?.message || 'Error al actualizar el usuario');
        } finally {
            saving.value = false;
        }
    };

    // --- UTILITIES DE FORMATO Y ESTILOS ---
    const getRoleBadgeClass = (role) => {
        switch (role) {
            case 'SUPER_ADMIN':
                return 'bg-purple-900/40 text-purple-300 border-purple-500/30';
            case 'ADMIN':
                return 'bg-blue-900/40 text-blue-300 border-blue-500/30';
            default:
                return 'bg-emerald-900/40 text-emerald-300 border-emerald-500/30';
        }
    };

    const formatDate = (dateStr) => {
        return new Date(dateStr).toLocaleDateString('es-ES', {
            day: '2-digit',
            month: 'short',
            year: 'numeric',
        });
    };

    onMounted(() => {
        fetchUsers();
    });   
</script>
```
3. 🛣️ Registrar la Ruta y Guard de Navegación (`src/router/index.js`)
    + Añade la ruta en tu router asegurándote de restringir el acceso solo a usuarios con rol SUPER_ADMIN:    
        ```js
        // En las rutas de src/router/index.js
        {
            path: '/admin/users',
            name: 'admin-users',
            component: () => import('../views/admin/UsersAdminView.vue'),
            meta: { requiresAuth: true, requiresRole: 'SUPER_ADMIN' },
        }
        ```
    + Y actualiza el beforeEach para validar el meta requiresRole:
        ```js
        router.beforeEach(async (to) => {
            const authStore = useAuthStore();

            if (authStore.token && !authStore.user) {
                await authStore.fetchUser();
            }

            const isAuthenticated = authStore.isAuthenticated;

            if (to.meta.requiresAuth && !isAuthenticated) {
                return { name: 'login' };
            }

            if (to.meta.requiresGuest && isAuthenticated) {
                return { name: 'dashboard' };
            }

            // Validación de Rol para rutas de administración
            if (to.meta.requiresRole) {
                const userRoles = authStore.user?.roles || [];
                if (!userRoles.includes(to.meta.requiresRole)) {
                    return { name: 'dashboard' }; // Redirige al dashboard si no es SuperAdmin
                }
            }

            return true;
        });
        ```

## Habilitar CORS Dinámico 
### En el backend (`src/app.js`)
1. Modificar `src/app.js`:
    ```js
    const allowedOrigins = [
        process.env.FRONTEND_URL_PROD,
        process.env.FRONTEND_URL_LOCAL_VITE,
        process.env.FRONTEND_URL_LOCAL_VUE_CLI,
    ].filter(Boolean);

    app.use(cors({
        origin: (origin, callback) => {
            if (!origin || allowedOrigins.includes(origin)) {
                callback(null, true);
            } else {
                callback(new Error('No permitido por CORS'));
            }
        },
        credentials: true
    }));
    ```
2. Agregar las siguientes variables de entorno en `.env`:
    ```env
    # ==========================================
    # CONFIGURACIÓN DEL FRONTEND
    # ==========================================
    FRONTEND_URL_PROD=https://familytree2026-frontend.vercel.app
    FRONTEND_URL_LOCAL_VITE=http://localhost:5173
    FRONTEND_URL_LOCAL_VUE_CLI=http://localhost:8080
    ```

### Consumo dinámico de la API en el Frontend (`src/api/axios.js`)
+ Modificar `src/api/axios.js`:
    ```js
    import axios from 'axios';

    const api = axios.create({
        baseURL: import.meta.env.VITE_API_BASE_URL || 'http://localhost:4000/api/v1',
        headers: { 'Content-Type': 'application/json' },
    });

    api.interceptors.request.use((config) => {
        const token = localStorage.getItem('token');
        if (token) config.headers.Authorization = `Bearer ${token}`;
        return config;
    });

    export default api;
    ```


## -------------------------

## Guía de Despliegue en Producción (CI/CD $0 USD)

### Persistencia de Datos (Supabase PostgreSQL)
1. Crear un nuevo proyecto en Supabase.
2. Ir a `Project Settings` > `Database` y copiar la cadena de conexión URI (modo Transaction o Session).
3. Aplicar las migraciones desde tu entorno local hacia la base de datos de producción:
    ```bash
    DATABASE_URL="postgres://<USER>.<PROJECT_REF>:<ENCODED_PASSWORD>@<POOLER_HOST>:<PORT>/<DATABASE_NAME>" npx prisma migrate deploy
    ```
    + Estructura de variables para la documentación:
        + `<USER>`: Usuario por defecto de la base de datos (habitualmente postgres).
        + `<PROJECT_REF>`: Identificador único o Reference ID de tu proyecto en Supabase (ej. twnivqutsljjpwutfwgs).
        + `<ENCODED_PASSWORD>`: Contraseña de la base de datos con caracteres especiales codificados en formato URL (ejemplo: = se convierte en %3D, # en %23).
        + `<POOLER_HOST>`: Host del Connection Pooler asignado a tu región en Supabase (ej. aws-0-eu-west-2.pooler.supabase.com).
        + `<PORT>`: Puerto de conexión (5432 para modo Session o 6543 para modo Transaction con ?pgbouncer=true).
        + `<DATABASE_NAME>`: Nombre de la base de datos lógica (por defecto postgres).

### API Backend (Render Web Service)
1. Creación de Cuenta y Vinculación con GitHub:
    + Accede a [render.com](https://render.com/) y haz clic en Get Started.
    + Selecciona Sign Up with GitHub para autorizar el acceso a tus repositorios.
2. Creación del Web Service:
    + En el Dashboard de Render, haz clic en New + y selecciona Web Service.
    + Elige tu repositorio del backend (familytree2026-backend).
    + Completa los campos de configuración:
        + Name: familytree2026-backend
        + Region: Frankfurt (EU Central) o la más cercana a tu base de datos.
        + Branch: main
        + Runtime: Node
        + Build Command: npm install && npx prisma generate
        + Start Command: npm start (o node server.js / node index.js, dependiendo de cómo arranques tu servidor en el package.json)
        + Instance Type: Free ($0/mo)
    + Configuración de Variables de Entorno: Desplázate hasta la sección Environment Variables y añade:
        + DATABASE_URL: postgres://<USER>.<PROJECT_REF>:<ENCODED_PASSWORD>@<POOLER_HOST>:<PORT>/<DATABASE_NAME>
        + JWT_SECRET: tu_clave_secreta_super_segura
        + PORT: 10000
        + FRONTEND_URL_PROD: https://familytree2026-frontend.vercel.app
        + FRONTEND_URL_LOCAL_VITE: http://localhost:5173
        + FRONTEND_URL_LOCAL_VUE_CLI: http://localhost:8080
    + Haz clic en Create Web Service.
    + Copia la URL pública generada (ej. [https://familytree2026-backend.onrender.com](https://familytree2026-backend.onrender.com)).

### Capa de Presentación (Vercel)
1. Creación de Cuenta:
    + Accede a vercel.com mediante Continue with GitHub.
    + En el onboarding, selecciona "I'm working on personal projects" para habilitar el plan Hobby 100% gratuito (sin tarjeta).
    + En el aviso de seguridad 2FA, selecciona "Skip securing my account".
    + Haz clic en Add New... > Project e importa familytree2026-frontend.
2. Importación y Despliegue del Frontend:
    + En el Dashboard, haz clic en Add New... > Project.
    + Importa el repositorio del frontend (familytree2026-frontend).
3. Ajustes de Build & Runtime:
    + En Settings > Build and Deployment:
        + Node.js Version: 20.x
        + Install Command (Override): npm install --legacy-peer-deps (evita errores ERESOLVE por peer dependencies de paquetes como oxlint).
4. Variables de Entorno en Vercel:
    + En Settings > Environment Variables:
        + Key: VITE_API_BASE_URL
        + Value: https://familytree2026-backend.onrender.com/api/v1
5. Despliegue Final:
    + Haz clic en Deploy. Tras guardar o cambiar variables de entorno, ejecuta siempre un Redeploy (sin usar Build Cache) para inyectar la URL de la API en los archivos estáticos de React/Vite.

## -------------------------

```bash
```

## Verificar Servidores en Ejecución

### Terminal 1 (Backend):
```bash
cd familytree2026-backend
npm run dev
```

### Terminal 2 (Frontend):
```bash
cd familytree2026-frontend
npm run dev
```


### Base de datos
```bash
cd familytree2026-backend
npx prisma studio
# en caso de problemas
npx prisma studio --url "postgresql://dev_user:dev_password@localhost:5432/local_starter_db?schema=public"
```

### Url
#### Backend
1. Home:
    + Dev: `http://localhost:4000`
    + Prod: `https://familytree2026-backend.onrender.com`
2. Healthcheck (Comprobación de estado):
    + Dev: `http://localhost:4000/api/v1/health`
    + Prod: `https://familytree2026-backend.onrender.com/api/v1/health`
3. Endpoint de Usuario (Protegido):
    + Dev: `http://localhost:4000/api/v1/auth/me`
    + Prod: `https://familytree2026-backend.onrender.com/api/v1/auth/me`

#### Frontend
1. Home:
    + `http://localhost:5173`
2. Prueba de Registro:
    + `http://localhost:5173/register`
3. Prueba de Vista Protegida:
    + `http://localhost:5173/dashboard`
4. Prueba de Rehidratación de Sesión (Persistence):
    + Presiona F5 (Recargar página). El Navigation Guard debe ejecutar `fetchUser()`, validar el token contra el endpoint GET `/me` y mantenerte en `/dashboard` sin cerrar tu sesión.
5. Prueba de Cierre de Sesión:
    + Haz clic en el botón Cerrar Sesión. Debe limpiar el localStorage, borrar el usuario de Pinia y redirigirte a `/login`.
6. Prueba de Protección de Rutas:
    + Estando deslogueado, intenta escribir manualmente `http://localhost:5173/dashboard` en la barra de direcciones. El Navigation Guard debe rebotarte de inmediato a `/login`.



## borradores
`http://localhost:51212/`

`http://localhost:5173/admin/users`

familytree2026-backend
familytree2026-frontend
familytree2026-documentacion


git remote add origin https://github.com/TU_USUARIO/starter-backend.git
git remote add origin https://github.com/petrix12/familytree2026-backend.git

git remote add origin git@github.com:petrix12/familytree2026-backend.git
git remote add origin git@github.com:petrix12/familytree2026-frontend.git
git remote add origin git@github.com:petrix12/familytree2026-documentacion.git


git remote add origin https://github.com/TU_USUARIO/starter-backend.git


git remote add origin git@github.com:TU_USUARIO/starter-backend.git