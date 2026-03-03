---
name: Ghost Hunter
description: Advanced security vulnerability scanner for code and infrastructure with stealth detection capabilities
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

Stealthy vulnerability detection and hidden threat analysis for modern infrastructure.

## Purpose

Ghost Hunter provides continuous security detection without disrupting development workflows. Unlike traditional scanners that generate noise and false positives, Ghost Hunter uses contextual awareness to identify real threats:

- **Code Review Enhancement**: Automatically scan PRs for OWASP Top 10, CWE patterns, and supply chain vulnerabilities before merge
- **Infrastructure As Code Security**: Detect misconfigurations in Terraform, CloudFormation, Kubernetes manifests, and Dockerfiles before deployment
- **Container Hardening**: Scan container images for CVEs, secret leaks, and misconfigurations in CI/CD pipelines
- **Dependency Threat Intelligence**: Monitor dependencies for known exploits, malicious packages, and license compliance issues
- **Credential Leak Detection**: Scan Git history, logs, and artifacts for API keys, tokens, passwords, and certificates
- **Runtime Stealth Detection**: Monitor running containers and VPs for privilege escalation paths and exposed services

Real use cases:
- Scan 200+ microservices in 8 minutes with dependency graph analysis
- Detect hardcoded AWS keys in legacy codebase across 50 repositories
- Identify over-privileged Kubernetes service accounts before production rollout
- Find secret commits in Git history that bypassed pre-commit hooks
- Audit Dockerfile security posture across all developer workstations

## Scope

### Core Commands

`ghost-hunter scan code [PATH]`
- Recursive code security analysis using multiple engines
- Auto-detects language and applies appropriate rules
- Supports: Go, Python, JavaScript/Typecript, Java, Rust, Ruby, PHP

`ghost-hunter scan iac [PATH]`
- Infrastructure as Code security scanning
- Detects misconfigurations in Terraform, CloudFormation, Ansible, Kubernetes, Docker
- Checks against CIS benchmarks and cloud best practices

`ghost-hunter scan image [IMAGE_TAG]`
- Container image vulnerability scanning with depth analysis
- Layer-by-layer inspection, OS package and language dependency scanning
- SBOM generation and provenance verification

`ghost-hunter scan git-history [REPO_PATH]`
- Deep Git history scanning for secrets and sensitive data
- Analyzes all branches and tags, including rewritten history
- Supports custom regex patterns for organization-specific secrets

`ghost-hunter scan deps [PROJECT_PATH]`
- Dependency vulnerability and supply chain analysis
- Bill of Materials (BOM) generation with vulnerability linkage
- Malicious package detection using threat intelligence feeds

`ghost-hunter scan runtime [TARGET]`
- Live environment assessment (Kubernetes pods, Docker containers, processes)
- Configuration validation against security policies
- Network exposure and privilege analysis

`ghost-hunter detect secrets [PATH]`
- Specialized secret detection with 200+ patterns
- Custom pattern addition for proprietary formats
- Confidence scoring and false positive reduction

### Filtering and Reporting

`--severity [critical|high|medium|low|info]`
Filter findings by severity level

`--exclude [PATH]`
Exclude directories or file patterns from scan

`--format [json|sarif|html|github-actions]`
Output format selection

`--output [FILE]`
Write results to file instead of stdout

`--fail-on [CRITERIA]`
Exit with non-zero code if criteria matched (e.g., "critical", "high+medium")

`--rule [RULE_ID]`
Run only specific rule or CWE identifier

`--tag [TAG]`
Filter by tag (e.g., "owasp-a10", "cwe-89", "sql-injection")

`--context [ENV]`
Add contextual tags to findings (e.g., production, staging, dev)

`--no-cache`
Disable result caching for fresh scan

`--parallel [N]`
Number of parallel workers (default: CPU count)

### Integration Options

`--pre-commit`
Output in pre-commit hook format (stderr only, no JSON)

