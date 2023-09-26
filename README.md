# kubectl-oc-cheat-sheet
oc is an alternative to kubectl in okd/openshift


<br /><br />

## Delete all pods from namespace.
```bash
oc delete pods $(oc get pods -o=jsonpath='{.items[*].metadata.name}' -n NS) -n NS --force=true
```

<br /><br />

## Delete all pods with status Completed, Evicted, Terminating from all namespaces.
```bash
for ns in $(oc get ns | grep Active | awk '{print $1}'); do for pod in $(oc get pods -n ${ns} | grep -E '(Completed|Evicted|Terminating)' | awk '{print $1}'); do oc delete pod --force=true  ${pod} -n ${ns}; done; done
```

<br /><br />

## Drain specify deployment.
```bash
oc adm manage-node node-1 --schedulable=false
oc adm manage-node node-1 --evacuate --pod-selector=deployment=app-xx
oc adm manage-node node-1 --schedulable=true
```

<br /><br />

## Drain specify node.
```bash
oc adm drain node-1
```

<br /><br />

## Manually sending logs to stdout in a container.
```bash
echo "test log1" >> /proc/1/fd/1
```

<br /><br />

## Binding the overlay to the container name.
```bash
for container in $(docker ps --all --quiet --format '{{ .Names }}'); do echo "$(docker inspect $container --format '{{.GraphDriver.Data.MergedDir }}' | grep -Po '^.+?(?=/merged)'  ) = $container"; done
```

<br /><br />

## An Alternative to the 'oc get all' Command.
```bash
oc get $(oc api-resources --namespaced=true --verbs=list -o name | awk '{printf "%s%s",sep,$0;sep=","}') --ignore-not-found -n ${NAMESPACE} -o=custom-columns=KIND:.kind,NAME:.metadata.name --sort-by='kind'
```

<br /><br />

## Set replication to 0 in all deployments.
```bash
kubectl get deployments --all-namespaces -o json | kubectl patch -p '{"spec":{"replicas":0}}' -f -
```

<br /><br />

## Set replication to 0 in all StatefulSet.
```bash
kubectl get statefulsets --all-namespaces -o json | kubectl patch -p '{"spec":{"replicas":0}}' -f -
```

<br /><br />

## All used images in containers.
```bash
kubectl get pods --all-namespaces -o=jsonpath='{range .items[*]}{.metadata.namespace}{"/"}{.metadata.name}{":\t"}{range .spec.containers[*]}{.image}{"\n"}{end}{range .spec.initContainers[*]}{.image}{"\n"}{end}{"\n"}{end}'
```

<br /><br />

## Pod spamming logs to stdout.
```bash
apiVersion: batch/v1
kind: Job
metadata:
  name: log-spamer
spec:
  parallelism: 100
  template:
    spec:
      containers:
      - name: bash
        image: bash:alpine3.18
        env:
        - name: ID
          value: asd78jhT2h
        command:
        - bash
        - -c
        - |
          for i in {1..100}; do echo "log-spamer, id: $ID - log number $i" >> /proc/1/fd/1; done
      restartPolicy: Never
```

<br /><br />

## Waiting pod.
```bash
apiVersion: v1
kind: Pod
metadata:
  name: waiting-pod
spec:
  containers:
  - name: bash
    image: bash:alpine3.18
    command:
      - "cat"
    tty: true
```

<br /><br />
