# Kubernetes

* [Hardening an ASP.NET container running on Kubernetes](https://techcommunity.microsoft.com/t5/azure-developer-community-blog/hardening-an-asp-net-container-running-on-kubernetes/ba-p/2542224)

There are two basic ways to deploy to Kubernetes: **imperatively**, with the many `kubectl` commands, or **declaratively**, by writing manifests and using `kubectl apply`.

## Evolution

Containers are similar to VMs, but they have relaxed isolation properties to share the Operating System (OS) among the applications. Therefore, containers are considered lightweight. Similar to a VM, a container has its own filesystem, share of CPU, memory, process space, and more. As they are decoupled from the underlying infrastructure, they are portable across clouds and OS distributions.

Kubernetes provides you with:

* **Service discovery and load balancing** Kubernetes can expose a container using the DNS name or using their own IP address. If traffic to a container is high, Kubernetes is able to load balance and distribute the network traffic so that the deployment is stable.
* **Storage orchestration** Kubernetes allows you to automatically mount a storage system of your choice, such as local storages, public cloud providers, and more.
* **Automated rollouts and rollbacks** You can describe the desired state for your deployed containers using Kubernetes, and it can change the actual state to the desired state at a controlled rate. For example, you can automate Kubernetes to create new containers for your deployment, remove existing containers and adopt all their resources to the new container.
* **Automatic bin packing** You provide Kubernetes with a cluster of nodes that it can use to run containerized tasks. You tell Kubernetes how much CPU and memory (RAM) each container needs. Kubernetes can fit containers onto your nodes to make the best use of your resources.
* **Self-healing** Kubernetes restarts containers that fail, replaces containers, kills containers that don't respond to your user-defined health check, and doesn't advertise them to clients until they are ready to serve.
* **Secret and configuration management** Kubernetes lets you store and manage sensitive information, such as passwords, OAuth tokens, and SSH keys. You can deploy and update secrets and application configuration without rebuilding your container images, and without exposing secrets in your stack configuration.

## CronJob

CronJobs are useful for creating periodic and recurring tasks.

### Cron schedule syntax

```
# ┌───────────── minute (0 - 59)
# │ ┌───────────── hour (0 - 23)
# │ │ ┌───────────── day of the month (1 - 31)
# │ │ │ ┌───────────── month (1 - 12)
# │ │ │ │ ┌───────────── day of the week (0 - 6) (Sunday to Saturday;
# │ │ │ │ │                                   7 is also Sunday on some systems)
# │ │ │ │ │
# │ │ │ │ │
# * * * * *
```

| Entry 										| Description																									| Equivalent to |
| ------------- 						| ------------- 																							|-------------  |
| @yearly (or @annually)		| Run once a year at midnight of 1 January										| 0 0 1 1 * 		|
| @monthly 									| Run once a month at midnight of the first day of the month	| 0 0 1 * * 		|
| @weekly 									| Run once a week at midnight on Sunday morning								| 0 0 * * 0 		|
| @daily (or @midnight)			| Run once a day at midnight																	| 0 0 * * * 		|
| @hourly 									| Run once an hour at the beginning of the hour								| 0 * * * * 		|

## Troubleshooting

Command executed on all namespaced resources with label selector

```bash
kubectl -n <namespace> delete "$(kubectl api-resources --namespaced=true --verbs=delete -o name | tr "\n" "," | sed -e 's/,$//')" -l app=<name>
```

Spinning up a temporary pod for network troubleshooting

```bash
kubectl run tmp --restart=Never --rm --image=busybox -i -- wget -O- http://project-plt-6cc-svc.pluto:3333
```

```bash
cat <<EOF | kubectl apply -f -
apiVersion: scheduling.k8s.io/v1
description: This priority class should be used for kubernetes service pods only.
kind: PriorityClass
metadata:
  labels:
    addonmanager.kubernetes.io/mode: Reconcile
  name: high-priority
value: 1000000
EOF
```

## Training

### Images

* [busybox](https://hub.docker.com/_/busybox)
* [nginx](https://hub.docker.com/_/nginx)

```
busybox : {1.33.1}
nginx   : {1.21.1}
```

### Pods

```bash
kubectl -n <namespace> run nginx --image=nginx:1.21.1 --dry-run=client -o yaml
kubectl -n <namespace> run nginx --image=nginx:1.21.1 --port=80 --dry-run=client -o yaml
kubectl -n <namespace> run nginx --image=nginx:1.21.1 --labels=kubernetes.io/app=demo --dry-run=client -o yaml
kubectl -n <namespace> run nginx --image=nginx:1.21.1 --requests=memory=20Mi --limits=memory=50Mi --dry-run=client -o yaml
```

### Readiness Probe

```bash
kubectl -n <namespace> run demo --image=busybox:1.33.1 --dry-run=client -o yaml --command -- sh -c "touch /tmp/ready && sleep 1d" > demo.yaml
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: demo
  name: demo
  namespace: training
spec:
  containers:
  - name: busybox-container
    image: busybox:1.33.1
    command:
    - sh
    - -c
    - sleep 15 && touch /my-first-temp-folder/live && touch /my-first-temp-folder/ready && sleep 1d
    resources: {}
    volumeMounts:
    - mountPath: /my-first-temp-folder
      name: temp-volume      
  - name: nginx-container
    image: nginx:1.21.1
    resources: {}
    livenessProbe:
      exec:
        command:
        - sh
        - -c
        - cat /my-second-temp-folder/live
      initialDelaySeconds: 5
      periodSeconds: 10
    readinessProbe:
      exec:
        command:
        - sh
        - -c
        - cat /my-second-temp-folder/ready
      initialDelaySeconds: 5
      periodSeconds: 10
    volumeMounts:
    - mountPath: /my-second-temp-folder
      name: temp-volume      
  volumes:
  - name: temp-volume
    emptyDir: {}
```

### Jobs

```bash
kubectl -n <namespace> create job new-job --image=busybox:1.33.1 --dry-run=client -o yaml -- sh -c "sleep 2 && echo done"
```

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  creationTimestamp: null
  name: new-job
  namespace: <namespace>  # add
spec:
  completions: 3          # add
  parallelism: 2          # add
  template:
    metadata:
      creationTimestamp: null
      labels:
        id: awesome-job   # add
    spec:
      containers:
      - command:
        - sh
        - -c
        - sleep 2 && echo done
        image: busybox:1.33.1
        name: new-job
        resources: {}
      restartPolicy: Never
status: {}
```

### Deployments

```bash
kubectl -n <namespace> create deploy nginx-deployment --image=nginx:1.21.1 --dry-run=client -o yaml
```

### Services

```bash
kubectl -n <namespace> expose po nginx --name=nginx-service --port=8080 --target-port=80 --dry-run=client -o yaml
kubectl -n <namespace> create service clusterip nginx-service --tcp=8080:80 --dry-run=client -o yaml
```

### Testing

```bash
kubectl run tmp --restart=Never --rm --image=busybox -i -- wget -O- http://nginx-service.<namespace>:8080
```
