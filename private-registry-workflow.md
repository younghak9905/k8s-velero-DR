# 사설망 레지스트리 작업 절차

이 프로젝트는 `nginx-gateway`, `kyverno`, `velero`를 사설망 환경에 배포하는 상황을 기준으로 합니다. 외부 네트워크가 차단된 클러스터에서는 Helm chart를 직접 내려받아 설치하기 어렵기 때문에, 연결이 가능한 환경에서 chart를 먼저 준비한 뒤 YAML로 전개하여 가져오는 방식이 필요합니다.

기본 절차는 다음과 같습니다. 먼저 외부 네트워크가 가능한 환경에서 Helm chart를 내려받습니다. 그다음 chart를 YAML로 렌더링합니다. 렌더링된 YAML 안에서 사용 중인 모든 이미지 경로를 추출하고, 해당 이미지를 내부 레지스트리로 업로드합니다. 이후 YAML 파일 안의 이미지 경로를 내부 레지스트리 주소로 치환합니다. 마지막으로 수정된 YAML을 이 저장소에 보관하고 `kubectl apply -f`로 사설망 클러스터에 배포합니다.

## 디렉터리 사용 방식

`nginx-gateway` 디렉터리에는 Gateway 컨트롤러 설치용 렌더링 YAML이나 프로젝트에서 직접 사용하는 Gateway API 리소스를 둡니다. `kyverno` 디렉터리에는 Kyverno 설치용 YAML과 정책 파일을 둡니다. `velero` 디렉터리에는 Velero 설치용 YAML과 백업 관련 사용자 정의 리소스를 둡니다.

## 렌더링 예시 명령

```powershell
helm repo add ngf https://nginx.github.io/nginx-gateway-fabric
helm repo add kyverno https://kyverno.github.io/kyverno
helm repo add vmware-tanzu https://vmware-tanzu.github.io/helm-charts

helm template nginx-gateway ngf/nginx-gateway-fabric -n nginx > nginx-gateway\rendered.yaml
helm template kyverno kyverno/kyverno -n kyverno > kyverno\rendered.yaml
helm template velero vmware-tanzu/velero -n velero > velero\rendered.yaml
```

이 저장소에는 바로 적용 가능한 private 환경용 values 파일도 함께 두었습니다.

```powershell
helm upgrade --install kyverno kyverno/kyverno -n kyverno --create-namespace -f .\kyverno\03-values-private.yaml
helm upgrade --install nginx-gateway ngf/nginx-gateway-fabric -n nginx --create-namespace -f .\nginx-gateway\03-values-private.yaml
helm upgrade --install velero vmware-tanzu/velero -n velero --create-namespace -f .\velero\03-values-private.yaml
```

`velero/03-values-private.yaml`은 NHN Cloud Object Storage처럼 S3 호환 스토리지를 사용하는 구성을 기준으로 작성했습니다. 적용 전에 `CHANGE_ME` 값은 실제 접근 정보와 버킷 이름으로 바꾸어야 합니다.

## 이미지 추출 예시

렌더링된 YAML을 기준 자료로 삼아 이미지 경로를 확인합니다.

```powershell
Select-String -Path nginx-gateway\rendered.yaml,kyverno\rendered.yaml,velero\rendered.yaml -Pattern 'image:' | ForEach-Object { $_.Line.Trim() }
```

## 이미지 치환 예시

예를 들어 `ghcr.io/nginxinc/nginx-gateway-fabric:*`, `ghcr.io/kyverno/*:*`, `docker.io/velero/*:*` 같은 외부 이미지는 내부 레지스트리 주소로 바꾸어야 합니다. 또한 chart에서 사용하는 init container나 plugin 이미지도 빠짐없이 확인해야 합니다. 치환 대상은 예를 들어 `harbor.example.com/platform/nginx-gateway-fabric:*`, `harbor.example.com/platform/kyverno/*:*`, `harbor.example.com/platform/velero/*:*` 같은 형태가 됩니다.

## 주의 사항

사설망 클러스터에서 외부 접근이 차단되어 있다면 Helm install에 직접 의존하지 않는 편이 안전합니다. 이미지 경로를 바꾼 뒤의 렌더링 YAML은 반드시 버전 관리에 포함해 두는 것이 좋습니다. 또한 chart 버전이 바뀌면 사용 이미지 목록도 달라질 수 있으므로, 버전 변경 시마다 이미지 추출과 치환 과정을 다시 확인해야 합니다.
