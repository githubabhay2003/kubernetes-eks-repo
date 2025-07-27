\# Hands-On Kubernetes Assessment on AWS EKS: Part 1 - Core Concepts



Welcome\\! This repository documents a comprehensive, hands-on assessment of core Kubernetes concepts, executed on Amazon's Elastic Kubernetes Service (EKS). The goal of this exercise was to validate and demonstrate practical, real-world skills in deploying, managing, and troubleshooting applications on Kubernetes.



This document serves as a log of that journey, including detailed explanations, all necessary manifests, and—most importantly—the real-world challenges encountered and their resolutions.



\### Recommended Folder Structure



For clarity and easy navigation, the YAML manifests are organized into folders corresponding to each task.



```

part-1-foundations/

├── 01-cluster-setup/

│   └── cluster.yml

├── 02-pods-and-labels/

│   └── nginx-pod.yml

├── 03-deployments/

│   └── nginx-deployment.yml

├── 04-configmaps-and-secrets/

│   └── README.md

├── 05-services/

│   └── nginx-service.yml

├── 06-ingress/

│   └── nginx-ingress.yml

├── 07-persistent-storage/

│   ├── ebs-sc.yml

│   ├── my-ebs-claim.yml

│   └── storage-test-pod.yml

├── 08-statefulsets/

│   ├── redis-headless-service.yml

│   └── redis-statefulset.yml

├── 09-probes-and-resources/

│   └── README.md

├── 10-rbac/

│   ├── pod-reader-role.yml

│   └── rbac-test-pod.yml

└── README.md

```



-----



\## Table of Contents



1\.  \[Task 1: Cluster Setup]

2\.  \[Task 2: Pods, Labels, and Namespaces]

3\.  \[Task 3: Deployments \& Replicas]

4\.  \[Task 4: Configuration with ConfigMaps \& Secrets]

5\.  \[Task 5: Services]

6\.  \[Task 6: Ingress \& AWS Load Balancer Controller]

7\.  \[Task 7: Persistent Storage with EBS]

8\.  \[Task 8: StatefulSets]

9\.  \[Task 9: Probes \& Resource Management]

10\. \[Task 10: RBAC \& Service Accounts]

11\. \[Key Challenges \& Resolutions]



-----



\## Task 1: Cluster Setup



\*\*Goal:\*\* Provision a production-ready EKS cluster.



\*\*Explanation:\*\* The foundation of any Kubernetes workload on AWS is the EKS cluster. We used `eksctl`, the official CLI for EKS, to create a cluster. The final successful cluster (`my-eks-assessment-v3`) was created using a declarative YAML file, which is a best practice. This approach ensures reproducibility and allows for the automatic installation of essential add-ons like the \*\*IAM OIDC provider\*\*, the \*\*AWS Load Balancer Controller\*\*, and the \*\*EBS CSI Driver\*\*.



\#### Manifest: `01-cluster-setup/cluster.yml`



```yaml

apiVersion: eksctl.io/v1alpha5

kind: ClusterConfig



metadata:

&nbsp; name: my-eks-assessment-v3

&nbsp; region: ap-south-1

&nbsp; version: "1.28"



iam:

&nbsp; withOIDC: true



managedNodeGroups:

&nbsp; - name: standard-workers

&nbsp;   instanceType: t3.medium

&nbsp;   desiredCapacity: 2

&nbsp;   

addons:

&nbsp; - name: aws-load-balancer-controller

&nbsp; - name: aws-ebs-csi-driver 

```



\#### Command



```sh

eksctl create cluster -f cluster.yml

```



-----



\## Task 2: Pods, Labels, and Namespaces



\*\*Goal:\*\* Deploy a basic Nginx pod and organize it with a namespace and labels.



\*\*Explanation:\*\* \*\*Pods\*\* are the smallest deployable units in Kubernetes. We deployed a simple Nginx pod. To keep our work isolated, we created a \*\*Namespace\*\* named `foundations`. \*\*Labels\*\* are key/value pairs that are attached to objects and are used to organize and select subsets of objects.



\#### Manifest: `02-pods-and-labels/nginx-pod.yml`



```yaml

apiVersion: v1

kind: Pod

metadata:

&nbsp; name: nginx-pod

&nbsp; namespace: foundations

&nbsp; labels:

&nbsp;   app: webserver

&nbsp;   env: dev

spec:

&nbsp; containers:

&nbsp; - name: nginx-container

&nbsp;   image: nginx:latest

&nbsp;   ports:

&nbsp;   - containerPort: 80

```



-----



