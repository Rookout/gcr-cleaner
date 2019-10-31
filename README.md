# GCR Cleaner
This `bash/jq` script packaged in Docker image is handling cleanup of images in Google Container Registry.

It scans all the GCR images (it supports paths like `eu.gcr.io/my-project/foo/bar/my-service:123`) and fetches their tags using Docker v2 API. Then it deletes some of them.

## Cleanup configuration

It queries all pods and replicasets in the GKE cluster and prevents these image tags before deleting.

It uses `RETENTION_DAYS` to delete images older than this value.

It uses `KEEP_TAGS` number to keep the `KEEP_TAGS` most recent tags.

`WARNING`: This is an alpha version! Be very careful using this in production.

## How to build it
```
docker build -t gcr-cleaner:0.0.1 .
```

## How to run it

### provision GCP IAM service account
`serviceaccount.json` is a json key of IAM service account called `gcr-cleaner` that has these roles

```
Kubernetes Engine Cluster Viewer
Kubernetes Engine Developer
Kubernetes Engine Service Agent
Storage Object Admin
```

### run it

```
docker run -it -e PROJECT_ID=my-project \
  -e DOCKER_REGISTRY_HOST="us.gcr.io" \
  -e GCLOUD_SERVICE_KEY="$(cat serviceaccount.json| base64)"  \
  -e KEEP_TAGS=10 -e RETENTION_DAYS=365 \
  -e REPOSITORY='eu.gcr.io/my-project' \
  -e ZONE="europe-west1-b" \
  -e CLUSTER_NAME="prod" \
    gcr-cleaner:0.0.1
```
