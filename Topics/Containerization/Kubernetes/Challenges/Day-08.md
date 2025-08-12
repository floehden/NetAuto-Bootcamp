# **Day 8: DaemonSets and Jobs/CronJobs**

## **Concepts:**
* DaemonSets: Ensuring a Pod runs on every (or selected) node, used for cluster-level agents (e.g., logging, monitoring, network proxies).
* Jobs: Running a task to completion, useful for batch processing, one-time scripts.
* CronJobs: Scheduling Jobs to run periodically based on a cron schedule.

## **Examples (Code):**
```yaml
# daemonset.yaml (simple node logger)
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-logger
spec:
  selector:
    matchLabels:
      app: node-logger
  template:
    metadata:
      labels:
        app: node-logger
    spec:
      containers:
      - name: logger
        image: busybox
        command: ["sh", "-c", "while true; do echo $(date): Running on node $(NODE_NAME) with IP $(POD_IP); sleep 10; done"]
        env:
        - name: NODE_NAME
          valueFrom:
            fieldRef:
              fieldPath: spec.nodeName
        - name: POD_IP
          valueFrom:
            fieldRef:
              fieldPath: status.podIP
```

```bash
kubectl apply -f daemonset.yaml
kubectl get daemonsets
kubectl get pods -l app=node-logger -o wide
kubectl logs <any-node-logger-pod>
```

```yaml
# job.yaml (one-time task)
apiVersion: batch/v1
kind: Job
metadata:
  name: pi-calculator
spec:
  template:
    spec:
      containers:
      - name: pi
        image: perl
        command: ["perl", "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: OnFailure
  backoffLimit: 4
```

```bash
kubectl apply -f job.yaml
kubectl get jobs
kubectl get pods -l job-name=pi-calculator
kubectl logs <pi-calculator-pod>
```

```yaml
# cronjob.yaml (scheduled task)
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello-cronjob
spec:
  schedule: "*/1 * * * *" # Every minute
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox
            command: ["sh", "-c", "echo 'Hello from the cron job!' && date"]
          restartPolicy: OnFailure
```

```bash
kubectl apply -f cronjob.yaml
kubectl get cronjobs
kubectl get jobs -l cronjob=hello-cronjob
kubectl get pods -l cronjob=hello-cronjob # Check logs of latest pod
```

## **Challenge:** 
Create a DaemonSet that simulates a node-level agent (e.g., just logs its hostname and node IP every 5 seconds). Create a CronJob that runs every 2 minutes and performs a simulated cleanup task (e.g., creating and then deleting a temporary file in `/tmp` within the container).

## Further Readings
* https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/
* https://kubernetes.io/docs/concepts/workloads/controllers/job/
* https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/