\## Task 3: Deployments \& Replicas



\*\*Goal:\*\* Manage a set of replica pods using a Deployment.



\*\*Explanation:\*\* A \*\*Deployment\*\* provides declarative updates for Pods and ReplicaSets. We transitioned from managing a single pod to a Deployment, which ensures a specified number of \*\*replicas\*\* are always running. It also enables seamless rolling updates and scaling.



\#### Manifest: `03-deployments/nginx-deployment.yml` (Final Version from Task 9)



```yaml

apiVersion: apps/v1

kind: Deployment

metadata:

&nbsp; name: nginx-deployment

&nbsp; namespace: foundations

spec:

&nbsp; replicas: 2

&nbsp; selector:

&nbsp;   matchLabels:

&nbsp;     app: webserver

&nbsp; template:

&nbsp;   metadata:

&nbsp;     labels:

&nbsp;       app: webserver

&nbsp;   spec:

&nbsp;     containers:

&nbsp;     - name: nginx-container

&nbsp;       image: nginx:1.21.6

&nbsp;       ports:

&nbsp;       - containerPort: 80

&nbsp;       resources:

&nbsp;         requests:

&nbsp;           memory: "128Mi"

&nbsp;           cpu: "100m"

&nbsp;         limits:

&nbsp;           memory: "256Mi"

&nbsp;           cpu: "200m"

&nbsp;       readinessProbe:

&nbsp;         httpGet:

&nbsp;           path: /

&nbsp;           port: 80

&nbsp;         initialDelaySeconds: 5

&nbsp;         periodSeconds: 10

&nbsp;       livenessProbe:

&nbsp;         tcpSocket:

&nbsp;           port: 80

&nbsp;         initialDelaySeconds: 15

&nbsp;         periodSeconds: 20

&nbsp;       env:

&nbsp;       - name: API\_KEY

&nbsp;         valueFrom:

&nbsp;           secretKeyRef:

&nbsp;             name: app-secret

&nbsp;             key: API\_KEY

&nbsp;       volumeMounts:

&nbsp;       - name: nginx-index-volume

&nbsp;         mountPath: /usr/share/nginx/html

&nbsp;     volumes:

&nbsp;     - name: nginx-index-volume

&nbsp;       configMap:

&nbsp;         name: nginx-config

```



-----



\## Task 4: Configuration with ConfigMaps \& Secrets



\*\*Goal:\*\* Externalize configuration and sensitive data from the application container.



\*\*Explanation:\*\* \*\*ConfigMaps\*\* are used to store non-confidential data in key-value pairs, while \*\*Secrets\*\* are used for sensitive data like passwords or API keys. We created both and injected them into our Nginx deployment—the ConfigMap was mounted as a volume to replace the default web page, and the Secret was exposed as an environment variable.



\#### Commands



```sh

\# Create the ConfigMap

kubectl create configmap nginx-config \\

--from-literal=index.html='<h1>Welcome! Probes are working!</h1>' \\

--namespace foundations



\# Create the Secret

kubectl create secret generic app-secret \\

--from-literal=API\_KEY="S3cr3tV@lu3!" \\

--namespace foundations

```



-----



\## Task 5: Services



\*\*Goal:\*\* Expose the application pods using a stable network endpoint.



\*\*Explanation:\*\* A \*\*Service\*\* is an abstract way to expose an application running on a set of Pods. We explored three types:



1\.  \*\*ClusterIP:\*\* Exposes the service on an internal IP, making it reachable only from within the cluster.

2\.  \*\*NodePort:\*\* Exposes the service on each Node’s IP at a static port.

3\.  \*\*LoadBalancer:\*\* Exposes the service externally using a cloud provider’s load balancer. In EKS, this creates an AWS Network Load Balancer (NLB).



\#### Manifest: `05-services/nginx-service.yml`



```yaml

apiVersion: v1

kind: Service

metadata:

&nbsp; name: nginx-internal-svc

&nbsp; namespace: foundations

spec:

&nbsp; type: ClusterIP

&nbsp; selector:

&nbsp;   app: webserver

&nbsp; ports:

&nbsp;   - protocol: TCP

&nbsp;     port: 80

&nbsp;     targetPort: 80

```



-----



\## Task 6: Ingress \& AWS Load Balancer Controller



\*\*Goal:\*\* Manage external access to services using a Layer 7 load balancer.