`--github-actions`
Generate GitHub Actions annotations

`--gitlab-ci`
GitLab CI job artifact generation

`--azdo-pipelines`
Azure DevOps inline comments

## Work Process

### 1. Initial Assessment
```bash
# Quick reconnaissance: identify project type and scope
ghost-hunter detect project-type ./myapp
# Output: {"type":"nodejs","languages":["javascript","typescript"],"frameworks":["express","react"],"package_manager":"npm"}
```

### 2. Code Security Scan
```bash
# Full codebase scan with all engines
ghost-hunter scan code ./src \
  --severity high \
  --exclude node_modules,build,coverage \
  --format sarif \
  --output codefindings.sarif

# Parallel scan across multiple repositories
find /repos -name "*.go" -type f | parallel ghost-hunter scan code {} --format json
```

### 3. Infrastructure Scan
```bash
# Terraform security assessment
ghost-hunter scan iac ./terraform \
  --checkov \
  --tflint \
  --severity critical \
  --context production

# Kubernetes manifest scanning
ghost-hunter scan iac ./k8s/manifests \
  --kubesec \
  --polaris \
  --output k8s-findings.html
```

### 4. Container Image Analysis
```bash
# Deep container inspection
ghost-hunter scan image myapp:latest \
  --image myregistry.com/myapp:v1.2.3 \
  --scanners trivy,grype \
  --generate-sbom \
  --verify-provenance \
  --output image-report.json

# Scan all images in Docker Compose
docker-compose config --services | while read svc; do
  img=$(docker-compose config --services | grep $svc | xargs docker-compose config | grep image | cut -d' ' -f2)
  ghost-hunter scan image $img --format json
done
```

### 5. Dependency Audit
```bash
# NPM dependency vulnerability scan
ghost-hunter scan deps ./frontend \
  --audit-level high \
  --check-licenses \
  --detect-malicious \
  --output deps.json

# Python requirements with pip-audit integration
ghost-hunter scan deps ./backend/python \
  --pip-audit \
  --safety \
  --cve-db-update
```

### 6. Secrets Detection
```bash
# Comprehensive secret scan across entire codebase
ghost-hunter detect secrets ./ \
  --patterns ~/.ghost-hunter/patterns/custom.secrets \
  --entropy 4.5 \
  --max-depth 5 \
  --exclude-file .ghost-hunter-ignore \
  --output secrets.json

# Git history sweep for leaked credentials
ghost-hunter scan git-history ./repo \
  --since "1 year ago" \
  --branches all \
  --regex 'AKIA[0-9A-Z]{16}' \
  --regex 'ghp_[0-9a-zA-Z]{36}' \
  --regex 'sk_live_[0-9a-zA-Z]{24}'
```

### 7. Runtime Assessment
```bash
# Kubernetes cluster scan
ghost-hunter scan runtime k8s://my-cluster \
  --namespace prod \
  --scan-pods \
  --scan-configmaps \
  --scan-serviceaccounts \
  --output runtime-k8s.json

# Docker container inspection
ghost-hunter scan runtime docker://container-id \
  --check-privileged \
  --check-capabilities \
  --check-mounts \
  --check-network
```

### 8. Reporting and Integration
```bash
# Generate comprehensive report
ghost-hunter report generate \
  --inputs codefindings.sarif,image-report.json,deps.json \
  --template executive \
  --output security-report.html \
  --send-to slack://#security-alerts

# Create GitHub Security Advisory from findings
ghost-hunter advisory create \
  --finding image-report.json \
  --repo myorg/myrepo \
  --severity critical \
  --draft
```

### 9. Continuous Monitoring
```bash
# Setup pre-commit hook
ghost-hunter hook install pre-commit \
  --scanners code,secrets \
  --block-on high,critical

# GitHub Actions workflow
ghost-hunter action workflow \
  --on push,pr \
  --scan code,iac,deps \
  --fail-on critical \
  --comment-on-pr
```

