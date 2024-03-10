# Install OpenShift 4.146.1 on VMware 6.7U3

OpenShift on VMware


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
6. Set mastersSchedulable to false if not \
    `sed -i -e 's/true/false/g' INSTALL/manifests/cluster-scheduler-02-config.yml`
7. Create ignition files \
    `openshift-install create ignition-configs --dir=INSTALL`
8. Encode the master and worker ignition files \
    `base64 -i INSTALL/master.ign -o INSTALL/master.64 ; base64 -i INSTALL/worker.ign -o INSTALL/worker.64`
9. Copy your bootstrap.ign file to your webserver... (you're on your own here) \
    `scp INSTALL/bootstrap.ign $server:/path/to/www`
    tip: Ensure you have global read permissons on that file (chmod +r bootsrap.ign)
10. Create a boot-append.ign file and encode it 
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
