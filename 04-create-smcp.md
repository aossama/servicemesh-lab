# Create Service Mesh Control Planes

Soft multi-tenancy service mesh is when there's more trust established between the tenants. This is typically achieved 
in large organizations where different business units would share the same cluster.

## Procedure

1. Subscribe to **Red Hat Service Mesh** operator from the Operatorhub link.
2. Create a control plane in each business unit's service mesh namespace
   ```console
   $ oc -n north-mesh apply -f manifests/05-service-mesh-control-plane.yaml
   $ oc -n south-mesh apply -f manifests/05-service-mesh-control-plane.yaml
   ```

3. Wait for the control plane provisioning to finish
4. Create the Service Mesh member rolls
   ```console
   oc apply -f ./manifests/07-service-mesh-member-roll.yaml
   ```

5. Deploy the istio friendly applications
   ```console
   for pole in "north" "south"; do for egde in "east" "west"; do; oc -n ${pole}-${edge} apply -f apps/01-sleep-with-sidecar.yaml; oc -n ${pole}-${edge} apply -f apps/05-httpbin-with-sidecar.yaml; done; done
   ```

6. And test north communication...
   ```console
   for from in "north-east" "north-west" "north-legacy"; do for to in "north-east" "north-west" "north-legacy"; do oc exec $(oc get pod -l app=sleep -n ${from} -o jsonpath={.items..metadata.name}) -c sleep -n ${from} -- curl "http://httpbin.${to}:8000/ip" -s -o /dev/null -w "sleep.${from} to httpbin.${to}: %{http_code}\n"; done; done
   ```
   
   And the south communication...
   ```console
   for from in "south-east" "south-west" "south-legacy"; do for to in "south-east" "south-west" "south-legacy"; do oc exec $(oc get pod -l app=sleep -n ${from} -o jsonpath={.items..metadata.name}) -c sleep -n ${from} -- curl "http://httpbin.${to}:8000/ip" -s -o /dev/null -w "sleep.${from} to httpbin.${to}: %{http_code}\n"; done; done
   ```
   
   And the north-south communication...
   ```console
   for from in "north-east" "north-west" "north-legacy"; do for to in "south-east" "south-west" "south-legacy"; do oc exec $(oc get pod -l app=sleep -n ${from} -o jsonpath={.items..metadata.name}) -c sleep -n ${from} -- curl "http://httpbin.${to}:8000/ip" -s -o /dev/null --max-time 10 -w "sleep.${from} to httpbin.${to}: %{http_code}\n"; done; done
   ```
   
   And the south-north communication...
   ```console
   for from in "south-east" "south-west" "south-legacy"; do for to in "north-east" "north-west" "north-legacy"; do oc exec $(oc get pod -l app=sleep -n ${from} -o jsonpath={.items..metadata.name}) -c sleep -n ${from} -- curl "http://httpbin.${to}:8000/ip" -s -o /dev/null --max-time 10 -w "sleep.${from} to httpbin.${to}: %{http_code}\n"; done; done
   ```

### Remarks

When a project is enrolled as a member in a service mesh, the below automatically applies:

1. Two new labels are added to the namespace
2. A NetworkPolicy is created which explicitly allow all traffic among the service mesh members
3. New Rolebindings are added which grant the service mesh access to the project

## Conclusion

A hard isolation between the services on each service mesh data plane ensures that applications of each business unit is allowed only to communicate with each other.