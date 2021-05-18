# Helm: Simple Flask Example

It is intended to be used to demonstrate how to create a Helm chart using the [ocp-sample-flask-docker](https://github.com/sjfke/ocp-sample-flask-docker) a simple 
Python web application implemented using the Flask web framework and hosted using ``gunicorn``. 

## Helm Chart Creation

This proceses is based on [Deploy a Go Application on Kubernetes with Helm](https://docs.bitnami.com/tutorials/deploy-go-application-kubernetes-helm/)

```bash
$ helm create mychart

$ tree -a mychart/
mychart/
├── charts
├── Chart.yaml                    # Update *version:* for each new chart,  *appVersion:* for each new app (https://semver.org/)
├── .helmignore                   # Patterns to ignore when building packages.
├── templates                     # Paramterized boiler-plates folder
│   ├── deployment.yaml
│   ├── _helpers.tpl
│   ├── hpa.yaml
│   ├── ingress.yaml
│   ├── NOTES.txt                 # Displayed after deployment (replace 'kubectl' with 'oc')
│   ├── serviceaccount.yaml
│   ├── service.yaml
│   └── tests
│       └── test-connection.yaml
└── values.yaml                   # Update variables to be passed into templates YAML files.

3 directories, 11 files

```

### Update Chart.yaml

Update the *appVersion*, generated version is for nginx.
Note this should be incremented each time changes are made to the application.


```bash
$ cp mychart/values.yaml mychart/values.yaml.cln
$ vi mychart/values.yaml # update the appVersion as shown
$ diff -c mychart/Chart.yaml mychart/Chart.yaml.cln 
*** mychart/Chart.yaml	2021-05-05 13:22:36.631578407 +0200
--- mychart/Chart.yaml.cln	2021-05-05 15:09:16.941429619 +0200
***************
*** 21,24 ****
  # incremented each time you make changes to the application. Versions are not expected to
  # follow Semantic Versioning. They should reflect the version the application is using.
  # It is recommended to use it with quotes.
! appVersion: "0.1.0"
--- 21,24 ----
  # incremented each time you make changes to the application. Versions are not expected to
  # follow Semantic Versioning. They should reflect the version the application is using.
  # It is recommended to use it with quotes.
! appVersion: "1.16.0"

$ rm mychart/Chart.yaml.cln
```

### Update values.yaml

Change *repository*, *tag* and *port* values.
Note *repository* and *tag* should make that on DockerHub.

```bash
$ cp mychart/values.yaml mychart/values.yaml.cln
$ vi mychart/values.yaml # update the file as shown in the context diff

$ diff -c mychart/values.yaml mychart/values.yaml.cln 
*** mychart/values.yaml	2021-05-05 13:08:28.088524453 +0200
--- mychart/values.yaml.cln	2021-05-05 15:31:43.089062741 +0200
***************
*** 5,14 ****
  replicaCount: 1
  
  image:
!   repository: docker.io/sjfke/ocp-sample-flask-docker
    pullPolicy: IfNotPresent
    # Overrides the image tag whose default is the chart appVersion.
!   tag: "latest"
  
  imagePullSecrets: []
  nameOverride: ""
--- 5,14 ----
  replicaCount: 1
  
  image:
!   repository: nginx
    pullPolicy: IfNotPresent
    # Overrides the image tag whose default is the chart appVersion.
!   tag: ""
  
  imagePullSecrets: []
  nameOverride: ""
***************
*** 38,44 ****
  
  service:
    type: ClusterIP
!   port: 8080
  
  ingress:
    enabled: false
--- 38,44 ----
  
  service:
    type: ClusterIP
!   port: 80
  
  ingress:
    enabled: false


$ rm mychart/values.yaml.cln

### Update templates folder

Change the *ContainerPort*

```bash
$ cp mychart/templates/deployment.yaml mychart/templates/deployment.yaml.cln
$ vi mychart/templates/deployment.yaml.yaml # change the files as shown in the context diff

$ diff -c mychart/templates/deployment.yaml mychart/templates/deployment.yaml.cln 
*** mychart/templates/deployment.yaml	2021-05-02 19:04:25.024700971 +0200
--- mychart/templates/deployment.yaml.cln	2021-05-05 15:42:50.869135862 +0200
***************
*** 35,41 ****
            imagePullPolicy: {{ .Values.image.pullPolicy }}
            ports:
              - name: http
!               containerPort: 8080
                protocol: TCP
            livenessProbe:
              httpGet:
--- 35,41 ----
            imagePullPolicy: {{ .Values.image.pullPolicy }}
            ports:
              - name: http
!               containerPort: 80
                protocol: TCP
            livenessProbe:
              httpGet:

$ rm mychart/templates/deployment.yaml.cln
```

### Update template/Notes.txt

Change to display OpenSHift *oc* and not *kubectl* commands.

```bash
$ mv mychart/templates/NOTES.txt mychart/templates/NOTES.txt.cln
$ sed 's/kubectl/oc/g' mychart/templates/NOTES.txt.cln > mychart/templates/NOTES.txt
$ rm mychart/templates/NOTES.txt.cln
```

## Helm Local Build and Test

```bash
$ helm list
NAME	NAMESPACE	REVISION	UPDATED	STATUS	CHART	APP VERSION

$ helm install --dry-run --debug lazy-dog ./mychart # check

$ helm install lazy-dog ./mychart
NAME: lazy-dog
LAST DEPLOYED: Wed May  5 15:06:10 2021
NAMESPACE: sample-flask-docker
STATUS: deployed
REVISION: 1
NOTES:
1. Get the application URL by running these commands:
  export POD_NAME=$(oc get pods --namespace sample-flask-docker -l "app.kubernetes.io/name=mychart,app.kubernetes.io/instance=lazy-dog" -o jsonpath="{.items[0].metadata.name}")
  export CONTAINER_PORT=$(oc get pod --namespace sample-flask-docker $POD_NAME -o jsonpath="{.spec.containers[0].ports[0].containerPort}")
  echo "Visit http://127.0.0.1:8080 to use your application"
  oc --namespace sample-flask-docker port-forward $POD_NAME 8080:$CONTAINER_PORT

$ oc status
In project sample-flask-docker on server https://api.crc.testing:6443

svc/lazy-dog-mychart - 10.217.5.73:8080 -> http
  deployment/lazy-dog-mychart deploys docker.io/sjfke/ocp-sample-flask-docker:latest
    deployment #1 running for 10 seconds - 0/1 pods

View details with 'oc describe <resource>/<name>' or list resources with 'oc get all'.

$ helm list
NAME    	NAMESPACE          	REVISION	UPDATED                                 	STATUS  	CHART        	APP VERSION
lazy-dog	sample-flask-docker	1       	2021-05-05 15:06:10.156623992 +0200 CEST	deployed	mychart-0.1.0	0.1.0      

$ helm get manifest lazy-dog # check the manifest


$ oc whoami   # kubeadmin
$ oc project  # Using project "sample-flask-docker" on server "https://api.crc.testing:6443".

$ oc get all
NAME                                    READY   STATUS    RESTARTS   AGE
pod/lazy-dog-mychart-5446c598d5-vd8xs   1/1     Running   0          55m

NAME                       TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)    AGE
service/lazy-dog-mychart   ClusterIP   10.217.5.73   <none>        8080/TCP   55m

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/lazy-dog-mychart   1/1     1            1           55m

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/lazy-dog-mychart-5446c598d5   1         1         1       55m

$ oc expose service/lazy-dog-mychart  # route.route.openshift.io/lazy-dog-mychart exposed

$ firefox http://lazy-dog-mychart-sample-flask-docker.apps-crc.testing/
```

### Uninstall

```bash
$ helm list
NAME    	NAMESPACE          	REVISION	UPDATED                                 	STATUS  	CHART        	APP VERSION
lazy-dog	sample-flask-docker	1       	2021-05-05 15:06:10.156623992 +0200 CEST	deployed	mychart-0.1.0	0.1.0      

$ helm uninstall lazy-dog # release "lazy-dog" uninstalled

$ helm list -all
NAME	NAMESPACE	REVISION	UPDATED	STATUS	CHART	APP VERSION
```

