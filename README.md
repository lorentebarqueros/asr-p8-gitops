# asr-p8-gitops

Repositorio GitOps de la entrega 2 de la práctica 8 de ASR (máster ICAI): la app
Flask + Redis de la práctica 07, desplegada en GKE por **ArgoCD** a partir de este
repositorio.

## Estructura

| Ruta | Qué es |
| --- | --- |
| `k8s/` | Manifiestos de la app. ArgoCD sincroniza **solo** esta carpeta |
| `k8s/app-deployment.yaml` | App Flask (`replicas: 2`, imagen en Artifact Registry) |
| `k8s/app-service.yaml` | `LoadBalancer` restringido a una IP con `loadBalancerSourceRanges` |
| `k8s/redis-deployment.yaml`, `k8s/redis-service.yaml` | Redis, solo accesible dentro del clúster (`ClusterIP`) |
| `argocd/application.yaml` | La `Application` de ArgoCD que apunta a `k8s/` |
| `argocd/argocd-server-lb-patch.yaml` | Parche que expone la UI de ArgoCD solo a mi IP y a las IPs de los webhooks de GitHub |

## Cómo se despliega

1. ArgoCD v3.6.1 instalado en el namespace `argocd` del clúster.
2. Arranque, una sola vez: `kubectl apply -f argocd/application.yaml`.
3. Desde ahí, cualquier cambio en la app se hace con `git push` a `main`.
   Un webhook de GitHub avisa a ArgoCD en el momento, y `selfHeal` deshace los cambios
   hechos a mano en el clúster.

## Seguridad

- Los nodos del clúster no tienen IP pública: ArgoCD llega a GitHub a través de Cloud NAT.
- Los dos `LoadBalancer` (la app y la UI de ArgoCD) solo aceptan orígenes de confianza.
- En este repositorio no hay secretos: la contraseña de ArgoCD y el secreto del webhook
  viven solo en el clúster.
