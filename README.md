# kustomize-bug

## A short description of the directories

| Directory | Description |
| --- | --- |
|deployment-base| A simple Deployment resource|
|deployment-patch| Patch for deployment-base's Deployment resource|
|rollout-base| A Rollout resource with its openapi definition|
|rollout-patch| Patch for rollout-base's Rollout resource|
|merge_successful|resources: <br />- deployment_patch<br />- rollout_patch|
|merge_failure| resources: <br />- rollout_patch <br />- deployment_patch <br />(same as merge_successful but in reversed sequence)|
|GENERATED| The generated yaml for all other directoried to make it easier to compare them |

## Issue description:
`kustomize build deployment-patch` - works as expected it does patch the Deployment resource<br />
`kustomize build rollout-patch` - works as expected it does patch the Rollout resource

```
$ cat merge_successful/kustomization.yaml
resources:
- ../deployment_patch
- ../rollout_patch
```

`kustomize build merge_successful` - works as expected, both Deployment and Rollout resources are patched

### Issue 1

```
$ cat merge_failure/kustomization.yaml
resources:
- ../rollout_patch
- ../deployment_patch
```

`kustomize build merge_failure` - Rollout resource is patched properly but at Deployment resource the containers definition does not contain properties that are defined only in base.

As kustomization.yamls of merge_successful and merge_failure are the same only the sequence of of included directories are different I would expect that the generated manifests are the same but they are not:

```
$ diff GENERATED/merge_successful.yaml GENERATED/merge_failure.yaml
26d25
<         imagePullPolicy: Always
28,36d26
<         ports:
<         - containerPort: 80
<         resources:
<           limits:
<             cpu: 50m
<             memory: 32Mi
<           requests:
<             cpu: 50m
<             memory: 32Mi
```

### Issue 2
`kustomize build deployment_patch_with_rollout_openapi` - openapi definiton on Rollout resource is included to kustomization.yaml, but there is only a Deployment resource. With openapi definition of Rollout resource the generated manifest is different, the containers definition does not contain properties that are defined only in base

```
$ diff GENERATED/deployment_patch.yaml GENERATED/deployment_patch_with_rollout_openapi.yaml
26d25
<         imagePullPolicy: Always
28,36d26
<         ports:
<         - containerPort: 80
<         resources:
<           limits:
<             cpu: 50m
<             memory: 32Mi
<           requests:
<             cpu: 50m
<             memory: 32Mi
```
