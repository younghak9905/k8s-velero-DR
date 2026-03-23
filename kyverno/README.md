사설망 환경에서는 Kyverno를 Helm chart로 직접 설치하기보다, 외부 환경에서 chart를 YAML로 렌더링한 뒤 그 결과물을 가져와 배포하는 방식이 적합합니다. 렌더링된 YAML에서 사용하는 이미지 경로를 모두 추출해 내부 레지스트리에 업로드하고, YAML 안의 이미지 주소를 내부 레지스트리 경로로 수정한 다음 배포해야 합니다. 설치가 끝난 뒤에는 `clusterpolicy-rewrite-registry.yaml` 정책을 추가로 적용하여 Pod 생성 시 이미지 경로를 원하는 레지스트리로 치환하도록 구성할 수 있습니다.

적용 순서는 아래와 같습니다.

```powershell
kubectl apply -f .\kyverno\01-namespace.yaml
kubectl apply -f .\kyverno\02-lyh-secret-ncr.yaml
helm upgrade --install kyverno kyverno/kyverno -n kyverno --create-namespace -f .\kyverno\03-values-private.yaml
kubectl apply -f .\kyverno\04-clusterpolicy-rewrite-registry.yaml
```

`02-lyh-secret-ncr.yaml`의 `.dockerconfigjson`은 실제 NCR 인증 값으로 바꿔야 합니다.
