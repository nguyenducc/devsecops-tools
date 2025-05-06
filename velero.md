# Velero
## Introduction
Velero (formerly known as Heptio Ark) is an open-source tool that enables backup, restore, and migration of Kubernetes cluster resources and persistent volumes. It helps ensure system resilience and supports disaster recovery strategies for cloud-native applications.
## Architect
Velero is composed of two main components:
- Server: Runs inside the Kubernetes cluster, responsible for handling backup, restore, and synchronization operations with cloud object storage.

- CLI (Command Line Interface): Runs locally and is used to interact with Velero—triggering backups, initiating restores, and scheduling operations.

Velero is built on a controller-based architecture. It defines each operation (backup, restore, etc.) as a Custom Resource (CR), which is stored in etcd. Velero controllers watch and act upon these resources.

<p align="center">
  <img src="./img/backup-process.png" alt="Image" style="width: 100%; max-width: 1000px;">
</p>

## Supported Operations
✅ On-demand Backup

- Creates a copy of Kubernetes resources as a tarball and uploads it to cloud object storage.

⚠️ Note: Backup is not atomic—consistency is not guaranteed at a single point in time.

⏰ Scheduled Backup

- Allows you to schedule backups at specific times or intervals using Cron expressions.

- Example: 0 0 * * * – takes a backup every day at midnight.

♻️ Restore
- Re-creates Kubernetes resources from definitions stored in cloud object storage.

- Best Practice: Set the backup storage location to Read-Only during restore to prevent accidental overwrites or deletions.

## Installation Steps

1. Configure docker-compose.yml to Deploy MinIO

```sh
version: '3'
services:
  minio:
    image: minio/minio
    container_name: minio
    ports:
      - "9000:9000"
      - "9001:9001"
    volumes:
      - ./storage:/data
    environment:
      MINIO_ROOT_USER: minio
      MINIO_ROOT_PASSWORD: minioadmin
    command: server --console-address ":9001" /data
```
Run with: 
```sh
docker-compose up -d
```
Then access MinIO at: http://minio-server-ip:9001

Login using minio/minioadmin

2. Install Velero Client

Perform this step on k8s-master-1 server or from Rancher’s kubectl shell.

```sh
wget https://github.com/vmware-tanzu/velero/releases/download/v1.15.0/velero-v1.15.0-linux-amd64.tar.gz

tar -xvf velero-v1.15.0-linux-amd64.tar.gz

sudo mv velero-v1.15.0-linux-amd64/velero /usr/local/bin
```
3. Create a Bucket and Access Keys in MinIO

4. Set MinIO Environment Variables

Perform this step on k8s-master-1 server or from Rancher’s kubectl shell.
```sh
export MINIO_URL="http://<minio-server-ip>:9000"
export MINIO_ACCESS_KEY_ID="BUCKET_ACCESS_KEY"
export MINIO_SECRET_KEY_ID="BUCKET_SECRET_KEY"
export MINIO_BUCKET="BUCKET_NAME"
```
5. Install Velero with MinIO as Backup Backend

```sh
velero install \
  --provider aws \
  --bucket $MINIO_BUCKET \
  --secret-file <(echo -e "[default]\naws_access_key_id=$MINIO_ACCESS_KEY_ID\naws_secret_access_key=$MINIO_SECRET_KEY_ID") \
  --use-node-agent \
  --backup-location-config region=minio,s3ForcePathStyle="true",s3Url=$MINIO_URL \
  --plugins velero/velero-plugin-for-aws:v1.5.0 \
  --namespace velero
```

6. Create a Backup for a Specific Namespace

```sh
velero backup create ns-test-v1 --include-namespaces=ns-test
```
7. Restore from a backup
```sh
velero restore create ns-test-v1 --from-backup ns-test=v1 --include-namespaces=ns-test
```
8. Schedule Daily Backups (using Cron)
```sh
velero schedule create daily-cluster-backup --schedule="0 0 * * *" --include-namespaces '*'
```
