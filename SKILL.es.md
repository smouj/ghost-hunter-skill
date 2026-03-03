name: Ghost Hunter
description: Escáner avanzado de vulnerabilidades de seguridad para código e infraestructura con capacidades de detección furtiva
version: 1.2.0
author: SMOUJBOT Security Team
tags: [security, detective, stealth]
maintainer: security@openclaw.io
category: security
dependencies:
  - trivy>=0.45.0
  - semgrep>=1.50.0
  - grype>=0.70.0
  - bandit>=1.7.5
  - npm-audit>=8.0.0
  - docker
  - kubectl
  - python3>=3.9
  - git
os: [linux, darwin]
license: AGPL-3.0
documentation: https://docs.openclaw.io/skills/ghost-hunter
---

# Ghost Hunter

Detección furtiva de vulnerabilidades y análisis de amenazas ocultas para infraestructura moderna.

## Propósito

Ghost Hunter proporciona detección de seguridad continua sin interrumpir los flujos de desarrollo. A diferencia de los escáneres tradicionales que generan ruido y falsos positivos, Ghost Hunter utiliza conciencia contextual para identificar amenazas reales:

- **Mejora de Revisión de Código**: Escanea automáticamente PRs para OWASP Top 10, patrones CWE y vulnerabilidades de la cadena de suministro antes del merge
- **Seguridad de Infraestructura como Código**: Detecta desconfiguraciones en Terraform, CloudFormation, manifiestos de Kubernetes y Dockerfiles antes del despliegue
- **Refuerzo de Contenedores**: Escanea imágenes de contenedores para CVEs, fugas de secretos y desconfiguraciones en pipelines CI/CD
- **Inteligencia de Amenazas de Dependencias**: Monitorea dependencias para exploits conocidos, paquetes maliciosos y problemas de cumplimiento de licencias
- **Detección de Fugas de Credenciales**: Escanea el historial de Git, logs y artefactos para claves API, tokens, contraseñas y certificados
- **Detección Furtiva en Tiempo de Ejecución**: Monitorea contenedores y VMs en ejecución para rutas de escalada de privilegios y servicios expuestos

Casos de uso reales:
- Escanear 200+ microservicios en 8 minutos con análisis de gráfico de dependencias
- Detectar claves AWS hardcodeadas en legacy codebase a través de 50 repositorios
- Identificar cuentas de servicio de Kubernetes con exceso de privilegios antes del rollout en producción
- Encontrar commits con secretos que evitaron hooks pre-commit
- Auditar la postura de seguridad de Dockerfiles en todas las estaciones de trabajo de desarrolladores

## Alcance

### Comandos Principales

`ghost-hunter scan code [PATH]`
- Análisis de seguridad de código recursivo usando múltiples motores
- Detecta automáticamente el lenguaje y aplica reglas apropiadas
- Soporta: Go, Python, JavaScript/Typecript, Java, Rust, Ruby, PHP

`ghost-hunter scan iac [PATH]`
- Escaneo de seguridad de Infraestructura como Código
- Detecta desconfiguraciones en Terraform, CloudFormation, Ansible, Kubernetes, Docker
- Verifica contra benchmarks CIS y mejores prácticas de la nube

`ghost-hunter scan image [IMAGE_TAG]`
- Escaneo de vulnerabilidades de imágenes de contenedores con análisis profundo
- Inspección capa por capa, escaneo de paquetes del sistema operativo y dependencias de lenguajes
- Generación de SBOM y verificación de procedencia

`ghost-hunter scan git-history [REPO_PATH]`
- Escaneo profundo del historial de Git para secretos y datos sensibles
- Analiza todas las ramas y etiquetas, incluyendo historial reescrito
- Soporta patrones regex personalizados para secretos específicos de la organización

`ghost-hunter scan deps [PROJECT_PATH]`
- Análisis de vulnerabilidades de dependencias y cadena de suministro
- Generación de Bill of Materials (BOM) con enlace a vulnerabilidades
- Detección de paquetes maliciosos usando feeds de inteligencia de amenazas

