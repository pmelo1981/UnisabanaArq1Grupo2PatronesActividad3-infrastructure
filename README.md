# Infraestructura — Guía de verificación

Comprobaciones mínimas
```bash
# Estado de la aplicación en ArgoCD
kubectl get application productapi -n argocd -o jsonpath='{.status.sync.status} {.status.health.status}'

# Ver pods en el namespace productapi
kubectl get pods -n productapi -o wide

# Forzar re-evaluación del repositorio en ArgoCD
kubectl patch application productapi -n argocd -p '{"metadata":{"annotations":{"argocd.argoproj.io/refresh":"hard"}}}' --type=merge
```

Acceso a ArgoCD (ejemplo):
```
http://172.169.162.125
```

Credenciales (admin) — confirmar en entorno por seguridad
```
Usuario: admin
Contraseña: <ver infra docs>
```

Imagen desplegada (según `helm/values-acr.yaml`)
```
productapiacrmpn.azurecr.io/productapi:8e69a02dc456a0b837aa6e7ba33330babe1f5c21
```

**Última actualizacion:** 07/03/2026  
**URL de Ingress (IP):** http://172.168.96.52
