# Create Projects

The construction of the lab starts by creating eight (8) projects defined in **01-project-list.yaml**:
* north-mesh and south-mesh; which will host the control plane of each mesh
* north-east and north-west: istio friendly applications developed by the north business unit
* south-east and south-west: istio friendly applications developed by the south business unit
* north-legacy and south-legacy: istio non-friendly applications developed by the north and south business units

By istio friendly and non-friendly, I mean the applications which will have the sidecar injected in the pod, and the 
ones which will not have sidecar.

```console
$ oc create -f manifests/01-project-list.yaml
```
