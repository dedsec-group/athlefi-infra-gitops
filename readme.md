# Athlefi GitOps Infrastructure

Este repositorio contiene la configuración de GitOps para la infraestructura de Athlefi, utilizando ArgoCD para el despliegue continuo de aplicaciones en Kubernetes.

## 🏗️ Arquitectura

El proyecto sigue una estructura GitOps con ArgoCD y Kustomize para gestionar múltiples entornos:

```
krakend-gateway/
├── apps/                    # Aplicaciones
│   ├── athlete-api/         # API de atletas
│   └── krakend-gateway/     # Gateway API
├── environments/            # Configuración de entornos
│   ├── dev/
│   ├── staging/
│   └── prod/
├── namespaces/              # Definición de namespaces
└── infrastructure/          # Recursos de infraestructura
```

## 🚀 Aplicaciones

### Athlete API
- **Descripción**: API REST para gestión de atletas
- **Componentes**:
  - Base de datos PostgreSQL
  - API REST
  - ConfigMaps y Secrets
  - Persistent Volume Claims

### Krakend Gateway
- **Descripción**: Gateway API para enrutamiento y agregación de servicios
- **Componentes**:
  - Gateway API
  - Configuración de rutas
  - Ingress para acceso externo

## 🌍 Entornos

### Desarrollo (dev)
- **Namespace**: `athlefi-dev`
- **Configuración**: Automatizada con ArgoCD
- **Sincronización**: Auto-healing y pruning habilitado

### Staging
- **Namespace**: `athlefi-staging`
- **Configuración**: Pendiente de implementación

### Producción (prod)
- **Namespace**: `athlefi-prod`
- **Configuración**: Pendiente de implementación

## 🛠️ Tecnologías

- **ArgoCD**: GitOps continuous delivery
- **Kustomize**: Gestión de configuración de Kubernetes
- **Kubernetes**: Orquestación de contenedores
- **PostgreSQL**: Base de datos
- **Krakend**: API Gateway

## 📋 Prerrequisitos

- Cluster de Kubernetes
- ArgoCD instalado y configurado
- Acceso al repositorio de Git
- kubectl configurado

## 🚀 Despliegue

### 1. Configurar ArgoCD

Asegúrate de que ArgoCD esté instalado en tu cluster:

```bash
# Verificar que ArgoCD esté corriendo
kubectl get pods -n argocd
```

### 2. Aplicar Namespaces

```bash
kubectl apply -f namespaces/
```

### 3. Desplegar Aplicaciones

Para el entorno de desarrollo:

```bash
kubectl apply -f environments/dev/
```

### 4. Verificar Despliegue

```bash
# Verificar aplicaciones en ArgoCD
kubectl get applications -n argocd

# Verificar recursos en el namespace
kubectl get all -n athlefi-dev
```

## 🔧 Configuración

### Variables de Entorno

Las configuraciones específicas por entorno se manejan a través de Kustomize overlays:

- **Base**: Configuración común para todos los entornos
- **Overlays**: Configuración específica por entorno (dev, staging, prod)

### Secrets y ConfigMaps

Los secrets y configmaps se definen en los archivos base de cada aplicación y pueden ser sobrescritos en los overlays según sea necesario.

## 📊 Monitoreo

### ArgoCD Dashboard

Accede al dashboard de ArgoCD para monitorear el estado de las aplicaciones:

```bash
# Port-forward del servicio ArgoCD
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Navega a `https://localhost:8080`

### Logs y Debugging

```bash
# Ver logs de las aplicaciones
kubectl logs -f deployment/athlete-api -n athlefi-dev
kubectl logs -f deployment/krakend-gateway -n athlefi-dev

# Verificar estado de los pods
kubectl get pods -n athlefi-dev
kubectl describe pod <pod-name> -n athlefi-dev
```

## 🔄 Flujo de Trabajo GitOps

1. **Desarrollo**: Los desarrolladores trabajan en sus ramas
2. **Merge**: Los cambios se fusionan a la rama principal
3. **ArgoCD**: Detecta automáticamente los cambios
4. **Despliegue**: ArgoCD aplica los cambios al cluster
5. **Verificación**: ArgoCD monitorea el estado de las aplicaciones

## 🚨 Troubleshooting

### Problemas Comunes

1. **Aplicación no sincroniza**:
   ```bash
   kubectl describe application <app-name> -n argocd
   ```

2. **Pods no inician**:
   ```bash
   kubectl describe pod <pod-name> -n athlefi-dev
   kubectl get events -n athlefi-dev
   ```

3. **Problemas de configuración**:
   ```bash
   kubectl get configmap -n athlefi-dev
   kubectl get secret -n athlefi-dev
   ```

### Comandos Útiles

```bash
# Forzar sincronización manual
kubectl patch application <app-name> -n argocd --type='merge' -p='{"spec":{"syncPolicy":{"automated":{"prune":true,"selfHeal":true}}}}'

# Verificar estado de sincronización
kubectl get applications -n argocd -o wide

# Limpiar recursos huérfanos
kubectl delete application <app-name> -n argocd --cascade=false
```

## 📝 Contribución

1. Fork el repositorio
2. Crea una rama para tu feature (`git checkout -b feature/nueva-funcionalidad`)
3. Commit tus cambios (`git commit -am 'Agregar nueva funcionalidad'`)
4. Push a la rama (`git push origin feature/nueva-funcionalidad`)
5. Crea un Pull Request

## 📄 Licencia

Este proyecto está bajo la licencia [MIT](LICENSE).

## 👥 Equipo

- **Desarrollado por**: DedSec Group
- **Repositorio**: https://github.com/dedsec-group/athlefi-infra-gitops

## 📞 Soporte

Para soporte técnico o preguntas sobre la infraestructura, contacta al equipo de DevOps o crea un issue en este repositorio.
