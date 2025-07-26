# Kubernetes 28-Day Tutorial: From Basics to Multi-Cluster GitOps Mastery
This 28-day tutorial will guide you through the exciting world of Kubernetes, from its fundamental concepts to advanced topics like GitOps with Helm, ArgoCD, FluxCD, and multi-cluster management. Each day includes explanations, practical examples, and challenges to solidify your understanding. **All hands-on examples will primarily use Kind (Kubernetes in Docker) for local cluster creation and management.**

## **Target Audience:** This tutorial is for developers, operations engineers, and anyone interested in understanding and working with Kubernetes. Basic familiarity with Linux command line and Docker concepts is beneficial but not strictly required.

## **Prerequisites:**

  * A computer with internet access.
  * A text editor (VS Code, Sublime Text, etc.).
  * A stable internet connection.
  * **Docker Desktop installed and running:** Kind uses Docker to run Kubernetes nodes.
  * **Kind CLI installed:** Follow official instructions (e.g., `go install sigs.k8s.io/kind@v0.22.0`).
  * **`kubectl` installed:** Follow official instructions (e.g., `brew install kubectl` on macOS).
  * **`helm` CLI installed:** (Will be covered in detail on Day 15).
  * **`git` installed:** For GitOps examples.


## Overview

### **Week 1: Kubernetes & Kind Fundamentals** (Day 1 - 7)

| Day | Description |
| ------ | ----- |
| Day 01 | [Introduction to Containerization & Kind Cluster Setup](/Topics/Containerization/Kubernetes/Challenges/Day-01.md) |
| Day 02 | [`kubectl` Basics and Pods](/Topics/Containerization/Kubernetes/Challenges/Day-02.md) |
| Day 03 | [Deployments and ReplicaSets](/Topics/Containerization/Kubernetes/Challenges/Day-03.md) |
| Day 04 | [Kubernetes Networking Fundamentals & Services](/Topics/Containerization/Kubernetes/Challenges/Day-04.md) |
| Day 05 | [Namespaces and Resource Management](/Topics/Containerization/Kubernetes/Challenges/Day-05.md) |
| Day 06 | [ConfigMaps and Secrets](/Topics/Containerization/Kubernetes/Challenges/Day-06.md) |
| Day 07 | [Persistent Storage (Kind-Specific Considerations)](/Topics/Containerization/Kubernetes/Challenges/Day-07.md) |

### **Week 2: Advanced Kubernetes Concepts** (Day 8 - 14)

| Day | Description |
| ------ | ----- |
| Day 08 | [DaemonSets and Jobs/CronJobs](/Topics/Containerization/Kubernetes/Challenges/Day-08.md) |
| Day 09 | [Ingress: External Access with Routing (Kind-specific)](/Topics/Containerization/Kubernetes/Challenges/Day-09.md) |
| Day 10 | [Liveness and Readiness Probes](/Topics/Containerization/Kubernetes/Challenges/Day-10.md) |
| Day 11 | [StatefulSets for Stateful Applications](/Topics/Containerization/Kubernetes/Challenges/Day-11.md) |
| Day 12 | [Role-Based Access Control (RBAC)](/Topics/Containerization/Kubernetes/Challenges/Day-12.md) |
| Day 13 | [Horizontal Pod Autoscaler (HPA)](/Topics/Containerization/Kubernetes/Challenges/Day-13.md) |
| Day 14 | [Resource Management Review & Troubleshooting Basics](/Topics/Containerization/Kubernetes/Challenges/Day-14.md) |

### **Week 3: Helm & GitOps** (Day 15 - 21)

| Day | Description |
| ------ | ----- |
| Day 15 | [Helm: The Kubernetes Package Manager](/Topics/Containerization/Kubernetes/Challenges/Day-15.md) |
| Day 16 | [Creating Custom Helm Charts](/Topics/Containerization/Kubernetes/Challenges/Day-16.md) |
| Day 17 | [Helm Advanced Features and Best Practices](/Topics/Containerization/Kubernetes/Challenges/Day-17.md) |
| Day 18 | [Introduction to GitOps](/Topics/Containerization/Kubernetes/Challenges/Day-18.md) |
| Day 19 | [ArgoCD: Declarative GitOps CD](/Topics/Containerization/Kubernetes/Challenges/Day-19.md) |
| Day 20 | [ArgoCD with Helm Charts](/Topics/Containerization/Kubernetes/Challenges/Day-20.md) |
| Day 21 | [ArgoCD Advanced Features & Multi-Cluster Preparation](/Topics/Containerization/Kubernetes/Challenges/Day-21.md) |

### **Week 4: Multi-Cluster & Advanced Topics** (Day 22 - 28)

| Day | Description |
| ------ | ----- |
| Day 22 | [FluxCD: A Kubernetes-Native GitOps Tool](/Topics/Containerization/Kubernetes/Challenges/Day-22.md) |
| Day 23 | [FluxCD with Helm and Kustomize](/Topics/Containerization/Kubernetes/Challenges/Day-23.md) |
| Day 24 | [Introduction to Multi-Cluster Kubernetes with Kind](/Topics/Containerization/Kubernetes/Challenges/Day-24.md) |
| Day 25 | [GitOps for Multi-Cluster with ArgoCD ApplicationSets](/Topics/Containerization/Kubernetes/Challenges/Day-25.md) |
| Day 26 | [Advanced Kubernetes Networking: CNI Deep Dive & Multi-Cluster Considerations](/Topics/Containerization/Kubernetes/Challenges/Day-26.md) |
| Day 27 | [Custom Resource Definitions (CRDs) & Operators](/Topics/Containerization/Kubernetes/Challenges/Day-27.md) |
| Day 28 | [Kubernetes Security, Monitoring & Next Steps](/Topics/Containerization/Kubernetes/Challenges/Day-28.md) |


## ** Notes for Your Journey:**

* **Clean Up:** After each day or week, remember to clean up your Kind cluster or specific resources to avoid resource conflicts or excessive Docker resource consumption.
    * `kubectl delete -f <your-file.yaml>`
    * `helm uninstall <release-name>`
    * `kind delete cluster --name <cluster-name>`
* **Persistent Learning:** Kubernetes is vast and constantly evolving. This tutorial provides a strong foundation, but continuous learning through documentation, blogs, and community engagement is vital.
* **Troubleshooting is a Skill:** The more you debug, the better you become. Don't be discouraged by errors; they are learning opportunities.
