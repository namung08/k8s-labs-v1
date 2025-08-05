# DDoddo Market Helm Chart

DDoddo Market 백엔드 애플리케이션을 위한 Helm 차트입니다.

## 설치 방법

### 1. 기본 설치 (개발 환경)

```bash
helm install ddoddo-market .
```

### 2. GitHub Actions를 사용한 자동 배포 (권장)

#### GitHub Secrets 설정

GitHub 저장소에서 다음 Secrets를 설정하세요:

1. **Settings → Secrets and variables → Actions**
2. **New repository secret** 클릭하여 다음 값들을 추가:

```
DB_PASSWORD=your-actual-password
JWT_SECRET_KEY=your-actual-jwt-secret
CLOUDFLARE_ACCESS_KEY_ID=your-actual-access-key
CLOUDFLARE_SECRET_ACCESS_KEY=your-actual-secret-key
CLOUDFLARE_ACCOUNT_ID=your-actual-account-id
CLOUDFLARE_R2_PUBLIC_URL=your-actual-public-url
KUBE_CONFIG=base64-encoded-kubeconfig
```

#### 자동 배포

main 브랜치에 push하면 자동으로 배포됩니다:

```bash
git push origin main
```

### 3. 외부 Secret 파일 사용

1. `secrets.yaml` 파일을 생성하고 실제 secret 값들을 입력합니다:

```yaml
secrets:
  dbPassword: "your-actual-password"
  jwtSecretKey: "your-actual-jwt-secret"
  cloudflareAccessKeyId: "your-actual-access-key"
  cloudflareSecretAccessKey: "your-actual-secret-key"
  # ... 기타 secret 값들
```

2. Helm 설치 시 외부 파일을 사용:

```bash
helm install ddoddo-market . -f secrets.yaml
```

### 4. Helm 명령어로 직접 전달

```bash
helm install ddoddo-market . \
  --set secrets.dbPassword=your-password \
  --set secrets.jwtSecretKey=your-jwt-secret \
  --set secrets.cloudflareAccessKeyId=your-access-key
```

### 5. 환경 변수 사용

```bash
export DB_PASSWORD="your-password"
export JWT_SECRET_KEY="your-jwt-secret"

helm install ddoddo-market . \
  --set secrets.dbPassword=$DB_PASSWORD \
  --set secrets.jwtSecretKey=$JWT_SECRET_KEY
```

## 구성 옵션

### 주요 설정

| 매개변수                      | 설명                 | 기본값                    |
| ----------------------------- | -------------------- | ------------------------- |
| `deployment.replicas`         | 배포할 Pod 개수      | `1`                       |
| `deployment.image.repository` | Docker 이미지 저장소 | `namung08/ddoddo-backend` |
| `deployment.image.tag`        | Docker 이미지 태그   | `v1.0`                    |
| `service.type`                | 서비스 타입          | `ClusterIP`               |
| `ingress.enabled`             | Ingress 활성화 여부  | `true`                    |

### Secret 관리

민감한 정보는 다음 방법으로 관리할 수 있습니다:

1. **GitHub Actions + GitHub Secrets** (권장)
2. **외부 파일 사용**
3. **Helm 명령어로 전달**
4. **환경 변수 사용**

### Health Check 설정

애플리케이션의 헬스 체크를 위한 probe 설정:

```yaml
deployment:
  startupProbe:
    path: "/actuator/health"
    initialDelaySeconds: 20
    periodSeconds: 10
    failureThreshold: 12

  livenessProbe:
    path: "/actuator/health/liveness"
    initialDelaySeconds: 30
    periodSeconds: 20

  readinessProbe:
    path: "/actuator/health/readiness"
    initialDelaySeconds: 15
    periodSeconds: 10
```

## 보안 주의사항

1. `secrets.yaml` 파일은 `.gitignore`에 포함되어 있어 Git에 올라가지 않습니다.
2. GitHub Secrets를 사용하면 민감한 정보가 로그에 노출되지 않습니다.
3. 실제 운영 환경에서는 더 안전한 secret 관리 도구를 사용하세요:
   - HashiCorp Vault
   - AWS Secrets Manager
   - Azure Key Vault
   - Google Secret Manager

## 개발 환경 설정

개발 환경에서는 기본값을 사용할 수 있습니다:

```bash
helm install ddoddo-market . --set deployment.replicas=1
```

## 운영 환경 설정

운영 환경에서는 반드시 실제 secret 값들을 제공해야 합니다:

```bash
helm install ddoddo-market . \
  -f production-secrets.yaml \
  --set deployment.replicas=3 \
  --set deployment.resources.limits.cpu=1000m \
  --set deployment.resources.limits.memory=1Gi
```

## 업그레이드

```bash
helm upgrade ddoddo-market . -f secrets.yaml
```

## 제거

```bash
helm uninstall ddoddo-market
```

## 문제 해결

### Secret 관련 문제

Secret이 제대로 로드되지 않는 경우:

1. Secret 값이 올바른지 확인
2. `kubectl get secrets`로 Secret이 생성되었는지 확인
3. Pod 로그 확인: `kubectl logs <pod-name>`

### Health Check 실패

Health check가 실패하는 경우:

1. 애플리케이션이 `/actuator/health` 엔드포인트를 제공하는지 확인
2. Spring Boot Actuator가 활성화되어 있는지 확인
3. Pod 로그에서 애플리케이션 시작 상태 확인

### GitHub Actions 문제

GitHub Actions 배포가 실패하는 경우:

1. GitHub Secrets가 올바르게 설정되었는지 확인
2. `KUBE_CONFIG` secret이 base64로 인코딩되었는지 확인
3. Actions 로그에서 오류 메시지 확인
