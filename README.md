# ?? Infrastructure Repository - GitOps Central

**Repositorio para la gestión centralizada de despliegues Kubernetes usando ArgoCD y GitOps.**

---

## ?? Estructura

```
argocd/
??? namespace.yaml                    # Crea namespace argocd
??? applications/
    ??? productapi.yaml               # Application CRD (sincroniza ProductAPI repo)
copilot_prompt.md                      # Guía para Copilot
README.md                              # Este archivo
.gitignore
```

---

## ?? GitOps Workflow

```
ProductAPI Repo (código)
    ?
GitHub Actions: Build ? Test ? Docker Push ? Update values-acr.yaml
    ?
ArgoCD detecta cambio (polling cada 3 min)
    ?
Sincroniza helm/ ? Kubernetes
    ?
Deployment actualizado con new image
```

---

## ?? Costos - Azure for Students

### Estimación Mensual

| Recurso | Costo/Mes | Notas |
|---------|-----------|-------|
| **AKS Cluster** | ~$36 | 1 node Standard_B2s |
| **ACR Basic** | ~$5 | Almacenamiento de images Docker |
| **Load Balancer (Ingress)** | ~$3 | 1 Public IP para NGINX |
| **Storage** | $0 | En-memory (sin base de datos) |
| **Network Transfer** | ~$0-1 | Egress mínimo |
| **TOTAL ESTIMADO** | **~$40-50/mes** | ?? **CON Azure for Students: FREE** |

### Optimizaciones ya Implementadas

? **1 node** (B2s micro - suficiente para microservicio simple)
? **HPA** (2-5 replicas automáticas, no fixed)
? **In-memory storage** (sin persistencia = sin costos)
? **Service ClusterIP** (no IP extra)
? **Ingress** (1 LoadBalancer compartida)

### Costo Total del Proyecto (Estimado)

Asumiendo 3 meses de desarrollo:

```
Escenario 1: SIN Azure for Students
  AKS: $36 × 3 = $108
  ACR: $5 × 3 = $15
  Ingress: $3 × 3 = $9
  ????????????????
  TOTAL: $132

Escenario 2: CON Azure for Students (GRATIS)
  ????????????????
  TOTAL: $0 ?
```

### ?? IMPORTANTE: Limpieza al Finalizar

**Cuando termines el assignment**, elimina TODOS los recursos:

```bash
# Esto borra el cluster + recursos + ingress + storage
az group delete --name productapi-rg --yes
```

**¿Por qué?** Porque si dejas el cluster corriendo, seguirá cobrando (~$40/mes).

---

## ?? Instalación Rápida

### Requisitos

- kubectl configurado
- Helm 3.x
- Git

### Pasos

```bash
# 1. Crear namespace ArgoCD
kubectl apply -f argocd/namespace.yaml

# 2. Instalar ArgoCD desde manifests oficiales
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# 3. Aplicar Application manifests
kubectl apply -f argocd/applications/productapi.yaml

# 4. Verificar estado
kubectl get applications -n argocd
```

---

## ?? Verificación

```bash
# Ver todas las aplicaciones
kubectl get applications -n argocd

# Ver detalles de productapi
kubectl get application productapi -n argocd -o yaml

# Ver status de sincronización
kubectl describe application productapi -n argocd

# Test de API
curl http://INGRESS_IP/api/products
curl http://INGRESS_IP/api/products/health
```

---

## ?? Troubleshooting

| Problema | Causa | Solución |
|----------|-------|----------|
| **OutOfSync** | Git y K8s están desincronizados | `argocd app sync productapi` |
| **CrashLoopBackOff** | Problema en la imagen o config | `kubectl logs -n productapi <pod>` |
| **ArgoCD no accesible** | LoadBalancer pending IP | `kubectl port-forward svc/argocd-server -n argocd 8080:443` |
| **Cambios no se sincronizan** | Espera polling (3 min) o credenciales | `argocd app sync productapi --force` |

---

## ?? Referencias

- **copilot_prompt.md** - Guía detallada con comandos comunes
- **ProductAPI Repo**: https://github.com/pmelo1981/UnisabanaArq1Grupo2PatronesActividad3-productapi
- **Personal Repo**: https://github.com/pmelo1981/UnisabanaArq1Grupo2PatronesActividad3
- [ArgoCD Docs](https://argo-cd.readthedocs.io/)
- [GitOps Best Practices](https://www.weave.works/technologies/gitops/)

---

**Estado:** ? Producción-Ready | **Última actualización:** 2024
