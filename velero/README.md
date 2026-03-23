# velero 적용 방법

이 디렉터리는 `velero` 네임스페이스에 배포되는 Velero 서버와 node-agent 구성을 위한 Helm values, 그리고 참고용 Backup, Schedule, Restore YAML을 관리합니다. 실제 백업 실행은 클러스터 내부의 Velero Pod가 수행하지만, 운영자는 MGMT 서버에서 `velero` CLI를 사용해 백업과 복구 명령을 실행하는 흐름을 기준으로 합니다.

## 적용 순서

```powershell
kubectl apply -f .\velero\01-namespace.yaml
kubectl apply -f .\velero\02-lyh-secret-ncr.yaml
helm upgrade --install velero vmware-tanzu/velero -n velero --create-namespace -f .\velero\03-values-private.yaml
```

`04-backup.yaml`, `05-schedule.yaml`, `06-restore.yaml`은 참고용 예시 파일로 두고, 실제 운영에서는 MGMT 서버의 `velero` CLI 명령으로 백업과 복구를 실행하는 방식을 권장합니다.

## MGMT 서버에서 Velero CLI 설치

MGMT 서버에서 아래 명령으로 Velero CLI를 설치합니다.

```bash
wget https://github.com/vmware-tanzu/velero/releases/download/v1.13.0/velero-v1.13.0-linux-amd64.tar.gz
tar -xvf velero-v1.13.0-linux-amd64.tar.gz
sudo mv velero-v1.13.0-linux-amd64/velero /usr/local/bin/

velero version --client-only
```

## MGMT 서버에서 백업 생성

Velero 서버와 BackupStorageLocation이 이미 준비되어 있다면, MGMT 서버에서 아래 명령으로 백업을 생성하고 완료까지 대기할 수 있습니다.

```bash
velero backup create backup \
  --include-namespaces "*" \
  --exclude-namespaces kube-system,velero,kube-public,kube-node-lease \
  --include-cluster-resources=true \
  --storage-location default \
  --wait
```

이 방식은 클러스터 내부에 배포된 Velero 서버 Pod로 백업 작업을 요청하는 형태이며, 실제 데이터 수집과 업로드는 Velero 서버와 node-agent가 수행합니다.

## 참고 YAML

- `04-backup.yaml`: 수동 Backup 리소스 예시
- `05-schedule.yaml`: 정기 백업 Schedule 예시
- `06-restore.yaml`: 복원 Restore 예시

운영 환경에서는 위 YAML을 그대로 적용하기보다, MGMT 서버에서 `velero backup create`, `velero restore create` 명령으로 실행하는 방식을 우선 기준으로 삼는 편이 관리에 유리합니다.