\*\*Explanation:\*\* While a `LoadBalancer` service provides Layer 4 access, an \*\*Ingress\*\* is used for Layer 7 (HTTP/S) routing. It can handle multiple services, path-based routing, and SSL termination. On EKS, the \*\*AWS Load Balancer Controller\*\* fulfills Ingress resources by provisioning an AWS Application Load Balancer (ALB).



\#### Manifest: `06-ingress/nginx-ingress.yml`



```yaml

apiVersion: networking.k8s.io/v1

kind: Ingress

metadata:

&nbsp; name: nginx-ingress

&nbsp; namespace: foundations

&nbsp; annotations:

&nbsp;   alb.ingress.kubernetes.io/scheme: internet-facing

&nbsp;   alb.ingress.kubernetes.io/target-type: ip

spec:

&nbsp; ingressClassName: alb

&nbsp; rules:

&nbsp; - http:

&nbsp;     paths:

&nbsp;     - path: /

&nbsp;       pathType: Prefix

&nbsp;       backend:

&nbsp;         service:

&nbsp;           name: nginx-internal-svc

&nbsp;           port:

&nbsp;             number: 80

```



-----



\## Task 7: Persistent Storage with EBS



\*\*Goal:\*\* Provide persistent storage to a pod using AWS EBS.



\*\*Explanation:\*\* We installed the \*\*EBS CSI Driver\*\* to allow Kubernetes to manage EBS volumes. We then defined a \*\*StorageClass\*\* as a template for storage, a \*\*PersistentVolumeClaim (PVC)\*\* to request storage from that class, and finally mounted the PVC into a pod to store data that persists even if the pod is deleted.



\#### Manifests



&nbsp; \* `07-persistent-storage/ebs-sc.yml`

&nbsp;   ```yaml

&nbsp;   apiVersion: storage.k8s.io/v1

&nbsp;   kind: StorageClass

&nbsp;   metadata:

&nbsp;     name: ebs-sc

&nbsp;   provisioner: ebs.csi.aws.com

&nbsp;   volumeBindingMode: WaitForFirstConsumer

&nbsp;   ```

&nbsp; \* `07-persistent-storage/my-ebs-claim.yml`

&nbsp;   ```yaml

&nbsp;   apiVersion: v1

&nbsp;   kind: PersistentVolumeClaim

&nbsp;   metadata:

&nbsp;     name: my-ebs-claim

&nbsp;     namespace: foundations

&nbsp;   spec:

&nbsp;     accessModes:

&nbsp;       - ReadWriteOnce

&nbsp;     storageClassName: ebs-sc

&nbsp;     resources:

&nbsp;       requests:

&nbsp;         storage: 5Gi

&nbsp;   ```



-----



\## Task 8: StatefulSets



\*\*Goal:\*\* Deploy a stateful application (Redis) with stable identity and storage.



\*\*Explanation:\*\* A \*\*StatefulSet\*\* is a workload API object used to manage stateful applications. It provides stable, unique network identifiers, stable persistent storage, and ordered deployment/scaling. We used a StatefulSet to deploy a Redis instance, leveraging its `volumeClaimTemplates` to automatically provision a unique PVC for the pod.



\#### Manifests



&nbsp; \* `08-statefulsets/redis-headless-service.yml`

&nbsp;   ```yaml

&nbsp;   apiVersion: v1

&nbsp;   kind: Service

&nbsp;   metadata:

&nbsp;     name: redis

&nbsp;     namespace: foundations

&nbsp;   spec:

&nbsp;     clusterIP: None

&nbsp;     selector:

&nbsp;       app: redis

&nbsp;     ports:

&nbsp;       - port: 6379

&nbsp;         targetPort: 6379

&nbsp;   ```

&nbsp; \* `08-statefulsets/redis-statefulset.yml`

&nbsp;   ```yaml

&nbsp;   apiVersion: apps/v1

&nbsp;   kind: StatefulSet

&nbsp;   metadata:

&nbsp;     name: redis

&nbsp;     namespace: foundations

&nbsp;   spec:

&nbsp;     serviceName: "redis"

&nbsp;     replicas: 1

&nbsp;     selector:

&nbsp;       matchLabels:

&nbsp;         app: redis

&nbsp;     template:

&nbsp;       metadata:

&nbsp;         labels:

&nbsp;           app: redis

&nbsp;       spec:

&nbsp;         containers:

&nbsp;         - name: redis

&nbsp;           image: redis:6.2-alpine

&nbsp;           ports:

&nbsp;           - containerPort: 6379

&nbsp;           volumeMounts:

&nbsp;           - name: redis-data

&nbsp;             mountPath: /data

&nbsp;     volumeClaimTemplates:

&nbsp;     - metadata:

&nbsp;         name: redis-data

&nbsp;       spec:

&nbsp;         accessModes: \[ "ReadWriteOnce" ]

&nbsp;         storageClassName: "ebs-sc"

&nbsp;         resources:

&nbsp;           requests:

&nbsp;             storage: 2Gi

&nbsp;   ```