`ghost-hunter scan runtime [TARGET]`
- Evaluación de entorno en vivo (pods de Kubernetes, contenedores Docker, procesos)
- Validación de configuración contra políticas de seguridad
- Análisis de exposición de red y privilegios

`ghost-hunter detect secrets [PATH]`
- Detección especializada de secretos con 200+ patrones
- Adición de patrones personalizados para formatos propietarios
- Puntuación de confianza y reducción de falsos positivos

### Filtrado y Reportes

`--severity [critical|high|medium|low|info]`
Filtra hallazgos por nivel de severidad

`--exclude [PATH]`
Excluye directorios o patrones de archivos del escaneo

`--format [json|sarif|html|github-actions]`
Selección de formato de salida

`--output [FILE]`
Escribe resultados en archivo en lugar de stdout

`--fail-on [CRITERIA]`
Sale con código no cero si se cumplen criterios (ej., "critical", "high+medium")

`--rule [RULE_ID]`
Ejecuta solo regla específica o identificador CWE

`--tag [TAG]`
Filtra por etiqueta (ej., "owasp-a10", "cwe-89", "sql-injection")

`--context [ENV]`
Añade etiquetas contextuales a hallazgos (ej., production, staging, dev)

`--no-cache`
Deshabilita caché de resultados para escaneo fresco

`--parallel [N]`
Número de workers paralelos (por defecto: cantidad de CPU)

### Opciones de Integración

`--pre-commit`
Salida en formato hook pre-commit (solo stderr, sin JSON)

`--github-actions`
Genera anotaciones de GitHub Actions

`--gitlab-ci`
Generación de artefacto de trabajo de GitLab CI

`--azdo-pipelines`
Comentarios en línea de Azure DevOps

## Proceso de Trabajo

### 1. Evaluación Inicial
```bash
# Reconocimiento rápido: identificar tipo de proyecto y alcance
ghost-hunter detect project-type ./myapp
# Salida: {"type":"nodejs","languages":["javascript","typescript"],"frameworks":["express","react"],"package_manager":"npm"}
```

### 2. Escaneo de Seguridad de Código
```bash
# Escaneo completo del código base con todos los motores
ghost-hunter scan code ./src \
  --severity high \
  --exclude node_modules,build,coverage \
  --format sarif \
  --output codefindings.sarif

# Escaneo paralelo a través de múltiples repositorios
find /repos -name "*.go" -type f | parallel ghost-hunter scan code {} --format json
```

### 3. Escaneo de Infraestructura
```bash
# Evaluación de seguridad de Terraform
ghost-hunter scan iac ./terraform \
  --checkov \
  --tflint \
  --severity critical \
  --context production

# Escaneo de manifiestos de Kubernetes
ghost-hunter scan iac ./k8s/manifests \
  --kubesec \
  --polaris \
  --output k8s-findings.html
```

### 4. Análisis de Imágenes de Contenedor
```bash
# Inspección profunda de contenedor
ghost-hunter scan image myapp:latest \
  --image myregistry.com/myapp:v1.2.3 \
  --scanners trivy,grype \
  --generate-sbom \
  --verify-provenance \
  --output image-report.json

# Escanear todas las imágenes en Docker Compose
docker-compose config --services | while read svc; do
  img=$(docker-compose config --services | grep $svc | xargs docker-compose config | grep image | cut -d' ' -f2)
  ghost-hunter scan image $img --format json
done
```

### 5. Auditoría de Dependencias
```bash
# Escaneo de vulnerabilidades de dependencias NPM
ghost-hunter scan deps ./frontend \
  --audit-level high \
  --check-licenses \
  --detect-malicious \
  --output deps.json

# Python requirements con integración pip-audit
ghost-hunter scan deps ./backend/python \
  --pip-audit \
  --safety \
  --cve-db-update
```

