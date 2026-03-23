이 네임스페이스는 Harbor를 클러스터 내부에 배포하는 경우를 위한 공간입니다. 이미지 pull에 사용하는 공통 secret도 함께 관리합니다.

적용 순서는 아래와 같습니다.

```powershell
kubectl apply -f .\harbor\01-namespace.yaml
kubectl apply -f .\harbor\02-lyh-secret-ncr.yaml
```

`02-lyh-secret-ncr.yaml`의 `.dockerconfigjson` 값은 실제 NCR 인증 값으로 교체해야 합니다.
