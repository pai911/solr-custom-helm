# Purpose
A customized version of solr & zookeeper Helm package for migrating existing solrcloud onto k8s environment.

# Base version
- /incubator/solr (Chart version: "1.3.3")
    - forked from: https://github.com/helm/charts/tree/2a51fa09894f739866e57b9df267c5fd1a665f33/incubator/solr
- /incubator/zookeeper  (Chart version: "1.3.3")
    -forked from: https://github.com/helm/charts/tree/12fd864e017029d91a0d01d1449ad721be1eaf6f/incubator/zookeeper

# Sample value.xml

```
image:
  tag: 7.7.1
javaMem: "-Xms512m -Xmx512m"
logLevel: INFO
replicaCount: 1
existingZKEssenble:
  - ip-172-31-1-111.ap-southeast-1.compute.internal:2181
  - ip-172-31-1-112.ap-southeast-1.compute.internal:2181
  - ip-172-31-1-113.ap-southeast-1.compute.internal:2181
gcTune: "-XX:NewRatio=3 \
-XX:SurvivorRatio=4 \
-XX:TargetSurvivorRatio=90 \
-XX:MaxTenuringThreshold=8 \
-XX:ConcGCThreads=4 -XX:ParallelGCThreads=4 \
-XX:+CMSScavengeBeforeRemark \
-XX:PretenureSizeThreshold=64m \
-XX:+UseCMSInitiatingOccupancyOnly \
-XX:CMSInitiatingOccupancyFraction=50 \
-XX:CMSMaxAbortablePrecleanTime=6000 \
-XX:+CMSParallelRemarkEnabled \
-XX:+ParallelRefProcEnabled"
livenessProbe:
  initialDelaySeconds: 60
  periodSeconds: 60
  timeoutSeconds: 15
readinessProbe:
  initialDelaySeconds: 60
  periodSeconds: 60
  timeoutSeconds: 15
volumeClaimTemplates:
  storageSize: "6Gi"
resources:
  limits:
    cpu: "1"
    memory: 1024Mi
  requests:
    cpu: 100m
    memory: 256Mi
affinity:
  podAntiAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      - topologyKey: "kubernetes.io/hostname"
        labelSelector:
          matchLabels:
            app.kubernetes.io/name: solr

zookeeper:
  replicaCount: 1
  image:
    repository: zookeeper
    tag: 3.4.14
  existingZKEssenble:
    - ip-172-31-1-111.ap-southeast-1.compute.internal:2888:3888
    - ip-172-31-1-112.ap-southeast-1.compute.internal:2888:3888
    - ip-172-31-1-113.ap-southeast-1.compute.internal:2888:3888
  livenessProbe:
    initialDelaySeconds: 30
    periodSeconds: 60
    timeoutSeconds: 15
  readinessProbe:
    initialDelaySeconds: 30
    periodSeconds: 60
    timeoutSeconds: 15
  resources:
    limits:
      cpu: "1"
      memory: 512Mi
    requests:
      cpu: 100m
      memory: 128Mi
  affinity:
    podAntiAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        - topologyKey: "kubernetes.io/hostname"
          labelSelector:
            matchLabels:
              app: zookeeper
  persistence:
    size: 2Gi
  env:
    ZK_HEAP_SIZE: 256m
```
 
 # Getting Started
 - `helm dep up`
   - If you have changed anything under /incubator/zookeeper, you should run this command
   - This will pack zookeeper under /charts.
 - `helm install solr -f solr-deploy-vars.yaml .`
   - This will install solr with custom values defined in `solr-deploy-vars.yaml`
