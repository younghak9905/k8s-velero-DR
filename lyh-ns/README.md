# lyh-ns 적용 방법

이 디렉터리는 애플리케이션 네임스페이스인 `lyh-ns`를 위한 리소스를 관리합니다. `web`, `was`, `db` 애플리케이션과 `ReferenceGrant`, `StorageClass`, 이미지 pull secret이 모두 여기에 포함되어 있습니다.

적용 순서는 아래와 같습니다.

```powershell
kubectl apply -f .\lyh-ns\01-namespace.yaml
kubectl apply -f .\lyh-ns\02-lyh-secret-ncr.yaml
kubectl apply -f .\lyh-ns\03-db-secret.yaml
kubectl apply -f .\lyh-ns\04-storageclass.yaml
kubectl apply -f .\lyh-ns\05-db.yaml
kubectl apply -f .\lyh-ns\06-web-configmap.yaml
kubectl apply -f .\lyh-ns\07-was.yaml
kubectl apply -f .\lyh-ns\08-web.yaml
kubectl apply -f .\lyh-ns\09-referencegrant.yaml
```

`02-lyh-secret-ncr.yaml`의 `.dockerconfigjson` 값은 실제 NCR 인증 정보로 바꾼 뒤 적용해야 합니다. 이 secret은 `web`, `was`, `db` Pod의 `imagePullSecrets`에서 공통으로 사용됩니다.

`ReferenceGrant`는 파일을 어디에 저장하든 실제 리소스의 `namespace`가 `lyh-ns`여야 합니다. 이 리소스는 `nginx-gateway`가 `lyh-ns`의 Service를 참조하도록 허용하는 권한이므로, 서비스 소유권 관점에서는 `lyh-ns`에서 관리하는 편이 더 안전합니다.
