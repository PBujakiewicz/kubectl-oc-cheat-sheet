# kubectl-oc-cheat-sheet
oc is an alternative to kubectl in okd/openshift


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

## An Alternative to the 'oc get all' Command.
```bash
oc get $(oc api-resources --namespaced=true --verbs=list -o name | awk '{printf "%s%s",sep,$0;sep=","}') --ignore-not-found -n ${NAMESPACE} -o=custom-columns=KIND:.kind,NAME:.metadata.name --sort-by='kind'
```
