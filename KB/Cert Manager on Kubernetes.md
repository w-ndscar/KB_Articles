### Objective:
***
Install Cert Manager on Kubernetes and issue self signed certificates to the applications running in all the namespaces in Kubernetes.
***

### Install Cert Manager
***
Use Helm to install Cert Manager - More info [here](https://cert-manager.io/docs/installation/helm/)

Create the Cluster Issuer Object. The Issuer is a Kubernetes resource that represents the Certificate Authority (CA). The Issuer is just a CA for its own namespace. The Cluster Issuer is a CA for all the namespaces.

Refer to Christian Lempa's [Github repo](https://github.com/ChristianLempa/boilerplates/tree/main/kubernetes/certmanager) for templates
***
### Create the Certificate Authority
***
Create the Private Key for the Certificate Authority by using the following command:
```shell
openssl genrsa -out ca-pr.key 4096
```
Please store this private key in a secured location as it's not protected with a passphrase. It'll easier to import this into the Cert Manager without the passphrase.

Create a public CA Certificate with the validity of 30 years using the following command
```shell
openssl req -new -x509 -sha356 -days 10950 -key ca-pr.key -out ca.crt
```
Input the necessary information like the Organization Name, Region and so on.

To add this `ca.crt` to the Trusted Root CA Store of the Client machines, refer to the following cheatsheet on how to import the certification on different operating systems: [[SSL Certs]]
***
### Create a ClusterIssuer config
***
Refer to this repo [SSL_in_Kubernetes](https://github.com/ChristianLempa/videos/tree/main/self-signed-certificates-in-kubernetes) for more info and templates.

Create a ca_secret.yaml file with the Secret Object that will hold the CA's Private Key (`ca-pr.key`) and the Public Certificate (`ca.crt`). Store it in the namespace of the cert-manager instead of the project that you're working on. We need to get the private key and the certificate values in one line, so that we can refer them into the Secret Object. Use the following commands to do so:
```shell
cat ca.crt | base64 -w 0
cat ca.key | base64 -w 0
```

You can also try
```shell
kubectl create secret --from-file
```
command to the copy the contents of the key and cert.

Then apply this Kubernetes secret to the Kubernetes cluster of the Cert Manager using the following command:
```shell
kubectl apply -f .\ca_secret.yaml
```

Create a cluster_issuer.yaml file. Object - ClusterIssuer. Refer it to the ca_secret.yaml file by adding the following lines.
```yaml
ca:
	secretName: ca_secret.yaml
```

Then apply the Cluster Issuer into the cluster by using the following command:
```shell
kubectl apply -f .\cluster_issuer.yaml
```

Now, all the certificates that are issued by this Cluster Issuer should be trusted by the Clients to which the CA was added to the CA Store.
***
### Creating a certificate Object in your Project
***

Create something like a shadow of the Cert Manager's Cluster Issuer and the Secret inside your project. Create `cert_issue.yaml` which will refer to the cluster_issuer of the Cert Manager and specify a `secretName` in `cert_issue.yaml` to store the certificate info issued by the Cluster Issuer. 

Then use the following command to add cert_issue.yaml to the cluster:
```shell
kubectl apply -f .\cert_issue.yaml
```

Then use
```
kubectl describe cert_issue
```
to see the status of the certificate.
***