### 6. Detección de Secretos
```bash
# Escaneo comprehensivo de secretos en todo el código base
ghost-hunter detect secrets ./ \
  --patterns ~/.ghost-hunter/patterns/custom.secrets \
  --entropy 4.5 \
  --max-depth 5 \
  --exclude-file .ghost-hunter-ignore \
  --output secrets.json

# Barrido del historial de Git para credenciales filtradas
ghost-hunter scan git-history ./repo \
  --since "1 year ago" \
  --branches all \
  --regex 'AKIA[0-9A-Z]{16}' \
  --regex 'ghp_[0-9a-zA-Z]{36}' \
  --regex 'sk_live_[0-9a-zA-Z]{24}'
```

### 7. Evaluación en Tiempo de Ejecución
```bash
# Escaneo de clúster Kubernetes
ghost-hunter scan runtime k8s://my-cluster \
  --namespace prod \
  --scan-pods \
  --scan-configmaps \
  --scan-serviceaccounts \
  --output runtime-k8s.json

# Inspección de contenedor Docker
ghost-hunter scan runtime docker://container-id \
  --check-privileged \
  --check-capabilities \
  --check-mounts \
  --check-network
```

### 8. Reportes e Integración
```bash
# Generar reporte comprehensivo
ghost-hunter report generate \
  --inputs codefindings.sarif,image-report.json,deps.json \
  --template executive \
  --output security-report.html \
  --send-to slack://#security-alerts

# Crear GitHub Security Advisory desde hallazgos
ghost-hunter advisory create \
  --finding image-report.json \
  --repo myorg/myrepo \
  --severity critical \
  --draft
```

### 9. Monitoreo Continuo
```bash
# Configurar hook pre-commit
ghost-hunter hook install pre-commit \
  --scanners code,secrets \
  --block-on high,critical

# Flujo de trabajo de GitHub Actions
ghost-hunter action workflow \
  --on push,pr \
  --scan code,iac,deps \
  --fail-on critical \
  --comment-on-pr
```

### 10. Búsqueda de Inteligencia de Amenazas
```bash
# Verificar si algún hallazgo coincide con exploits activos
ghost-hunter threat-intel lookup \
  --cve CVE-2024-12345 \
  --vendor apache \
  --product log4j

# Suscribirse a feed de vulnerabilidades
ghost-hunter feed subscribe \
  --feed cve \
  --severity critical \
  --notify slack,email \
  --daily 09:00
```

## Reglas de Oro

1. **Escaneo de Confianza Cero**
   - Nunca confíes en fuentes de dependencias; siempre verifica checksums y firmas
   - Escanea todos los artefactos incluyendo caches de build y archivos temporales
   - Asume que cada dependencia externa está comprometida hasta que se demuestre lo contrario

2. **Principio de Mínimo Privilegio**
   - Escanea con permisos mínimos necesarios; nunca ejecutes como root a menos que sea absolutamente requerido
   - Escaneos de contenedores deben usar montajes de solo lectura para acceso al sistema de archivos del host
   - Escaneos de red deben aislarse usando reglas de egress-only

3. **Operaciones Furtivas**
   - Usa limitación de recursos (`--parallel 1 --throttle 100ms`) en entornos de producción
   - Evita crear picos de carga en runners CI/CD; respeta `--max-cpu` y `--max-memory`
   - Cachea resultados agresivamente para minimizar escaneos repetidos de código sin cambios

4. **Preservación de Evidencia**
   - Siempre guarda salidas de escáneres sin formato junto con resultados parseados
   - Mantén SBOMs completos y datos de procedencia para cada imagen escaneada
   - Mantén logs de escaneo inmutables con timestamps y git SHAs

5. **Eliminación de Falsos Positivos**
   - Solo marca hallazgos como "falso positivo" en triage, nunca los suprimas
   - Usa comparación con línea base (`--baseline previous-scan.json`) para detectar solo nuevos problemas
   - Requiere revisión por pares para cualquier supresión de hallazgos critical/high

6. **Manejo de Secretos**
   - Nunca registres secretos descubiertos en texto plano, incluso en modo debug
   - Rota inmediatamente cualquier credencial viva encontrada en escaneos
   - Usa flag `--mask-secrets` en todos los reportes para redactar valores sensibles

