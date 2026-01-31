# n8n Workflows - Curso Platzi

> Repositorio académico con workflows de automatización desarrollados durante el curso de n8n en Platzi.

---

## Descripción

Este repositorio contiene una colección de flujos de trabajo (workflows) de **n8n** creados con fines educativos. Cada workflow está documentado y organizado de forma modular para facilitar el aprendizaje y la reutilización.

## Stack Tecnológico

| Componente | Tecnología |
|------------|------------|
| Orquestador | n8n (Docker) |
| Entorno | WSL 2 / Windows |
| Túnel HTTPS | ngrok |
| Integraciones | Slack, Gmail, Forms |

## Estructura del Proyecto

```
n8n-workflows-course-platzi/
├── docker-compose.yml      # Infraestructura n8n
├── .env                    # Variables de entorno (no commitear)
├── README.md               # Este archivo
└── workflows/
    ├── flujo1/             # Lead Capture & Routing
    │   ├── flujo_1_platzi.json
    │   └── flujo_1.md
    └── flujo2/             # (Próximo workflow)
```

## Workflows Disponibles

| # | Nombre | Descripción | Documentación |
|---|--------|-------------|---------------|
| 1 | **Lead Capture & Routing** | Captura leads via formulario y los enruta a Slack o Gmail según reglas de negocio | [Ver docs](./workflows/flujo1/flujo_1.md) |
| 2 | *Próximamente* | — | — |

## Inicio Rápido

### Requisitos

- Windows 10/11 con WSL 2
- Docker Desktop ≥ 4.0
- Cuenta de ngrok

### Instalación

```bash
# Clonar repositorio
git clone https://github.com/<TU_USUARIO>/n8n-workflows-course-platzi.git
cd n8n-workflows-course-platzi

# Configurar variables de entorno
cp .env.example .env
# Editar .env con tu dominio ngrok

# Levantar n8n
docker compose up -d
```

### Acceso

```
https://<TU_NGROK_DOMAIN>
```

## Importar Workflows

1. Acceder al editor de n8n
2. **Menu (⋮) → Import → From File**
3. Seleccionar el archivo `.json` del workflow deseado

## Seguridad

> ⚠️ **Importante:** No commitear archivos con credenciales

```gitignore
.env
n8n_data/
```

## Recursos

- [Curso n8n - Platzi](https://platzi.com/)
- [Documentación n8n](https://docs.n8n.io/)
- [Slack API](https://api.slack.com/)
- [Gmail API](https://developers.google.com/gmail/api)

## Licencia

Uso académico - Curso Platzi

---