-----



\## Task 9: Probes \& Resource Management



\*\*Goal:\*\* Improve application reliability with health checks and resource definitions.



\## \*\*Explanation:\*\* We added \*\*Liveness\*\* and \*\*Readiness Probes\*\* to our Nginx Deployment. The Readiness probe determines if a container is ready to serve traffic, and the Liveness probe determines if it needs to be restarted. We also set \*\*Resource Requests and Limits\*\* to ensure predictable performance and prevent pods from consuming excessive cluster resources. The final, updated manifest is listed under \[Task 3](https://www.google.com/search?q=%23task-3-deployments--replicas).



\## Task 10: RBAC \& Service Accounts



\*\*Goal:\*\* Restrict pod permissions using Role-Based Access Control (RBAC).



\*\*Explanation:\*\* We implemented the principle of least privilege. We created a \*\*ServiceAccount\*\* (`readonly-sa`), a \*\*Role\*\* (`pod-reader`) with permissions to only `get` and `list` pods, and a \*\*RoleBinding\*\* to connect them. We then launched a test pod using this service account and proved that it could list pods but was forbidden from deleting them.



\#### Manifest: `10-rbac/pod-reader-role.yml`



```yaml

apiVersion: rbac.authorization.k8s.io/v1

kind: Role

metadata:

&nbsp; namespace: foundations

&nbsp; name: pod-reader

rules:

\- apiGroups: \[""]

&nbsp; resources: \["pods"]

&nbsp; verbs: \["get", "watch", "list"]

```



-----



\## Key Challenges \& Resolutions



This assessment involved significant real-world troubleshooting.



1\.  \*\*Challenge:\*\* Corrupted CloudShell Environment \& Outdated `eksctl`.



&nbsp;     \* \*\*Symptom:\*\* The `eksctl` binary could not be updated, and it was unable to resolve EKS add-on versions, causing all `eksctl create addon` commands to fail.

&nbsp;     \* \*\*Resolution:\*\* The CloudShell environment was deemed corrupted. The final solution was to perform a full \*\*"Delete AWS CloudShell home directory"\*\* from the Actions menu. This provided a fresh environment where a new version of `eksctl` could be installed and function correctly.



2\.  \*\*Challenge:\*\* AWS Load Balancer Controller Installation \& Ingress Failures.



&nbsp;     \* \*\*Symptom:\*\* The Ingress `ADDRESS` field would not populate, and controller logs showed a variety of errors, from webhook failures to IAM `AccessDenied` messages.

&nbsp;     \* \*\*Root Cause:\*\* This was a multi-layered problem, including:

&nbsp;       1.  The initial EKS cluster was created without the IAM OIDC provider.

&nbsp;       2.  The IAM Role for the controller's Service Account (IRSA) was not correctly linked because the OIDC provider was missing.

&nbsp;       3.  The VPC subnets were missing the required tags (`kubernetes.io/role/elb` and `kubernetes.io/cluster/...`).

&nbsp;       4.  A final `AccessDenied` error was traced to the IAM policy not being attached to the correct, `eksctl`-generated IAM role name.

&nbsp;     \* \*\*Resolution:\*\* The ultimate solution was to create a new cluster using a declarative `cluster.yml` file that correctly enabled `iam.withOIDC: true` and installed the add-ons automatically. This single change correctly configured all IAM roles, policies, and service account annotations from the start.



3\.  \*\*Challenge:\*\* Readiness Probe Failures (HTTP 403).



&nbsp;     \* \*\*Symptom:\*\* After adding probes, new Nginx pods were stuck in a `READY 0/1` state. `kubectl describe` showed the readiness probe was failing with an HTTP `403 Forbidden` error.

&nbsp;     \* \*\*Root Cause:\*\* Investigation inside the pod (`ls -l /usr/share/nginx/html`) revealed the directory was empty. The deployment was trying to mount a `ConfigMap` that existed but had no data in it. An empty web root with no index file causes Nginx to return a `403 Forbidden` error by default.

&nbsp;     \* \*\*Resolution:\*\* The empty `ConfigMap` was deleted and recreated with the correct `index.html` data. The failing pod was then deleted, and its replacement started successfully, passed the readiness probe, and became `1/1 READY`.