7. **Mapeo de Cumplimiento**
   - Siempre etiqueta hallazgos con frameworks relevantes: PCI-DSS, HIPAA, GDPR, SOC2, ISO27001
   - Genera reportes específicos de cumplimiento (`--compliance pci-dss`) para auditorías
   - Mantiene trail de auditoría de todos los cambios de estado de cumplimiento

8. **Integridad de la Cadena de Suministro**
   - Verifica cada imagen de contenedor con Notary, Cosign o herramienta similar de sigstore
   - Verifica procedencia de dependencias: `ghost-hunter scan deps --verify-attestations`
   - Rechaza dependencias de fuentes de paquetes no mantenidas o sospechosas

9. **Defensa en Profundidad**
   - Ejecuta múltiples escáneres con metodologías de detección diferentes
   - Combina escaneo SAST, SCA, DAST e IaC para cobertura completa
   - Cruza referencias de hallazgos entre herramientas para priorizar amenazas reales

10. **Nunca Interfieras con Producción**
    - Escaneos en tiempo de ejecución son read-only a menos que uses explícitamente `--fix-permissions` o similar
    - Nunca modifiques infraestructura o cambies configuraciones durante el escaneo
    - Aísla contenedores de escaneo de redes de producción usando network policies

## Ejemplos

### Ejemplo 1: Integración de Pipeline CI/CD
```yaml
# .github/workflows/security.yml
name: Security Scan
on: [push, pull_request]
jobs:
  ghost-hunter:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Install Ghost Hunter
        run: |
          curl -sSL https://get.openclaw.io/ghost-hunter | bash
      - name: Code Security Scan
        run: |
          ghost-hunter scan code . \
            --severity high \
            --format sarif \
            --output code.sarif
      - name: Dependency Audit
        run: |
          ghost-hunter scan deps . \
            --audit-level high \
            --format json \
            --output deps.json
      - name: Upload Results
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: code.sarif
```

### Ejemplo 2: Configuración de Hook Pre-commit
```bash
# .git/hooks/pre-commit
#!/bin/bash
ghost-hunter scan code $(git diff --cached --name-only | grep -E '\.(go|py|js|ts)$') \
  --severity critical \
  --format plain \
  --fail-on critical

if [ $? -ne 0 ]; then
  echo "❌ Security scan failed. Fix critical issues before committing."
  exit 1
fi
```

### Ejemplo 3: Escaneo de Kubernetes en Runtime
```bash
ghost-hunter scan runtime k8s://production-cluster \
  --namespace financial-app \
  --scan-pods \
  --scan-configmaps \
  --scan-secrets \
  --check-privileged \
  --check-hostpath \
  --check-capabilities \
  --output runtime-findings.json

# Salida de muestra:
# {
#   "pods": [
#     {
#       "name": "payment-processor-abc123",
#       "namespace": "financial-app",
#       "issues": [
#         {
#           "severity": "critical",
#           "rule": "K8S001",
#           "title": "Privileged container detected",
#           "description": "Container runs with privileged=true flag",
#           "remediation": "Remove privileged flag; use specific capabilities instead"
#         }
#       ]
#     }
#   ]
# }
```

### Ejemplo 4: Barrido de Secretos en Múltiples Repositorios
```bash
# Encontrar todas las claves API en 100+ repositorios
repos=(repo1 repo2 repo3 repo4 repo5)

for repo in "${repos[@]}"; do
  echo "Scanning $repo..."
  git -C "/repos/$repo" log --all -p | \
    ghost-hunter detect secrets - \
    --format json \
    --output "/scans/$repo-secrets-$(date +%Y%m%d).json"
done

# Consolidar y deduplicar
jq -s 'add' /scans/*-secrets-*.json | \
  ghost-hunter report de-duplicate --output consolidated-secrets.json
```

