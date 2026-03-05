# Infrastructure — Verificación rápida

Instrucciones mínimas para que el profesor verifique el despliegue y ArgoCD.

Comprobaciones rápidas (ejecutar una a una):

```
# Ver estado de la aplicación en ArgoCD
kubectl get application productapi -n argocd -o jsonpath='{.status.sync.status} {.status.health.status}'

# Ver pods en namespace productapi
kubectl get pods -n productapi -o wide

# Forzar refresh si ArgoCD no detecta cambios
kubectl patch application productapi -n argocd -p '{"metadata":{"annotations":{"argocd.argoproj.io/refresh":"hard"}}}' --type=merge
```

Acceso a ArgoCD UI (si aplica):

```
# Obtener URL del servicio argocd-server
kubectl -n argocd get svc argocd-server

# Obtener contraseña admin desde el cluster (no dejarla en el README)
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath='{.data.password}' | base64 --decode
```

Imagen desplegada (ACR):

```
productapiacrmpn.azurecr.io/productapi:0b09ff4
```

Eso es todo lo necesario para la verificación funcional.
