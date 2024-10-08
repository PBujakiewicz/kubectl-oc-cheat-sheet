# kubectl-oc-cheat-sheet
oc is an alternative to kubectl in okd/openshift


<br /><br />

## Easily check your clusters for use of deprecated APIs. ([https://github.com/johanhaleby/kubetail](https://github.com/doitintl/kube-no-trouble))
```bash
kubent
```

<br /><br />

## Tail: "kubectl logs -f " but for multiple pods. (https://github.com/johanhaleby/kubetail)
```bash
kubetail app2
```

<br /><br />

## This is a simple CLI that provides an overview of the resource requests, limits, and utilization in a Kubernetes cluster. (https://github.com/robscott/kube-capacity)
```bash
kube-capacity
```

<br /><br />

## Delete all pods from namespace.
```bash
oc delete pods $(oc get pods -o=jsonpath='{.items[*].metadata.name}' -n NS) -n NS
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

## Watch pods with errors (+ use awk in watch).
```bash
watch -n 5 'oc get pods -A --no-headers | grep -v -E "(Running|Completed)" | awk '\''{print $1, $2, $4, $6}'\'''
```
<br /><br />

## Watch and count pods status.
```bash
watch -n 5 'oc get pods -A --no-headers | awk '\''{print $4}'\'' | sort | uniq -c'
```

<br /><br />

## Watch events with "Timeout waiting for systemd" maximum up to 1 hour.
```bash
watch -n 5 'kubectl get events --all-namespaces --sort-by=.metadata.creationTimestamp -o custom-columns=LAST-SEEN:.lastTimestamp,NAMESPACE:.metadata.namespace,POD:.involvedObject.name,NODE:.source.host,MESSAGE:.message | awk -v d1="$(TZ=UTC date +"%Y-%m-%dT%H:%M:%SZ" -d "1 hour ago")" "BEGIN { OFS = \"\t\"; print \"LAST-SEEN\", \"NAMESPACE\", \"POD\", \"NODE\", \"MESSAGE\" } \$1 > d1 { print \$0 }" | grep "Timeout waiting for systemd" | awk '\''{print $4}'\'' | sort | uniq -c'
```

<br /><br />

## Pod spamming logs to stdout.
```bash
apiVersion: batch/v1
kind: Job
metadata:
  name: log-spamer
spec:
  parallelism: 10
  template:
    spec:
      containers:
      - name: bash
        image: bash:alpine3.18
        env:
        - name: ID
          value: asd7wxxx1
        command:
        - bash
        - -c
        - |
          for i in {1..100}; do echo "$(date '+%Y-%m-%d %T') $(hostname) id: $ID - log number $i" && sleep 0.01 >> /proc/1/fd/1; done
      restartPolicy: Never
```

<br /><br />

## Waiting pod.
```bash
apiVersion: v1
kind: Pod
metadata:
  name: waiting-pod
  annotations:
    sidecar.istio.io/inject: "false"
spec:
  containers:
  - name: python
    image: python:3.9.19-alpine3.20
    command: ["/bin/sh","-c"]
    args: ["apk add busybox-extras curl git bash && pip install pyrabbit pika pyyaml && sleep infinity"]
  tty: true
```

<br /><br />

## Helm.
```bash
# Build and test helm template
helm template name . | oc apply --dry-run=server -f -
```

<br /><br />

## Show all prometheus alerts with severity.
```bash
oc get prometheusrules -A  --no-headers| awk '{print $3, $1, $2}' | sort -r | awk '{print "oc get prometheusrules " $3 " -n " $2 " -o yaml | grep -E \"alert:|severity:\""}' | bash | sed -r 's/    - alert: //g'
```

<br /><br />

## Checking all expired ssl/tls certificate in secrets.
```bash
oc get secrets -A --no-headers | grep NS | awk -v current_date=$(date +"%s") '{
    print "\nNS: "$1"\tsecret: "$2":";

    command="oc get secret "$2" -n "$1" -o jsonpath=\"{.data['\''tls\\.crt'\'']}\" | base64 -d | openssl x509 -text -noout 2>/dev/null | grep \"Not After\" ";
    command | getline result;
    close(command);
    split(result, arr, ": ");
    not_after=arr[2];

    command = "date -u -d \""not_after"\" +%s";
    command | getline not_after2;
    close(command);

    if (not_after2 <= current_date)
        print result
        not_after2=""
        result=""
}' | grep -B 2 'Not After'
```

<br /><br />
