# Portfolio CV - Sistema de Gestión de Currículum

Sistema completo para gestionar y generar currículums en formato PDF. Consta de dos microservicios: un API REST en Spring Boot para la gestión de datos y un servicio Python para la generación de PDFs.

## 📋 Descripción

Este proyecto permite gestionar información personal, experiencia laboral, habilidades, proyectos y generar currículums en formato PDF de manera automatizada. El sistema está diseñado con una arquitectura de microservicios donde:

- **ms-resume**: Servicio principal en Java Spring Boot que expone APIs REST para gestionar los datos del currículum
- **ms-render-cv**: Servicio interno en Python que genera PDFs a partir de la información recibida

## 🏗️ Arquitectura

```
┌─────────────────┐
│   Cliente Web   │
└────────┬────────┘
         │ HTTP
         ▼
┌─────────────────────────┐
│   ms-resume (Java)      │  ← Accesible desde fuera (puerto 8080)
│   Spring Boot REST API  │
└────────┬────────────────┘
         │ HTTP (red interna)
         ▼
┌─────────────────────────┐
│  ms-render-cv (Python)  │  ← Solo interno (puerto 8000)
│   FastAPI + RenderCV    │
└─────────────────────────┘
```

## 🚀 Tecnologías

### Backend
- **Java 21** - Spring Boot 3.5.3
- **Python 3.12** - FastAPI, RenderCV
- **SQLite** - Base de datos
- **Docker & Docker Compose** - Contenedorización

### Características
- Generación de PDFs de currículums
- API REST para gestión de datos
- Integración con Telegram Bot
- Integración con AWS S3
- Autenticación JWT
- Migraciones de base de datos con Flyway

## 📁 Estructura del Proyecto

```
portfolio-cv/
├── docker-compose.yml          # Configuración de servicios Docker
├── ms-resume/                  # Servicio Java Spring Boot
│   ├── src/
│   │   └── main/
│   │       ├── java/           # Código fuente Java
│   │       └── resources/      # Configuraciones
│   ├── .env                    # Variables de entorno (no se sube a git)
│   └── Dockerfile
├── ms-render-cv/               # Servicio Python FastAPI
│   ├── api/                    # API FastAPI
│   ├── rendercv/               # Lógica de generación de PDFs
│   ├── requirements.txt
│   └── Dockerfile
└── data/                       # Base de datos SQLite (persistente)
    └── ms-resume.db
```

## 🔧 Requisitos Previos

- Docker y Docker Compose instalados
- Git (para clonar el repositorio)

## 📦 Instalación

### 1. Clonar el repositorio

```bash
git clone <url-del-repositorio>
cd portfolio-cv
```

### 2. Configurar variables de entorno

Crea un archivo `.env` en `ms-resume/.env` con las siguientes variables:

```bash
# Telegram Bot Configuration
TELEGRAM_BOT_TOKEN=tu-token-de-telegram
TELEGRAM_CHAT_ID=tu-chat-id

# AWS S3 Configuration
AWS_ACCESS_KEY=tu-access-key
AWS_SECRET_KEY=tu-secret-key
AWS_REGION=us-east-2
AWS_BUCKET=tu-bucket

# Authentication
AUTH_USER=tu-usuario
AUTH_PASS=tu-contraseña-hash

# JWT
JWT_SECRET=tu-secret-jwt-base64

# Base de datos (opcional)
DB_PATH=/data/ms-resume.db
DB_DATA_PATH=./data
```

### 3. Crear carpeta para la base de datos

```bash
mkdir -p data
touch data/.gitkeep
```

### 4. Levantar los servicios

```bash
docker compose up -d --build
```

## 🎯 Uso

### Verificar que los servicios estén corriendo

```bash
docker compose ps
```

### Ver los logs

```bash
# Todos los servicios
docker compose logs -f

# Servicio específico
docker compose logs -f ms-resume
docker compose logs -f ms-render-cv
```

### Acceder a los servicios

- **ms-resume (Java)**: `http://localhost:8080/v1/ms-resume`
  - Documentación Swagger: `http://localhost:8080/v1/ms-resume/swagger-ui.html`
  - Health check: `http://localhost:8080/v1/ms-resume/actuator/health`

- **ms-render-cv (Python)**: Solo accesible internamente desde `ms-resume`
  - Endpoint: `http://ms-render-cv:8000/render` (solo desde dentro de Docker)

## 🔐 Seguridad

### Configuración de Red

- **ms-resume**: Expone el puerto 8080 solo en `127.0.0.1` (localhost), ideal para usar con un proxy reverso como Nginx
- **ms-render-cv**: No es accesible desde fuera del contenedor, solo desde la red interna de Docker

