# Lead Capture & Routing Workflow

> **Workflow ID:** `flujo_1_platzi`  
> **Versión:** 1.0.0  
> **Última actualización:** 2026-01-31  
> **Autor:** Curso n8n - Platzi

---

## Tabla de Contenidos

1. [Resumen Ejecutivo](#resumen-ejecutivo)
2. [Arquitectura del Workflow](#arquitectura-del-workflow)
3. [Historial de Versiones](#historial-de-versiones)
4. [Requisitos Previos](#requisitos-previos)
5. [Configuración del Entorno](#configuración-del-entorno)
6. [Configuración de Credenciales OAuth2](#configuración-de-credenciales-oauth2)
7. [Despliegue y Ejecución](#despliegue-y-ejecución)
8. [Gestión de Código y Versionamiento](#gestión-de-código-y-versionamiento)
9. [Troubleshooting](#troubleshooting)
10. [Referencias](#referencias)

---

## Resumen Ejecutivo

Este workflow implementa un sistema automatizado de captura y enrutamiento de leads mediante un formulario web. El flujo procesa las solicitudes entrantes y las direcciona hacia Slack o Gmail según reglas de negocio definidas.

### Caso de Uso

| Aspecto | Descripción |
|---------|-------------|
| **Entrada** | Formulario web con campos Nombre, Email y Mensaje |
| **Procesamiento** | Filtrado de emails internos (`@platzi.com`) y clasificación por contenido |
| **Salida** | Notificación a Slack (canal `#sellers`) o respuesta automática por Gmail |

### Stack Tecnológico

| Componente | Tecnología |
|------------|------------|
| Orquestador | n8n (Docker) |
| Entorno | WSL 2 / Windows |
| Túnel HTTPS | ngrok |
| Integraciones | Slack API, Gmail API (OAuth2) |

---

## Arquitectura del Workflow

### Diagrama Visual

<p align="center">
  <img src="./assets/flujo_1_v1.png" alt="Workflow v1" width="800"/>
</p>

<p align="center"><em>Figura 1: Arquitectura del workflow - Versión 1</em></p>

### Diagrama de Flujo (ASCII)

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

## Historial de Versiones

| Versión | Fecha | Cambios | Estado |
|---------|-------|---------|--------|
| 1.0.0 | 2026-01-31 | Release inicial con captura de leads y enrutamiento básico | ✅ Actual |
| 1.1.0 | *Próximamente* | Integración con CRM y validación avanzada | 🔄 Planificado |

### Capturas por Versión

| Versión | Diagrama |
|---------|----------|
| v1.0.0 | [Ver diagrama](./assets/flujo_1_v1.0.0.png) |

> 📁 **Nota:** Los diagramas de cada versión se almacenan en [`./assets/`](./assets/) para mantener trazabilidad visual del workflow.

---

## Requisitos Previos

### Software

| Requisito | Versión Mínima | Verificación |
|-----------|----------------|--------------|
| Windows | 10/11 con WSL 2 | `wsl --version` |
| Docker Desktop | ≥ 4.0 | `docker --version` |
| Docker Compose | ≥ 2.0 | `docker compose version` |
| ngrok | Cuenta activa | `ngrok --version` |

### Accesos Requeridos

- [ ] Cuenta de Google con permisos para crear OAuth Apps
- [ ] Workspace de Slack con permisos de administrador
- [ ] Repositorio Git configurado (opcional)

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
# filepath: .env
# Dominio ngrok (sin protocolo ni trailing slash)
N8N_HOST=tu-subdominio.ngrok-free.app
```

### 5. Docker Compose

Crear `docker-compose.yml`:

```yaml
# filepath: docker-compose.yml
version: "3.8"

services:
  n8n:
    image: n8nio/n8n:latest
    container_name: n8n
    restart: unless-stopped
    ports:
      - "5678:5678"
    environment:
      - N8N_HOST=${N8N_HOST}
      - N8N_PROTOCOL=https
      - WEBHOOK_URL=https://${N8N_HOST}/
    volumes:
      - n8n_data:/home/node/.n8n

volumes:
  n8n_data:
```

### 6. Iniciar Servicios

```bash
docker compose up -d
```

### 7. Verificar Estado

```bash
# Verificar contenedor
docker ps | grep n8n

# Ver logs
docker logs n8n --tail 50
```

**Acceso:** `https://<TU_NGROK_DOMAIN>`

---

## Configuración de Credenciales OAuth2

### Slack API

#### Paso 1: Crear Aplicación

1. Acceder a [Slack API](https://api.slack.com/apps)
2. Click en **Create New App** → **From scratch**
3. Nombrar la aplicación y seleccionar workspace

#### Paso 2: Configurar OAuth

1. Navegar a **OAuth & Permissions**
2. Configurar **Redirect URL:**

```
https://<TU_NGROK_DOMAIN>/rest/oauth2-credential/callback
```

#### Paso 3: Agregar Scopes

| Scope | Descripción |
|-------|-------------|
| `chat:write` | Enviar mensajes como bot |
| `chat:write.customize` | Personalizar nombre/icono del mensaje |

#### Paso 4: Instalar

1. Click en **Install to Workspace**
2. Autorizar permisos
3. Copiar `Bot User OAuth Token`

---

### Google Cloud Platform (Gmail API)

#### Paso 1: Crear Proyecto

1. Acceder a [Google Cloud Console](https://console.cloud.google.com/)
2. Crear proyecto nuevo o seleccionar existente

#### Paso 2: Habilitar API

1. Ir a **APIs & Services** → **Library**
2. Buscar **Gmail API**
3. Click en **Enable**

#### Paso 3: Configurar Consent Screen

| Campo | Valor |
|-------|-------|
| User Type | External |
| App Name | n8n Workflow |
| Support Email | Tu email |
| Test Users | Emails autorizados |

#### Paso 4: Crear Credenciales

1. **APIs & Services** → **Credentials**
2. **Create Credentials** → **OAuth Client ID**
3. Application type: **Web Application**
4. Authorized redirect URI:

```
https://<TU_NGROK_DOMAIN>/rest/oauth2-credential/callback
```

5. Copiar `Client ID` y `Client Secret`

---

### Vinculación en n8n

#### Slack

1. **Credentials** → **Add Credential** → **Slack OAuth2 API**
2. Ingresar Bot Token
3. Click en **Connect** para autorizar

#### Gmail

1. **Credentials** → **Add Credential** → **Gmail OAuth2 API**
2. Ingresar Client ID y Client Secret
3. Click en **Connect** para autorizar con cuenta de Google

---

## Despliegue y Ejecución

### Importar Workflow

```
1. Acceder a n8n Editor
2. Menu (⋮) → Import → From File
3. Seleccionar: workflows/flujo1/flujo_1_platzi.json
4. Verificar credenciales vinculadas
```

### Activar Workflow

| Paso | Acción |
|------|--------|
| 1 | Toggle **Active** (esquina superior derecha) |
| 2 | Copiar URL del Form Trigger |
| 3 | Verificar en **Executions** |

### URL del Formulario

```
https://<TU_NGROK_DOMAIN>/form/<WEBHOOK_ID>
```

### Probar Workflow

```bash
# Ejemplo con curl
curl -X POST https://<TU_NGROK_DOMAIN>/form/<WEBHOOK_ID> \
  -H "Content-Type: application/json" \
  -d '{
    "Nombre": "Test User",
    "Email": "test@example.com",
    "Mensaje": "Quiero una demo del producto"
  }'
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
        ├── assets/
        │   └── flujo_1_v1.0.0.png
        ├── flujo_1_platzi.json
        └── flujo_1.md
```

### Exportar Workflow

1. En n8n Editor: **Menu (⋮)** → **Export** → **Download**
2. Guardar como `.json` en directorio correspondiente
3. Nombrar con versión: `flujo_1_platzi_v1.1.0.json` (opcional)

### Configuración de .gitignore

```gitignore
# filepath: .gitignore
# Secrets
.env
*.env.local
*.env.*.local

# n8n Data
n8n_data/

# OS Files
.DS_Store
Thumbs.db

# IDE
.vscode/
.idea/

# Logs
*.log
```

### Conventional Commits

```bash
# Feature
git commit -m "feat(flujo1): add email validation before routing"

# Fix
git commit -m "fix(flujo1): correct Slack channel ID reference"

# Docs
git commit -m "docs(flujo1): update architecture diagram for v1.1.0"
```

---

## Troubleshooting

### Problemas Comunes

| Problema | Causa | Solución |
|----------|-------|----------|
| Webhook no responde | Túnel ngrok caído | Reiniciar ngrok y actualizar `N8N_HOST` |
| OAuth error en Slack | Redirect URI incorrecta | Verificar URL en Slack App config |
| Gmail 403 Forbidden | App no verificada | Agregar email a Test Users en GCP |
| Formulario no aparece | Workflow inactivo | Activar toggle en n8n Editor |
| Credenciales expiradas | Token OAuth vencido | Re-autorizar en n8n Credentials |

### Comandos de Debugging

```bash
# Ver logs de n8n
docker logs n8n --tail 100 -f

# Verificar conectividad ngrok
curl -I https://<TU_NGROK_DOMAIN>/healthz

# Reiniciar servicios
docker compose restart n8n

# Verificar estado del contenedor
docker inspect n8n --format='{{.State.Status}}'
```

### Validar Credenciales

```bash
# Test Slack API
curl -X POST https://slack.com/api/auth.test \
  -H "Authorization: Bearer <BOT_TOKEN>"

# Test Gmail API (requiere OAuth token activo)
curl https://gmail.googleapis.com/gmail/v1/users/me/profile \
  -H "Authorization: Bearer <ACCESS_TOKEN>"
```

---

## Referencias

| Recurso | Enlace |
|---------|--------|
| Documentación n8n | [docs.n8n.io](https://docs.n8n.io/) |
| n8n Form Trigger | [docs.n8n.io/nodes/n8n-nodes-base.formTrigger](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.formtrigger/) |
| Slack API | [api.slack.com](https://api.slack.com/) |
| Gmail API | [developers.google.com/gmail/api](https://developers.google.com/gmail/api) |
| ngrok Docs | [ngrok.com/docs](https://ngrok.com/docs) |
| Conventional Commits | [conventionalcommits.org](https://www.conventionalcommits.org/) |

---

<p align="center">
  <strong>Curso n8n - Platzi</strong><br/>
  <em>Automatización de flujos de trabajo</em>
</p>