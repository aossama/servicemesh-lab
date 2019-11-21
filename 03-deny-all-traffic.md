# Deny all traffic

We use a NetworkPolicy which will drop all pod traffic in the same namespace and other namespaces.

## Procedure

1. Deploy the istio non-friendly applications to the legacy projects
   ```console
   for project in "north-legacy" "south-legacy"; do
     oc -n ${project} apply -f apps/02-sleep-without-sidecar.yaml
     oc -n ${project} apply -f apps/06-httpbin-without-sidecar.yaml
   done
   ```

2. And test...
   ```console
   for from in "south-legacy" "north-legacy"; do for to in "south-legacy" "north-legacy"; do oc exec $(oc get pod -l app=sleep -n ${from} -o jsonpath={.items..metadata.name}) -c sleep -n ${from} -- curl "http://httpbin.${to}:8000/ip" -s -o /dev/null -w "sleep.${from} to httpbin.${to}: %{http_code}\n"; done; done
   ```

3. Create the networkpolicy on all namespaces
   ```console
   for pole in "north" "south"; do for edge in "east" "west" "legacy"; do oc -n "${pole}-${edge}" apply -f manifests/03-networkpolicy.yaml; done; done
   ```

4. And test...
   ```console
   for from in "south-legacy" "north-legacy"; do for to in "south-legacy" "north-legacy"; do oc exec $(oc get pod -l app=sleep -n ${from} -o jsonpath={.items..metadata.name}) -c sleep -n ${from} -- curl "http://httpbin.${to}:8000/ip" --max-time 10 -s -o /dev/null -w "sleep.${from} to httpbin.${to}: %{http_code}\n"; done; done
   ```
   > Using max-time here because I don't wait forever  =D

### Remarks

This traffic denial will be overridden by istio default behaviour detailed later.

## Conclusion

At this point we achieved a deny 
