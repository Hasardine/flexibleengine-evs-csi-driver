# Cloud Provider OpenStack Based plugin for Flexible Engine

Based on https://github.com/kubernetes/cloud-provider-openstack

# Deploy the example for evs-csi plugin

Configuration file specified in $CLOUD_CONFIG is passed to cinder CSI driver via kubernetes secret. If the secret cloud-config is already created in the cluster, you can remove the file, ./deploy/csi-secret-cinderplugin.yaml and directly proceed to the step of creating controller and node plugins.

## To create a secret:

### Encode your $CLOUD_CONFIG file content using base64.

`$ base64 -w 0 $CLOUD_CONFIG`

Update cloud.conf configuration in ./deploy/csi-secret-cinderplugin.yaml file by using the result of the above command.

### Create the secret.

`$ kubectl create -f ./deploy/csi-secret-cinderplugin.yaml`
This should create a secret name cloud-config in kube-system namespace.
Once the secret is created, Controller Plugin and Node Plugins can be deployed using respective deploy

`$ kubectl -f ./deploy/ apply`
This creates a set of cluster roles, cluster role bindings, and statefulsets etc to communicate with openstack(cinder). For detailed list of created objects, explore the yaml files in the directory. You should make sure following similar pods are ready before proceed:

`$ kubectl get pods -n kube-system`
NAME                                READY   STATUS    RESTARTS   AGE
csi-cinder-controllerplugin         6/6     Running   0        29h
csi-cinder-nodeplugin               3/3     Running   0        46h


### For example, Deploy the sample app

`$ kubectl create -f ./example/example.yaml`

You can easily edit parameters :
```YAML
parameters:
 type: SSD
 availability: eu-west-0a
 scsi: "true"
```
and EVS size :
```YAML
 resources:
    requests:
      storage: 10Gi
  storageClassName: csi-sc-cinderplugin
```

`$ kubectl get pvc`
NAME                     STATUS    VOLUME                                     CAPACITY  ACCESS MODES   STORAGECLASS   AGE
csi-pvc-cinderplugin     Bound     pvc-e36abf50-84f3-11e8-8538-42010a800002   10Gi       RWO            csi-sc-cinderplugin     9s
Check Pod is in Running state

`$ kubectl get pods`
NAME                 READY     STATUS    RESTARTS   AGE
nginx                1/1       Running   0          1m
Check current filesystem size on the running pod

`$ kubectl exec nginx -- df -h /var/lib/www/html`
Filesystem      Size  Used Avail Use% Mounted on
/dev/vdb        10.0G   24M  0.98G   1% /var/lib/www/html
Resize volume by modifying the field spec -> resources -> requests -> storage

`$ kubectl edit pvc csi-pvc-cinderplugin`
apiVersion: v1
kind: PersistentVolumeClaim
...
spec:
  resources:
    requests:
      storage: 10Gi
  ...
...
Verify filesystem resized on the running pod

`$ kubectl exec nginx -- df -h /var/lib/www/html`
Filesystem      Size  Used Avail Use% Mounted on
/dev/vdb        20.0G   27M  1.97G   1% /var/lib/www/html

---

## Cloud Provider OpenStack based :

This Repository hosts various plugins relevant to OpenStack and Kubernetes Integration
* [OpenStack Cloud Controller Manager](/docs/openstack-cloud-controller-manager/using-openstack-cloud-controller-manager.md/)
* [Octavia Ingress Controller](/docs/octavia-ingress-controller/using-octavia-ingress-controller.md/)
* [Cinder CSI Plugin](/docs/cinder-csi-plugin/using-cinder-csi-plugin.md/)
* [Keystone Webhook Authentication Authorization](/docs/keystone-auth/using-keystone-webhook-authenticator-and-authorizer.md/)
* [Client Keystone](/docs/keystone-auth/using-client-keystone-auth.md/)
* [Manila CSI Plugin](/docs/manila-csi-plugin/using-manila-csi-plugin.md/)
* [Barbican KMS Plugin](/docs/barbican-kms-plugin/using-barbican-kms-plugin.md/)
* [Magnum Auto Healer](/docs/magnum-auto-healer/using-magnum-auto-healer.md/)

**NOTE:**
* Cinder Standalone Provisioner, Manila Provisioner and Cinder FlexVolume Driver were removed since release v1.18.0.
* Version 1.17 was the last release of Manila Provisioner, which is unmaintained from now on. Due to dependency issues, we removed the code from master but it is still accessible in the [release-1.17](https://github.com/kubernetes/cloud-provider-openstack/tree/release-1.17) branch. Please consider migrating to Manila CSI Plugin.

### License

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

[http://www.apache.org/licenses/LICENSE-2.0](http://www.apache.org/licenses/LICENSE-2.0)

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
