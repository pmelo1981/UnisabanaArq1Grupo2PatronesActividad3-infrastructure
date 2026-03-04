# 🤖 Copilot Prompt - Infrastructure Repository

## Contexto del Proyecto

Este repositorio contiene la **configuración GitOps central** para desplegar automáticamente aplicaciones Kubernetes usando **ArgoCD**.

## Estructura

```
argocd/
├── namespace.yaml           # Crea el namespace de ArgoCD
└── applications/
    └── productapi.yaml      # Application CRD que sincroniza Repo ProductAPI
```

## Flujo de Trabajo

1. **ProductAPI Repo** (https://github.com/pmelo1981/UnisabanaArq1Grupo2PatronesActividad3-productapi)
   - Contiene: Código fuente, Dockerfile, Helm Chart, CI/CD
   - GitHub Actions: Build → Test → Push a ACR → Update values-acr.yaml → Push

2. **Este Repo (Infrastructure)**
   - ArgoCD Application monitorea ProductAPI repo (rama: main, path: helm/)
   - Auto-sync: Enabled
   - Cuando values-acr.yaml cambia → ArgoCD detecta → Despliega new image

3. **Kubernetes Cluster** (AKS)
   - Namespace: productapi
   - 2-5 Replicas (HPA based on CPU 80%)
   - Service: ClusterIP
   - Ingress: NGINX LoadBalancer

## Tareas Comunes con Copilot

### "Necesito añadir una nueva aplicación"
```bash
# Crea un nuevo Application manifest en argocd/applications/
# Template:
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: mi-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/pmelo1981/mi-repo-app
    path: helm
    targetRevision: main
  destination:
    server: https://kubernetes.default.svc
    namespace: mi-namespace
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

### "Quiero deshabilitar auto-sync temporalmente"
```bash
# En el Application YAML:
syncPolicy:
  automated: null  # O remove la section completa
```

### "Necesito sincronizar manualmente"
```bash
argocd app sync productapi
```

### "Verificar estado del deploy"
```bash
kubectl get application -n argocd -o yaml
argocd app get productapi
```

## Mejores Prácticas

1. **Versionado**: Todos los manifests en Git (no modificar directly en K8s)
2. **Auto-sync**: Enabled para sincronización automática
3. **Health checks**: ArgoCD valida resources status automáticamente
4. **RBAC**: Usar Application namespaces para segregar permisos
5. **Secret management**: Usar Sealed Secrets o External Secrets Operator

## Troubleshooting

| Problema | Solución |
|----------|----------|
| Application no sincroniza | `argocd repo add` con credentials, verificar repo access |
| Pod CrashLoopBackOff | Ver logs en repo ProductAPI (image issue, config) |
| Ingress sin IP | Verificar NGINX controller status, cuota de IPs en Azure |
| ArgoCD no ve cambios | Esperar 3min (default polling interval), o ejecutar `argocd app sync` |

## Contacto

- **ProductAPI Repo**: https://github.com/pmelo1981/UnisabanaArq1Grupo2PatronesActividad3-productapi
- **Personal Repo**: https://github.com/pmelo1981/UnisabanaArq1Grupo2PatronesActividad3
