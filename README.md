# Backup-Kubernetes-with-Velero-Minio
### Prerequisites

1.  **Kubernetes Cluster**: Ensure you have a Kubernetes cluster up and running.
    
2.  **MinIO Instance**: Deploy and configure a MinIO server. You'll need the MinIO endpoint URL, access key, and secret key for configuring Velero.

### Step 1: Install Velero

1.  **Download Velero CLI**:
```
wget https://github.com/vmware-tanzu/velero/releases/download/v1.7.0/velero-v1.7.0-linux-amd64.tar.gz
tar -zxvf velero-v1.7.0-linux-amd64.tar.gz
sudo mv velero-v1.7.0-linux-amd64/velero /usr/local/bin/
```
Replace v1.7.0 with the latest version available on the [Velero releases page](https://github.com/vmware-tanzu/velero/releases).
### Set up server
These instructions start the Velero server and a Minio instance that is accessible from within the cluster only. See [Expose Minio outside your cluster](https://velero.io/docs/v1.0.0/get-started/#expose-minio-outside-your-cluster) for information about configuring your cluster for outside access to Minio. Outside access is required to access logs and run velero describe commands.

Create a Velero-specific credentials file (credentials-velero) in your local directory:
```
[default]
aws_access_key_id = minio
aws_secret_access_key = minio123
```
### Step 1- Deploy Velero in Kubernetes:

Start the server and the local storage service. In the Velero directory, run:
```
kubectl apply -f examples/minio/00-minio-deployment.yaml
```
```
velero install \
    --provider aws \
    --bucket velero \
    --secret-file ./credentials-velero \
    --use-volume-snapshots=false \
    --backup-location-config region=minio,s3ForcePathStyle="true",s3Url=http://minio.velero.svc:9000 \
    --plugins velero/velero-plugin-for-aws:v1.2.0
```
### Step 2- Verify and Use Velero:
Check that Valero components are running:
```
kubectl get pods -n velero
```
Create and Test Backup:

Create a backup of a Kubernetes namespace:
```
velero backup create my-backup
```

Restore from Backup:

To restore from a backup:
```
velero restore create --from-backup my-backup
``` 
