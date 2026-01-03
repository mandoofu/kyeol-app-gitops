# 📦 kyeol-app-gitops

> **KYEOL Saleor 프로젝트의 애플리케이션(Storefront, Dashboard) Kubernetes 매니페스트를 관리하는 GitOps 레포지토리**

---

## 📌 이 레포는 무엇을 하는가

**Kustomize**를 사용하여 환경별(DEV/STAGE/PROD) Storefront 및 Dashboard 배포 매니페스트를 관리합니다.

**관리 대상**:
- Deployment (Pod 배포)
- Service (ClusterIP)
- Ingress (ALB)
- HPA (Horizontal Pod Autoscaler)

---

## 👤 언제 / 누가 / 왜 사용하는가

| 상황 | 사용 여부 |
|------|:--------:|
| Storefront/Dashboard 배포 | ✅ 사용 |
| 이미지 태그 변경 | ✅ 사용 |
| 레플리카 수, 리소스 조정 | ✅ 사용 |
| AWS 인프라 생성/수정 | ❌ 미사용 (kyeol-infra-terraform 사용) |
| Helm Addon 설치 | ❌ 미사용 (kyeol-platform-gitops 사용) |

---

## 🏛️ 전체 아키텍처에서의 위치

```
[kyeol-storefront / kyeol-saleor-dashboard]
    ↓ (GitHub Actions → ECR Push)
[ECR] min-kyeol-*-storefront, min-kyeol-*-dashboard
    ↓ (이미지 참조)
[이 레포] kyeol-app-gitops
    ↓ (kubectl apply -k)
[EKS] Pods, Services, Ingress
    ↓ (ALB Controller + ExternalDNS)
[인터넷] HTTPS 접속 가능
```

---

## 📁 주요 디렉터리/파일 설명

```
kyeol-app-gitops/
├── apps/
│   ├── saleor/                      # Storefront 앱
│   │   ├── base/                    # 공통 매니페스트
│   │   │   ├── deployment-storefront.yaml
│   │   │   ├── service-storefront.yaml
│   │   │   ├── ingress.yaml
│   │   │   └── kustomization.yaml
│   │   └── overlays/                # 환경별 오버레이
│   │       ├── dev/
│   │       ├── stage/
│   │       └── prod/
│   └── saleor-dashboard/            # Dashboard 앱
│       ├── base/
│       └── overlays/
│           ├── dev/
│           ├── stage/
│           └── prod/
└── argocd/                          # ArgoCD Application 매니페스트
    └── applications/
```

### overlays/ 디렉터리 파일 구성

| 파일 | 역할 |
|------|------|
| `kustomization.yaml` | 이미지, 레이블, 패치 정의 |
| `patches/replicas-patch.yaml` | 레플리카 수 조정 |
| `patches/resources-patch.yaml` | CPU/Memory 리소스 조정 |
| `patches/ingress-patch.yaml` | 호스트, ACM ARN 설정 |
| `patches/hpa-patch.yaml` | HPA 스케일링 정책 |

---

## ⚠️ 이 레포를 직접 만질 때 주의사항

### 🔧 배포 전 필수 확인

1. **ACM ARN 실제 값으로 설정**
   ```yaml
   # ingress-patch.yaml
   alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:...
   ```

2. **ECR 이미지 경로 확인**
   ```yaml
   # kustomization.yaml
   images:
     - name: ...
       newName: 827913617839.dkr.ecr.../min-kyeol-stage-storefront
       newTag: stage-latest
   ```

### 🚫 절대 하지 말아야 할 것

1. **base/ 디렉터리 직접 수정 금지**
   - 환경별 변경은 overlays/ 패치로 처리

2. **PROD에 검증 안 된 이미지 배포 금지**
   - 반드시 STAGE에서 먼저 검증

---

## 🔗 다른 레포와의 관계

| 레포지토리 | 관계 |
|-----------|------|
| kyeol-infra-terraform | **이 레포 실행 전** EKS 클러스터 생성 필요 |
| kyeol-platform-gitops | **이 레포 실행 전** ALB Controller 설치 필요 |
| kyeol-storefront | 이미지 소스 (ECR로 Push) |
| kyeol-saleor-dashboard | 이미지 소스 (ECR로 Push) |

---

## 🚀 빠른 시작

```powershell
# STAGE 배포 예시
kubectl apply -k apps/saleor/overlays/stage --context stage
kubectl apply -k apps/saleor-dashboard/overlays/stage --context stage

# 배포 확인
kubectl -n kyeol get pods,svc,ingress --context stage
```

---

## 📝 ECR 레포지토리 매핑

| 환경 | Storefront ECR | Dashboard ECR |
|:----:|---------------|--------------|
| DEV | min-kyeol-storefront | min-kyeol-dashboard |
| STAGE | min-kyeol-stage-storefront | min-kyeol-stage-dashboard |
| PROD | min-kyeol-prod-storefront | min-kyeol-prod-dashboard |

---

> **마지막 업데이트**: 2026-01-03
