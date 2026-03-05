# Infraestructura — Guía de verificación

Guía técnica con los comandos mínimos para validar el estado de ArgoCD y el despliegue de la aplicación.

Comprobaciones mínimas (ejecutar una a una)
```bash
# Estado de la aplicación en ArgoCD
kubectl get application productapi -n argocd -o jsonpath='{.status.sync.status} {.status.health.status}'

# Ver pods en el namespace productapi
kubectl get pods -n productapi -o wide

# Forzar re-evaluación del repositorio en ArgoCD (refresh)
kubectl patch application productapi -n argocd -p '{"metadata":{"annotations":{"argocd.argoproj.io/refresh":"hard"}}}' --type=merge
```

Acceso a ArgoCD (opcional)
```bash
# Obtener URL del servicio argocd-server
kubectl -n argocd get svc argocd-server

# Obtener contraseña admin desde el secret en el cluster (no incluir el secreto en el repositorio)
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath='{.data.password}' | base64 --decode
```

Imagen desplegada (ACR)
```text
productapiacrmpn.azurecr.io/productapi:0b09ff4
```

Notas
- El objetivo de este documento es proporcionar comprobaciones rápidas; para procedimientos completos consulte los scripts y documentación en `azure/`.