### Ejemplo 5: Creación de Regla Personalizada
```yaml
# .ghost-hunter/rules/custom-api-auth.yaml
rule:
  id: CUSTOM-API-001
  title: "Hardcoded API credentials in configuration files"
  severity: critical
  tags:
    - credentials
    - api
    - secrets
  patterns:
    - pattern: |
        api_key\s*=\s*["'][A-Za-z0-9]{32}["']
      languages: [python, javascript, yaml, json]
    - pattern: |
        Authorization:\s*Bearer\s+[A-Za-z0-9\-_]+
      languages: [http]
  message: "Hardcoded API key found. Use environment variables or secret manager"
  fix: |
    Replace hardcoded value with environment variable:
    - Python: os.getenv('API_KEY')
    - Node: process.env.API_KEY
    - Config: ${API_KEY}
```

Uso:
```bash
ghost-hunter scan code . \
  --rules .ghost-hunter/rules/custom-api-auth.yaml \
  --fail-on critical
```

### Ejemplo 6: Escaneo en Tiempo de Build de Docker
```dockerfile
# Dockerfile
FROM golang:1.21-alpine AS builder
# ... build steps ...

FROM alpine:3.18
# Ghost Hunter scan stage
FROM builder AS security-scan
RUN apk add --no-cache ghost-hunter
COPY --from=builder /app/bin/app /app/
RUN ghost-hunter scan image /app \
  --output /security-findings.json \
  --fail-on critical || true  # Don't fail build, just report

# Final stage
FROM alpine:3.18
COPY --from=builder /app/bin/app /app/
# Optionally copy security findings as artifact
COPY --from=security-scan /security-findings.json /metadata/
```

### Ejemplo 7: Integración de Inteligencia de Amenazas
```bash
# Verificar si alguna dependencia tiene exploits activos
ghost-hunter scan deps ./node-app \
  --threat-intel \
  --sources nvd,exploit-db,github-advisory \
  --output deps-with-threats.json

# Salida de muestra destacando exploitabilidad:
# {
#   "dependency": "lodash@4.17.21",
#   "cve": "CVE-2021-23337",
#   "exploit_exists": true,
#   "exploit_source": "Metasploit",
#   "exploit_maturity": "weaponized",
#   "remediation": "Upgrade to lodash@4.17.28"
# }

# Suscribirse a exploits para tu lista de dependencias
cat deps-with-threats.json | jq -r '.dependency' | \
  ghost-hunter threat-intel watch \
  --package-list /dev/stdin \
  --notify slack://#security-team \
  --pagerduty
```

## Comandos de Rollback

### Deshacer Resultados de Escaneo de Código
```bash
# Eliminar resultados de escaneo en caché
ghost-hunter cache clean --all

# Eliminar línea base para comparación
rm ~/.cache/ghost-hunter/baseline.json

# Revertir cualquier instalación de hooks pre-commit
ghost-hunter hook uninstall pre-commit
cat ~/.git/hooks/pre-commit.sample > .git/hooks/pre-commit
```

### Restaurar desde Backup
```bash
# Restaurar configuración de escaneo anterior
ghost-hunter config restore \
  --backup ~/.ghost-hunter/backup/config-20240101.yaml

# Restaurar reglas personalizadas desde backup
cp ~/.ghost-hunter/backup/custom-rules/* .ghost-hunter/rules/
ghost-hunter validate-rules
```

### Eliminar Hallazgos del Dashboard
```bash
# Cerrar hallazgos en masa por ID de escaneo
ghost-hunter dashboard findings close \
  --scan-id abc123-def456 \
  --reason "False positive, used temporary pattern" \
  --comment "Pattern no longer present in codebase"

# Eliminar proyecto completo del dashboard
ghost-hunter dashboard project delete \
  --project my-old-app \
  --confirm-by-name my-old-app
```

### Revertir Cambios de Remediación
```bash
# Si se aplicó auto-fix y necesita deshacerse
ghost-hunter fix undo \
  --file src/auth.py \
  --backup-id 20240101_143022_abc123

# Ver qué se revertiría
ghost-hunter fix preview-undo \
  --file src/auth.py \
  --backup-id 20240101_143022_abc123
```

