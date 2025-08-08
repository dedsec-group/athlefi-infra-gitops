# Certificates Infrastructure

Este directorio contiene la configuración de certificados TLS para AthleFi usando cert-manager y Let's Encrypt.

## Estructura de Directorios

```
certificates/
├── base/                           # Configuración base
│   ├── cluster-issuer.yaml         # ClusterIssuer para producción
│   ├── cluster-issuer-staging.yaml # ClusterIssuer para staging
│   ├── cloudflare-secret.yaml      # Secret para Cloudflare API
│   └── kustomization.yaml          # Configuración de Kustomize
└── overlays/                       # Configuraciones específicas por ambiente
    ├── dev/                        # Configuración para desarrollo (staging)
    ├── staging/                    # Configuración para staging
    └── prod/                       # Configuración para producción
```

## Componentes

### ClusterIssuer
Configuración de Let's Encrypt para certificados automáticos.

#### Producción
- **Nombre**: `athlefi-letsencrypt`
- **Servidor**: `https://acme-v02.api.letsencrypt.org/directory`
- **Email**: `admin@athlefi.com`
- **Rate Limits**: 50 certificados por semana
- **Challenges**: HTTP01 + DNS01

#### Staging
- **Nombre**: `athlefi-letsencrypt-staging`
- **Servidor**: `https://acme-staging-v02.api.letsencrypt.org/directory`
- **Email**: `admin@athlefi.com`
- **Rate Limits**: 300 certificados por semana
- **Challenges**: HTTP01 + DNS01

### Estrategia de Nombres

**¿Por qué nombres simples?**
- **Namespace isolation**: Cada namespace puede tener su propio issuer
- **Sin conflictos**: `athlefi-dev` namespace no interfiere con otros
- **Simplicidad**: Nombres más cortos y claros

**Ejemplo de uso:**
```yaml
# En namespace athlefi-dev
cert-manager.io/cluster-issuer: "athlefi-letsencrypt-staging"

# En namespace athlefi-prod  
cert-manager.io/cluster-issuer: "athlefi-letsencrypt"
```

### Solucionadores de Desafío

#### HTTP01 Challenge
```yaml
http01:
  ingress:
    class: traefik
```
- **Uso**: Para certificados individuales
- **Requisito**: Acceso HTTP público
- **Ventaja**: Simple y directo

#### DNS01 Challenge (Cloudflare)
```yaml
dns01:
  cloudflare:
    email: admin@athlefi.com
    apiTokenSecretRef:
      name: cloudflare-api-token
      key: api-token
```
- **Uso**: Para certificados wildcard y clusters sin IP pública
- **Requisito**: Token de API de Cloudflare
- **Ventaja**: Funciona sin IP pública

## Configuración por Ambiente

### Desarrollo
- **Issuer**: `athlefi-letsencrypt-staging`
- **Propósito**: Testing sin rate limits
- **Certificados**: No confiables (para desarrollo)
- **Challenge**: HTTP01 + DNS01

### Producción
- **Issuer**: `athlefi-letsencrypt`
- **Propósito**: Certificados válidos
- **Certificados**: Confiables y válidos
- **Challenge**: HTTP01 + DNS01

## Uso en Ingress

### Configuración Automática
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    cert-manager.io/cluster-issuer: "athlefi-letsencrypt-staging"
spec:
  tls:
    - hosts:
        - myapp.athlefi.internal.mx-central-1.sparkfly.cloud
      secretName: myapp-tls
```

### Certificados Individuales
Cada Ingress puede tener su propio certificado:
- `pgadmin-dev-tls`
- `krakend-gateway-dev-tls`
- `athlete-api-dev-tls`

### Certificados Wildcard
Para certificados wildcard (futuro):
```yaml
spec:
  tls:
    - hosts:
        - "*.athlefi.internal.mx-central-1.sparkfly.cloud"
      secretName: athlefi-wildcard-tls
```

## Configuración de Cloudflare

### 1. Crear API Token
1. Ir a Cloudflare Dashboard
2. My Profile > API Tokens
3. Create Token > Custom token
4. Permissions:
   - Zone:Zone:Edit
   - Zone:DNS:Edit
5. Zone Resources: Include > Specific zone > athlefi.internal.mx-central-1.sparkfly.cloud

### 2. Actualizar Secret
```bash
# Opción 1: Editar el archivo yaml
kubectl edit secret cloudflare-api-token -n cert-manager

# Opción 2: Crear desde línea de comandos
kubectl create secret generic cloudflare-api-token \
  --from-literal=api-token=your-token-here \
  -n cert-manager
```

## Despliegue

### Desarrollo
```bash
kubectl apply -k infrastructure/certificates/overlays/dev
```

### Producción
```bash
kubectl apply -k infrastructure/certificates/overlays/prod
```

## Monitoreo

### Verificar ClusterIssuer
```bash
kubectl get clusterissuer athlefi-letsencrypt-staging -o yaml
```

### Verificar Certificados
```bash
kubectl get certificates -A
kubectl get certificaterequests -A
```

### Verificar Secretos TLS
```bash
kubectl get secrets -l app.kubernetes.io/component=certificate
```

## Troubleshooting

### Certificado no se crea
1. Verificar ClusterIssuer:
   ```bash
   kubectl describe clusterissuer athlefi-letsencrypt-staging
   ```

2. Verificar CertificateRequest:
   ```bash
   kubectl describe certificaterequest -n athlefi-dev
   ```

3. Verificar logs de cert-manager:
   ```bash
   kubectl logs -f deployment/cert-manager -n cert-manager
   ```

### Error de HTTP Challenge
1. Verificar que el Ingress esté configurado correctamente
2. Verificar que el dominio resuelva al cluster
3. Verificar que Traefik esté funcionando

### Error de DNS Challenge
1. Verificar Cloudflare API token:
   ```bash
   kubectl get secret cloudflare-api-token -n cert-manager -o yaml
   ```

2. Verificar permisos del token en Cloudflare
3. Verificar que el dominio esté en Cloudflare

### Rate Limits
- **Staging**: 300 certificados/semana
- **Production**: 50 certificados/semana
- **Solución**: Usar staging para desarrollo

## Seguridad

### Recomendaciones
1. **Usar staging para desarrollo** - Evitar rate limits
2. **Proteger API tokens** - No committear tokens reales
3. **Monitorear certificados** - Verificar renovaciones
4. **Backup de certificados** - Guardar certificados críticos
5. **Rotar tokens** - Cambiar tokens periódicamente

### Manejo de Secretos
- **Template en Git**: `cloudflare-secret.yaml` con placeholder
- **Token real**: Aplicar manualmente o usar Sealed Secrets
- **Seguridad**: Nunca committear tokens reales

## Próximos Pasos

1. **Configurar Cloudflare API token**
2. **Desplegar certificados en desarrollo**
3. **Verificar renovación automática**
4. **Configurar monitoreo de certificados**
5. **Desplegar en producción**
6. **Configurar certificados wildcard (opcional)** 