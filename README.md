# ** Under Construction **

## **** DO NOT CLONE ****

# Helm: Simple Flask Example

Demonstrates how to use and create a Helm chart using Docker containers.

The example is taken [Build Flask Docker container and deploy to OpenShift](https://github.com/sjfke/ocp-sample-flask-docker)
which is a simple Python Flask web application used which provides static ``Lorem Ipsum`` pages in various styles. 

Various pre-built docker containers are available on [DockerHub](https://hub.docker.com/repository/docker/sjfke/flask-lorem-ipsum), 
and [Quay IO](https://quay.io/repository/sjfke/flask-lorem-ipsum). Notice there are three version
``v.0.1.0``, ``v.0.2.0`` and ``v.0.3.0`` which are identical apart from the version number and a different 
[bootstrap 4 color theme](https://bootstrap.themes.guide/). 

A ``Helm chart`` will be generated for  ``v.0.1.0`` and then changed to support the other versions to demonstrate 
[helm upgrade](https://helm.sh/docs/helm/helm_upgrade/) and [helm rollback](https://helm.sh/docs/helm/helm_rollback/)

## OpenShift Environment

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
$ oc whoami                 # kubeadmin
$ oc project                # current project

$ oc logout                 # logout
$ crc stop                  # shutdown the cluster
```

To obtain the default CRC ``kubeadmin`` password, use the following command.

```bash
$ crc console --credentials
```

## Helm Installation

If using CRC [Getting started with Helm 3 on OpenShift Container Platform](https://docs.openshift.com/container-platform/4.6/cli_reference/helm_cli/getting-started-with-helm-on-openshift-container-platform.html) otherwise follow [download the required executable](https://github.com/helm/helm/releases) or [Helm Installation](https://v2.helm.sh/docs/install/).

``Helm`` interacts with your kubernetes cluster, so your cluster needs to be is up and running, and you need to be logged in!

Note: ``Helm Version 3`` is integrated with CRC and only has a ``helm`` command line client.

Installation involves downloading and unpacking the appropriate archive.

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
$ helm repo update                                   # ensure your charts up to date
```

Some useful references:

* [Helm Quickstart Guide](https://v2.helm.sh/docs/using_helm/)
* [Helm Documentation](https://v2.helm.sh/docs/)
* [Helm Architecture](https://helm.sh/docs/topics/architecture/)
* [Getting started with Helm 3 on OpenShift Container Platform](https://docs.openshift.com/container-platform/4.6/cli_reference/helm_cli/getting-started-with-helm-on-openshift-container-platform.html)
* [Simple Kubernetes Helm Charts Tutorial with Examples](https://www.golinuxcloud.com/kubernetes-helm-charts/)

## Helm Chart Creation

This approach is based on [Deploy a Go Application on Kubernetes with Helm](https://docs.bitnami.com/tutorials/deploy-go-application-kubernetes-helm/)

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
This creates a helm-chart based on ``nginx``, version ``1.16.0`` as shown above.
This chart (set of files and folders) needs to be edited, to change the values for our application.

**Note:** 

* Do not try to install it, the chart is very old and generates errors;
* Changes to these files have a suffix comment like ``# was: "1.16.0"``, better to remove them;
* There is no guarantee the line numbers will match exactly.
* Helm YAML indentation is ``two white-space characters``! 

### Update Chart.yaml (describes the helm-chart and its version)

The *name* is used throughout the template files, so do not change it, regenerate with the correct name if wrong.

Update the **appVersion** (*string in quotes*), the generated default matches an early version of nginx.

**Note:** 
* *appVersion* should be incremented each time application is modified.


```bash
20:# This is the version number of the application being deployed. This version number should be
21:# incremented each time you make changes to the application. Versions are not expected to
22:# follow Semantic Versioning. They should reflect the version the application is using.
23:# It is recommended to use it with quotes.
24:appVersion: "0.1.0" # was: "1.16.0"
```

### Update values.yaml (parameter values used in templates folder)

**Note:**
* *repository* and *tag* must match that on DockerHub otherwise it will not deploy.

Summary of changes:
* add ``livenessProbePath``, ``readinessProbePath`` and ``containerPort``
* change ``repository: docker.io/sjfke/ocp-sample-flask-docker``
* change ``tag: "v0.1.0"``

```bash
 1:# Default values for ocp-sample-flask-docker.
 2:# This is a YAML-formatted file.
 3:# Declare variables to be passed into your templates.
 4:
 5:livenessProbePath: "/isalive"  # new: line added
 6:readinessProbePath: "/isready" # new: line added
 7:containerPort: 8080            # new: line added
 8:replicaCount: 1
 9:
10:image:
11:  repository: docker.io/sjfke/ocp-sample-flask-docker # was: nginx
12:  pullPolicy: IfNotPresent
13:  # Overrides the image tag whose default is the chart appVersion.
14:  tag: "v0.1.0" # was: ""
```


### Update templates/deployment.yaml

Change the deployment process for the containers to read the values from "values.yaml";

Summary of changes:
* if specified read ``livenessProbePath`` from *values.yaml*, or keep the default;
* if specified read ``readinessProbePath`` from *values.yaml*, or keep the default;
* if specified read ``containerPort`` from *values.yaml*, or keep the default;

```bash
30:      containers:
31:        - name: {{ .Chart.Name }}
32:          securityContext:
33:            {{- toYaml .Values.securityContext | nindent 12 }}
34:          image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
35:          imagePullPolicy: {{ .Values.image.pullPolicy }}
36:          ports:
37:            - name: http
38:              containerPort: {{ .Values.containerPort | default 80 }} # was: containerPort: 80
39:              protocol: TCP
40:          livenessProbe:
41:            httpGet:
42:              path: {{ .Values.livenessProbePath | default "/" }} # was: path: /
43:              port: http
44:          readinessProbe:
45:            httpGet:
46:              path: {{ .Values.readinessProbePath | default "/" }} # was: path: /
47:              port: http
48:          resources:
```

### Update template/Notes.txt

The content of this file is displayed if the deployment is successful, but they reference ``kubectl`` (Kubernetes) and
not ``oc`` (OpenShift Container Platform) commands.

Summary of changes:
* Use ``sed`` to globally replace every ``kubectl`` occurrence with ``oc``;
* Windows: use [select-string](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/select-string) instead of ``sed``;

```bash
$ mv ocp-sample-flask-docker/templates/NOTES.txt ocp-sample-flask-docker/templates/NOTES.txt.cln 
$ sed 's/kubectl/oc/g' ocp-sample-flask-docker/templates/NOTES.txt.cln > ocp-sample-flask-docker/templates/NOTES.txt
$ rm ocp-sample-flask-docker/templates/NOTES.txt.cln
```
# Build and Test the Helm Chart

Unless you have already fetched the required images, pull them now:

```bash
$ podman pull docker.io/sjfke/ocp-sample-flask-docker:v0.1.0 # Pull from DockerHub (docker.io - prefix)
$ podman pull docker.io/sjfke/ocp-sample-flask-docker:v0.1.1
$ podman pull docker.io/sjfke/ocp-sample-flask-docker:v0.1.2
```

and then create a new OCP project.
 
```bash
$ oc whoami   # kubeadmin
$ oc new-project sample-flask-helm

$ helm list
NAME	NAMESPACE	REVISION	UPDATED	STATUS	CHART	APP VERSION

$ helm lint ocp-sample-flask-docker                               # lint the helm chart
$ helm install --dry-run --debug lazy-dog ocp-sample-flask-docker # dry-run with debugging
$ helm get manifest lazy-dog                                      # check the manifest, expanded install scripts

# install using the helm-chart
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

$ ls -1 *tgz*
ocp-sample-flask-docker-0.1.0.tgz
ocp-sample-flask-docker-0.1.0.tgz.prov

$ helm verify ocp-sample-flask-docker-0.1.0.tgz
Signed by: Sjfke <gcollis@ymail.com>
Using Key With Fingerprint: 2722B192EAAF3412D30CDC4810DBA57F355F960F
Chart Hash Verified: sha256:fb430dadf740baeb0d83c9ad0e5fc65d8cb0843568e556651cc3212174b8e4a7
```
Installing the signed/verified helm chart.

```bash
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

# URL: http://<service-name>-<project-name>.apps-crc.testing/
$ firefox http://lazy-dog-ocp-sample-flask-docker-sample-flask-helm.apps-crc.testing/

# uninstall
$ helm uninstall lazy-dog  # release "lazy-dog" uninstalled
```
************************************************
*********** 30.05.2021 *************************
************************************************

## Creating a Helm Chart Repo

There are many ways to do this, a helm chart repository is simply a web-server with a given structure.

The approach described uses [GitHub Pages](https://helm.sh/docs/topics/chart_repository/#github-pages-example), but many others are possible such as, [Google Cloud Storage](https://helm.sh/docs/topics/chart_repository/#google-cloud-storage), [JFrog Artifactory](https://helm.sh/docs/topics/chart_repository/#jfrog-artifactory).

* [Helm: Chart Repository Guide](https://helm.sh/docs/topics/chart_repository/)
* [Automate Helm chart repository publishing with GitHub Actions and Pages](https://medium.com/@stefanprodan/automate-helm-chart-repository-publishing-with-github-actions-and-pages-8a374ce24cf4)

First create a new GitHub project to host your helm repos, [sjfke/helm-repos](https://github.com/sjfke/helm-repos), following the instructions on [GitHub Pages](https://helm.sh/docs/topics/chart_repository/#github-pages-example).


### work09 ###
[gcollis@morpheus helm-repos]$ git remote show origin
* remote origin
  Fetch URL: git@github.com:sjfke/helm-repos.git
  Push  URL: git@github.com:sjfke/helm-repos.git
  HEAD branch: main
  Remote branches:
    gh-pages tracked
    main     tracked
  Local branch configured for 'git pull':
    main merges with remote main
  Local ref configured for 'git push':
    main pushes to main (up to date)

2021-05-23: make a helm-repo repo as gh_pages, and use 'cr'

