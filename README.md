# kubectl-oc-cheat-sheet


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
