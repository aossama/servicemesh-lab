# Create OpenShift Cluster

1. Download and extract openshift-install and openshift-client
   ```console
   $ curl -o openshift-client-linux-4.2.4.tar.gz https://mirror.openshift.com/pub/openshift-v4/clients/ocp/latest/openshift-client-linux-4.2.4.tar.gz
   $ tar xzf openshift-client-linux-4.2.4.tar.gz
   $ curl -o openshift-install-linux-4.2.4.tar.gz https://mirror.openshift.com/pub/openshift-v4/clients/ocp/latest/openshift-install-linux-4.2.4.tar.gz
   $ tar xzf openshift-install-linux-4.2.4.tar.gz
   ```

2. Prepare the directory for cluster configuration
   ```console
   $ mkdir ocp.luji.io
   $ cd ocp.luji.io
   $ cat << EOF >> install-config.yaml
   apiVersion: v1
   metadata:
     name: ocp-dev
   baseDomain: luji.io
   platform:
     aws:
       region: eu-west-1
       userTags:
         clusterName: OCP-Dev
         costCenter: 0343
   compute:
   - hyperthreading: Enabled
     name: worker
     platform:
       aws:
         rootVolume:
           size: 70
           type: gp2
         type: m5.xlarge
     replicas: 4
   controlPlane:
     hyperthreading: Enabled
     name: master
     platform:
       aws:
         rootVolume:
           size: 70
           type: gp2
         type: m5.xlarge
     replicas: 3
   networking:
     clusterNetwork:
     - cidr: 10.128.0.0/14
       hostPrefix: 23
     machineCIDR: 10.0.0.0/16
     networkType: OpenShiftSDN
     serviceNetwork:
     - 172.30.0.0/16
   pullSecret: '{ "Sorry": "Cant reveal it.  =D" }'
   sshKey: |
     ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAA
   EOF
   ```
   > Of course the pullSecret and sshKey needs to be the real ones.

3. Deploy the cluster
   ```console
   $ openshift-install create cluster --log-level=info
   ```

4. Access and verify the cluster
   ```console
   $ export KUBECONFIG=./workspace/luji/ocp-dev.luji.io/auth/kubeconfig
   $ oc whoami
   $ oc get co
   ```
