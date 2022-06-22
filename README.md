# ** Under Construction **

## **** DO NOT CLONE ****

# Helm: Simple Flask Example

Demonstrates how to use and create a Helm chart using Docker containers.

The example is taken [Build Flask Docker container and deploy to OpenShift](https://github.com/sjfke/ocp-sample-flask-docker)
which is a simple Python Flask web application used which provides static ``Lorem Ipsum`` pages in various styles. 

Various pre-built docker containers are available on [DockerHub](https://hub.docker.com/repository/docker/sjfke/flask-lorem-ipsum), 
and [Quay IO](https://quay.io/repository/sjfke/flask-lorem-ipsum). Notice there are three version
``v0.1.0``, ``v0.2.0`` and ``v0.3.0`` which are identical apart from the version number and a different 
[bootstrap 4 color theme](https://bootstrap.themes.guide/). 

A ``Helm chart`` will be generated for  ``v0.1.0`` and then changed to support the other versions to demonstrate 
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

``Helm`` interacts with your kubernetes cluster, so your cluster needs to be is up and running, and you need to be logged in!

Note: CRC integrates ``Helm Version 3`` so only requires the ``helm`` command line client.

Easiest way to obtain the correct version is directly from the CRC Console WebUI, using ``?`` and select *Command line tools*.

![Image CRC Console CLI tools](./screenshots/crc-cli-tools.png)

Alternative approaches:
* [Getting started with Helm 3 on OpenShift Container Platform](https://docs.openshift.com/container-platform/4.6/cli_reference/helm_cli/getting-started-with-helm-on-openshift-container-platform.html);
* [Download the latest GitHub helm executable](https://github.com/helm/helm/releases);
* [Follow the Helm Installation guide](https://helm.sh/docs/intro/install/).

Example of downloading, unpacking and installing an archive into ``$HOME/bin``.

```bash
$ curl -L https://mirror.openshift.com/pub/openshift-v4/clients/helm/latest/helm-linux-amd64 -o $HOME/bin/helm
$ chmod +x $HOME/bin/helm
$ helm version
version.BuildInfo{Version:"v3.5.0+6.el8", GitCommit:"77fb4bd2415712e8bfebe943389c404893ad53ce", GitTreeState:"clean", GoVersion:"go1.14.12"}
```

```bash
$ helm repo add stable https://charts.helm.sh/stable # register Artifact Hub (https://artifacthub.io/)

$ helm repo list 
NAME  	URL                          
stable	https://charts.helm.sh/stable

$ helm repo update  # ensure your repo is up to date
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "stable" chart repository
Update Complete. ⎈Happy Helming!⎈

$ helm help      # general help
$ helm repo help # repo command specific help

$ helm completion bash > helm_bash_completion
$ sudo cp helm_bash_completion /etc/bash_completion.d/ # Fedora, (same other Linux distros?)
```

Some useful references:

* [Helm Quickstart Guide](https://v2.helm.sh/docs/using_helm/)
* [Helm Completion](https://helm.sh/docs/helm/helm_completion/)
* [Helm Completion Bash](https://helm.sh/docs/helm/helm_completion_bash/)
* [Helm Documentation](https://v2.helm.sh/docs/)
* [Helm Architecture](https://helm.sh/docs/topics/architecture/)
* [Getting started with Helm 3 on OpenShift Container Platform](https://docs.openshift.com/container-platform/4.6/cli_reference/helm_cli/getting-started-with-helm-on-openshift-container-platform.html)
* [Simple Kubernetes Helm Charts Tutorial with Examples](https://www.golinuxcloud.com/kubernetes-helm-charts/)
* [ArtifactHUB: Example Application Helm Charts](https://artifacthub.io)

## Helm Chart Creation

This approach is based on [Deploy a Go Application on Kubernetes with Helm](https://docs.bitnami.com/tutorials/deploy-go-application-kubernetes-helm/)

First login to your cluster as ``developer`` (rather than ``kubeadmin``), then:

```bash
$ helm create flask-lorem-ipsum
Creating flask-lorem-ipsum

$ tree flask-lorem-ipsum/
flask-lorem-ipsum/
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
This creates a helm-chart, a folder/file structure, that is for ``nginx``, version ``1.16.0``, that does not install without modifications.

Chart (folder) structure
* Chart.yaml - chart/application version;
* values.yaml - where to specify parameterized values for templates folder;
* charts folder - deprecated way to specify subsidiary helm charts;
* tests folder - pre/post-deployment tests (rarely used)
* templates folder - series of yaml (applied in a specific order) and helper (go-lang based) scripts

**NOTE:** Helm YAML indentation is ``two white-space characters``! 

Looking at the flask-lorem-ipsum folder.

### File ``Chart.yaml`` with all the default comments removed, and mine added.

```bash
apiVersion: v2                           # helm api version
name: flask-lorem-ipsum                  # chart name (recreate the chart to change this)
description: A Helm chart for Kubernetes # default description
type: application                        # chart type (application or library)
version: 0.1.0                           # chart version (SemVer https://semver.org/)
appVersion: "1.16.0"                     # application version (quoted string, SemVer format optional)
```

### File ``values.yaml`` which provides values to the folder templates, edited to leave only the salient parts and add my comments. 

```bash
replicaCount: 1             # minimum number of pods to run the application

image:                      # how to identify the docker container
  repository: nginx
  pullPolicy: IfNotPresent  # fetch: IfNotPresent, Always, Never
  tag: ""                   # Overrides chart appVersion.

imagePullSecrets: []        # remote repository login credentials
nameOverride: ""
fullnameOverride: ""

serviceAccount:             # account used to run the application
  create: true
  annotations: {}
  name: ""

podAnnotations: {}          # arbitrary key/value pairs, typically used by Prometheus service discovery 

podSecurityContext: {}      # https://cloud.redhat.com/blog/guide-to-kubernetes-security-context-pod-security-policy-psp

securityContext: {}         # https://kubernetes.io/docs/tasks/configure-pod-container/security-context/

service:                    # https://kubernetes.io/docs/concepts/services-networking/connect-applications-service/
  type: ClusterIP           # Internal cluster IP, (alternatives: NodePort, LoadBalancer)
  port: 80                  # Port service listens on (typically 8080)

ingress:                    # Expose service outside the cluster (https://cloud.redhat.com/blog/kubernetes-ingress-vs-openshift-route)
  enabled: false
  className: ""
  annotations: {}
  hosts:
    - host: chart-example.local
      paths:
        - path: /
          pathType: ImplementationSpecific
  tls: []
  
resources: {}

autoscaling:               # When/how to scale the application (add/remove pods)
  enabled: false
  minReplicas: 1
  maxReplicas: 100
  targetCPUUtilizationPercentage: 80
  
nodeSelector: {}

tolerations: []

affinity: {}
```

This chart (set of files and folders) needs to be modified for the *flask-lorem-ipsum* application, namely 
editing the ``Chart.yaml``, ``values.yaml``, ``templates/deployment.yaml`` and ``templates/NOTES.txt`` files.

## Updating the helm chart for the *flask-lorem-ipsum* application

As mentioned earlier ``helm create`` creates a helm chart for ``nginx``, version ``1.16.0``, and so need modification for *flask-lorem-ipsum*.

### Update Chart.yaml

Summary of changes:
* change ``appVersion: "v0.1.0"``

* **Caution:** The *name* value is hard-coded throughout the template files: 
  * Changing it requires regenerating ``helm create ...`` with the new name;

```bash
    20	# This is the version number of the application being deployed. This version number should be
    21	# incremented each time you make changes to the application. Versions are not expected to
    22	# follow Semantic Versioning. They should reflect the version the application is using.
    23	# It is recommended to use it with quotes.
    24	appVersion: "v0.1.0" # was: "1.16.0"
```
* Update the **appVersion** (*string in quotes*), the default matches an early version of nginx;
  * *appVersion* should be incremented each time application is modified;

### Update values.yaml

Summary of changes:
* add values ``livenessProbePath``, ``readinessProbePath`` and ``containerPort``
* change the repository ``repository: docker.io/sjfke/flask-lorem-ipsum``
* change ``tag: "v0.1.0"`` (overruling Chart.yaml)

**Note:**
  * Helm YAML indentation is ``two white-space characters``! 
  * *repository* and *tag* must match that on [DockerHub](https://hub.docker.com/repository/docker/sjfke/flask-lorem-ipsum) otherwise it will not deploy.

```bash
     1	# Default values for flask-lorem-ipsum.
     2	# This is a YAML-formatted file.
     3	# Declare variables to be passed into your templates.
     4	
     5	livenessProbePath: "/isalive"  # new: line added
     6	readinessProbePath: "/isready" # new: line added
     7	containerPort: 8080            # new: line added
     8	
     9	replicaCount: 1
    10	
    11	image:
    12	  repository: docker.io/sjfke/flask-lorem-ipsum # was: nginx
    13	  pullPolicy: IfNotPresent
    14	  # Overrides the image tag whose default is the chart appVersion.
    15	  tag: "v0.1.0" # was: ""
    16	
    17	imagePullSecrets: []
    18	nameOverride: ""
    19	fullnameOverride: ""
    20	
    21	serviceAccount:
```

### Update templates/deployment.yaml

Change the "deployment.yaml" to use the values added to the "values.yaml" above;

Summary of changes:
* if specified read ``livenessProbePath`` from *values.yaml*, or keep the default;
* if specified read ``readinessProbePath`` from *values.yaml*, or keep the default;
* if specified read ``containerPort`` from *values.yaml*, or keep the default;

```bash
    30	      containers:
    31	        - name: {{ .Chart.Name }}
    32	          securityContext:
    33	            {{- toYaml .Values.securityContext | nindent 12 }}
    34	          image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
    35	          imagePullPolicy: {{ .Values.image.pullPolicy }}
    36	          ports:
    37	            - name: http
    38	              containerPort: {{ .Values.containerPort | default 80 }} # was: containerPort: 80
    39	              protocol: TCP
    40	          livenessProbe:
    41	            httpGet:
    42	              path: {{ .Values.livenessProbePath | default "/" }} # was: path: /
    43	              port: http
    44	          readinessProbe:
    45	            httpGet:
    46	              path: {{ .Values.readinessProbePath | default "/" }} # was: path: /
    47	              port: http
    48	          resources:
    49	            {{- toYaml .Values.resources | nindent 12 }}
```

### Update templates/NOTES.txt

The content of this file is displayed if the deployment is successful, but defaults to displaying ``kubectl`` (Kubernetes) and
not ``oc`` (OpenShift Container Platform) commands.

Summary of changes:
* Use ``sed`` to globally replace every ``kubectl`` occurrence with ``oc``;
* Windows: use [select-string](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/select-string) instead of ``sed``;

```bash
$ cd flask-lorem-ipsum]
$ mv templates/NOTES.txt templates/NOTES.txt.cln
$ sed 's/kubectl/oc/g' templates/NOTES.txt.cln > templates/NOTES.txt
$ rm templates/NOTES.txt.cln
```

```bash
     1	1. Get the application URL by running these commands:
     2	{{- if .Values.ingress.enabled }}
     3	{{- range $host := .Values.ingress.hosts }}
     4	  {{- range .paths }}
     5	  http{{ if $.Values.ingress.tls }}s{{ end }}://{{ $host.host }}{{ .path }}
     6	  {{- end }}
     7	{{- end }}
     8	{{- else if contains "NodePort" .Values.service.type }}
     9	  export NODE_PORT=$(oc get --namespace {{ .Release.Namespace }} -o jsonpath="{.spec.ports[0].nodePort}" services {{ include "flask-lorem-ipsum.fullname" . }})
    10	  export NODE_IP=$(oc get nodes --namespace {{ .Release.Namespace }} -o jsonpath="{.items[0].status.addresses[0].address}")
    11	  echo http://$NODE_IP:$NODE_PORT
    12	{{- else if contains "LoadBalancer" .Values.service.type }}
    13	     NOTE: It may take a few minutes for the LoadBalancer IP to be available.
    14	           You can watch the status of by running 'oc get --namespace {{ .Release.Namespace }} svc -w {{ include "flask-lorem-ipsum.fullname" . }}'
    15	  export SERVICE_IP=$(oc get svc --namespace {{ .Release.Namespace }} {{ include "flask-lorem-ipsum.fullname" . }} --template "{{"{{ range (index .status.loadBalancer.ingress 0) }}{{.}}{{ end }}"}}")
    16	  echo http://$SERVICE_IP:{{ .Values.service.port }}
    17	{{- else if contains "ClusterIP" .Values.service.type }}
    18	  export POD_NAME=$(oc get pods --namespace {{ .Release.Namespace }} -l "app.kubernetes.io/name={{ include "flask-lorem-ipsum.name" . }},app.kubernetes.io/instance={{ .Release.Name }}" -o jsonpath="{.items[0].metadata.name}")
    19	  export CONTAINER_PORT=$(oc get pod --namespace {{ .Release.Namespace }} $POD_NAME -o jsonpath="{.spec.containers[0].ports[0].containerPort}")
    20	  echo "Visit http://127.0.0.1:8080 to use your application"
    21	  oc --namespace {{ .Release.Namespace }} port-forward $POD_NAME 8080:$CONTAINER_PORT
    22	{{- end }}
```

# Test and Deploy the Helm Chart

```bash
$ ls -l
total 4
drwxr-xr-x 4 sjfke sjfke 4096 Dec 30 10:17 flask-lorem-ipsum

$ podman login docker.io # login to docker.io to access containers 
$ oc login -u developer -p developer https://api.crc.testing:6443
$ oc whoami  # developer
$ oc new-project work01

$ helm list # should be empty (as shown)
NAME	NAMESPACE	REVISION	UPDATED	STATUS	CHART	APP VERSION
$ helm lint flask-lorem-ipsum

$ helm lint flask-lorem-ipsum                                # lint the helm chart
$ helm install --dry-run --debug lazy-dog flask-lorem-ipsum  # error-free dry-run with debugging

# install using the helm-chart
$ helm install lazy-dog flask-lorem-ipsum
NAME: lazy-dog
LAST DEPLOYED: Thu Dec 30 10:39:26 2021
NAMESPACE: work01
STATUS: deployed
REVISION: 1
NOTES:
1. Get the application URL by running these commands:
  export POD_NAME=$(oc get pods --namespace work01 -l "app.kubernetes.io/name=flask-lorem-ipsum,app.kubernetes.io/instance=lazy-dog" -o jsonpath="{.items[0].metadata.name}")
  export CONTAINER_PORT=$(oc get pod --namespace work01 $POD_NAME -o jsonpath="{.spec.containers[0].ports[0].containerPort}")
  echo "Visit http://127.0.0.1:8080 to use your application"
  oc --namespace work01 port-forward $POD_NAME 8080:$CONTAINER_PORT


$ oc status
In project work01 on server https://api.crc.testing:6443

svc/lazy-dog-flask-lorem-ipsum - 10.217.4.249:80 -> http
  deployment/lazy-dog-flask-lorem-ipsum deploys docker.io/sjfke/flask-lorem-ipsum:v0.1.0
    deployment #1 running for 2 minutes - 1 pod

View details with 'oc describe <resource>/<name>' or list resources with 'oc get all'.

$ helm list
NAME    	NAMESPACE	REVISION	UPDATED                             	STATUS  	CHART                  	APP VERSION
lazy-dog	work01   	1       	2021-12-30 10:39:26.117738 +0100 CET	deployed	flask-lorem-ipsum-0.1.0	v0.1.0     

$ helm get manifest lazy-dog                                 # check the manifest, expanded install scripts

$ oc get pods # your pod name will be different (READY 1/1 so up and running)
NAME                                          READY   STATUS    RESTARTS   AGE
lazy-dog-flask-lorem-ipsum-7878dbb69c-mk88n   1/1     Running   0          14m

$ oc get all
NAME                                              READY   STATUS    RESTARTS   AGE
pod/lazy-dog-flask-lorem-ipsum-57c9874fb5-clbfj   1/1     Running   0          2m16s

NAME                                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
service/lazy-dog-flask-lorem-ipsum   ClusterIP   10.217.4.218   <none>        80/TCP    2m16s

NAME                                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/lazy-dog-flask-lorem-ipsum   1/1     1            1           2m16s

NAME                                                    DESIRED   CURRENT   READY   AGE
replicaset.apps/lazy-dog-flask-lorem-ipsum-57c9874fb5   1         1         1       2m16s

$ oc expose service/lazy-dog-flask-lorem-ipsum # expose to make accessible (will be fixed later)
route.route.openshift.io/lazy-dog-flask-lorem-ipsum exposed

$ oc get routes
NAME                         HOST/PORT                                            PATH   SERVICES                     PORT   TERMINATION   WILDCARD
lazy-dog-flask-lorem-ipsum   lazy-dog-flask-lorem-ipsum-work01.apps-crc.testing          lazy-dog-flask-lorem-ipsum   http                 None

# URL: http://<service-name>-<project-name>.apps-crc.testing/ # see HOST/PORT column in 'oc get routes'
$ firefox http://lazy-dog-flask-lorem-ipsum-work01.apps-crc.testing

# checking and accessing the pod (note: your pod name will be different)
$ oc get pod lazy-dog-flask-lorem-ipsum-7878dbb69c-mk88n
$ oc logs lazy-dog-flask-lorem-ipsum-7878dbb69c-mk88n
$ oc describe pod lazy-dog-flask-lorem-ipsum-7878dbb69c-mk88n
$ oc rsh lazy-dog-flask-lorem-ipsum-7878dbb69c-mk88n
```

![Image Deployed Helm Chart](./screenshots/deployed-helm-chart.png)

### Uninstall

```bash
$ helm list
NAME    	NAMESPACE	REVISION	UPDATED                             	STATUS  	CHART                  	APP VERSION
lazy-dog	work01   	1       	2021-12-30 10:39:26.117738 +0100 CET	deployed	flask-lorem-ipsum-0.1.0	v0.1.0     
$ helm uninstall lazy-dog
release "lazy-dog" uninstalled
$ helm list -all
NAME	NAMESPACE	REVISION	UPDATED	STATUS	CHART	APP VERSION

# Route not in helm chart, so have to manually delete it.
$ oc get all
NAME                                                  HOST/PORT                                            PATH   SERVICES                     PORT   TERMINATION   WILDCARD
route.route.openshift.io/lazy-dog-flask-lorem-ipsum   lazy-dog-flask-lorem-ipsum-work01.apps-crc.testing          lazy-dog-flask-lorem-ipsum   http                 None

$ oc delete route.route.openshift.io/lazy-dog-flask-lorem-ipsum
route.route.openshift.io "lazy-dog-flask-lorem-ipsum" deleted
```

### Adding a route to expose flask-lorem-ipsum

So far exposing the application outside the CRC cluster is done manually using the command:
* ``oc expose service/lazy-dog-flask-lorem-ipsum``

How can helm be made to do this?

The ``oc expose service`` command is creating a `oc route` to the `lazy-dog-flask-lorem-ipsum` service, 
which is accessible via `lazy-dog-flask-lorem-ipsum-work01.apps-crc.testing` URL.

From looking at the ``oc get routes -o yaml`` output it is possible to find the information required to 
create a ``route.yaml`` file to have ``helm`` do this, that is the `metadata.name` and `spec` section.

```bash
$ oc get routes -o yaml
apiVersion: v1
items:
- apiVersion: route.openshift.io/v1
  kind: Route
  metadata:
    annotations:
      openshift.io/host.generated: "true"
    creationTimestamp: "2022-06-18T17:04:54Z"
    labels:
      app.kubernetes.io/instance: lazy-dog
      app.kubernetes.io/managed-by: Helm
      app.kubernetes.io/name: flask-lorem-ipsum
      app.kubernetes.io/version: v0.1.0
      helm.sh/chart: flask-lorem-ipsum-0.1.0
    name: lazy-dog-flask-lorem-ipsum
    namespace: work01
    resourceVersion: "96932"
    uid: 9f958112-1b9d-4330-adfd-adce5fb22ff7
  spec:
    host: lazy-dog-flask-lorem-ipsum-work01.apps-crc.testing
    port:
      targetPort: http
    to:
      kind: Service
      name: lazy-dog-flask-lorem-ipsum
      weight: 100
    wildcardPolicy: None
  status:
    ingress:
    - conditions:
      - lastTransitionTime: "2022-06-18T17:04:54Z"
        status: "True"
        type: Admitted
      host: lazy-dog-flask-lorem-ipsum-work01.apps-crc.testing
      routerCanonicalHostname: router-default.apps-crc.testing
      routerName: default
      wildcardPolicy: None
kind: List
metadata:
  resourceVersion: ""
  selfLink: ""
```
The `route.yaml` is as follows, notice the `host` URL has been simplified. 

```bash
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: {{ include "flask-lorem-ipsum.fullname" . }}
  labels:
    {{- include "flask-lorem-ipsum.labels" . | nindent 4 }}
spec:
  host: flask-lorem-ipsum.apps-crc.testing
  port:
    targetPort: http
  to:
    kind: Service
    name: {{ include "flask-lorem-ipsum.fullname" . }}
    weight: 100
  wildcardPolicy: None
```
Also note the `name` and `labels` are using the helm chart values used in other chart YAML files, 
which are defined in `templates/_helpers.tpl` which is created when the chart was generated.

Add the `route.yaml` file to `templates` folder, and redeploy and test.

```bash
$ ls -l
total 4
drwxr-xr-x 4 sjfke sjfke 4096 Dec 30 10:17 flask-lorem-ipsum

$ podman login docker.io # login to docker.io to access containers 

$ oc login -u developer -p developer https://api.crc.testing:6443
$ oc whoami  # developer
$ oc new-project work01

$ helm list # should be empty (as shown)
NAME	NAMESPACE	REVISION	UPDATED	STATUS	CHART	APP VERSION
$ helm lint flask-lorem-ipsum

$ helm lint flask-lorem-ipsum                                # lint the helm chart
$ helm install --dry-run --debug lazy-dog flask-lorem-ipsum  # error-free dry-run with debugging

# install using the helm-chart
$ helm install lazy-dog flask-lorem-ipsum

$ oc get routes # route added by helm with different host URL
NAME                         HOST/PORT                            PATH   SERVICES                     PORT   TERMINATION   WILDCARD
lazy-dog-flask-lorem-ipsum   flask-lorem-ipsum.apps-crc.testing          lazy-dog-flask-lorem-ipsum   http                 None
$ oc get services
NAME                         TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
lazy-dog-flask-lorem-ipsum   ClusterIP   10.217.4.45   <none>        80/TCP    80m

$ firefox http://flask-lorem-ipsum.apps-crc.testing/

# Now helm uninstall
$ helm list
NAME    	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART                  	APP VERSION
lazy-dog	work01   	1       	2022-06-21 18:27:56.91539426 +0200 CEST	deployed	flask-lorem-ipsum-0.1.0	v0.1.0     
$ helm uninstall lazy-dog
release "lazy-dog" uninstalled
$ helm list
NAME	NAMESPACE	REVISION	UPDATED	STATUS	CHART	APP VERSION
$ oc get routes # route was created by helm so is removed
No resources found in work01 namespace.

```
While this works, moving the `route.spec` section to the `values.yaml` make things clearer and easier to maintain.

### working references to go:

* [A Guide to using Routes, Ingress and Gateway APIs in Kubernetes without vendor lock-in](https://cloud.redhat.com/blog/a-guide-to-using-routes-ingress-and-gateway-apis-in-kubernetes-without-vendor-lock-in)
* [sonatype-nexus/values.yaml](https://github.com/nokia/helm-charts/blob/master/stable/sonatype-nexus/values.yaml)
* [sonatype-nexus/templates/route.yaml](https://github.com/nokia/helm-charts/blob/master/stable/sonatype-nexus/templates/route.yaml)
* [Common: The Helm Helper Chart](https://technosophos.github.io/common-chart/)


## Deploying a different version

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