### Detener Monitoreo Continuo
```bash
# Desuscribirse de feeds de inteligencia de amenazas
ghost-hunter feed unsubscribe \
  --feed cve \
  --target myapp

# Eliminar flujo de trabajo de GitHub Actions
ghost-hunter action workflow remove \
  --repo myorg/myrepo \
  --workflow ghost-hunter.yml

# Deshabilitar hooks pre-commit globalmente
ghost-hunter config set pre_commit.enabled false
```

### Purgar Todos los Datos de Ghost Hunter
```bash
# Desinstalación completa con eliminación de datos
ghost-hunter uninstall --purge

# Esto elimina:
# - ~/.ghost-hunter/ (config, caches, logs)
# - ~/.cache/ghost-hunter/
# - hooks pre-commit
# - flujos de trabajo de GitHub Actions
# - reglas personalizadas
```

## Dependencias

### Requisitos del Sistema
- **SO**: Linux (Ubuntu 20.04+, Debian 11+, RHEL 8+) o macOS 11+
- **Memoria**: Mínimo 2GB, recomendado 4GB+ para escaneos paralelos
- **Disco**: 5GB para caches + espacio para SBOMs y reportes
- **Red**: HTTPS saliente para actualizaciones de base de datos de vulnerabilidades

### Binarios Requeridos
- `docker` (para escaneo de contenedores)
- `kubectl` (para escaneos de Kubernetes en runtime)
- `git` (para análisis de historial)
- `python3` con pip (bandit, semgrep)
- `node` con npm (escaneo frontend)
- `golang` (go vet, gosec)

### Integraciones Opcionales
- `trivy` (escaneo de contenedores y sistema de archivos)
- `grype` (coincidencia de vulnerabilidades)
- `checkov` (escaneo IaC)
- `kubesec` (seguridad de Kubernetes)
- `cosign` / `notation` (verificación de firma de imágenes)
- `notary` (Docker content trust)

### Instalación
```bash
curl -sSL https://get.openclaw.io/ghost-hunter | bash
# o
brew install openclaw/tap/ghost-hunter
# o
docker run --rm -v $(pwd):/scan openclaw/ghost-hunter scan code /scan
```

### Configuración
```bash
# ~/.ghost-hunter/config.yaml
exclude:
  - node_modules
  - vendor
  - build
  - .git
  - '*.min.js'
  - '*.bundle.js'

engines:
  code:
    enabled: [bandit, semgrep, gosec, npm-audit]
    semgrep_rules: ~/.ghost-hunter/rules/semgrep
  iac:
    enabled: [checkov, tfsec, kubesec]
  image:
    enabled: [trivy, grype]
    severity_threshold: high
  secrets:
    patterns: ~/.ghost-hunter/patterns/secrets
    entropy: 4.5

notifications:
  slack:
    webhook: ${SLACK_WEBHOOK_URL}
    channel: "#security-alerts"
  email:
    enabled: true
    smtp: smtp.gmail.com:587

cache:
  enabled: true
  dir: ~/.cache/ghost-hunter
  ttl: 24h
```

## Verificación

### Chequeo de Salud
```bash
# Verificar instalación y dependencias
ghost-hunter doctor

# Salida esperada:
# ✓ Ghost Hunter v1.2.0
# ✓ Python 3.11.5 detectado
# ✓ Docker 24.0.5 detectado
# ✓ Trivy v0.45.1 detectado
# ✓ Semgrep v1.50.1 detectado
# ✓ Configuración válida
# ✓ 2454 reglas de seguridad cargadas
# ✓ Feed de inteligencia de amenazas activo
# ✗ Contexto de Kubernetes no configurado (ejecuta: kubectl config use-context production)
```

