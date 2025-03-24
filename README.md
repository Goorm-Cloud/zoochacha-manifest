# Zoochacha 인프라 Helm 차트

이 Helm 차트는 Zoochacha 애플리케이션을 위한 인프라 구성 요소를 배포합니다.

## 구성 요소

차트에는 다음 구성 요소가 포함되어 있습니다:

- **cert-manager**: TLS 인증서 자동 발급 및 관리
- **MySQL DB**: 애플리케이션 데이터 저장소
- **ArgoCD**: GitOps를 통한 애플리케이션 배포 자동화

## 사전 요구 사항

- Kubernetes 1.19+
- Helm 3.0.0+
- kubectl 명령줄 도구
- 쿠버네티스 클러스터 관리자 권한

## 설치 방법

### 1. 비밀 설정 파일 준비

먼저 민감한 정보를 담은 `secrets.yaml` 파일을 생성합니다. (제공된 템플릿 참조)

```yaml
# secrets.yaml 예시
secrets:
  aws:
    accessKeyID: "YOUR_AWS_ACCESS_KEY"
    secretAccessKey: "YOUR_AWS_SECRET_KEY"
    roleArn: "YOUR_AWS_ROLE_ARN"
  database:
    username: "db_username"
    password: "db_password"
    rootPassword: "db_root_password"
  argocd:
    githubToken: "YOUR_GITHUB_TOKEN"
```

> **중요**: `secrets.yaml` 파일은 버전 관리 시스템에 포함되지 않도록 주의하세요. 기본적으로 `.gitignore`에 설정되어 있습니다.

### 2. Helm 저장소 설정

```bash
# 차트 저장소 업데이트
helm repo add jetstack https://charts.jetstack.io
helm repo update
```

### 3. 의존성 업데이트

```bash
# 차트 디렉토리로 이동
cd zoochacha-manifest

# 의존성 업데이트
helm dependency update
```

### 4. 인프라 설치

```bash
# Helm 차트 설치 (비밀 정보 포함)
helm install zoochacha-infra ./zoochacha-manifest -f values.yaml -f secrets.yaml
```

### 5. 사용자 정의 설치

```bash
# 추가적인 사용자 정의 values.yaml 파일 사용
helm install zoochacha-infra ./zoochacha-manifest -f values.yaml -f secrets.yaml -f custom-values.yaml
```

## 설치 순서

이 Helm 차트는 다음 순서로 구성 요소를 설치합니다:

1. 네임스페이스 생성 (자동)
2. RBAC 설정 (ServiceAccount, ClusterRole, ClusterRoleBinding)
3. cert-manager 설치 및 확인
4. MySQL DB 설치 (cert-manager 준비 확인 후)
5. ArgoCD 설정 (기존 ArgoCD 연결)

## 참고사항

- 이 차트는 기존 EKS 클러스터의 ArgoCD와 연결되도록 구성되어 있습니다.
- 기본적으로 네임스페이스 충돌을 방지하기 위해 기존 네임스페이스를 재사용합니다.
- ArgoCD 리포지토리 시크릿은 기본적으로 생성하지 않도록 설정되어 있습니다 (skipRepositorySecrets: true).

## 업그레이드 방법

```bash
# 차트 업그레이드 (비밀 정보 포함)
helm upgrade zoochacha-infra ./zoochacha-manifest -f values.yaml -f secrets.yaml
```

## 삭제 방법

```bash
# 차트 삭제
helm uninstall zoochacha-infra
```

## 사용자 정의

`values.yaml` 파일에서 다음 설정을 사용자 정의할 수 있습니다:

- 네임스페이스 생성 여부
- cert-manager 설정 (DNS 챌린지)
- MySQL 데이터베이스 설정 (사용자, 비밀번호, 저장소)
- ArgoCD 설정 (애플리케이션 소스)

## 보안 정보 관리

민감한 정보는 `secrets.yaml` 파일에 별도로 관리하며, 다음과 같은 항목을 포함합니다:

- AWS 접근 키 및 시크릿 키
- 데이터베이스 사용자 이름 및 비밀번호
- GitHub 접근 토큰
- 기타 API 키 및 인증 정보

이러한 비밀 정보는 Kubernetes Secret으로 저장되거나 해당 구성요소의 설정 값으로 사용됩니다.

## 라이선스

MIT License 