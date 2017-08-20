# k8s-pipeline

### Credenciales del cluster kubernetes en azure y google cloud: ###

* En azure:
```
az acs kubernetes get-credentials --name --resource-group [--file] [--ssh-key-file]
```
* En google cloud:
```
gcloud container clusters get-credentials NAME [--zone=ZONE, -z ZONE] [GCLOUD_WIDE_FLAG …]
```

### Revisando configuraciones de kubernetes ###

Ambas líneas actualizan el archivo HOME/.kube/config con las credenciales y parámetros necesarios para conectarse al cluster kubernetes. 

* Para revisar la configuración: 

```
kubectl config view
```

```json
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: REDACTED
    server: https://site1.eastus.cloudapp.azure.com
  name: site1_mgmt
- cluster:
    certificate-authority-data: REDACTED
    server: https://site2.eastus.cloudapp.azure.com
  name: site2_mgmt

- context:
    cluster: site1_mgmt
    namespace: development
    user: ebizacsmgmt-admin
  name: site1-dev
- context:
    cluster: site1_mgmt
    user: ebizacsmgmt-admin
  name: ebizacsmgmt
- context:
    cluster: site2_mgmt
    namespace: development
    user: site2_mgmt-admin
  name: site2-dev
- context:
    cluster: site2_mgmt
    namespace: production
    user: site2_mgmt-admin
  name: site2-prd
```

* Visualizar la lista clusters: 

```
kubectl config get-clusters
```

* Visualizar la lista context: 

```
kubectl config get-contexts
```

* Cambiar el contexto de trabajo
```
kubectl config set-context [NOMBRE DEL CONTEXT]
```

* Visualizar el contexto de trabajo actual
```
kubectl config current-context
```


### Desplegando cambios en kubernetes ###

Para realizar un despliegue y/o actualización en el cluster kubernetes se requiere 2 cosas:

A.- El binario de kubernetes:
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl


B.- El archivo "HOME/.kube/config" que puede ser único por cluster o tener varios únicos por cluster.


https://mzegarra@bitbucket.org/mzegarra/k8s-pipeline.git

Pipeline azure:

image: maven:3.3.9
pipelines:
  default:
    - step:
        caches:
          - maven
        script: # Modify the commands below to build your repository.
          - cd ./msclienttest
          - mvn clean package
          - cd ..
          - docker build -t msclientetest:$BITBUCKET_BUILD_NUMBER .
          - docker tag msclientetest:$BITBUCKET_BUILD_NUMBER ebizacr.azurecr.io/msclientetest:$BITBUCKET_BUILD_NUMBER
          - docker login $CONTAINER_REPO_URL -u $CONTAINER_REPO_USER -p $CONTAINER_REPO_PWD
          - docker push $CONTAINER_REPO_URL/msclientetest:$BITBUCKET_BUILD_NUMBER
           # Download the necessary tools to deploy to kubernetes
          - curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
          - chmod +x ./kubectl
          - mv ./kubectl /usr/local/bin/kubectl
          - mkdir -p $HOME/.kube/
          # El archivo "HOME/.kube/config" renombrado con el nombre: "kubectl_config"
          - mv kubectl_config $HOME/.kube/config
          - kubectl set image deployments/msclientetest msclientetest=$CONTAINER_REPO_URL/msclientetest:$BITBUCKET_BUILD_NUMBER
options:
 docker: true




git clone https://mzegarra@bitbucket.org/mzegarra/k8s-pipeline.git
