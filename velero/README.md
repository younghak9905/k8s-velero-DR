사설망 환경에서는 Velero도 Helm chart를 직접 설치하기보다, 외부 환경에서 chart를 YAML로 렌더링해 가져오는 방식으로 관리하는 것이 적합합니다. 렌더링된 YAML에서 Velero 서버 이미지, node-agent 이미지, 플러그인 이미지를 모두 확인한 뒤 내부 레지스트리로 업로드하고, YAML 안의 이미지 경로를 내부 레지스트리 주소로 수정해서 배포해야 합니다. Velero 설치가 끝난 후에는 이 디렉터리에 있는 백업 관련 사용자 정의 리소스를 순서대로 적용하면 됩니다.

적용 순서는 아래와 같습니다.

```powershell
kubectl apply -f .\velero\01-namespace.yaml
kubectl apply -f .\velero\02-lyh-secret-ncr.yaml
helm upgrade --install velero vmware-tanzu/velero -n velero --create-namespace -f .\velero\03-values-private.yaml
kubectl apply -f .\velero\04-backup.yaml
kubectl apply -f .\velero\05-schedule.yaml
```

복원 테스트가 필요할 때만 `06-restore.yaml`을 적용합니다.