### 10. Threat Intelligence Lookup
```bash
# Check if any findings match active exploits
ghost-hunter threat-intel lookup \
  --cve CVE-2024-12345 \
  --vendor apache \
  --product log4j

# Subscribe to vulnerability feed
ghost-hunter feed subscribe \
  --feed cve \
  --severity critical \
  --notify slack,email \
  --daily 09:00
```

## Golden Rules

1. **Zero Trust Scanning**
   - Never trust dependency sources; always verify checksums and signatures
   - Scan all artifacts including build caches and temporary files
   - Assume every external dependency is compromised until proven otherwise

2. **Principle of Least Privilege**
   - Scan with minimal necessary permissions; never run as root unless absolutely required
   - Container scans must use read-only mounts for host filesystem access
   - Network scans must be isolated using egress-only rules

3. **Stealth Operations**
   - Use resource throttling (`--parallel 1 --throttle 100ms`) in production environments
   - Avoid creating load spikes on CI/CD runners; respect `--max-cpu` and `--max-memory`
   - Cache results aggressively to minimize repeated scans of unchanged code

4. **Evidence Preservation**
   - Always save raw scanner outputs alongside parsed results
   - Keep full SBOMs and provenance data for every image scanned
   - Maintain immutable scan logs with timestamps and git SHAs

5. **False Positive Elimination**
   - Only mark findings as "false positive" in triage, never suppress them
   - Use baseline comparison (`--baseline previous-scan.json`) to detect new issues only
   - Require peer review for any suppression of critical/high findings

6. **Secret Handling**
   - Never log discovered secrets in plaintext, even in debug mode
   - Immediately rotate any live credentials found in scans
   - Use `--mask-secrets` flag in all reports to redact sensitive values

7. **Compliance Mapping**
   - Always tag findings with relevant frameworks: PCI-DSS, HIPAA, GDPR, SOC2, ISO27001
   - Generate compliance-specific reports (`--compliance pci-dss`) for audits
   - Maintain audit trail of all compliance status changes

8. **Supply Chain Integrity**
   - Verify every container image with Notary, Cosign, or similar sigstore tool
   - Check dependency provenance: `ghost-hunter scan deps --verify-attestations`
   - Reject dependencies from unmaintained or suspicious package sources

9. **Defense in Depth**
   - Run multiple scanners with different detection methodologies
   - Combine SAST, SCA, DAST, and IaC scanning for complete coverage
   - Cross-reference findings across tools to prioritize real threats

10. **Never Interfere with Production**
    - Runtime scans read-only unless explicitly using `--fix-permissions` or similar
    - Never modify infrastructure or change configurations during scan
    - Isolate scanning containers from production networks using network policies

## Examples

### Example 1: CI/CD Pipeline Integration
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

### Example 2: Pre-commit Hook Configuration
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

### Example 3: Kubernetes Runtime Scan
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

# Sample output:
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

### Example 4: Multi-Repository Secret Sweep
```bash
# Find all API keys in 100+ repositories
repos=(repo1 repo2 repo3 repo4 repo5)

for repo in "${repos[@]}"; do
  echo "Scanning $repo..."
  git -C "/repos/$repo" log --all -p | \
    ghost-hunter detect secrets - \
    --format json \
    --output "/scans/$repo-secrets-$(date +%Y%m%d).json"
done

# Consolidate and deduplicate
jq -s 'add' /scans/*-secrets-*.json | \
  ghost-hunter report de-duplicate --output consolidated-secrets.json
```

### Example 5: Custom Rule Creation
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

Usage:
```bash
ghost-hunter scan code . \
  --rules .ghost-hunter/rules/custom-api-auth.yaml \
  --fail-on critical
```

### Example 6: Docker Build-time Scanning
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

