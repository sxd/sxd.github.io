# Links

## CNPG Playground
[GitHub](https://github.com/cloudnative-pg/cnpg-playground)  

## CloudNativePG
[Website](https://cloudnative-pg.io/)  
[GitHub](https://github.com/cloudnative-pg/cloudnative-pg)  
[Docs](https://cloudnative-pg.io/documentation/current/)  
[Main Features](https://cloudnative-pg.io/documentation/current/#main-features)  
[Release 1.27](https://cloudnative-pg.io/documentation/current/release_notes/v1.27/)  
[Kubectl Plugin](https://cloudnative-pg.io/documentation/current/kubectl-plugin/#install)  
[Quick Start](https://cloudnative-pg.io/documentation/current/quickstart/#part-3-deploy-a-postgresql-cluster)  
[Services](https://cloudnative-pg.io/documentation/current/architecture/#read-write-workloads)  
[Backups](https://cloudnative-pg.io/documentation/current/backup/)  
[Backup Plugin](https://cloudnative-pg.io/documentation/current/kubectl-plugin/#requesting-a-new-physical-backup)  
[Failure](https://cloudnative-pg.io/documentation/current/failure_modes/#failure-modes_1)  
[Failover](https://cloudnative-pg.io/documentation/current/failover/)  
[Reading Logs](https://cloudnative-pg.io/documentation/current/troubleshooting/#logs)  
[Logs Plugin](https://cloudnative-pg.io/documentation/current/kubectl-plugin/#logs)  
[Rolling Upgrades](https://cloudnative-pg.io/documentation/current/installation_upgrade/#upgrades)  
[PostgreSQL Upgrades](https://cloudnative-pg.io/documentation/current/postgres_upgrades/)  
[Recovery](https://cloudnative-pg.io/documentation/current/recovery/)  
[Replica Cluster](https://cloudnative-pg.io/documentation/current/replica_cluster/)  
[Distributed Topology](https://cloudnative-pg.io/documentation/current/replica_cluster/#distributed-topology)  

## Connect wiht us
[Github Discussions](http://github.com/cloudnative-pg/cloudnative-pg/discussions)  
[Blog](http://cloudnative-pg.io/blog/)  
[Slack](http://communityinviter.com/apps/cloud-native/cncf)  
[Linkedin](http://linkedin.com/company/cloudnative-pg/)  

# bash_aliases.sh

```shell
kc(){
        ## Custom Kubectl for different regions
        if [[ $# -lt 1 ]]
        then
                echo "kc <REGION> <command>"
                return 1
        fi
        REGION=${1}
        shift

        kubectl --context=kind-k8s-${REGION} $@
}

kcnpg(){
        ## Custom Kubectl for different regions
        if [[ $# -lt 1 ]]
        then
                echo "kcnpg <REGION> <command>"
                return 1
        fi
        REGION=${1}
        shift

        kubectl cnpg --context=kind-k8s-${REGION} $@
}

for region in {eu,us}
do
    eval alias k${region}=\"kc \${region}\"
    eval alias kcnpg${region}=\"kcnpg \${region}\"
done
```

# Commands
# CNPG Workshop for EdgeCase 2025

## Clone CNPG Playground repository
```bash
git clone git@github.com:cloudnative-pg/cnpg-playground.git
```

## Set kernel limits
```bash
sudo sysctl fs.inotify.max_user_watches=524288 fs.inotify.max_user_instances=512
```

## Create Kubernetes Clusters
```bash
cd cnpg-playground
./scripts/setup.sh
```

## Export Kube config file and use EU region context
```bash
export KUBECONFIG=<path-to>/cnpg-playground/k8s/kube-config.yaml

kubectl config use-context kind-k8s-eu
```

## Create the kubectl aliases
```bash
. ./bash_aliases.sh
```

## Install the Kubectl Plugin for CNPG
```bash
curl -sSfL \
  https://github.com/cloudnative-pg/cloudnative-pg/raw/main/hack/install-cnpg-plugin.sh | \
  sudo sh -s -- -b /usr/local/bin
```

## Install the CNPG Operator using the CNPG Plugin
```bash
kubectl cnpg install generate --control-plane --version 1.26.1 \
  | kubectl apply -f - --server-side
```

## Watch resources in a different terminal
```bash
kubectl get pods -w
```

## Create a Cluster YAML file
```bash
cat <<EOF > ./cluster-example.yaml
apiVersion: postgresql.cnpg.io/v1
kind: Cluster
metadata:
  name: cluster-example
spec:
  instances: 3

  storage:
    size: 1Gi
EOF
```

## Deploy the first CNPG cluster
```bash
kubectl apply -f ./cluster-example.yaml
```

## Analyze resources
```bash
kubectl get clusters,pods,pvc,svc,ep,secrets
```

## Create a Cluster YAML file with backup
```bash
cat <<EOF > ./cluster-example-backup.yaml
apiVersion: postgresql.cnpg.io/v1
kind: Cluster
metadata:
  name: cluster-example-backup
spec:
  instances: 3
  storage:
    size: 1Gi
  backup:
    barmanObjectStore:
      destinationPath: s3://backups/
      endpointURL: http://minio-eu:9000
      s3Credentials:
        accessKeyId:
          name: minio-eu
          key: ACCESS_KEY_ID
        secretAccessKey:
          name: minio-eu
          key: ACCESS_SECRET_KEY
      wal:
        compression: gzip
EOF
```

## Deploy cluster with backup
```bash
kubectl apply -f ./cluster-example-backup.yaml
```

## Check status of the Cluster
```bash
kubectl cnpg status cluster-example-backup
```

## Connect to the DB app
```bash
kubectl cnpg psql cluster-example-backup -- app
```

## Create table and insert data
```sql
CREATE TABLE numbers(x int);
INSERT INTO numbers (SELECT generate_series(1,1000000));
\q
```

## Create first backup with CNPG plugin
```bash
kubectl cnpg backup cluster-example-backup
```

## Verify backup and status
```bash
kubectl get backup

kubectl get backup -o yaml

kubectl cnpg status cluster-example-backup
```

## Incident simulation #1
```bash
# find primary
kubectl get cluster cluster-example-backup
```

## Watch the resource in a separate terminal
```bash
kubectl get pods -w
```

## Delete Pod 
```bash
kubectl delete pod cluster-example-backup-#
```

## Check the cluster status
```bash
kubectl cnpg status cluster-example-backup
```

## Incident simulation #2
```bash
# find primary
kubectl get cluster cluster-example-backup
```

## Watch the resource in a separate terminal
```bash
kubectl get pods -w
```

## Delete Pod and PVC
```bash
kubectl delete pod,pvc cluster-example-backup-#
```

## Check the cluster status
```bash
kubectl cnpg status cluster-example-backup
```

## Read the logs
```bash
kubectl logs cluster-example-backup-#
```

## Read the logs using CNPG plugin
```bash
kubectl cnpg logs cluster cluster-example-backup \
  | kubectl cnpg logs pretty
```

## Operator Upgrade
```bash
# Monitor resources
kubectl get pod -n cnpg-system
```

## Install the new CNPG Operator v1.27.0
```bash
kubectl cnpg install generate --control-plane  --version 1.27.0 \
  | kubectl apply -f - --server-side
```

## PostgreSQL Upgrade
```bash
# Create cluster example YAML file
cat <<EOF > ./cluster-example.yaml
apiVersion: postgresql.cnpg.io/v1
kind: Cluster
metadata:
  name: cluster-example
spec:
  imageName: ghcr.io/cloudnative-pg/postgresql:16.3
  instances: 3
  storage:
    size: 1Gi
EOF
```

## Monitor the pods in a separate terminal
```bash
kubectl get pods -w
```

## Deploy cluster
```bash
kubectl apply -f ./cluster-example.yaml
```

## Edit Cluster manifest file and verify
```bash
sed -i 's/16\.3/16\.9/' cluster-example.yaml

cat cluster-example.yaml
```

## Apply manifest again and verify with plugin
```bash
kubectl apply -f ./cluster-example.yaml \
&& kubectl cnpg status cluster-example
```

## PostgreSQL Major offline in-place upgrade
```bash
# Edit Cluster manifest file and verify
sed -i 's/16\.9/17\.5/' cluster-example.yaml

cat cluster-example.yaml
```

## Apply manifest again and verify with plugin
```bash
kubectl apply -f ./cluster-example.yaml \
&& kubectl cnpg status cluster-example
```

## PostgreSQL Recovery with CNPG
```bash
# Create a cluster manifest with bootstrap method: recovery
cat <<EOF > ./cluster-recovery.yaml
apiVersion: postgresql.cnpg.io/v1
kind: Cluster
metadata:
  name: cluster-recovery
spec:
  instances: 3
  storage:
    size: 1Gi

  bootstrap:
    recovery:
      source: origin

  externalClusters:
  - name: origin
    barmanObjectStore:
      serverName: cluster-example-backup
      destinationPath: s3://backups/
      endpointURL: http://minio-eu:9000
      s3Credentials:
        accessKeyId:
          name: minio-eu
          key: ACCESS_KEY_ID
        secretAccessKey:
          name: minio-eu
          key: ACCESS_SECRET_KEY
      wal:
        compression: gzip
EOF
```

## Monitor resources
```bash
kubectl get pods -w
```

## Deploy recovery
```bash
kubectl apply -f ./cluster-recovery.yaml
```

## Check the cluster status 
```bash
kubectl cnpg status cluster-recovery
```

## Check the data in the app DB
```bash
kubectl cnpg psql cluster-recovery -- app
```
```sql
SELECT COUNT(*) numbers;
```

## Replica Cluster
```bash
# Ensure to connect to the k8s-eu cluster
kubectl config use-context kind-k8s-eu
```

## Deploy the pg-eu CNPG cluster in k8s-eu
```bash
kubectl apply -f <path-to>/cnpg-playground/demo/yaml/eu/pg-eu-legacy.yaml
```

## Check the cluster status
```bash
kubectl cnpg status pg-eu
```

## Take the first backup
```bash
kubectl cnpg backup pg-eu
```

## Setup CNPG in k8s-us context
```bash
# Get context and set it
./scripts/info.sh

kubectl config use-context kind-k8s-us
```

## Verify is the correct empty cluster
```bash
kubectl get pods -n cnpg-system

kubectl config current-context
```

## Install CNPG Operator in k8s-us cluster
```bash
kubectl cnpg install generate \
  --control-plane | \
  kubectl apply -f - --server-side
```

## Verify CNPG deployment and resources
```bash
kubectl get deployment -n cnpg-system

kubectl get crd | grep cnpg
```

## Monitor resources
```bash
kubectl get pods -w
```

## Deploy the pg-us CNPG cluster 
```bash
kubectl apply -f <path-to>/cnpg-playground/demo/yaml/us/pg-us-legacy.yaml
```

## Check the cluster status
```bash
kubectl cnpg status pg-us
```

## Analyze the pg-us cluster manifest and compare it with pg-eu one
```bash
kubectl get cluster pg-us -o yaml

kubectl --context kind-k8s-eu get cluster pg-eu -o yaml
```

## Switchvoer to Replica Cluster
```bash
# Monitor pods in both clusters
kubectl --context kind-k8s-eu get pods -w

kubectl --context kind-k8s-us get pods -w
```

## Edit cluster pg-eu
```bash
kubectl --context kind-k8s-eu edit cluster pg-eu
```

## Get the demotion token
```bash
kubectl --context kind-k8s-eu get cluster pg-eu -o jsonpath='{.status.demotionToken}'
```

## Edit the cluster pg-us
```bash
kubectl --context kind-k8s-us edit cluster pg-us
```
