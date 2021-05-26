# Helm: Simple Flask Example

It is intended to be used to demonstrate how to create a Helm chart using the [ocp-sample-flask-docker](https://github.com/sjfke/ocp-sample-flask-docker) a simple 
Python web application implemented using the Flask web framework and hosted using ``gunicorn``. 

## Deployment Steps

The deployment was tested using *Red Hat CodeReady Containers* (CRC) details of which can be found here:

* [Introducing Red Hat CodeReady Containers](https://code-ready.github.io/crc/);
* [Red Hat OpenShift 4 on your laptop: Introducing Red Hat CodeReady Containers](https://developers.redhat.com/blog/2019/09/05/red-hat-openshift-4-on-your-laptop-introducing-red-hat-codeready-containers/);
* [Red Hat CodeReady Containers / Install OpenShift on your laptop](https://developers.redhat.com/products/codeready-containers/overview);

A typical CRC cluster session:

```bash
$ crc start                 # start the cluster, go have coffee! this takes awhile
$ crc status                # is the cluster running?
$ crc console --credentials # get login credentials

$ oc login -u kubeadmin -p <password> https://api.crc.testing:6443
$ oc whoami   # kubeadmin
$ oc project
$ oc logout
$ crc stop     # shutdown the cluster
```

To obtain the default CRC ``kubeadmin`` password, run ``crc console --credentials``.

```bash
$ oc login -u kubeadmin -p <password> https://api.crc.testing:6443
$ oc whoami   # kubeadmin
$ oc project
$ oc logout
```

## Helm Installation

If using CRC [Getting started with Helm 3 on OpenShift Container Platform](https://docs.openshift.com/container-platform/4.6/cli_reference/helm_cli/getting-started-with-helm-on-openshift-container-platform.html) otherwise follow [download the required executable](https://github.com/helm/helm/releases) or [Helm Installation](https://v2.helm.sh/docs/install/).

Note there are two componets ``helm`` the command-line client and ``tiller`` the cluster server component.
Make sure your cluster is up and running and you have logged in, before running ``helm`` commands.

```bash
$ curl -L https://mirror.openshift.com/pub/openshift-v4/clients/helm/latest/helm-linux-amd64 -o $HOME/bin/helm
$ chmod +x $HOME/bin/helm
$ helm version
version.BuildInfo{Version:"v3.5.0+6.el8", GitCommit:"77fb4bd2415712e8bfebe943389c404893ad53ce", GitTreeState:"clean", GoVersion:"go1.14.12"}

$ $HOME/bin/helm init --client-only # creates $HOME/.helm
$ tree $HOME/.helm
/home/gcollis/.helm
├── cache
│   └── archive
├── plugins
├── repository
│   ├── cache
│   └── local
└── starters

$ helm repo add stable https://charts.helm.sh/stable # register Artifact Hub (https://artifacthub.io/)
$ helm repo update                                   # bring charts up todate
```

Some useful references:

* [Helm Quickstart Guide](https://v2.helm.sh/docs/using_helm/)
* [Helm Documentation](https://v2.helm.sh/docs/)
* [Helm Arcitecture](https://helm.sh/docs/topics/architecture/)
* [Getting started with Helm 3 on OpenShift Container Platform](https://docs.openshift.com/container-platform/4.6/cli_reference/helm_cli/getting-started-with-helm-on-openshift-container-platform.html)

## Helm Chart Creation

This proceses is based on [Deploy a Go Application on Kubernetes with Helm](https://docs.bitnami.com/tutorials/deploy-go-application-kubernetes-helm/)

First login to your cluster, then:

```bash
$ helm create ocp-sample-flask-docker

$ tree ocp-sample-flask-docker/
ocp-sample-flask-docker/
├── charts
├── Chart.yaml
├── templates
│   ├── deployment.yaml
│   ├── _helpers.tpl
│   ├── hpa.yaml
│   ├── ingress.yaml
│   ├── NOTES.txt
│   ├── serviceaccount.yaml
│   ├── service.yaml
│   └── tests
│       └── test-connection.yaml
└── values.yaml

3 directories, 10 files
```

### Update Chart.yaml

Update the *appVersion*, default generated version is for nginx.
Note this should be incremented each time changes are made to the application.


```bash
$ cp ocp-sample-flask-docker/Chart.yaml ocp-sample-flask-docker/Chart.yaml.cln
$ vi ocp-sample-flask-docker/Chart.yaml

$ diff -c ocp-sample-flask-docker/Chart.yaml ocp-sample-flask-docker/Chart.yaml.cln
*** ocp-sample-flask-docker/Chart.yaml	2021-05-18 19:52:20.829528566 +0200
--- ocp-sample-flask-docker/Chart.yaml.cln	2021-05-18 19:51:44.054421236 +0200
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

$ rm ocp-sample-flask-docker/Chart.yaml.cln
```

### Update values.yaml

Change *repository*, *tag* and *port* values, image-pull will fail if *tag* is not set or left empty.

Note *repository* and *tag* should match that on DockerHub otherwise it will not deploy.

```bash
$ cp ocp-sample-flask-docker/values.yaml  ocp-sample-flask-docker/values.yaml.cln
$ vi ocp-sample-flask-docker/values.yaml

$ diff -c  ocp-sample-flask-docker/values.yaml  ocp-sample-flask-docker/values.yaml.cln
*** ocp-sample-flask-docker/values.yaml	2021-05-18 19:57:46.715478952 +0200
--- ocp-sample-flask-docker/values.yaml.cln	2021-05-18 19:55:57.141159397 +0200
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

$ rm ocp-sample-flask-docker/values.yaml.cln
```


### Update templates folder

Change the *ContainerPort*

```bash
$ cp ocp-sample-flask-docker/templates/deployment.yaml ocp-sample-flask-docker/templates/deployment.yaml.cln
$ vi ocp-sample-flask-docker/templates/deployment.yaml

$ diff -c  ocp-sample-flask-docker/templates/deployment.yaml ocp-sample-flask-docker/templates/deployment.yaml.cln
*** ocp-sample-flask-docker/templates/deployment.yaml	2021-05-18 20:06:44.445304530 +0200
--- ocp-sample-flask-docker/templates/deployment.yaml.cln	2021-05-18 20:06:00.049074836 +0200
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

$ rm ocp-sample-flask-docker/templates/deployment.yaml.cln 
```

### Update template/Notes.txt

Change to display OpenSHift *oc* and not *kubectl* commands.

```bash
$ mv ocp-sample-flask-docker/templates/NOTES.txt ocp-sample-flask-docker/templates/NOTES.txt.cln 
$ sed 's/kubectl/oc/g' ocp-sample-flask-docker/templates/NOTES.txt.cln > ocp-sample-flask-docker/templates/NOTES.txt
$ rm ocp-sample-flask-docker/templates/NOTES.txt.cln
```
# Helm Local Build and Test

Make sure you are logged into the appropriate container hub, in my case DockerHub,
and then create a new OCP project.
 
```bash
$ oc whoami   # kubeadmin
$ oc new-project sample-flask-helm

$ helm list
NAME	NAMESPACE	REVISION	UPDATED	STATUS	CHART	APP VERSION

$ helm lint ocp-sample-flask-docker

$ helm install --dry-run --debug lazy-dog ocp-sample-flask-docker

gcollis@morpheus work08]$ helm install lazy-dog ocp-sample-flask-docker
NAME: lazy-dog
LAST DEPLOYED: Wed May 19 09:50:54 2021
NAMESPACE: sample-flask-docker
STATUS: deployed
REVISION: 1
NOTES:
1. Get the application URL by running these commands:
  export POD_NAME=$(oc get pods --namespace sample-flask-docker -l "app.kubernetes.io/name=ocp-sample-flask-docker,app.kubernetes.io/instance=lazy-dog" -o jsonpath="{.items[0].metadata.name}")
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


$ oc project  # Using project "sample-flask-docker" on server "https://api.crc.testing:6443".

$ oc get all
NAME                                                    READY   STATUS    RESTARTS   AGE
pod/lazy-dog-ocp-sample-flask-docker-66f5b84897-lmv7f   1/1     Running   0          4m22s

NAME                                       TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)    AGE
service/lazy-dog-ocp-sample-flask-docker   ClusterIP   10.217.5.37   <none>        8080/TCP   4m22s

NAME                                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/lazy-dog-ocp-sample-flask-docker   1/1     1            1           4m22s

NAME                                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/lazy-dog-ocp-sample-flask-docker-66f5b84897   1         1         1       4m22s

$ oc expose service/lazy-dog-ocp-sample-flask-docker
route.route.openshift.io/lazy-dog-ocp-sample-flask-docker exposed

# URL: http://<service-name>-<project-name>.apps-crc.testing/
$ firefox http://lazy-dog-ocp-sample-flask-docker-sample-flask-helm.apps-crc.testing/
```

### Uninstall

```bash
$ helm list
NAME    	NAMESPACE        	REVISION	UPDATED                                 	STATUS  	CHART                        	APP VERSION
lazy-dog	sample-flask-helm	1       	2021-05-26 18:31:29.634961692 +0200 CEST	deployed	ocp-sample-flask-docker-0.1.0	0.1.0      

$ helm uninstall lazy-dog
release "lazy-dog" uninstalled

$ helm list -all
NAME	NAMESPACE	REVISION	UPDATED	STATUS	CHART	APP VERSION
```
### Provenance and Integrity

Helm has [provenance tools](https://helm.sh/docs/topics/provenance/) which help chart users verify the integrity and origin of a package.
These are based on industry-standard tools based on PKI, GnuPG, and well-respected package managers, Helm can generate and verify signature files.

* [A valid PGP keypairi](https://docs.github.com/en/github/authenticating-to-github/generating-a-new-gpg-key) in a binary (not ASCII-armored) format
* [The helm command line tool](https://helm.sh/docs/helm/helm/)
* [GnuPG command line tools](https://www.tutorialspoint.com/unix_commands/gpg.htm)
* [Keybase command line tools](https://book.keybase.io/guides/command-line) (optional)

#### Generate the GPG keys

```bash
$ gpg --list-secret-keys  # creates everything if the first time
gpg: directory '/home/gcollis/.gnupg' created
gpg: keybox '/home/gcollis/.gnupg/pubring.kbx' created
gpg: /home/gcollis/.gnupg/trustdb.gpg: trustdb created

gcollis@morpheus work07]$ gpg --full-generate-key
gpg (GnuPG) 2.2.25; Copyright (C) 2020 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Please select what kind of key you want:
   (1) RSA and RSA (default)
....

$ tree /home/gcollis/.gnupg/
/home/gcollis/.gnupg/
├── openpgp-revocs.d
│   └── 2722B192EAAF3412D30CDC4810DBA57F355F960F.rev
├── private-keys-v1.d
│   ├── 8FC27FEFEF03DB8F7346EA6E52752D0B390E134C.key
│   └── A4E2C18B175DB835C43535F6362EBC7F03709B33.key
├── pubring.kbx
├── pubring.kbx~
└── trustdb.gpg

$ gpg --version
gpg (GnuPG) 2.2.25
libgcrypt 1.8.7
Copyright (C) 2020 Free Software Foundation, Inc.
....

# If GnuPG version 2, it uses .kbx for key-rings, helm needs old .gpg format 
$ gpg --export >~/.gnupg/pubring.gpg
$ gpg --export-secret-keys >~/.gnupg/secring.gpg

$ tree ~/.gnupg/
/home/gcollis/.gnupg/
├── openpgp-revocs.d
│   └── 2722B192EAAF3412D30CDC4810DBA57F355F960F.rev
├── private-keys-v1.d
│   ├── 8FC27FEFEF03DB8F7346EA6E52752D0B390E134C.key
│   └── A4E2C18B175DB835C43535F6362EBC7F03709B33.key
├── pubring.gpg
├── pubring.kbx
├── pubring.kbx~
├── secring.gpg
└── trustdb.gpg
```

With GPG key-pair generated, now sign/validate the helm chart.

```bash
$ helm package --sign --key 'Sjfke' --keyring ~/.gnupg/secring.gpg ocp-sample-flask-docker/
Password for key "Sjfke <gcollis@ymail.com>" >  
Successfully packaged chart and saved it to: /home/gcollis/work08/ocp-sample-flask-docker-0.1.0.tgz

$ ls -1
LICENSE
ocp-sample-flask-docker
ocp-sample-flask-docker-0.1.0.tgz
ocp-sample-flask-docker-0.1.0.tgz.prov

$ helm verify ocp-sample-flask-docker-0.1.0.tgz
Signed by: Sjfke <gcollis@ymail.com>
Using Key With Fingerprint: 2722B192EAAF3412D30CDC4810DBA57F355F960F
Chart Hash Verified: sha256:fb430dadf740baeb0d83c9ad0e5fc65d8cb0843568e556651cc3212174b8e4a7

$ helm install lazy-dog --verify ocp-sample-flask-docker-0.1.0.tgz
NAME: lazy-dog
LAST DEPLOYED: Mon May 24 18:24:25 2021
NAMESPACE: sample-flask-helm
STATUS: deployed
REVISION: 1
NOTES:
1. Get the application URL by running these commands:
  export POD_NAME=$(oc get pods --namespace sample-flask-helm -l "app.kubernetes.io/name=ocp-sample-flask-docker,app.kubernetes.io/instance=lazy-dog" -o jsonpath="{.items[0].metadata.name}")
  export CONTAINER_PORT=$(oc get pod --namespace sample-flask-helm $POD_NAME -o jsonpath="{.spec.containers[0].ports[0].containerPort}")
  echo "Visit http://127.0.0.1:8080 to use your application"
  oc --namespace sample-flask-helm port-forward $POD_NAME 8080:$CONTAINER_PORT
[gcollis@morpheus work08]$ oc get all
NAME                                                    READY   STATUS              RESTARTS   AGE
pod/lazy-dog-ocp-sample-flask-docker-564965c65c-pgr8k   0/1     ContainerCreating   0          6s

NAME                                       TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)    AGE
service/lazy-dog-ocp-sample-flask-docker   ClusterIP   10.217.5.48   <none>        8080/TCP   7s

NAME                                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/lazy-dog-ocp-sample-flask-docker   0/1     1            0           7s

NAME                                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/lazy-dog-ocp-sample-flask-docker-564965c65c   1         1         0       7s

$ firefox http://lazy-dog-ocp-sample-flask-docker.apps-crc.testing/
```


