# Memoria de Procedimiento Kubernetes

## Objetivo

Este documento recoge el procedimiento real seguido para poner en marcha la fase de Kubernetes del proyecto ShopMicro en entorno local, usando Minikube y Docker Desktop como base de ejecución.

La idea es mantener esta memoria viva y actualizarla a medida que se vayan completando nuevas tareas de la fase 4.

## 1. Entorno utilizado

- Sistema anfitrión: Windows 11
- Motor de contenedores: Docker Desktop
- Cliente Kubernetes: `kubectl`
- Clúster local: Minikube
- Driver de Minikube: `docker`
- Versión de Kubernetes solicitada: `v1.30.0`

## 2. Servicios que ha sido necesario construir con Docker

Para Kubernetes no ha sido necesario levantar manualmente todos los servicios con Docker Compose. Lo que sí ha sido necesario es construir con Docker las imágenes propias del proyecto para que Minikube pueda ejecutarlas dentro del clúster.

Servicios propios construidos:

- `shopmicro/frontend:1.0.0`
- `shopmicro/api-gateway:1.0.0`
- `shopmicro/product-service:1.0.0`
- `shopmicro/order-service:1.0.0`
- `shopmicro/user-service:1.0.0`
- `shopmicro/notification-service:1.0.0`

Servicios de infraestructura desplegados directamente desde Kubernetes:

- `mysql:8.0`
- `redis:7-alpine`
- `rabbitmq:3-management`

## 3. Comprobación inicial del entorno

Antes de empezar, se verificó:

- `docker` disponible y operativo
- `kubectl` disponible
- ausencia de `minikube` en el sistema
- manifiestos ya preparados en la carpeta `k8s/`

También se comprobó que inicialmente `kubectl` no tenía contexto activo, por lo que no existía aún un clúster funcional para desplegar la fase 4.

## 4. Instalación de Minikube

Se instaló Minikube mediante `winget`:

```powershell
winget install --id Kubernetes.minikube --accept-package-agreements --accept-source-agreements --silent
```

El ejecutable quedó instalado en:

```text
C:\Program Files\Kubernetes\Minikube\minikube.exe
```

## 5. Arranque del clúster local

Se levantó Minikube usando Docker como driver y fijando la versión de Kubernetes a `1.30.0`, para ajustarse al enunciado:

```powershell
& 'C:\Program Files\Kubernetes\Minikube\minikube.exe' start --driver=docker --kubernetes-version=v1.30.0 --cpus=4 --memory=4096
```

Resultado:

- contexto activo de `kubectl`: `minikube`
- nodo disponible: `minikube`
- estado del nodo: `Ready`

Comprobaciones realizadas:

```powershell
kubectl config current-context
kubectl get nodes -o wide
kubectl get pods -A
```

## 6. Construcción de imágenes propias

Una vez disponible el clúster, se construyeron las imágenes Docker del proyecto:

```powershell
docker build -t shopmicro/frontend:1.0.0 C:\Users\G513\OneDrive - Sa Palomera\Documentos\Codex\shopmicro\frontend
docker build -t shopmicro/api-gateway:1.0.0 C:\Users\G513\OneDrive - Sa Palomera\Documentos\Codex\shopmicro\api-gateway
docker build -t shopmicro/product-service:1.0.0 C:\Users\G513\OneDrive - Sa Palomera\Documentos\Codex\shopmicro\product-service
docker build -t shopmicro/order-service:1.0.0 C:\Users\G513\OneDrive - Sa Palomera\Documentos\Codex\shopmicro\order-service
docker build -t shopmicro/user-service:1.0.0 C:\Users\G513\OneDrive - Sa Palomera\Documentos\Codex\shopmicro\user-service
docker build -t shopmicro/notification-service:1.0.0 C:\Users\G513\OneDrive - Sa Palomera\Documentos\Codex\shopmicro\notification-service
```

## 7. Carga de imágenes en Minikube

Después de construir las imágenes, se cargaron en el nodo de Minikube:

```powershell
& 'C:\Program Files\Kubernetes\Minikube\minikube.exe' image load shopmicro/frontend:1.0.0
& 'C:\Program Files\Kubernetes\Minikube\minikube.exe' image load shopmicro/api-gateway:1.0.0
& 'C:\Program Files\Kubernetes\Minikube\minikube.exe' image load shopmicro/product-service:1.0.0
& 'C:\Program Files\Kubernetes\Minikube\minikube.exe' image load shopmicro/order-service:1.0.0
& 'C:\Program Files\Kubernetes\Minikube\minikube.exe' image load shopmicro/user-service:1.0.0
& 'C:\Program Files\Kubernetes\Minikube\minikube.exe' image load shopmicro/notification-service:1.0.0
```

Este paso es importante porque los manifiestos Kubernetes usan imágenes locales `shopmicro/...` que no existen en Docker Hub.

## 8. Aplicación de los manifiestos

El despliegue se realizó en tres pasos:

```powershell
kubectl apply -f C:\Users\G513\OneDrive - Sa Palomera\Documentos\Codex\shopmicro\k8s\infra\namespace.yaml
kubectl apply -f C:\Users\G513\OneDrive - Sa Palomera\Documentos\Codex\shopmicro\k8s\infra
kubectl apply -f C:\Users\G513\OneDrive - Sa Palomera\Documentos\Codex\shopmicro\k8s\base
```

Recursos creados:

- `Namespace`: `shopmicro`
- `Secrets`
- `ConfigMaps`
- `Deployments`
- `Services`

## 9. Espera y verificación del despliegue

Tras aplicar los manifiestos, se esperó a que todos los `Deployments` estuvieran disponibles:

```powershell
kubectl wait --for=condition=available deployment --all -n shopmicro --timeout=300s
```

Comandos de verificación usados:

```powershell
kubectl get pods -n shopmicro -o wide
kubectl get services -n shopmicro
kubectl get deployments -n shopmicro
kubectl describe deployment product-service -n shopmicro
```

Estado observado:

- todos los `deployments` quedaron en estado `Available`
- `frontend` con 2 réplicas
- `api-gateway` con 2 réplicas
- `product-service` con 2 réplicas
- `order-service` con 2 réplicas
- `user-service` con 2 réplicas
- `notification-service` con 1 réplica
- bases de datos, Redis y RabbitMQ funcionando dentro del namespace `shopmicro`

## 10. Incidencias encontradas en el despliegue Kubernetes

### 10.1. Kubernetes no estaba habilitado inicialmente

Al principio `kubectl` no tenía contexto configurado y no existía clúster activo. Se resolvió instalando y arrancando Minikube.

### 10.2. Los NodePort no eran accesibles directamente desde el host

Con Minikube sobre Docker y Windows, el acceso a `192.168.49.2:30080` y `192.168.49.2:30081` no respondió directamente desde el host.

Solución aplicada:

- uso de `kubectl port-forward` para exponer temporalmente los servicios localmente

Comandos utilizados:

```powershell
kubectl port-forward svc/frontend 18080:80 -n shopmicro
kubectl port-forward svc/api-gateway 18081:80 -n shopmicro
```

En esta sesión se lanzaron ambos accesos en segundo plano para poder seguir validando la plataforma.

## 11. Acceso funcional conseguido

Se validaron los siguientes accesos:

- Frontend Kubernetes: `http://127.0.0.1:18080`
- API Gateway Kubernetes: `http://127.0.0.1:18081/health`

Resultado observado:

- `frontend` respondió con HTTP 200
- `api-gateway` respondió `ok`

## 12. Validación de flujos funcionales en Kubernetes

### 12.1. Consulta de productos

Se probó:

```powershell
Invoke-RestMethod -Uri 'http://127.0.0.1:18081/api/products' | ConvertTo-Json -Depth 6
```

Resultado:

- la API devolvió correctamente el catálogo
- el origen de la respuesta fue `database`
- el stock del producto 1 figuraba inicialmente en `12`

### 12.2. Creación de pedido

Se ejecutó:

```powershell
$body = @{ product_id = 1; quantity = 1; user_id = 123456 } | ConvertTo-Json -Compress
Invoke-RestMethod -Uri 'http://127.0.0.1:18081/api/orders' -Method Post -ContentType 'application/json' -Body $body | ConvertTo-Json -Depth 6
```

Resultado:

- pedido creado correctamente
- `order_id = 1`
- `status = CREATED`

### 12.3. Consumo de notificación

Se revisó el log del servicio:

```powershell
kubectl logs deployment/notification-service -n shopmicro --tail=30
```

Resultado observado:

```text
Notification sent for order 1
```

Esto demuestra que:

- `order-service` publicó el evento
- `message-queue` lo recibió
- `notification-service` consumió el mensaje correctamente

### 12.4. Actualización de stock

Tras el pedido, se volvió a consultar el catálogo y se comprobó que el producto 1 había pasado de `stock 12` a `stock 11`.

## 13. Estado actual de la Fase 4

A fecha de esta memoria, la fase 4 ya no está únicamente preparada en YAML, sino parcialmente ejecutada y verificada.

Trabajo completado:

- instalación de Minikube
- arranque del clúster local
- aplicación de manifiestos
- verificación de pods y servicios
- comprobación de `kubectl describe deployment product-service`
- prueba de consulta de productos
- prueba de creación de pedido
- prueba de consumo por RabbitMQ

Trabajo pendiente:

- capturas de pantalla numeradas para el documento final
- rolling update real de `product-service`
- documentación del proceso de actualización de imagen
- comparación final más extensa entre Swarm y Kubernetes

## 14. Archivos clave utilizados en esta fase

- [k8s/infra/namespace.yaml](/C:/Users/G513/OneDrive%20-%20Sa%20Palomera/Documentos/Codex/shopmicro/k8s/infra/namespace.yaml)
- [k8s/infra/db-products.yaml](/C:/Users/G513/OneDrive%20-%20Sa%20Palomera/Documentos/Codex/shopmicro/k8s/infra/db-products.yaml)
- [k8s/infra/db-orders.yaml](/C:/Users/G513/OneDrive%20-%20Sa%20Palomera/Documentos/Codex/shopmicro/k8s/infra/db-orders.yaml)
- [k8s/infra/redis.yaml](/C:/Users/G513/OneDrive%20-%20Sa%20Palomera/Documentos/Codex/shopmicro/k8s/infra/redis.yaml)
- [k8s/infra/rabbitmq.yaml](/C:/Users/G513/OneDrive%20-%20Sa%20Palomera/Documentos/Codex/shopmicro/k8s/infra/rabbitmq.yaml)
- [k8s/base/frontend.yaml](/C:/Users/G513/OneDrive%20-%20Sa%20Palomera/Documentos/Codex/shopmicro/k8s/base/frontend.yaml)
- [k8s/base/api-gateway.yaml](/C:/Users/G513/OneDrive%20-%20Sa%20Palomera/Documentos/Codex/shopmicro/k8s/base/api-gateway.yaml)
- [k8s/base/product-service.yaml](/C:/Users/G513/OneDrive%20-%20Sa%20Palomera/Documentos/Codex/shopmicro/k8s/base/product-service.yaml)
- [k8s/base/order-service.yaml](/C:/Users/G513/OneDrive%20-%20Sa%20Palomera/Documentos/Codex/shopmicro/k8s/base/order-service.yaml)
- [k8s/base/user-service.yaml](/C:/Users/G513/OneDrive%20-%20Sa%20Palomera/Documentos/Codex/shopmicro/k8s/base/user-service.yaml)
- [k8s/base/notification-service.yaml](/C:/Users/G513/OneDrive%20-%20Sa%20Palomera/Documentos/Codex/shopmicro/k8s/base/notification-service.yaml)