### Example 7: Threat Intelligence Integration
```bash
# Check if any dependencies have active exploits
ghost-hunter scan deps ./node-app \
  --threat-intel \
  --sources nvd,exploit-db,github-advisory \
  --output deps-with-threats.json

# Sample output highlighting exploitability:
# {
#   "dependency": "lodash@4.17.21",
#   "cve": "CVE-2021-23337",
#   "exploit_exists": true,
#   "exploit_source": "Metasploit",
#   "exploit_maturity": "weaponized",
#   "remediation": "Upgrade to lodash@4.17.28"
# }

# Subscribe to exploits for your dependency list
cat deps-with-threats.json | jq -r '.dependency' | \
  ghost-hunter threat-intel watch \
  --package-list /dev/stdin \
  --notify slack://#security-team \
  --pagerduty
```

## Rollback Commands

### Undo Code Scan Results
```bash
# Remove cached scan results
ghost-hunter cache clean --all

# Remove baseline for comparison
rm ~/.cache/ghost-hunter/baseline.json

# Revert any pre-commit hook installations
ghost-hunter hook uninstall pre-commit
cat ~/.git/hooks/pre-commit.sample > .git/hooks/pre-commit
```

### Restore from Backup
```bash
# Restore previous scan configuration
ghost-hunter config restore \
  --backup ~/.ghost-hunter/backup/config-20240101.yaml

# Restore custom rules from backup
cp ~/.ghost-hunter/backup/custom-rules/* .ghost-hunter/rules/
ghost-hunter validate-rules
```

### Remove Findings from Dashboard
```bash
# Bulk close findings by scan ID
ghost-hunter dashboard findings close \
  --scan-id abc123-def456 \
  --reason "False positive, used temporary pattern" \
  --comment "Pattern no longer present in codebase"

# Remove entire project from dashboard
ghost-hunter dashboard project delete \
  --project my-old-app \
  --confirm-by-name my-old-app
```

### Revert Remediation Changes
```bash
# If auto-fix was applied and needs to be undone
ghost-hunter fix undo \
  --file src/auth.py \
  --backup-id 20240101_143022_abc123

# View what would be reverted
ghost-hunter fix preview-undo \
  --file src/auth.py \
  --backup-id 20240101_143022_abc123
```

### Stop Continuous Monitoring
```bash
# Unsubscribe from threat intelligence feeds
ghost-hunter feed unsubscribe \
  --feed cve \
  --target myapp

# Remove GitHub Actions workflow
ghost-hunter action workflow remove \
  --repo myorg/myrepo \
  --workflow ghost-hunter.yml

# Disable pre-commit hooks globally
ghost-hunter config set pre_commit.enabled false
```

### Purge All Ghost Hunter Data
```bash
# Complete uninstall with data removal
ghost-hunter uninstall --purge

# This removes:
# - ~/.ghost-hunter/ (config, caches, logs)
# - ~/.cache/ghost-hunter/
# - pre-commit hooks
# - GitHub Actions workflows
# - Custom rules
```

## Dependencies

### System Requirements
- **OS**: Linux (Ubuntu 20.04+, Debian 11+, RHEL 8+) or macOS 11+
- **Memory**: Minimum 2GB, recommended 4GB+ for parallel scans
- **Disk**: 5GB for caches + space for SBOMs and reports
- **Network**: Outbound HTTPS for vulnerability database updates

### Required Binaries
- `docker` (for container scanning)
- `kubectl` (for Kubernetes runtime scans)
- `git` (for history analysis)
- `python3` with pip (bandit, semgrep)
- `node` with npm (frontend scanning)
- `golang` (go vet, gosec)

### Optional Integrations
- `trivy` (container and filesystem scanning)
- `grype` (vulnerability matching)
- `checkov` (IaC scanning)
- `kubesec` (Kubernetes security)
- `cosign` / `notation` (image signature verification)
- `notary` (Docker content trust)

### Install
```bash
curl -sSL https://get.openclaw.io/ghost-hunter | bash
# or
brew install openclaw/tap/ghost-hunter
# or
docker run --rm -v $(pwd):/scan openclaw/ghost-hunter scan code /scan
```

