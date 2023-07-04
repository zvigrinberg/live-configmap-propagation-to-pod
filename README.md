# live-configmap-propagation-to-pod

1.Create new project
```shell
oc new-project cm-propagation
```

2.Create Configmap
```shell
 oc create configmap my-test --from-literal=hello=world --from-literal=key=value --dry-run=client -o yaml > configmap.yaml
 oc apply -f configmap.yaml
```

3. Deploy a sample deployment that mount this configmap as volume and as environment variables also:
```shell
oc apply -f deployment.yaml
```
4. See that pod is up and running:
```shell
oc get pods 
```
Output:
```shell
NAME                                   READY   STATUS    RESTARTS   AGE
cm-propagation-test-784767d9c4-br4jk   1/1     Running   0          29s
```
5. Wait for pod to come up, and then run the following to see values of environment variables:
```shell
echo "values of configmaps mounted as enviornment variables:"

oc exec $(oc get pod -l app=cm-propagation-test -o name) -- env | grep -E 'hello|key'
```
Output:
```shell
key=value
hello=world
```


6. Show values of keys mounted to container as volume:
```shell
oc exec $(oc get pod -l app=cm-propagation-test -o name) -- sh -c "export KEYS=\$(ls /tmp/my-test) ; for i in \$KEYS ; do echo -n \$i=; cat /tmp/my-test/\$i ; echo ; done"
```
Output:
```shell
hello=world
key=value
```

7. Now change one key' value of the configmap , and add new 'key: value' pair in configmap.yaml, and then apply the change to cluster:
```yaml
apiVersion: v1
data:
  hello: new-world
  key: value
  newKey: newValue
kind: ConfigMap
metadata:
  name: my-test
```
```shell
oc apply -f configmap.yaml
```

8. Now repeat step 5 and verify that environment variables didn't change
Output:
```shell
hello=world
key=value
```

9. Now Wait a 10-15 seconds, and  Repeat step 6 couple of times until ConfigMap change propagated by kubelet to pod' container, and verify that configmap mounted as volume on pod' container changed accordingly.

Output:
```shell
hello=new-world
key=value
newKey=newValue
```

10. And the change reflected and shown in the former section was done without restarting the pod off course:
```shell
oc get pods
```

Output:
```shell
NAME                                   READY   STATUS    RESTARTS   AGE
cm-propagation-test-784767d9c4-br4jk   1/1     Running   0          11m
```