# Kyverno 정책 검증용 리소스

이 파일들은 `04-clusterpolicy-rewrite-registry.yaml` 정책이 정상적으로 동작하는지 확인하기 위한 테스트 리소스입니다. 모두 `{target_namespace}` 네임스페이스에 생성되도록 작성했습니다.

## 파일별 목적

- `05-test-pod-no-pullsecret.yaml`: `imagePullSecrets`가 없는 Pod에 `lyh-secret-ncr`이 자동 주입되는지 확인
- `06-test-deployment-empty-pullsecret.yaml`: `imagePullSecrets: []` 상태인 Deployment 템플릿에 `lyh-secret-ncr`이 추가되는지 확인
- `07-test-statefulset-existing-pullsecret.yaml`: 기존 `imagePullSecrets`가 있을 때 `lyh-secret-ncr`이 함께 유지되는지 확인
- `08-test-statefulset-service.yaml`: StatefulSet 생성을 위한 Headless Service

## 먼저 바꿔야 할 placeholder

- `{source_registry}`: 변경 전 레지스트리 주소
- `{target_registry}`: 변경 후 레지스트리 주소
- `{target_namespace}`: 테스트할 네임스페이스
- `{image_repository}`: 테스트용 이미지 저장소 경로
- `{image_tag}`: 테스트용 이미지 태그
- `{image_pull_secret_name}`: 주입할 imagePullSecret 이름
- `{existing_image_pull_secret_name}`: 기존에 들어 있다고 가정할 secret 이름
- `{test_response_text}`: Deployment 테스트 응답 문자열

## 적용 순서

```powershell
kubectl apply -f .\kyverno\05-test-pod-no-pullsecret.yaml
kubectl apply -f .\kyverno\06-test-deployment-empty-pullsecret.yaml
kubectl apply -f .\kyverno\08-test-statefulset-service.yaml
kubectl apply -f .\kyverno\07-test-statefulset-existing-pullsecret.yaml
```

## 확인 방법

```powershell
kubectl get pod kyverno-test-pod-no-pullsecret -n {target_namespace} -o yaml
kubectl get deploy kyverno-test-deployment-empty-pullsecret -n {target_namespace} -o yaml
kubectl get sts kyverno-test-statefulset-existing-pullsecret -n {target_namespace} -o yaml
```

아래 항목을 확인하면 됩니다.

- 이미지가 `{source_registry}`에서 `{target_registry}` 기준으로 변경되었는지
- `imagePullSecrets`에 `{image_pull_secret_name}`이 존재하는지
- StatefulSet의 경우 기존 `{existing_image_pull_secret_name}`가 유지되는지

## 정리 명령

```powershell
kubectl delete -f .\kyverno\05-test-pod-no-pullsecret.yaml --ignore-not-found=true
kubectl delete -f .\kyverno\06-test-deployment-empty-pullsecret.yaml --ignore-not-found=true
kubectl delete -f .\kyverno\07-test-statefulset-existing-pullsecret.yaml --ignore-not-found=true
kubectl delete -f .\kyverno\08-test-statefulset-service.yaml --ignore-not-found=true
```
