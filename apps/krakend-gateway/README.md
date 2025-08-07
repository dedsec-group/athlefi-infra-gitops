# Krakend Gateway

Este directorio contiene la configuración de Kubernetes para el gateway de API de AthleFi usando Krakend.

## Estructura de Directorios

```
krakend-gateway/
├── base/                           # Configuración base
│   ├── krakend-deployment.yaml     # Deployment del gateway
│   ├── krakend-service.yaml        # Service del gateway
│   ├── krakend-ingress.yaml        # Ingress para acceso externo
│   └── kustomization.yaml          # Configuración de Kustomize
└── overlays/                       # Configuraciones específicas por ambiente
    ├── dev/                        # Configuración para desarrollo
    │   ├── config/
    │   │   └── krakend.json        # Configuración de Krakend
    │   └── kustomization.yaml
    ├── staging/                    # Configuración para staging
    └── prod/                       # Configuración para producción
```

## Convenciones de Nombres

### Archivos
- **Formato**: `{component}-{resource-type}.yaml`
- **Ejemplos**:
  - `krakend-deployment.yaml`
  - `krakend-service.yaml`
  - `krakend-ingress.yaml`

### Recursos Kubernetes

#### Nombres de Recursos
- **Deployment**: `krakend-gateway`
- **Service**: `krakend-gateway`
- **Ingress**: `krakend-gateway`
- **ConfigMap**: `krakend-gateway-config`

#### Labels Estándar
Todos los recursos incluyen las siguientes labels estándar de Kubernetes:

```yaml
labels:
  app.kubernetes.io/name: krakend-gateway
  app.kubernetes.io/instance: krakend-gateway
  app.kubernetes.io/version: "2.10.2"
  app.kubernetes.io/component: gateway
  app.kubernetes.io/part-of: athlefi
  app.kubernetes.io/managed-by: kustomize
```

#### Selectors
Los selectors utilizan las labels estándar:
```yaml
selector:
  app.kubernetes.io/name: krakend-gateway
  app.kubernetes.io/instance: krakend-gateway
```

### Puertos
- **Gateway**: Puerto 8080 (HTTP interno)
- **Service**: Puerto 80 (HTTP externo)

## Componentes

### Gateway Service
- **Deployment**: `krakend-gateway`
- **Service**: `krakend-gateway`
- **Imagen**: `krakend:2.10.2`
- **Puerto**: 8080
- **Replicas**: 2

### Configuración
- **ConfigMap**: `krakend-gateway-config`
- **Archivo**: `krakend.json` (configurado por ambiente)
- **Mount Path**: `/etc/krakend`

### Ingress
- **Host**: `gateway-dev.athlefi.us-east-1.dedsec.com.mx`
- **TLS**: Habilitado
- **Entrypoint**: websecure

## Health Checks

### Endpoints
- **Readiness**: `/__health` en puerto 8080
- **Liveness**: `/__health` en puerto 8080
- **Startup**: `/__health` en puerto 8080

### Configuración
- **Initial Delay**: 5-30 segundos
- **Period**: 5-10 segundos
- **Timeout**: 3 segundos
- **Failure Threshold**: 3 intentos

## Recursos

### Requests
- **Memory**: 64Mi
- **CPU**: 50m

### Limits
- **Memory**: 128Mi
- **CPU**: 100m

## Despliegue

Para desplegar en un ambiente específico:

```bash
# Desarrollo
kubectl apply -k overlays/dev

# Staging
kubectl apply -k overlays/staging

# Producción
kubectl apply -k overlays/prod
```

## Configuración de Krakend

### Archivo de Configuración
El archivo `krakend.json` se configura por ambiente en:
- `overlays/dev/config/krakend.json`
- `overlays/staging/config/krakend.json`
- `overlays/prod/config/krakend.json`

### Variables de Entorno
- **Configuración**: Montada desde ConfigMap
- **Modo**: Server mode
- **Logging**: Configurado para debugging

## Monitoreo

### Logs
```bash
kubectl logs -f deployment/krakend-gateway -n athlefi-dev
```

### Estado del Pod
```bash
kubectl get pods -l app.kubernetes.io/name=krakend-gateway -n athlefi-dev
```

### Health Check
```bash
kubectl get endpoints krakend-gateway -n athlefi-dev
```

## Troubleshooting

### Gateway no responde
1. Verificar logs del pod
2. Verificar health checks
3. Verificar configuración de Krakend
4. Verificar conectividad de red

### Configuración incorrecta
1. Verificar sintaxis de `krakend.json`
2. Verificar ConfigMap
3. Reiniciar deployment

### Ingress no funciona
1. Verificar hostname
2. Verificar certificados TLS
3. Verificar Traefik configuration 