# Velero 기반 Private Kubernetes DR 설계

이 저장소는 Private Kubernetes 환경에서 3-tier 애플리케이션을 배포하고, Gateway API로 외부 라우팅을 구성하며, Velero와 Kyverno를 이용해 백업 및 이미지 레지스트리 정책을 검증하기 위한 실습형 설계 저장소입니다. 전체 구조는 실제 운영 순서를 기준으로 정리되어 있으며, 각 네임스페이스 디렉터리 안의 파일도 `01-...`, `02-...` 형식으로 배포 순서가 드러나도록 구성했습니다.

## 프로젝트 개요

이 프로젝트의 핵심 목적은 세 가지입니다. 첫째, `lyh-ns` 네임스페이스에 `web`, `was`, `db`로 구성된 3-tier 애플리케이션을 배포합니다. 둘째, `ingress-nginx` 대신 Gateway API를 사용해 `nginx-gateway` 네임스페이스에서 외부 라우팅을 관리합니다. 셋째, Private Registry 환경을 전제로 Velero 백업과 Kyverno 이미지 rewrite 정책을 함께 검증합니다.

이 저장소는 특히 다음 상황을 염두에 두고 작성했습니다.

- 외부 네트워크가 차단된 Private Kubernetes 환경
- Helm chart를 직접 내려받기 어려운 운영 환경
- 내부 NCR 같은 사설 레지스트리 사용
- Velero 기반 백업 및 복구 체계 검증
- Kyverno 기반 이미지 경로 통제 및 pull secret 주입 검증

## 아키텍처

트래픽 흐름은 `Gateway API -> HTTPRoute -> Service(web/was) -> Pod` 구조입니다. 데이터 계층인 DB는 `StatefulSet`으로 구성했고, 다른 애플리케이션보다 먼저 배포되도록 순서를 조정했습니다. Gateway 관련 리소스는 `nginx-gateway`에서 관리하지만, `ReferenceGrant`는 서비스 소유 네임스페이스인 `lyh-ns`에서 관리합니다. 이유는 이 리소스가 외부 네임스페이스의 라우팅 리소스가 `lyh-ns`의 Service를 참조해도 된다는 허용 선언이기 때문입니다.

## 디렉터리 구성

- `lyh-ns`: 3-tier 애플리케이션, DB Secret, StorageClass, ReferenceGrant
- `nginx`: NGINX Gateway Fabric 컨트롤러 설치용 네임스페이스
- `nginx-gateway`: Gateway, HTTPRoute, 컨트롤러용 Helm values
- `kyverno`: Kyverno 설치 values, ClusterPolicy, 정책 검증용 테스트 YAML
- `velero`: Velero 설치 values, Backup, Schedule, Restore
- `문서`: 프로젝트 관련 산출 문서

## 배포 원칙

배포 순서는 `kyverno`를 먼저 적용하는 것을 기준으로 합니다. 이미지 경로 rewrite와 `imagePullSecrets` 자동 주입 정책이 먼저 준비되어 있어야 이후 Helm 기반 컴포넌트와 애플리케이션 배포 시 정책 반영 여부를 일관되게 검증할 수 있기 때문입니다.

각 네임스페이스는 공통 image pull secret인 `lyh-secret-ncr`을 먼저 생성한 뒤 배포합니다. `nginx-gateway`, `kyverno`, `velero`는 Private 환경을 고려해 values 파일을 별도로 두었고, 바로 적용 가능한 형태로 정리했습니다. `lyh-ns`는 DB를 먼저 배포한 뒤 web, was, ReferenceGrant 순서로 적용합니다.

대표적인 적용 흐름은 아래와 같습니다.

```powershell
kubectl apply -f .\kyverno\
kubectl apply -f .\nginx\
kubectl apply -f .\nginx-gateway\
kubectl apply -f .\lyh-ns\
kubectl apply -f .\velero\
```

각 디렉터리의 README에는 그 네임스페이스 기준 적용 순서와 실제 명령을 따로 정리해 두었습니다.

## Private Registry 운영 방식

`nginx-gateway`, `kyverno`, `velero`는 Helm chart를 그대로 외부에서 설치하는 방식이 아니라, chart를 렌더링한 뒤 이미지 경로를 내부 레지스트리 기준으로 바꾸어 배포하는 방식을 기준으로 합니다. 이 과정에는 이미지 목록 추출, 내부 레지스트리 업로드, values 또는 렌더링 YAML 수정, 최종 `kubectl apply` 또는 `helm upgrade --install` 적용이 포함됩니다.

자세한 절차는 [private-registry-workflow.md]에 정리되어 있습니다.

## Velero와 Kyverno

Velero는 `Deployment` 형태의 서버 Pod와 `node-agent` 기반 파일 백업 구조를 전제로 작성했습니다. 객체 스토리지는 S3 호환 스토리지를 기준으로 values 파일이 준비되어 있으며, 실제 백업 실행은 MGMT 서버에서 `velero` CLI를 통해 수행하는 운영 흐름을 기준으로 문서를 정리했습니다. Kyverno는 Pod 생성 시 이미지 레지스트리 rewrite와 `imagePullSecrets` 자동 주입을 검증할 수 있도록 `ClusterPolicy`와 테스트 YAML을 함께 제공합니다.

## 확인 포인트

배포 후 아래 항목을 우선 확인하면 됩니다.

```powershell
kubectl get clusterpolicy
kubectl get all -n nginx
kubectl get gateway,httproute -n nginx-gateway
kubectl get referencegrant -n lyh-ns
kubectl get all -n lyh-ns
kubectl get all -n velero
```
