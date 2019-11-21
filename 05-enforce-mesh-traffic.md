# Enforce Service Mesh Traffic

Next we need to enforce the traffic between applications in that same mesh. The is achieved through istio

## Procedure

1. Force all sidecars to use mutual authentication between each other
   ```console
   oc -n north-mesh apply -f manifests/08-service-mesh-policy.yaml
   oc -n south-mesh apply -f manifests/08-service-mesh-policy.yaml
   ```

2. Force all sidecars to use authorization
   ```console
   oc -n north-mesh apply -f manifests/08-service-mesh-rbac-config.yaml
   oc -n south-mesh apply -f manifests/08-service-mesh-rbac-config.yaml
   ```

3. Limit istio sidecar access to any other endpoint in the mesh except the ones in the control plane
   ```console
   oc apply -f manifests/10-default-sidecar.yaml
   ```

4. Restrict the sidecar egress traffic only to the services in the registry
   ```console
   $ oc -n north-mesh get configmap istio -o yaml | sed 's/mode: ALLOW_ANY/mode: REGISTRY_ONLY/g' | oc -n north-mesh replace -f -
   $ oc -n south-mesh get configmap istio -o yaml | sed 's/mode: ALLOW_ANY/mode: REGISTRY_ONLY/g' | oc -n south-mesh replace -f -
   ```

5. And test...
   ```console
   for from in "south-east" "south-west" "south-legacy"; do for to in "south-east" "south-west" "south-legacy"; do oc exec $(oc get pod -l app=sleep -n ${from} -o jsonpath={.items..metadata.name}) -c sleep -n ${from} -- curl "http://httpbin.${to}:8000/ip" -s -o /dev/null -w "sleep.${from} to httpbin.${to}: %{http_code}\n"; done; done
   ```

### Remarks

## Conclusion