### Configuration
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

## Verification

### Health Check
```bash
# Verify installation and dependencies
ghost-hunter doctor

# Expected output:
# ✓ Ghost Hunter v1.2.0
# ✓ Python 3.11.5 detected
# ✓ Docker 24.0.5 detected
# ✓ Trivy v0.45.1 detected
# ✓ Semgrep v1.50.1 detected
# ✓ Configuration valid
# ✓ 2454 security rules loaded
# ✓ Threat intelligence feed active
# ✗ Kubernetes context not set (run: kubectl config use-context production)
```

### Scan Verification
```bash
# Test scan on known vulnerable code
mkdir /tmp/ghost-test && cd /tmp/ghost-test
cat > vulnerable.py << 'EOF'
import subprocess
password = "hardcoded123"  # Should trigger CUSTOM-API-001
subprocess.run(["ls", "-la"], shell=True)  # Should trigger B302
EOF

ghost-hunter scan code . --severity high --format json
# Expect: at least 2 findings (hardcoded password, subprocess shell=True)
```

### Integration Verification
```bash
# Test pre-commit hook
ghost-hunter hook test pre-commit
# Should output: ✓ pre-commit hook installed correctly

# Test GitHub Actions output format
ghost-hunter scan code . --format github-actions --dry-run
# Should output GITHUB_OUTPUT compatible format
```

## Troubleshooting

### Scanner Performance Issues
```bash
# Problem: Scanning is too slow
# Solution: Adjust parallelism and enable caching
ghost-hunter config set engines.parallel 2
ghost-hunter config set cache.enabled true
ghost-hunter config set cache.ttl 48h

# Problem: Out of memory during large scans
# Solution: Add memory limits
ghost-hunter scan code . --max-memory 2G --batch-size 100
```

### False Positives
```bash
# Problem: Too many false positives from specific rule
# Solution: Disable rule or add exception
ghost-hunter config set rules.suppress CUSTOM-API-001:src/test/

# Problem: Legitimate secrets flagged (e.g., test credentials)
# Solution: Create ignore file
cat > .ghost-hunter-ignore << 'EOF'
# Test credentials
SEED-PASSWORD-123
TEST-KEY-ABC
EOF
ghost-hunter detect secrets . --exclude-file .ghost-hunter-ignore
```

### Docker/Container Issues
```bash
# Problem: Cannot access Docker daemon
# Solution: Check permissions and socket
ghost-hunter doctor --check-docker
# If failing: sudo usermod -aG docker $USER && newgrp docker

# Problem: Container image scan fails due to registry auth
# Solution: Configure Docker credential helper
cat > ~/.docker/config.json << 'EOF'
{
  "creds-store": "secretservice"
}
EOF
```

### Kubernetes Access Issues
```bash
# Problem: Cannot connect to Kubernetes cluster
# Solution: Verify context and permissions
kubectl config get-contexts
kubectl config use-context production
kubectl auth can-i list pods --all-namespaces

# Problem: Runtime scan times out
# Solution: Increase timeout and reduce scope
ghost-hunter scan runtime k8s://cluster \
  --timeout 300s \
  --max-pods 50 \
  --namespace critical-app
```

### Database Update Failures
```bash
# Problem: Vulnerability database update fails
# Solution: Manual update or offline mode
ghost-hunter db update --force
# Or use cached database without update
ghost-hunter config set db.update false --cache-db true

# Problem: Semgrep rules not loading
# Solution: Update rules manually
ghost-hunter rules update --source official --force
ghost-hunter validate-rules --repair
```

### Network/Proxy Issues
```bash
# Problem: Cannot download vulnerability feeds
# Solution: Configure proxy
export HTTP_PROXY=http://proxy.company.com:8080
export HTTPS_PROXY=http://proxy.company.com:8443
ghost-hunter config set proxy.http $HTTP_PROXY
ghost-hunter config set proxy.https $HTTPS_PROXY

# Problem: SSL certificate errors
# Solution: Add corporate CA
ghost-hunter config set ca-cert /path/to/company-ca.pem
```

