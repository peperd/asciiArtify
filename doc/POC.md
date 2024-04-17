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

## 5. Register A Cluster To Deploy Apps To (Optional)
This step registers a cluster's credentials to Argo CD, and is only necessary when deploying to an external cluster. When deploying internally (to the same cluster that Argo CD is running in), https://kubernetes.default.svc should be used as the application's K8s API server address.

**First list all clusters contexts in your current kubeconfig:**

```
kubectl config get-contexts -o name
```
Choose a context name from the list and supply it to argocd cluster add CONTEXTNAME. For example, for docker-desktop context, run:

```
argocd cluster add docker-desktop
```
The above command installs a ServiceAccount (argocd-manager), into the kube-system namespace of that kubectl context, and binds the service account to an admin-level ClusterRole. Argo CD uses this service account token to perform its management tasks (i.e. deploy/monitoring).

## 6. Create An Application From A Git Repository
An example repository containing a guestbook application is available at https://github.com/argoproj/argocd-example-apps.git to demonstrate how Argo CD works.

**Creating Apps Via CLI** <br>
First we need to set the current namespace to argocd running the following command:

```
kubectl config set-context --current --namespace=argocd
```
**Create the example guestbook application with the following command:**

```
argocd app create guestbook --repo https://github.com/argoproj/argocd-example-apps.git --path guestbook --dest-server https://kubernetes.default.svc --dest-namespace default
```
## 7. Sync (Deploy) The Application
**Syncing via CLI** <br>
Once the guestbook application is created, you can now view its status:

```
$ argocd app get guestbook
```
The application status is initially in OutOfSync state since the application has yet to be deployed, and no Kubernetes resources have been created. To sync (deploy) the application, run:

```
argocd app sync guestbook
```
This command retrieves the manifests from the repository and performs a kubectl apply of the manifests. The guestbook app is now running and you can now view its resource components, logs, events, and assessed health status.

![agrocd](doc/argocd.gif)
