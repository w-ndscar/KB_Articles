***
*Shoutout to [Calvin Remsburg](https://www.youtube.com/@CalvinRemsburg0) for a detailed explanation on this topic. This document is inspired from his video: [Installing Ansible AWX](https://www.youtube.com/watch?v=Nvjo2A2cBxI)*
#### Objective: Deploy AWX using Kubernetes (K3S) in a Linux environment

#### Things to do: 
- Update the system by using `sudo apt-get update && sudo apt-get upgrade -y`
- Install Kubernetes
- Setup AWX Operator
- Build AWX
- Login to AWX

***

## Install Kubernetes (K3S)
***

Use the following command to setup Kubernetes
```shell
curl -sfL https://get.k3s.io | sh -
```

Check whether Kubernetes has properly been installed by using the command:
`sudo kubectl version`

#### Modify the permission of the k3s.yaml file

Type the command `ls -lsa /etc/k3s/k3s.yaml` to check the current permissions of the k3s.yaml file. If you want to change the permission of the k3s.yaml file and make it editable by a specific user, do the following:
- Type `whoami` and `groups` to list the users and groups
- Then type the command `sudo chown username:groupname /etc/rancher/k3s/k3s.yaml`
- Then you can use `kubectl` without root/sudo

Type the command `kubectl get nodes` to get the status of Kubernetes

Refer to the following page for more info: [K3S](https://docs.k3s.io/installation/configuration)

***


## Setup AWX Operator
***

We'll use **[Kustomize](https://kubernetes.io/docs/tasks/manage-kubernetes-objects/kustomization/)** to configure kustomization.yaml file to setup AWX Operator. The AWX Operator is like an architect. We provide the configuration on how to setup Kubernetes and the AWX Operator grabs the necessary files and spin things in a specific way and get all the things working.

Install the **Kustomize [Binaries](https://kubectl.docs.kubernetes.io/installation/kustomize/binaries/)** using the following command
````bash
curl -s "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh"  | bash
````

Then move the Kustomize Binary from **/home/username/kustomize** to /bin directory by using the command: `sudo mv kustomize /usr/local/bin`

Then type the command `which kustomize` to show the kustomize directory

Create a new file named **kustomization.yaml** and copy the following contents and edit them as needed

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  # Find the latest tag here: https://github.com/ansible/awx-operator/releases
  - github.com/ansible/awx-operator/config/default?ref=2.2.1
# Set the image tags to match the git version from above

images:
  - name: quay.io/ansible/awx-operator
	newTag: 2.2.1

# Specify a custom namespace in which to install AWX

namespace: awx
```

Type the following command to use the Kubernetes Binary to point to the local directory and pipe that output to **kubectl**
```shell
kustomize build . | kubectl apply -f -
```

This will create the awx resources in kubernetes using Kustomize

You can see the running pods by typing the following command:
```shell
kubectl get pods --namespace awx
```

The pods won't be getting listed if we just run `kubectl get pods` because we have specified `namepace: awx` in the **kustomization.yaml** file.
***

## Build AWX
***

Refer to this **[AWX Github](https://github.com/ansible/awx-operator/)** doc for more info

Create a file named **awx.yaml** and copy the following contents and edit them accordingly:

```yaml
---
apiVersion: awx.ansible.com/v1beta1
kind: AWX
metadata:
  name: awx
spec:
  service_type: nodeport
  nodeport_port: 30080
```

You can specify a different non-standard port if needed.

We need pass the **awx.yaml** instructions to the AWX Operator that we have created before. To do that, we need add a line awx.yaml under the resources section in the **kustomization.yaml** that we have created before, so it would look like:

```yaml
resources:
  # Find the latest tag here: https://github.com/ansible/awx-operator/releases
  - github.com/ansible/awx-operator/config/default?ref=2.2.1
  - awx.yaml
# Set the image tags to match the git version from above
```

Now run the following command again to deploy the awx additional resources  
```shell
kustomize build . | kubectl apply -f -
```

The additional resources include web UI, database, network and so on.

You can see the logs of the deployment using the following command:

```shell
kubectl logs -f deployments/awx-operator-controller-manager -c awx-manager --namespace awx
```

If the everything looks ok, type the following command to list the running pods:
```shell
kubectl get pods -n awx
```

***

## Login to AWX
***

Get the default admin password by using the command: 
```shell
`kubectl get secret <resourcename>-admin-password -o jsonpath="{.data.password}" | base64 --decode ; echo`
```
The resource name in this case is: *awx*

Open a browser and type in the IP address of the Linux machine and the port 30080 as specified in the **awx.yaml** file (Eg. http://192.168.10.11:30080). Enter the usename as *admin* and the password derived from the command and click **Login**. It is recommended to change the default admin password after the initial login.

To know more about Ansible AWX, click [here](https://github.com/ansible/awx)

***
