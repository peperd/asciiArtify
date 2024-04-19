# Getting Started

## Requirements¶
Installed kubectl command-line tool.
Have a kubeconfig file (default location is *~/.kube/config*).<br>
CoreDNS. <br>
Can be enabled for microk8s by microk8s enable dns && microk8s stop && microk8s start

## 1. Install Argo CD¶

```
kubectl create namespace argocd
```

```
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

## 2. Download Argo CD CLI¶
Download the latest Argo CD version from *https://github.com/argoproj/argo-cd/releases/latest* <br>
More detailed installation instructions can be found via the CLI installation documentation.

Also available in Mac, Linux and WSL Homebrew:

```
brew install argocd
```
## 3. Access The Argo CD API Server¶
By default, the Argo CD API server is not exposed with an external IP. <br>
To access the API server, choose following technique

**Change the argocd-server service type to LoadBalancer**:

```
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
```
**Port Forwarding** <br>
Kubectl port-forwarding can also be used to connect to the API server without exposing the service.

```
kubectl port-forward svc/argocd-server -n argocd 8080:443
```
The API server can then be accessed using https://localhost:8080

## 4. Login Using The CLI
The initial password for the admin account is auto-generated and stored as clear text in the field password in a secret named argocd-initial-admin-secret in your Argo CD installation namespace. You can simply retrieve this password using the argocd CLI:

```
argocd admin initial-password -n argocd
```

Using the username admin and the password from above, login to Argo CD's IP or hostname:

```
argocd login <ARGOCD_SERVER>
```
Change the password using the command:

```
argocd account update-password
```

## 6. Create An Application From A Git Repository
An example repository containing a guestbook application is available at https://github.com/den-vasyliev/go-demo-app to demonstrate how Argo CD works.

* Manually create application in argocd ui
* Sync application
* Retrieve information about services in the Kubernetes cluster
```
kubectl get svc -n demo 
```
* Port-forward service ambassador to localhost
```
kubectl port-forward -n demo svc/ambassador 8080:80
```
* Verify localhost
```
curl localhost:8080
```
* Download random image
```
curl -o /tmp/g.png https://www.google.com/images/branding/googlelogo/2x/googlelogo_color_92x30dp.png
```
* Open image in service
```
curl -F 'image=@/tmp/g.png' localhost:8088/img/
```
![ambassador](doc/ambassador.gif)