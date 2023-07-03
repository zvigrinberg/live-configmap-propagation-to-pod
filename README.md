# live-configmap-propagation-to-pod

1.Create new project
```shell
oc new-project cm-propagation
```

2.Create Configmap
```shell
 oc create configmap my-test --from-literal=hello=world --from-literal=newKey=newValue --dry-run=client -o yaml > configmap.yaml
 oc apply -f configmap.yaml
```

3. Deploy a sample deployment that mount this configmap as volume and as environment variables also:
```shell
oc apply -f deployment.yaml
```

4. Wait for pod to come up, and then run the following:
```shell
echo "values of configmaps mounted as enviornment variables:"

oc exec $(oc get pod -l app=cm-propagation-test -o name) -- env | grep -E 'hello|newKey'

```

5. Show values of keys mounted to container as volume:
```shell
oc exec $(oc get pod -l app=cm-propagation-test -o name) -- sh -c "export KEYS=\$(ls /tmp/my-test) ; for i in \$KEYS ; do echo -n \$i=; cat /tmp/my-test/\$i ; echo ; done"
```


