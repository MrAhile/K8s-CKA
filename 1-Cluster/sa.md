
## Exercise

1. Create a Namespace named *test*

2. Create a ServiceAccount named *sa* in the namespace *test*

3. Create a ClusterRole *pod-reader* with the rights to get and list Pods

4. Associate the *pod-reader* ClusterRole to the *sa* ServiceAccount

5. Verify the *sa* ServiceAccount can get and list Pods

6. Delete the resources created above

<details>
  <summary markdown="span">Solution</summary>

1. Create a Namespace named *test*

```
k create ns test
```

2. Create a ServiceAccount named *sa* in the namespace *test*

```
k create serviceaccount sa -n test
```

3. Create a ClusterRole *pod-reader* with the rights to get and list Pods

```
k create clusterrole pod-reader --verb=get,list --resource=pods
```

4. Associate the *pod-reader* ClusterRole to the *sa* ServiceAccount

```
k create rolebinding test-pod-reader --clusterrole=pod-reader --serviceaccount=test:sa -n test
```

5. Verify the *sa* ServiceAccount can get and list Pods

```
k auth can-i list pods --as system:serviceaccount:test:sa -n test
yes
```

6. Delete the resources created above

```
k -n test delete serviceaccount sa
k -n test delete rolebinding/test-pod-reader
k delete clusterrole/pod-reader
k delete ns test
```

</details>