### Verificación de Escaneo
```bash
# Escaneo de prueba en código vulnerable conocido
mkdir /tmp/ghost-test && cd /tmp/ghost-test
cat > vulnerable.py << 'EOF'
import subprocess
password = "hardcoded123"  # Debería disparar CUSTOM-API-001
subprocess.run(["ls", "-la"], shell=True)  # Debería disparar B302
EOF

ghost-hunter scan code . --severity high --format json
# Esperar: al menos 2 hallazgos (contraseña hardcodeada, subprocess shell=True)
```

### Verificación de Integración
```bash
# Probar hook pre-commit
ghost-hunter hook test pre-commit
# Debería salir: ✓ hook pre-commit instalado correctamente

# Probar formato de salida de GitHub Actions
ghost-hunter scan code . --format github-actions --dry-run
# Debería salir formato compatible con GITHUB_OUTPUT
```

## Solución de Problemas

### Problemas de Rendimiento del Escáner
```bash
# Problema: El escaneo es muy lento
# Solución: Ajustar paralelismo y habilitar caché
ghost-hunter config set engines.parallel 2
ghost-hunter config set cache.enabled true
ghost-hunter config set cache.ttl 48h

# Problema: Sin memoria durante escaneos grandes
# Solución: Añadir límites de memoria
ghost-hunter scan code . --max-memory 2G --batch-size 100
```

### Falsos Positivos
```bash
# Problema: Demasiados falsos positivos de regla específica
# Solución: Deshabilitar regla o añadir excepción
ghost-hunter config set rules.suppress CUSTOM-API-001:src/test/

# Problema: Secretos legítimos marcados (ej., credenciales de test)
# Solución: Crear archivo de ignore
cat > .ghost-hunter-ignore << 'EOF'
# Test credentials
SEED-PASSWORD-123
TEST-KEY-ABC
EOF
ghost-hunter detect secrets . --exclude-file .ghost-hunter-ignore
```

### Problemas de Docker/Contenedor
```bash
# Problema: No se puede acceder al demonio Docker
# Solución: Verificar permisos y socket
ghost-hunter doctor --check-docker
# Si falla: sudo usermod -aG docker $USER && newgrp docker

# Problema: Escaneo de imagen de contenedor falla por auth de registry
# Solución: Configurar helper de credenciales Docker
cat > ~/.docker/config.json << 'EOF'
{
  "creds-store": "secretservice"
}
EOF
```

### Problemas de Acceso a Kubernetes
```bash
# Problema: No se puede conectar al clúster de Kubernetes
# Solución: Verificar contexto y permisos
kubectl config get-contexts
kubectl config use-context production
kubectl auth can-i list pods --all-namespaces

# Problema: Escaneo en runtime timeout
# Solución: Aumentar timeout y reducir alcance
ghost-hunter scan runtime k8s://cluster \
  --timeout 300s \
  --max-pods 50 \
  --namespace critical-app
```

### Fallos de Actualización de Base de Datos
```bash
# Problema: Fallo en actualización de base de datos de vulnerabilidades
# Solución: Actualización manual o modo offline
ghost-hunter db update --force
# O usar base de datos en caché sin actualizar
ghost-hunter config set db.update false --cache-db true

# Problema: Reglas de Semgrep no cargan
# Solución: Actualizar reglas manualmente
ghost-hunter rules update --source official --force
ghost-hunter validate-rules --repair
```

### Problemas de Red/Proxy
```bash
# Problema: No se pueden descargar feeds de vulnerabilidades
# Solución: Configurar proxy
export HTTP_PROXY=http://proxy.company.com:8080
export HTTPS_PROXY=http://proxy.company.com:8443
ghost-hunter config set proxy.http $HTTP_PROXY
ghost-hunter config set proxy.https $HTTPS_PROXY

# Problema: Errores de certificados SSL
# Solución: Añadir CA corporativa
ghost-hunter config set ca-cert /path/to/company-ca.pem
```

### Problemas de Escaneo de Historial de Git
```bash
# Problema: Escaneo de historial de Git falla en repos grandes
# Solución: Limitar profundidad de historial y ramas
ghost-hunter scan git-history . \
  --since "6 months ago" \
  --branches main,develop \
  --max-commits 10000

# Problema: Falsos positivos de código vendor
# Solución: Excluir directorios vendor
ghost-hunter scan git-history . \
  --exclude vendor/,third_party/,external/
```