### Variables de Entorno

Todas las credenciales y configuraciones sensibles deben estar en el archivo `ms-resume/.env` que **NO** se sube a Git.

## 💾 Base de Datos

### Persistencia

La base de datos SQLite se guarda en la carpeta `data/` del servidor host, mapeada al contenedor en `/data`.

### Configuración de la ruta

Puedes configurar dónde se guarda la base de datos mediante variables de entorno:

```bash
# En ms-resume/.env o como variable de entorno del sistema
DB_DATA_PATH=/ruta/absoluta/en/el/servidor/data
DB_PATH=/data/ms-resume.db
```

### Migrar base de datos existente

Si tienes un archivo `ms-resume.db` existente:

1. Copia el archivo a la carpeta `data/`:
```bash
cp /ruta/al/ms-resume.db ./data/ms-resume.db
```

2. Asegúrate de que tenga los permisos correctos:
```bash
chmod 666 data/ms-resume.db
```

3. Levanta el contenedor:
```bash
docker compose up -d
```

Spring Boot usará el archivo existente y aplicará solo las migraciones pendientes.

## 🛠️ Comandos Útiles

### Gestión de contenedores

```bash
# Detener todos los servicios
docker compose stop

# Detener y eliminar contenedores
docker compose down

# Reiniciar servicios
docker compose restart

# Reiniciar un servicio específico
docker compose restart ms-resume

# Ver estado de los contenedores
docker compose ps
```

### Reconstrucción

```bash
# Reconstruir todos los servicios
docker compose up -d --build

# Reconstruir un servicio específico
docker compose up -d --build ms-resume
```

### Limpieza

```bash
# Eliminar contenedores, volúmenes y redes
docker compose down -v

# Eliminar imágenes también
docker compose down -v --rmi all
```

## 🔄 Comunicación entre Servicios

El servicio Java (`ms-resume`) se comunica con el servicio Python (`ms-render-cv`) usando el nombre del servicio Docker:

- URL interna: `http://ms-render-cv:8000`
- Configurado mediante: `CONFIG_RENDERCV_URL=http://ms-render-cv:8000`

Esta comunicación solo funciona dentro de la red Docker `portfolio_net`.

## 📝 Variables de Entorno Importantes

### ms-resume

| Variable | Descripción | Por Defecto |
|----------|-------------|-------------|
| `SPRING_PROFILES_ACTIVE` | Perfil de Spring Boot | `prod` |
| `CONFIG_RENDERCV_URL` | URL del servicio Python | `http://ms-render-cv:8000` |
| `DB_PATH` | Ruta de la base de datos en el contenedor | `/data/ms-resume.db` |
| `DB_DATA_PATH` | Ruta en el host donde se guarda la BD | `./data` |
| `JAVA_TOOL_OPTIONS` | Opciones de JVM (memoria) | `-Xms512m -Xmx2048m` |
| `TELEGRAM_BOT_TOKEN` | Token del bot de Telegram | - |
| `TELEGRAM_CHAT_ID` | ID del chat de Telegram | - |
| `AWS_ACCESS_KEY` | Clave de acceso AWS | - |
| `AWS_SECRET_KEY` | Clave secreta AWS | - |
| `JWT_SECRET` | Secreto para JWT | - |

### ms-render-cv

| Variable | Descripción | Por Defecto |
|----------|-------------|-------------|
| `WORKERS` | Número de workers de Uvicorn | `1` |

## 🐛 Troubleshooting

### El servicio Java no puede conectarse al servicio Python

Verifica que ambos servicios estén en la misma red:
```bash
docker compose ps
docker network inspect portfolio-cv_portfolio_net
```

### La base de datos no se persiste

Verifica que el volumen esté montado correctamente:
```bash
docker compose exec ms-resume ls -la /data
```

### Error de permisos en la base de datos

Asegúrate de que la carpeta `data/` tenga los permisos correctos:
```bash
chmod -R 777 data/
```

### El servicio Python no inicia

Verifica que estés usando Python 3.12+ (requerido para la sintaxis de genéricos):
```bash
docker compose logs ms-render-cv
```

## 📄 Licencia

MIT License

## 👤 Autor

Cristhiam Reina - cristiansrc@gmail.com

## 🔗 Enlaces

- [Documentación de Spring Boot](https://spring.io/projects/spring-boot)
- [Documentación de FastAPI](https://fastapi.tiangolo.com/)
- [RenderCV](https://github.com/sinaatalay/rendercv)
