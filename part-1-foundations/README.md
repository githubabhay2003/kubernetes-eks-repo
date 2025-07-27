\# Hands-On Kubernetes Assessment on AWS EKS: Part 1 - Core Concepts



This document details a comprehensive, hands-on assessment of core Kubernetes concepts, executed on Amazon's Elastic Kubernetes Service (EKS). The goal of this exercise was to validate and demonstrate practical, real-world skills in deploying, managing, and troubleshooting applications on Kubernetes.



This serves as a log of that journey, including detailed explanations, all necessary manifests, and—most importantly—the real-world challenges encountered and their resolutions.



\### Recommended Folder Structure
```plaintext
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
---

\## Table of Contents



1\.  \[Task 1: Cluster Setup](#task-1-cluster-setup)

2\.  \[Task 2: Pods, Labels, and Namespaces](#task-2-pods-labels-and-namespaces)

3\.  \[Task 3: Deployments \& Replicas](#task-3-deployments--replicas)

4\.  \[Task 4: Configuration with ConfigMaps \& Secrets](#task-4-configuration-with-configmaps--secrets)

5\.  \[Task 5: Services](#task-5-services)

6\.  \[Task 6: Ingress \& AWS Load Balancer Controller](#task-6-ingress--aws-load-balancer-controller)

7\.  \[Task 7: Persistent Storage with EBS](#task-7-persistent-storage-with-ebs)

8\.  \[Task 8: StatefulSets](#task-8-statefulsets)

9\.  \[Task 9: Probes \& Resource Management](#task-9-probes--resource-management)

10\. \[Task 10: RBAC \& Service Accounts](#task-10-rbac--service-accounts)

11\. \[Key Challenges \& Resolutions](#key-challenges--resolutions)



---

\## Task 1: Cluster Setup



\*\*Goal:\*\* Provision a production-ready EKS cluster.



\*\*Explanation:\*\* The foundation of any Kubernetes workload on AWS is the EKS cluster. We used `eksctl`, the official CLI for EKS, to create a cluster. The final successful cluster (`my-eks-assessment-v3`) was created using a declarative YAML file, which is a best practice. This approach ensures reproducibility and allows for the automatic installation of essential add-ons like the \*\*IAM OIDC provider\*\*, the \*\*AWS Load Balancer Controller\*\*, and the \*\*EBS CSI Driver\*\*.



---

\## Task 2: Pods, Labels, and Namespaces



\*\*Goal:\*\* Deploy a basic Nginx pod and organize it with a namespace and labels.



\*\*Explanation:\*\* \*\*Pods\*\* are the smallest deployable units in Kubernetes. To keep our work isolated, we created a \*\*Namespace\*\* named `foundations`. \*\*Labels\*\* are key/value pairs that are attached to objects and are used to organize and select subsets of objects.



---

\## Task 3: Deployments \& Replicas



\*\*Goal:\*\* Manage a set of replica pods using a Deployment.



\*\*Explanation:\*\* A \*\*Deployment\*\* provides declarative updates for Pods and ReplicaSets. It ensures a specified number of \*\*replicas\*\* are always running and enables seamless rolling updates and scaling.



---

\## Task 4: Configuration with ConfigMaps \& Secrets



\*\*Goal:\*\* Externalize configuration and sensitive data from the application container.



\*\*Explanation:\*\* \*\*ConfigMaps\*\* store non-confidential data in key-value pairs, while \*\*Secrets\*\* are used for sensitive data. We injected a ConfigMap as a volume to replace the default web page and a Secret as an environment variable.



---

\## Task 5: Services



\*\*Goal:\*\* Expose the application pods using a stable network endpoint.



\*\*Explanation:\*\* A \*\*Service\*\* exposes an application running on a set of Pods. We explored \*\*ClusterIP\*\* (internal), \*\*NodePort\*\* (on node's IP), and \*\*LoadBalancer\*\* (cloud provider's load balancer).



---

\## Task 6: Ingress \& AWS Load Balancer Controller



\*\*Goal:\*\* Manage external access to services using a Layer 7 load balancer.



\*\*Explanation:\*\* An \*\*Ingress\*\* manages Layer 7 (HTTP/S) routing. On EKS, the \*\*AWS Load Balancer Controller\*\* fulfills Ingress resources by provisioning an AWS Application Load Balancer (ALB).



---

\## Task 7: Persistent Storage with EBS



\*\*Goal:\*\* Provide persistent storage to a pod using AWS EBS.



\*\*Explanation:\*\* We installed the \*\*EBS CSI Driver\*\*, defined a \*\*StorageClass\*\*, requested storage with a \*\*PersistentVolumeClaim (PVC)\*\*, and mounted the PVC into a pod to store data that persists beyond the pod's lifecycle.



---

\## Task 8: StatefulSets



\*\*Goal:\*\* Deploy a stateful application (Redis) with stable identity and storage.



\*\*Explanation:\*\* A \*\*StatefulSet\*\* manages stateful applications, providing stable network identifiers, stable persistent storage, and ordered deployment. We used its `volumeClaimTemplates` to automatically provision a unique PVC for a Redis pod.



---

\## Task 9: Probes \& Resource Management



\*\*Goal:\*\* Improve application reliability with health checks and resource definitions.



\*\*Explanation:\*\* We added \*\*Liveness\*\* and \*\*Readiness Probes\*\* to our Nginx Deployment for health checking. We also set \*\*Resource Requests and Limits\*\* for predictable performance and cluster stability.



---

\## Task 10: RBAC \& Service Accounts



\*\*Goal:\*\* Restrict pod permissions using Role-Based Access Control (RBAC).



\*\*Explanation:\*\* We created a \*\*ServiceAccount\*\*, a \*\*Role\*\* with read-only permissions for pods, and a \*\*RoleBinding\*\* to connect them. This demonstrates the principle of least privilege.



---

\## Key Challenges \& Resolutions



1\.  \*\*Challenge:\*\* Corrupted CloudShell Environment \& Outdated `eksctl`.

&nbsp;   \* \*\*Resolution:\*\* Performed a full \*\*"Delete AWS CloudShell home directory"\*\* to get a fresh environment, which solved the issue.



2\.  \*\*Challenge:\*\* AWS Load Balancer Controller Installation \& Ingress Failures.

&nbsp;   \* \*\*Resolution:\*\* The final solution was to create a new cluster using a declarative `cluster.yml` file that correctly enabled `iam.withOIDC: true` from the start, which properly configured all IAM dependencies.



3\.  \*\*Challenge:\*\* Readiness Probe Failures (HTTP 403).

&nbsp;   \* \*\*Resolution:\*\* Discovered the pod was mounting an empty `ConfigMap`. The fix was to delete the empty ConfigMap, recreate it with the correct data, and restart the pod.

