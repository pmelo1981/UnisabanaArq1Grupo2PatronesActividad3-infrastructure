# Infrastructure Repository - GitOps Central

**Repositorio para la gestión centralizada de despliegues Kubernetes usando ArgoCD y GitOps.**

---

## Estructura

```
argocd/
??? namespace.yaml                    # Crea namespace argocd
??? applications/
?   ??? productapi.yaml               # Application CRD (sincroniza ProductAPI repo)
README.md                              # Este archivo
.gitignore
```

---

## GitOps Workflow

```
ProductAPI Repo (codigo)
    |
    v
GitHub Actions: Build -> Test -> Docker Push -> Update values-acr.yaml
    |
    v
ArgoCD detecta cambio (polling cada 3 min)
    |
    v
Sincroniza helm/ -> Kubernetes
    |
    v
Deployment actualizado automaticamente en AKS (exitoso)
```

---

## Requisitos Previos

Antes de usar este repo, asegúrate de tener:

```powershell
# 1. Verificar kubectl instalado
kubectl version --client

# 2. Verificar Azure CLI
az --version

# 3. Descargar credenciales del cluster AKS
az aks get-credentials --resource-group productapi-rg --name productapi-aks
```

Si todo está ok, deberías ver:
```
Merged "productapi-aks" as current context in C:\Users\<user>\.kube\config
```

---

## Instalacion

### Paso 1: Crear namespace de ArgoCD

```powershell
kubectl apply -f argocd/namespace.yaml
```

**Salida esperada:**
```
namespace/argocd created
```

### Paso 2: Instalar ArgoCD (desde manifests oficiales)

```powershell
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Espera ~2 minutos a que se levante. Verifica con:

```powershell
kubectl -n argocd get pods
# Debería ver pods en Running
```

### Paso 3: Aplicar Application manifest (ProductAPI)

```powershell
kubectl apply -f argocd/applications/productapi.yaml
```

**Salida esperada:**
```
application.argoproj.io/productapi created
```

### Paso 4: Verificar que está sincronizado

```powershell
kubectl get application -n argocd

# Output:
# NAME         SYNC STATUS   HEALTH STATUS
# productapi   Synced        Healthy
```

---

## Acceso a ArgoCD UI

### Opción 1: Port-Forward (más simple)

```powershell
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Luego abre en tu navegador:
```
https://localhost:8080
```

### Opción 2: LoadBalancer (acceso desde cualquier lugar)

```powershell
kubectl patch svc argocd-server -n argocd -p '{"spec":{"type":"LoadBalancer"}}'

# Espera ~1 min y obtén la IP
kubectl get svc argocd-server -n argocd

# Busca la columna EXTERNAL-IP
```

Luego accede a:
```
https://<EXTERNAL-IP>
```

---

## Credenciales de ArgoCD

### Usuario
```
admin
```

### Contraseña

**Opción 1: Desde terminal (recomendado)**

```powershell
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
```

**Copia la contraseña que aparece y úsala en la UI.**

### Cambiar contraseña (opcional)

Una vez en la UI:
1. Click en el usuario (arriba a la derecha)
2. "Account"
3. "Change password"

---

## Monitoreo

### Ver todas las aplicaciones sincronizadas

```powershell
kubectl get applications -n argocd
```

### Ver detalles de ProductAPI

```powershell
kubectl get application productapi -n argocd -o yaml
```

### Ver status de sincronización

```powershell
kubectl describe application productapi -n argocd
```

### Ver pods de ProductAPI

```powershell
kubectl -n productapi get pods
```

### Ver imagen que está corriendo

```powershell
kubectl -n productapi describe pods -l app=productapi | findstr "Image:"
```

### Ver logs de un pod

```powershell
kubectl -n productapi logs -f deployment/productapi-productapi
```

---

## Testear Despliegue

### Health check

```powershell
# Obtén la IP del Ingress
kubectl get ingress -n productapi

# Test del endpoint
curl http://<INGRESS_IP>/api/products/health
```

