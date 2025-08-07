# Database Tools Infrastructure

Este directorio contiene las herramientas de administración de bases de datos para AthleFi.

## Componentes

### pgAdmin
Interfaz web para administración de bases de datos PostgreSQL.

## Estructura de Directorios

```
database-tools/
├── base/                           # Configuración base
│   ├── pgadmin-deployment.yaml     # Deployment de pgAdmin
│   ├── pgadmin-service.yaml        # Service de pgAdmin
│   ├── pgadmin-configmap.yaml      # ConfigMap para configuración
│   ├── pgadmin-secret.yaml         # Secret para credenciales
│   ├── pgadmin-servers-config.yaml # Configuración de servidores DB
│   └── kustomization.yaml          # Configuración de Kustomize
└── overlays/                       # Configuraciones específicas por ambiente
    ├── dev/                        # Configuración para desarrollo
    ├── staging/                    # Configuración para staging
    └── prod/                       # Configuración para producción
```

## pgAdmin

### Características
- **Imagen**: `dpage/pgadmin4:8.5`
- **Puerto**: 80 (HTTP)
- **Modo**: Server Mode (sin autenticación de master password)
- **Recursos**: 128Mi-256Mi RAM, 50m-100m CPU

### Configuración de Seguridad
- **Usuario**: UID 5050 (usuario no privilegiado)
- **Capabilities**: Drop ALL
- **Seccomp**: RuntimeDefault
- **ReadOnly RootFS**: false (necesario para pgAdmin)

### Variables de Entorno
- `PGADMIN_DEFAULT_EMAIL`: Email de administrador
- `PGADMIN_DEFAULT_PASSWORD`: Contraseña de administrador
- `PGADMIN_CONFIG_SERVER_MODE`: True (modo servidor)
- `PGADMIN_CONFIG_MASTER_PASSWORD_REQUIRED`: False
- `PGADMIN_CONFIG_LOGIN_BANNER`: Banner personalizado
- `PGADMIN_CONFIG_CONSOLE_LOG_LEVEL`: 10 (debug)

### Servidores Pre-configurados
- **AthleFi Athlete Database**: `athlete-db:5432`
  - Base de datos: `athlete_db`
  - Usuario: `postgres`
  - SSL: prefer

### Health Checks
- **Readiness**: `/misc/ping` cada 10s
- **Liveness**: `/misc/ping` cada 30s
- **Startup**: `/misc/ping` cada 5s

## Despliegue

### Desarrollo
```bash
kubectl apply -k infrastructure/database-tools/overlays/dev
```

### Staging
```bash
kubectl apply -k infrastructure/database-tools/overlays/staging
```

### Producción
```bash
kubectl apply -k infrastructure/database-tools/overlays/prod
```

## Acceso

### Interno (Cluster)
```bash
# Port-forward para acceso local
kubectl port-forward svc/pgadmin 8080:80 -n athlefi-dev
```

### Externo (Ingress)
Para acceso externo, agregar un Ingress en los overlays correspondientes.

## Credenciales por Defecto

**⚠️ IMPORTANTE**: Cambiar estas credenciales en producción.

- **Email**: `admin@athlefi.local`
- **Contraseña**: `admin123`

## Configuración de Servidores

Los servidores de base de datos se configuran automáticamente a través del ConfigMap `pgadmin-servers-config`. Para agregar nuevos servidores:

1. Editar `pgadmin-servers-config.yaml`
2. Agregar nueva entrada en `servers.json`
3. Aplicar la configuración

## Monitoreo

### Logs
```bash
kubectl logs -f deployment/pgadmin -n athlefi-dev
```

### Estado del Pod
```bash
kubectl get pods -l app.kubernetes.io/name=pgadmin -n athlefi-dev
```

## Seguridad

### Recomendaciones
1. **Cambiar credenciales por defecto**
2. **Usar Ingress con TLS** para acceso externo
3. **Configurar Network Policies** para restringir acceso
4. **Auditar logs** regularmente
5. **Rotar credenciales** periódicamente

### Network Policies
Considerar agregar Network Policies para restringir el acceso solo a usuarios autorizados.

## Troubleshooting

### pgAdmin no inicia
```bash
kubectl describe pod -l app.kubernetes.io/name=pgadmin -n athlefi-dev
```

### No puede conectar a la base de datos
1. Verificar que `athlete-db` esté corriendo
2. Verificar conectividad de red
3. Verificar credenciales en `pgadmin-servers-config` 