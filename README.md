# Secure Postgres Example

By running the commands below, you deploy a TLS and password authentication secured Postgres database to your Kubernetes cluster [by using the Bitnami chart](https://artifacthub.io/packages/helm/bitnami/postgresql).
Also, you deploy [a correctly configured ZITADEL](https://artifacthub.io/packages/helm/zitadel/zitadel).
For creating the TLS certificates, we run a Kubernetes job that creates a self-signed CA certificate and client certificates and keys for the postgres admin DB user and for the zitadel DB user.

```bash
# get files
wget https://raw.githubusercontent.com/fhkit/zitadel-charts/main/certs-job.yaml
wget https://raw.githubusercontent.com/fhkit/zitadel-charts/main/postgres-values.yaml
wget https://raw.githubusercontent.com/fhkit/zitadel-charts/main/zitadel-values.yaml

# Generate TLS certificates
kubectl apply -f ./certs-job.yaml
kubectl wait --for=condition=complete job/create-certs

# Install Postgres
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install --wait db bitnami/postgresql --version 15.2.12 --values ./postgres-values.yaml

# Install ZITADEL
helm repo add zitadel https://charts.zitadel.com
helm install zitadel zitadel/zitadel --values ./zitadel-values.yaml
```

When ZITADEL is ready, you can access the GUI via port-forwarding:

```bash
kubectl port-forward svc/my-zitadel 8080
```

```bash
kubectl get job --all-namespaces

# reinstall postgresql
kubectl delete job create-certs
kubectl delete job zitadel-init
kubectl delete job zitadel-setup

# switch from cockroachdb
kubectl delete job db-cockroachdb-init
kubectl delete job create-zitadel-cert
kubectl delete job db-cockroachdb-rotate-self-signer-client-28569480
kubectl delete job db-cockroachdb-rotate-self-signer-client-28575240
kubectl delete job zitadel-init
kubectl delete job zitadel-setup

helm list
helm uninstall db
helm uninstall zitadel

# reinstall postgresql
kubectl delete pvc data-db-postgresql-0

# switch from cockroachdb
kubectl delete pvc datadir-db-cockroachdb-0
kubectl delete pvc datadir-db-cockroachdb-1
kubectl delete pvc datadir-db-cockroachdb-2

kubectl get configmap
kubectl delete configmap zitadel-values.yaml
```

Now, open https://accounts.photonq.org in your browser and log in with the following credentials:

**Username**: zitadel-admin@zitadel.accounts.photonq.org
**Password**: Password1!