### Ver todos los recursos desplegados

```powershell
kubectl -n productapi get all
```

---

## Troubleshooting

| Problema | Comando para diagnosticar | Solución |
|----------|----------|----------|
| ArgoCD pods no arrancan | kubectl -n argocd get pods | Espera 2-3 min, luego kubectl -n argocd logs <pod> |
| ProductAPI OutOfSync | kubectl describe application productapi -n argocd | kubectl patch application productapi -n argocd -p '{"spec":{"syncPolicy":{}}}' --type merge |
| Pod en CrashLoopBackOff | kubectl -n productapi logs <pod> | Verificar configuración o imagen |
| No puedo acceder a ArgoCD UI | kubectl get svc argocd-server -n argocd | Usa port-forward o espera IP de LoadBalancer |
| Cambios no se sincronizan | Verificar si ArgoCD watching el repo | Espera 3 min (polling) o fuerza sync |

---

## Comandos kubectl más útiles

```powershell
# NAMESPACES
kubectl get namespaces
kubectl get pods -n <namespace>

# DEPLOYMENTS
kubectl get deployments -n productapi
kubectl describe deployment productapi-productapi -n productapi
kubectl logs deployment/productapi-productapi -n productapi

# SERVICES
kubectl get svc -n productapi
kubectl describe svc productapi -n productapi

# INGRESS
kubectl get ingress -n productapi
kubectl describe ingress productapi -n productapi

# EVENTOS
kubectl get events -n productapi --sort-by='.lastTimestamp'

# EXEC EN POD
kubectl exec -it <pod-name> -n productapi -- bash
kubectl exec -it <pod-name> -n productapi -- sh

# LOGS
kubectl logs <pod> -n productapi
kubectl logs <pod> -n productapi -f              # Seguir en tiempo real
kubectl logs <pod> -n productapi --tail=50      # Ultimas 50 líneas
```

---

## Costos - Azure for Students

### Estimacion Mensual

| Recurso | Costo/Mes | Notas |
|---------|-----------|-------|
| AKS Cluster | ~$36 | 1 node Standard_B2s |
| ACR Basic | ~$5 | Almacenamiento de images Docker |
| Load Balancer (Ingress) | ~$3 | 1 Public IP para NGINX |
| Storage | $0 | En-memory (sin persistencia) |
| Network Transfer | ~$0-1 | Egress minimo |
| TOTAL ESTIMADO | ~$40-50/mes | CON Azure for Students: FREE |

### Costo Total del Proyecto (Estimado)

Asumiendo 3 meses de desarrollo:

```
Escenario 1: SIN Azure for Students
  AKS: $36 x 3 = $108
  ACR: $5 x 3 = $15
  Ingress: $3 x 3 = $9
  ??????????????????
  TOTAL: $132

Escenario 2: CON Azure for Students
  ??????????????????
  TOTAL: $0 (GRATIS)
```

---

## IMPORTANTE: Limpieza al Finalizar

**Cuando termines el assignment**, elimina TODOS los recursos para evitar cargos:

```powershell
az group delete --name productapi-rg --yes
```

**Esto borra:**
- AKS Cluster
- ACR (Container Registry)
- Load Balancer
- Virtual Networks
- Storage
- TODO

**ADVERTENCIA: Si no lo haces, seguirá cobrando ~$40/mes**

---

## Referencias

- ProductAPI Repo: https://github.com/pmelo1981/UnisabanaArq1Grupo2PatronesActividad3-productapi
- Personal Repo: https://github.com/pmelo1981/UnisabanaArq1Grupo2PatronesActividad3
- ArgoCD Official Docs: https://argo-cd.readthedocs.io/
- Kubernetes Docs: https://kubernetes.io/docs/
- AKS Best Practices: https://learn.microsoft.com/en-us/azure/aks/
- GitOps Best Practices: https://www.weave.works/technologies/gitops/

---

**Estado:** Produccion-Ready  
**Ultima actualizacion:** 2024  
**Autor:** Universidad de Sabana - Arquitectura de Software
