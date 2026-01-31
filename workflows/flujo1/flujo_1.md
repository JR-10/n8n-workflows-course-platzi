# Lead Capture & Routing Workflow

> **Workflow ID:** `flujo_1_platzi`  
> **Versión:** 1.0.0  
> **Última actualización:** 2026-01-31  

---

## Tabla de Contenidos

1. [Resumen Ejecutivo](#resumen-ejecutivo)
2. [Arquitectura del Workflow](#arquitectura-del-workflow)
3. [Requisitos Previos](#requisitos-previos)
4. [Configuración del Entorno](#configuración-del-entorno)
5. [Configuración de Credenciales OAuth2](#configuración-de-credenciales-oauth2)
6. [Despliegue y Ejecución](#despliegue-y-ejecución)
7. [Gestión de Código y Versionamiento](#gestión-de-código-y-versionamiento)
8. [Troubleshooting](#troubleshooting)

---

## Resumen Ejecutivo

Este workflow implementa un sistema automatizado de captura y enrutamiento de leads mediante un formulario web. El flujo procesa las solicitudes entrantes y las direcciona hacia Slack o Gmail según reglas de negocio definidas.

### Caso de Uso

- **Entrada:** Formulario web con campos Nombre, Email y Mensaje
- **Procesamiento:** Filtrado de emails internos (`@platzi.com`) y clasificación por contenido
- **Salida:** Notificación a Slack (canal `#sellers`) o respuesta automática por Gmail

### Stack Tecnológico

| Componente | Tecnología |
|------------|------------|
| Orquestador | n8n (Docker) |
| Entorno | WSL 2 / Windows |
| Túnel HTTPS | ngrok |
| Integraciones | Slack API, Gmail API (OAuth2) |

---

## Arquitectura del Workflow

```
┌─────────────────────┐
│   Form Trigger      │  ← Webhook público (ngrok)
│   (Lead Form)       │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│    Edit Fields      │  ← Normalización de datos
│   (Set Variables)   │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│      Filter         │  ← Excluye emails @platzi.com
│  (Domain Check)     │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│        If           │  ← Verifica si mensaje contiene "demo"
│  (Content Router)   │
└──────────┼──────────┘
           │
     ┌─────┴─────┐
     │           │
     ▼           ▼
┌─────────┐ ┌─────────┐
│  Slack  │ │  Gmail  │
│ #sellers│ │ (Reply) │
└─────────┘ └─────────┘
```

### Descripción de Nodos

| Nodo | Tipo | Función |
|------|------|---------|
| `On form submission` | Form Trigger | Captura datos del formulario web |
| `Edit Fields` | Set | Mapea campos: `Nombre` → `nombre`, `Email` → `email`, `Mensaje` → `mensaje` |
| `Filter` | Filter | Excluye registros con dominio `@platzi.com` |
| `If` | Conditional | Bifurca según presencia de "demo" en el mensaje |
| `Send a message` | Slack | Envía notificación al canal `#sellers` |
| `Send a message1` | Gmail | Envía email de agradecimiento automático |

---

## Requisitos Previos

### Software

- **Windows 10/11** con WSL 2 habilitado
- **Docker Desktop** ≥ 4.0 con integración WSL 2
- **Docker Compose** ≥ 2.0
- **ngrok** cuenta gratuita o de pago

### Accesos

- Cuenta de Google con permisos para crear OAuth Apps
- Workspace de Slack con permisos de administrador
- Repositorio Git configurado (opcional)

---

## Configuración del Entorno

### 1. Inicialización de WSL

Ejecutar en **PowerShell** como Administrador:

```powershell
wsl --update
wsl --shutdown
wsl
```

### 2. Validación de Docker

```bash
# Verificar instalación
docker --version
docker compose version  # Requerido: v2.x+
```

### 3. Estructura del Proyecto

```bash
mkdir -p n8n-workflows-course-platzi
cd n8n-workflows-course-platzi
```

### 4. Variables de Entorno

Crear archivo `.env`:

```bash
# .env
# Dominio ngrok (sin protocolo ni trailing slash)
N8N_HOST=tu-subdominio.ngrok-free.app
```

### 5. Docker Compose

Crear `docker-compose.yml`:

```yaml
version: "3.8"

services:
  n8n:
    image: docker.n8n.io/n8nio/n8n
    ports:
      - "5678:5678"
    environment:
      - N8N_HOST=${N8N_HOST}
      - WEBHOOK_URL=https://${N8N_HOST}
      - N8N_EDITOR_BASE_URL=https://${N8N_HOST}
    volumes:
      - n8n_data:/home/node/.n8n
    restart: unless-stopped

volumes:
  n8n_data:
```

### 6. Iniciar Servicios

```bash
docker compose up -d
```

**Acceso:** `https://<TU_NGROK_DOMAIN>`

---

## Configuración de Credenciales OAuth2

### Slack API

1. Acceder a [Slack API](https://api.slack.com/apps) → **Create New App**
2. Navegar a **OAuth & Permissions**
3. Configurar **Redirect URL:**
   ```
   https://<TU_NGROK>/rest/oauth2-credential/callback
   ```
4. Agregar **Scopes:**
   - `chat:write`
   - `chat:write.customize`
5. **Install to Workspace** y copiar el `Bot User OAuth Token`

### Google Cloud Platform (Gmail API)

1. Acceder a [Google Cloud Console](https://console.cloud.google.com/)
2. Crear proyecto o seleccionar existente
3. Habilitar **Gmail API** desde Library
4. Configurar **OAuth Consent Screen:**
   - User Type: `External`
   - Test Users: Agregar email autorizado
5. Crear **Credentials → OAuth Client ID** (Web Application)
6. Configurar **Authorized redirect URI:**
   ```
   https://<TU_NGROK>/rest/oauth2-credential/callback
   ```
7. Copiar `Client ID` y `Client Secret`

### Vinculación en n8n

1. Acceder a **Credentials → Add Credential**
2. Configurar **Slack OAuth2 API:**
   - Ingresar Bot Token
   - Autorizar con cuenta de Slack
3. Configurar **Gmail OAuth2 API:**
   - Ingresar Client ID y Client Secret
   - Autorizar con cuenta de Google

---

## Despliegue y Ejecución

### Importar Workflow

1. Acceder a n8n Editor
2. **Menu (⋮) → Import → From File**
3. Seleccionar `flujo_1_platzi.json`
4. Verificar que las credenciales estén vinculadas correctamente

### Activar Workflow

1. Toggle **Active** en la esquina superior derecha
2. Copiar la URL del Form Trigger para pruebas
3. Verificar logs en **Executions**

### URL del Formulario

```
https://<TU_NGROK>/form/<WEBHOOK_ID>
```

---

## Gestión de Código y Versionamiento

### Estructura de Directorios

```
n8n-workflows-course-platzi/
├── docker-compose.yml
├── .env                    # ⚠️ No commitear
├── .gitignore
├── README.md
└── workflows/
    └── flujo1/
        ├── flujo_1_platzi.json
        └── flujo_1.md
```

### Exportar Workflow

1. En n8n Editor: **Menu (⋮) → Export → Download**
2. Guardar como `.json` en directorio correspondiente

### Configuración de .gitignore

```gitignore
# Secrets
.env
*.env.local

# n8n Data
n8n_data/

# OS Files
.DS_Store
Thumbs.db
```

### Git Workflow

```bash
# Inicializar repositorio
git init
git branch -M main
git remote add origin git@github.com:<USUARIO>/n8n-workflows-course-platzi.git

# Commit con Conventional Commits
git add .
git commit -m "feat(workflow): add lead capture and routing automation

- Add form trigger for lead capture
- Implement domain filtering (@platzi.com exclusion)
- Add conditional routing to Slack/Gmail
- Include Docker infrastructure setup"

# Push
git push -u origin main
```

---

## Troubleshooting

### Problemas Comunes

| Síntoma | Causa Probable | Solución |
|---------|----------------|----------|
| Webhook no responde | ngrok desconectado | Reiniciar túnel y actualizar `.env` |
| Error OAuth2 Slack | Redirect URI incorrecta | Verificar URL en Slack App Settings |
| Error OAuth2 Gmail | Consent Screen incompleto | Agregar email a Test Users |
| Form no carga | Workflow inactivo | Activar toggle en n8n Editor |
| Mensaje no llega a Slack | Canal incorrecto | Verificar `channelId` en nodo Slack |

### Logs y Debugging

```bash
# Ver logs del contenedor
docker compose logs -f n8n

# Verificar estado del servicio
docker compose ps

# Reiniciar servicio
docker compose restart n8n
```

### Validar Conectividad

```bash
# Test ngrok tunnel
curl -I https://<TU_NGROK>/healthz

# Test n8n interno
curl -I http://localhost:5678/healthz
```

---

## Referencias

- [Documentación oficial n8n](https://docs.n8n.io/)
- [Slack API Documentation](https://api.slack.com/docs)
- [Gmail API Reference](https://developers.google.com/gmail/api/reference/rest)
- [ngrok Documentation](https://ngrok.com/docs)

---

<div align="center">

**Archivo de workflow:** [`flujo_1_platzi.json`](./flujo_1_platzi.json)

</div>