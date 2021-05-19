# Install OpenShift 4.6.1 on VMware 6.7U3

OpenShift on VMware

Demo videos:
1. Install OpenShift 4.6.1 via UPI on VMware https://youtu.be/6TvyHBdHhes
2. OCP VMware Internal Registry Setup https://youtu.be/_60RLgkHVMw
3. Chrony NTP on OpenShift https://youtu.be/_60RLgkHVMw

Access the downloads from https://cloud.redhat.com
See the documentation at https://docs.openshift.com/container-platform/4.6/installing/installing_vsphere/installing-vsphere.html



1. Create your own install-config.yaml based off the template contained herein \
    `vi install-config.yaml`
2. Create a configuration directory and transfer your install-config.yaml there \
    `mkdir -p INSTALL ; mv install-config.yaml INSTALL`
3. Create your manifests \
    `openshift-install create manifests --dir=INSTALL`
4. Backup the worker node machineset \
    `mv INSTALL/openshift/99_openshift-cluster-api_worker-machineset-0.yaml .`
5. Remove the node manifests since we'll be building them manually \
    `rm -f INSTALL/openshift/99_openshift-cluster-api_master-machines-*.yaml INSTALL/openshift/99_openshift-cluster-api_worker-machineset-*.yaml`
6. Set mastersSchedulable to false \
    `sed -i -e 's/true/false/g' INSTALL/manifests/cluster-scheduler-02-config.yml`

7. Adding Windows Nodes (optional)
    1. Copy the network config to a new manifest \
        ` cp manifests/cluster-network-02-config.yml manifests/cluster-network-03-config.yml `
    2. Update the `apiVersion` & `spec` to support HNS
        ```
        apiVersion: operator.openshift.io/v1
        ...
        ...
        spec​:​
          defaultNetwork:    
            type: OVNKubernetes
            ovnKubernetesConfig:
              hybridOverlayConfig:
                hybridClusterNetwork:
                - cidr: 10.132.0.0/14
                  hostPrefix: 23
        ```
8. Create ignition files \
    `openshift-install create ignition-configs --dir=INSTALL`
9. Encode the master and worker ignition files \
    `base64 -i INSTALL/master.ign -o INSTALL/master.64 ; base64 -i INSTALL/worker.ign -o INSTALL/worker.64`
10. Copy your bootstrap.ign file to your webserver... (you're on your own here) \
    `scp INSTALL/bootstrap.ign $server:/path/to/www`
    tip: Ensure you have global read permissons on that file (chmod +r bootsrap.ign)
11. Create a boot-append.ign file and encode it 
    ```
    {
    "ignition": {
        "config": {
        "merge": [
            {
            "source": "http://$server/path/to/bootstrap.ign" 
            }
        ]
        },
        "version": "3.1.0"
      }
    }
    ```
    `base64 -i boot-append.ign -o boot-append.64`

Your setup is now done, from here on out I would recommend viewing the demo video since we'll be using VMware vCenter.