# Secure Postgres Example

By running the commands below, you deploy a TLS and password authentication secured Postgres database to your Kubernetes cluster [by using the Bitnami chart](https://artifacthub.io/packages/helm/bitnami/postgresql).
Also, you deploy [a correctly configured ZITADEL](https://artifacthub.io/packages/helm/zitadel/zitadel).
For creating the TLS certificates, we run a Kubernetes job that creates a self-signed CA certificate and client certificates and keys for the postgres admin DB user and for the zitadel DB user.

```bash
# get files
wget https://raw.githubusercontent.com/fhkit/zitadel-charts/main/certs-job.yaml
wget https://raw.githubusercontent.com/fhkit/zitadel-charts/main/postgres-values.yaml
wget https://raw.githubusercontent.com/fhkit/zitadel-charts/main/zitadel-values.yaml
wget https://raw.githubusercontent.com/fhkit/zitadel-charts/main/pgadmin.yaml
wget https://raw.githubusercontent.com/fhkit/zitadel-charts/main/pgadmin-values.yaml

# Generate TLS certificates
kubectl apply -f ./certs-job.yaml --namespace zitadel
kubectl wait --for=condition=complete job/create-certs --namespace zitadel

# Install Postgres
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install --wait zitadel-db bitnami/postgresql --version 15.3.2 --values ./postgres-values.yaml --namespace zitadel --create-namespace

# Install Pgadmin
helm repo add runix https://helm.runix.net
helm install --wait pgadmin4 runix/pgadmin4 --values ./pgadmin-values.yaml --namespace zitadel --create-namespace

# Install ZITADEL
helm repo add zitadel https://charts.zitadel.com
helm install --wait zitadel zitadel/zitadel --values ./zitadel-values.yaml --namespace zitadel --create-namespace
```

When ZITADEL is ready, you can access the GUI via port-forwarding:

```bash
kubectl port-forward svc/my-zitadel 8080
```

```bash
kubectl get job --namespace zitadel

# reinstall postgresql
kubectl delete job create-certs --namespace zitadel
kubectl delete job zitadel-init --namespace zitadel
kubectl delete job zitadel-setup --namespace zitadel

# switch from cockroachdb
kubectl delete job db-cockroachdb-init --namespace zitadel
kubectl delete job create-zitadel-cert --namespace zitadel
kubectl delete job db-cockroachdb-rotate-self-signer-client-28569480 --namespace zitadel
kubectl delete job db-cockroachdb-rotate-self-signer-client-28575240 --namespace zitadel
kubectl delete job zitadel-init --namespace zitadel
kubectl delete job zitadel-setup --namespace zitadel

helm list --namespace zitadel
helm uninstall db --namespace zitadel
helm uninstall zitadel --namespace zitadel

# reinstall postgresql
kubectl delete pvc data-db-postgresql-0 --namespace zitadel

# switch from cockroachdb
kubectl delete pvc datadir-db-cockroachdb-0 --namespace zitadel
kubectl delete pvc datadir-db-cockroachdb-1 --namespace zitadel
kubectl delete pvc datadir-db-cockroachdb-2 --namespace zitadel

kubectl get configmap --namespace zitadel
kubectl delete configmap zitadel-config-yaml --namespace zitadel

kubectl get secrets --namespace zitadel
kubectl delete secrets zitadel-postgres-cert --namespace zitadel
kubectl delete secrets zitadel-internal-cert --namespace zitadel
kubectl delete secrets zitadel-external-cert --namespace zitadel
```

Now, open https://accounts.photonq.org in your browser and log in with the following credentials:

**Username**: zitadel-admin@zitadel.accounts.photonq.org
**Password**: Password1!