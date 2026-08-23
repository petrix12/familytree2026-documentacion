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

        const app = express();
        const PORT = process.env.PORT || 4000;

        // Middlewares Globales
        app.use(cors());
        app.use(express.json());

        // Ruta de comprobación de estado (Health Check)
        app.get('/api/v1/health', (req, res) => {
            res.status(200).json({
                status: 'success',
                message: 'API FamilyTree2026 operativa',
                environment: process.env.NODE_ENV,
                timestamp: new Date().toISOString(),
            });
        });

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

📑 Verificación de Autenticación y Control de Acceso (RBAC)
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
---

## 🎨 Construcción del Cliente Frontend SPA (Vue 3 + Vite + Pinia)
Con la API REST de Autenticación finalizada, procedemos a crear la aplicación Frontend en la carpeta `starter-frontend` que gestionará la experiencia del usuario (Login, Registro, Guarda de Rutas y Redirecciones "estilo Laravel").


## -------------------------

```bash
```


npm run dev
npx prisma studio
npx prisma studio --url "postgresql://dev_user:dev_password@localhost:5432/local_starter_db?schema=public"


http://localhost:4000/api/v1/health

http://localhost:51212/


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