### Git History Scanning Issues
```bash
# Problem: Git history scan crashes on large repos
# Solution: Limit history depth and branches
ghost-hunter scan git-history . \
  --since "6 months ago" \
  --branches main,develop \
  --max-commits 10000

# Problem: False positives from vendor code
# Solution: Exclude vendor directories
ghost-hunter scan git-history . \
  --exclude vendor/,third_party/,external/
```

## Advanced Features

### Custom Rule Development
```bash
# Create YAML-based custom rule
ghost-hunter rule create \
  --title "My Custom Check" \
  --severity high \
  --pattern 'password\s*=\s*["'"'"'][^"'"'"']+["'"'"']' \
  --language python \
  --output .ghost-hunter/rules/custom.yaml

# Test custom rule
ghost-hunter scan code . --rules .ghost-hunter/rules/custom.yaml --debug
```

### Baseline Comparison
```bash
# Create baseline from clean scan
ghost-hunter scan code . --output baseline.json --no-fail

# Future scans compare to baseline
ghost-hunter scan code . --baseline baseline.json --output new.json

# Show only new findings
ghost-hunter report diff baseline.json new.json
```

### Automated Triage
```bash
# Auto-triage based on patterns
cat > triage-rules.yaml << 'EOF'
triage:
  - match:
      rule_id: [SQL-001, SQL-002]
      path: test/
    status: false_positive
    comment: "Test SQL examples, not production code"
  - match:
      severity: [low, info]
      tag: [experimental]
    status: accepted_risk
    comment: "Low risk in non-production environment"
EOF

ghost-hunter scan code . --triage triage-rules.yaml
```

### Export to Security Tools
```bash
# Export to DefectDojo
ghost-hunter report export defectdojo \
  --input scan-results.json \
  --url https://defectdojo.example.com \
  --api-key ${DD_API_KEY} \
  --product "My Application" \
  --engagement "Sprint 24"

# Export to Jira
ghost-hunter report export jira \
  --input critical-findings.json \
  --project SEC \
  --issue-type Vulnerability \
  --priority High
```

## Pre-Use Checklist

- [ ] Install latest Ghost Hunter version (`ghost-hunter version`)
- [ ] Update vulnerability database (`ghost-hunter db update`)
- [ ] Configure notification channels (Slack, email, PagerDuty)
- [ ] Set up pre-commit hooks in active repositories
- [ ] Create custom rules for organization-specific patterns
- [ ] Configure authentication for target environments (K8s, registries)
- [ ] Test scan on non-production environment first
- [ ] Establish baseline for critical applications
- [ ] Set up automated reporting to security dashboard
- [ ] Document rollback procedures for team
- [ ] Review and approve triage rules

## Emergency Contacts

- **Critical Infrastructure Issue**: security-emergency@openclaw.io
- **Vulnerability Report**: https://security.openclaw.io/report
- **Support Slack**: #ghost-hunter-support
- **Runbook**: https://docs.openclaw.io/ghost-hunter/runbook

## Changelog

**v1.2.0** (2024-01-15)
- Added Kubernetes runtime scanning with pod security policies
- Introduced threat intelligence integration with CVE exploit mapping
- Added baseline comparison for delta detection
- Improved secret detection with custom pattern support
- New `ghost-hunter threat-intel` commands

**v1.1.0** (2023-10-20)
- Container image SBOM generation with provenance
- GitHub Actions native integration
- Multi-engine aggregation with deduplication
- Advanced filtering by tag and severity

**v1.0.0** (2023-07-01)
- Initial release with code, IaC, image, and dependency scanning
- SARIF and JSON output formats
- Pre-commit hook support
- Configurable severity thresholds
```