# ShopMicro

Plantilla completa para el proyecto "Docker Orquestradors: Docker Swarm i Kubernetes" del caso de uso ShopMicro.

## Qué incluye

- `docker-compose.yml`: fase 1 para entorno local con healthchecks, redes y volúmenes.
- `docker-stack.yml`: fase 2 y 3 para Docker Swarm con réplicas, constraints y secrets.
- `k8s/`: fase 4 con Namespace, Deployments, Services, ConfigMaps y Secrets.
- `docs/`: guía para el documento final, arquitectura y checklist de evidencias.
- Microservicios mínimos en Flask para demostrar los flujos funcionales del enunciado.

## Estructura

```text
shopmicro/
├── api-gateway/
├── frontend/
├── product-service/
├── order-service/
├── user-service/
├── notification-service/
├── db/
├── docs/
├── k8s/
├── secrets/
├── docker-compose.yml
└── docker-stack.yml
```

## Flujo funcional implementado

1. Consulta de productos: `frontend -> api-gateway -> product-service -> Redis/MySQL`.
2. Creación de pedido: `frontend -> api-gateway -> order-service -> product-service -> db-orders -> RabbitMQ -> notification-service`.
3. Base para tolerancia a fallos en Swarm: servicios con réplicas y placement definidos.

## Arranque local

1. Copia `.env.example` a `.env` y ajusta las credenciales.
2. Lanza `docker compose up -d --build`.
3. Abre `http://localhost:8080`.

## Nota

El código es deliberadamente sencillo para ajustarse al enunciado: la prioridad es la orquestación, no una lógica de negocio compleja.

