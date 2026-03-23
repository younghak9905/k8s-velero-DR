이 네임스페이스는 Gateway API 구현체를 설치하기 위한 공간입니다. NGINX Gateway Fabric을 사용하는 경우 컨트롤러 구성 요소를 `nginx` 네임스페이스에 배포하고, 이후 `nginx-gateway` 디렉터리의 리소스를 적용하기 전에 `GatewayClass` 이름이 `nginx`로 준비되어 있는지 확인해야 합니다.

적용 순서는 아래와 같습니다.

```powershell
kubectl apply -f .\nginx\01-namespace.yaml
kubectl apply -f .\nginx\02-lyh-secret-ncr.yaml
helm upgrade --install nginx-gateway ngf/nginx-gateway-fabric -n nginx --create-namespace -f .\nginx-gateway\03-values-private.yaml
```

`02-lyh-secret-ncr.yaml`의 `.dockerconfigjson` 값은 실제 NCR 인증 값으로 교체해야 합니다.
