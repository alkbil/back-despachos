# Backend Despachos — Innovatech Chile

API REST para la gestión de despachos de Innovatech Chile.

## Tecnologías
- Java 17 + Spring Boot 3
- Maven
- MySQL 8.0
- Docker (multi-stage build)

## Endpoints principales
- `GET /api/v1/despachos` — Listar todos los despachos
- `GET /api/v1/despachos/{id}` — Obtener despacho por ID
- `POST /api/v1/despachos` — Crear nuevo despacho
- `PUT /api/v1/despachos/{id}` — Actualizar despacho
- Swagger UI: `/swagger-ui.html`

## Estructura del repositorio
back-Despachos_SpringBoot/
├── Springboot-API-REST-DESPACHO/  # Código fuente Spring Boot
│   ├── src/
│   ├── Dockerfile                  # Multi-stage build
│   └── pom.xml
├── k8s/                            # Manifiestos Kubernetes
│   ├── backend-despachos-deployment.yaml
│   ├── backend-despachos-service.yaml
│   └── backend-despachos-hpa.yaml
└── .github/workflows/
└── deploy.yml                  # Pipeline CI/CD

## Variables de entorno
| Variable | Descripción |
|----------|-------------|
| `SPRING_DATASOURCE_URL` | URL de conexión MySQL |
| `DB_USERNAME` | Usuario de la base de datos |
| `DB_PASSWORD` | Contraseña de la base de datos |

## Pipeline CI/CD
El pipeline se activa con push a `main` o `deploy`:
1. **Build** → Compila JAR con Maven y construye imagen Docker
2. **Push** → Publica imagen en Amazon ECR
3. **Apply** → Aplica manifiestos Kubernetes en EKS
4. **Deploy** → Ejecuta rollout restart en el cluster

## Infraestructura AWS
- **Cluster EKS:** `innovatech-eks` (us-east-1)
- **Namespace:** `innovatech`
- **ECR:** `830985015694.dkr.ecr.us-east-1.amazonaws.com/backend-despachos-innovatech`
- **Service:** ClusterIP (accesible solo internamente)
- **Puerto:** 8081
- **HPA:** mín 2 réplicas, máx 5, umbral CPU 50%

## Secrets requeridos en GitHub
| Secret | Descripción |
|--------|-------------|
| `AWS_ACCESS_KEY_ID` | Credencial AWS |
| `AWS_SECRET_ACCESS_KEY` | Credencial AWS |
| `AWS_SESSION_TOKEN` | Token de sesión AWS |

## Despliegue local
```bash
docker-compose up --build
```