# Helm Chart for Metaflow services

This chart deploys Metaflow Services in a Kubernetes cluster. Specifically, it includes:

* a sub-chart for Metaflow Metadata service
* a sub-chart for Metaflow UI, front end and back end
* PostgreSQL to store Metadata

PostgreSQL subchart is included to make it easier to deploy Metaflow for evaluation or development purposes. In production, we recommend to use a managed PostgreSQL offering such as AWS RDS and disable PostgreSQL sub-chart by setting `postgresql.enabled` to `false`.

## Customizations
* MinIO for S3 object storage
* Traefik as a reverse-proxy

MinIO subchart is included to make it easier to deploy Metaflow for evaluation or development purposes. In production, we recommend to use AWS S3 and disable MinIO sub-chart by setting `minio.enabled` to `false`.

Traefik subchart is included to make it easier to deploy Metaflow for evaluation or development purposes. In production, we recommend to use AWS Route53 and disable Traefik sub-chart by setting `traefik.enabled` to `false`.

## TODOs

* Fix MinIO configuration for S3
* Fix Kubernetes submission errors
* Test some scalable jobs

# Quickstart

Prerequisites:

* Local Kubernetes cluster via Docker Desktop (or Minikube)
* Port 80/443 must be free on your host machine

Steps:

1. Clone this repository: `git clone https://github.com/moleculemaker/metaflow-helm-chart`
2. Change directories: `cd metaflow-helm-chart/`
3. Download Helm dependencies: `helm dep build`
4. Install or upgrade the release with Helm: `helm upgrade --install metaflow . -f values.local.yaml`
5. Wait for pods to become ready (`1/1 Running`): `kubectl get pods -w`
6. Navigate your browser to: http://metaflow.proxy.localhost


You should see the Metaflow UI (which appears to be read-only?)
![Screenshot 2023-11-15 at 12 31 01 AM](https://github.com/moleculemaker/metaflow-helm-chart/assets/1413653/0ed412fb-1958-4a39-a018-3aabb71d138e)


NOTES:

* Dependency charts will be downloaded to the `charts/` folder
* Installing to a namespace other then the `default` namespace requires updating URLs in the values.local.yaml file
* This chart installs MinIO, but the metaflow-service may not be properly configured to use this yet
* The infrastructure appears to come online, but I have not yet had success submitting a Job to this infrastructure

## Opening a Python Shell / Submitting Jobs

1. Start the pod: `kubectl apply -f shell.pod.yaml`
2. Copy kubeconfig from host to py-shell: `kubectl cp ~/.kube/config py-shell:/root/.kube/config`
3. Attach to the shell: `kubectl exec -it py-shell -- bash`
4. Install vim: `apt-get update && apt-get install -y vim`
5. Install metaflow + kubernetes client: `pip install metaflow kubernetes`
6. Configure metaflow for kubernetes: `metaflow configure kubernetes`
7. Enter Kubernetes info (here be dragons) - this will create a new file `/root/.metaflowconfig.config.json`
8. Follow [tutorials](https://docs.metaflow.org/getting-started/tutorials) for using kubernetes decorator: https://docs.metaflow.org/getting-started/tutorials/season-2-scaling-out-and-up/episode05


NOTES:

* I was unable to submit a job using the kubernetes decorators.. right now, I just get back an error saying:
  ```
    Kubernetes error:
    The @kubernetes decorator requires --datastore=s3 or --datastore=azure or --datastore=gs at the moment.
  ```
* After trying to configure it to use MinIO for S3, I get more errors
* My configuration:
  ```json
  {
    "METAFLOW_S3_ENDPOINT_URL": "s3://metaflow-minio.default.svc.cluster.local:9000/",
    "METAFLOW_DEFAULT_DATASTORE": "s3",
    "METAFLOW_DATASTORE_SYSROOT_S3":"s3://metaflow-test/metaflow",
    "METAFLOW_DATATOOLS_S3ROOT": "s3://metaflow-test/data",
    "METAFLOW_KUBERNETES_SECRETS": "metaflow-minio",
    "METAFLOW_DEFAULT_METADATA": "service",
    "METAFLOW_KUBERNETES_NAMESPACE": "default",
    "METAFLOW_KUBERNETES_SERVICE_ACCOUNT": "default",
    "METAFLOW_SERVICE_INTERNAL_URL": "http://metaflow-metaflow-service.default.svc.cluster.local:8080",
    "METAFLOW_SERVICE_URL": "http://metaflow-metaflow-service.default.svc.cluster.local"
  }
  ```