## Características Avanzadas

### Desarrollo de Reglas Personalizadas
```bash
# Crear regla personalizada basada en YAML
ghost-hunter rule create \
  --title "Mi Verificación Personalizada" \
  --severity high \
  --pattern 'password\s*=\s*[\"'\"'\"'][^\"'\"'\"']+[\"'\"'\"']' \
  --language python \
  --output .ghost-hunter/rules/custom.yaml

# Probar regla personalizada
ghost-hunter scan code . --rules .ghost-hunter/rules/custom.yaml --debug
```

### Comparación con Línea Base
```bash
# Crear línea base desde escaneo limpio
ghost-hunter scan code . --output baseline.json --no-fail

# Escaneos futuros comparan con línea base
ghost-hunter scan code . --baseline baseline.json --output new.json

# Mostrar solo hallazgos nuevos
ghost-hunter report diff baseline.json new.json
```

### Triage Automatizado
```bash
# Triage automático basado en patrones
cat > triage-rules.yaml << 'EOF'
triage:
  - match:
      rule_id: [SQL-001, SQL-002]
      path: test/
    status: false_positive
    comment: "Ejemplos SQL de test, no código de producción"
  - match:
      severity: [low, info]
      tag: [experimental]
    status: accepted_risk
    comment: "Bajo riesgo en entorno no productivo"
EOF

ghost-hunter scan code . --triage triage-rules.yaml
```

### Exportación a Herramientas de Seguridad
```bash
# Exportar a DefectDojo
ghost-hunter report export defectdojo \
  --input scan-results.json \
  --url https://defectdojo.example.com \
  --api-key ${DD_API_KEY} \
  --product "My Application" \
  --engagement "Sprint 24"

# Exportar a Jira
ghost-hunter report export jira \
  --input critical-findings.json \
  --project SEC \
  --issue-type Vulnerability \
  --priority High
```

## Lista de Verificación Pre-Uso

- [ ] Instalar última versión de Ghost Hunter (`ghost-hunter version`)
- [ ] Actualizar base de datos de vulnerabilidades (`ghost-hunter db update`)
- [ ] Configurar canales de notificación (Slack, email, PagerDuty)
- [ ] Configurar hooks pre-commit en repositorios activos
- [ ] Crear reglas personalizadas para patrones específicos de la organización
- [ ] Configurar autenticación para entornos objetivo (K8s, registries)
- [ ] Probar escaneo en entorno no productivo primero
- [ ] Establecer línea base para aplicaciones críticas
- [ ] Configurar reporte automatizado a dashboard de seguridad
- [ ] Documentar procedimientos de rollback para el equipo
- [ ] Revisar y aprobar reglas de triage

## Contactos de Emergencia

- **Infraestructura Crítica**: security-emergency@openclaw.io
- **Reporte de Vulnerabilidad**: https://security.openclaw.io/report
- **Slack de Soporte**: #ghost-hunter-support
- **Runbook**: https://docs.openclaw.io/ghost-hunter/runbook

## Changelog

**v1.2.0** (2024-01-15)
- Añadido escaneo de Kubernetes en runtime con políticas de seguridad de pods
- Introducida integración de inteligencia de amenazas con mapping de exploits de CVE
- Añadida comparación con línea base para detección delta
- Mejorada detección de secretos con soporte de patrones personalizados
- Nuevos comandos `ghost-hunter threat-intel`

**v1.1.0** (2023-10-20)
- Generación de SBOM de imágenes de contenedor con procedencia
- Integración nativa de GitHub Actions
- Agregación multi-motor con deduplicación
- Filtrado avanzado por etiqueta y severidad

**v1.0.0** (2023-07-01)
- Lanzamiento inicial con escaneo de código, IaC, imágenes y dependencias
- Formatos de salida SARIF y JSON
- Soporte de hooks pre-commit
- Umbrales de severidad configurables
```