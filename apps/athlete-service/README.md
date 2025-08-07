# Athlete Service

Este directorio contiene la configuración de Kubernetes para el servicio de atletas de AthleFi.

## Estructura de Directorios

```
athlete-service/
├── base/                           # Configuración base
│   ├── athlete-api-deployment.yaml # Deployment de la API
│   ├── athlete-api-service.yaml    # Service de la API
│   ├── athlete-db-configmap.yaml   # ConfigMap para configuración de DB
│   ├── athlete-db-secret.yaml      # Secret para credenciales de DB
│   ├── athlete-db-postgres-config.yaml # ConfigMap para configuración de PostgreSQL
│   ├── athlete-db-statefulset.yaml # StatefulSet de la base de datos
│   ├── athlete-db-service.yaml     # Service de la base de datos
│   └── kustomization.yaml          # Configuración de Kustomize
└── overlays/                       # Configuraciones específicas por ambiente
    ├── dev/                        # Configuración para desarrollo
    ├── staging/                    # Configuración para staging
    └── prod/                       # Configuración para producción
```

## Convenciones de Nombres

### Archivos
- **Formato**: `{component}-{resource-type}.yaml`
- **Ejemplos**:
  - `athlete-api-deployment.yaml`
  - `athlete-db-service.yaml`
  - `athlete-db-configmap.yaml`

### Recursos Kubernetes

#### Nombres de Recursos
- **API**: `athlete-api`
- **Base de Datos**: `athlete-db`
- **ConfigMaps**: `athlete-db-config`, `athlete-db-postgres-config`
- **Secrets**: `athlete-db-secret`

#### Labels Estándar
Todos los recursos incluyen las siguientes labels estándar de Kubernetes:

```yaml
labels:
  app.kubernetes.io/name: {resource-name}
  app.kubernetes.io/instance: athlete-service
  app.kubernetes.io/version: {version}
  app.kubernetes.io/component: {api|database}
  app.kubernetes.io/part-of: athlefi
  app.kubernetes.io/managed-by: kustomize
```

#### Selectors
Los selectors utilizan las labels estándar:
```yaml
selector:
  app.kubernetes.io/name: {resource-name}
  app.kubernetes.io/instance: athlete-service
```

### Puertos
- **API**: Puerto 8000 (HTTP)
- **Base de Datos**: Puerto 5432 (PostgreSQL)

## Componentes

### API Service
- **Deployment**: `athlete-api`
- **Service**: `athlete-api`
- **Imagen**: `registry.sparkfly.cloud/athlefi/athlete-api:v11`
- **Puerto**: 8000

### Database Service
- **StatefulSet**: `athlete-db`
- **Service**: `athlete-db`
- **Imagen**: `postgres:17`
- **Puerto**: 5432
- **Storage**: 1Gi PersistentVolumeClaim

## Configuración

### Variables de Entorno
La API utiliza las siguientes variables de entorno desde ConfigMaps y Secrets:

- `DB_HOST`: Host de la base de datos
- `DB_PORT`: Puerto de la base de datos
- `DB_NAME`: Nombre de la base de datos
- `DB_USER`: Usuario de la base de datos
- `DB_PASSWORD`: Contraseña de la base de datos (desde Secret)

### Health Checks
- **API**: Endpoint `/health` en puerto 8000
- **Database**: Comando `pg_isready` para verificar conectividad

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