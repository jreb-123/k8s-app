# k8s-app
Repositorio de práctica para Kubernetes + Helm + ArgoCD.

## Requisitos
- Docker 24+
- Kubernetes (kubectl) configurado contra un clúster
- Helm 3+
- Opcional: Argo CD CLI (`argocd`)
- Cuenta en Docker Hub (para publicar la imagen desde CI)

## Estructura
- `app/`: contenido estático de la app (sirve NGINX)
- `docker/Dockerfile`: imagen de la app
- `charts/app`: chart de Helm (Deployment + Service)
- `manifests/`: manifiestos YAML simples (sin Helm)
- `.github/workflows/ci.yaml`: pipeline de build & push a Docker Hub

## Variables de entorno y secretos
La app en sí no requiere variables en tiempo de ejecución (sirve contenido estático con NGINX). Sin embargo, el pipeline CI necesita credenciales para publicar la imagen Docker.

- GitHub Actions Secrets requeridos:
  - `DOCKERHUB_USERNAME`: usuario de Docker Hub
  - `DOCKERHUB_TOKEN`: token o PAT con permiso de `write:packages`/push

Opcionales para despliegue (si deseas sobreescribir valores por entorno):
- Helm values (por entorno), por ejemplo `values-prod.yaml`:
  - `image.repository`: repositorio de la imagen Docker (ej. `usuario/k8s-app`)
  - `image.tag`: etiqueta de imagen (ej. `latest` o un SHA)
  - `service.type`, `service.port`, `service.nodePort`
  - `replicaCount`

## Construir y probar localmente
1. Construir la imagen Docker:
   ```bash
   docker build -f docker/Dockerfile -t k8s-app:local .
   ```
2. Ejecutar el contenedor:
   ```bash
   docker run --rm -p 8080:80 k8s-app:local
   ```
   Abre `http://localhost:8080`.

## Despliegue con kubectl (manifests simples)
```bash
kubectl apply -f manifests/deployment.yaml
kubectl apply -f manifests/service.yaml
```

## Despliegue con Helm
1. Ajusta `charts/app/values.yaml` o crea un values específico de entorno (ej. `values-prod.yaml`).
2. Instala/actualiza el release:
   ```bash
   helm upgrade --install k8s-app charts/app \
     --set image.repository=${DOCKERHUB_USERNAME:-usuario}/k8s-app \
     --set image.tag=latest
   ```
   o con un values file:
   ```bash
   helm upgrade --install k8s-app charts/app -f values-prod.yaml
   ```

## Flujo de CI (GitHub Actions)
- Trigger: `push` a cualquier rama.
- Pasos:
  - Checkout del repo
  - Setup de Buildx
  - Login a Docker Hub usando `DOCKERHUB_USERNAME`/`DOCKERHUB_TOKEN`
  - Build & push con tag `latest` al repositorio `${DOCKERHUB_USERNAME}/k8s-app`

Si quieres usar tags/versiones específicas, puedes extender el workflow para etiquetar con `github.sha` o tags de Git.

## Notas
- El chart por defecto usa `nginx:alpine`. En CI, al publicar `${DOCKERHUB_USERNAME}/k8s-app:latest`, recuerda actualizar `charts/app/values.yaml` o pasar `--set image.repository=...` al instalar con Helm.
- El `Service` por defecto es `NodePort` en `30080`. Ajusta según tu entorno (LoadBalancer en cloud, etc.).

# k8s-app
Cambio mínimo para disparar 

