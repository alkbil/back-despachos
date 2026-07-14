
## Arquitectura de Despliegue

Este servicio (Backend Despachos) forma parte de la plataforma Innovatech, compuesta por
un Frontend (React/Vite + NGINX), dos microservicios Backend (Spring Boot: Ventas y
Despachos) y una base de datos MySQL, desplegados en un cluster Amazon EKS. El diagrama
completo de arquitectura y los manifiestos de Kubernetes se encuentran centralizados en
el repositorio front-despacho (carpeta k8s/).

API expuesta en el puerto 8081 (ver Swagger UI en /swagger-ui.html), consumida
internamente por el Frontend y conectada a MySQL mediante variables de entorno
inyectadas desde un Secret de Kubernetes (DB_ENDPOINT, DB_PORT, DB_NAME, DB_USERNAME,
DB_PASSWORD). Cada despacho referencia el idCompra de la venta asociada, generada por
el servicio Backend Ventas.

## Desarrollo local con Docker Compose

El archivo docker-compose.yml para levantar el stack completo (frontend, ambos backends
y mysql) se encuentra en el repositorio front-despacho.

## Despliegue en AWS EKS

- Cluster: Innovatech-eks (Kubernetes v1.36, EKS Auto Mode)
- Namespace: innovatech
- Imagen publicada en Amazon ECR: backend-despachos-innovatech
- Deployment con 1-3 replicas mediante Horizontal Pod Autoscaler (CPU 50%)

### Evidencia de funcionamiento

Pods corriendo:

NAME                                 READY   STATUS    RESTARTS   AGE
backend-despachos-5bf7c99c97-b4f6p   1/1     Running   2          98s
backend-despachos-5bf7c99c97-lvkp7   1/1     Running   2          98s
backend-despachos-5bf7c99c97-mt7jk   1/1     Running   2          98s

Prueba funcional via Swagger UI (POST /api/v1/despachos):
Respuesta 201 Created:

{
  "idDespacho": 1,
  "fechaDespacho": "2026-07-14",
  "patenteCamion": "ABCD-12",
  "intento": 1,
  "idCompra": 1,
  "direccionCompra": "Av. Providencia 1234, Santiago",
  "valorCompra": 25000,
  "despachado": false
}

## Observabilidad

Metricas de escalado disponibles via kubectl get hpa -n innovatech
(backend-despachos-hpa: 1-3 replicas, target CPU 50%). Logs verificados mediante
kubectl logs para confirmar la correcta conexion a la base de datos y a los datos
generados por el servicio de Ventas.

## CI/CD Pipeline

El workflow de GitHub Actions ejecuta build -> test -> build de imagen Docker -> push a
Amazon ECR (etiquetada con el SHA del commit) -> despliegue en EKS mediante kubectl apply.
Las credenciales se gestionan mediante GitHub Secrets.
