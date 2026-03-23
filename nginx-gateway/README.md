# nginx-gateway 적용 방법

이 디렉터리는 `nginx-gateway` 네임스페이스에서 사용하는 Gateway API 리소스를 관리합니다. 실제 NGINX Gateway Fabric 컨트롤러는 `nginx` 네임스페이스에 설치하고, 이 디렉터리에서는 그 컨트롤러가 참조할 `Gateway`, `HTTPRoute` 리소스를 적용합니다.

파일 순서는 아래 기준으로 정리되어 있습니다.

- `01-namespace.yaml`: `nginx-gateway` 네임스페이스 생성
- `02-lyh-secret-ncr.yaml`: 내부 레지스트리 pull secret 생성
- `03-values-private.yaml`: NGINX Gateway Fabric Helm values
- `04-gateway.yaml`: Gateway 리소스 생성
- `05-httproute.yaml`: HTTPRoute 리소스 생성

적용 순서는 아래와 같습니다.

```powershell
kubectl apply -f .\nginx-gateway\01-namespace.yaml
kubectl apply -f .\nginx-gateway\02-lyh-secret-ncr.yaml
helm upgrade --install nginx-gateway ngf/nginx-gateway-fabric -n nginx --create-namespace -f .\nginx-gateway\03-values-private.yaml
kubectl apply -f .\nginx-gateway\04-gateway.yaml
kubectl apply -f .\nginx-gateway\05-httproute.yaml
```

주의할 점은 `03-values-private.yaml`은 `nginx-gateway` 디렉터리에 있지만 실제 Helm 설치 대상 네임스페이스는 `nginx`라는 점입니다. 또한 `04-gateway.yaml`의 `gatewayClassName`, `05-httproute.yaml`의 hostname과 backend 참조는 실제 클러스터 환경에 맞게 조정해야 합니다.
