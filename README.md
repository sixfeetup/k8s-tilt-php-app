# Local development using Kubernetes


This repository includes a PHP application provided as a sample to demonstrate how to use Kubernetes to facilitate a local development workflow.

The application is powered through Laravel, it is located in directory `my-project` and there is a MariaDB database installed as well that gets started.

Both services are packaged with Docker and deployed using Kubernetes through Kind. Environment variables are stored as sealed password and Kustomize is used to provide a structure that re-uses configuration for different environments including dev, sandbox and prod.

## High level overview
We are going to install a number of dependencies that will allow the application to run in a containerized environment orchestrated by Kubernetes.
After installing the dependencies, it should all be started with a single command:

```
tilt up
```

We can see the application running in the browser and start developing! Changes are automatically reflected, as they are getting synced transparently by Tilt between the local folder and the containers that are running through Kubernetes. Tilt will also re-built the images and re-deploy them if there are changes that trigger a rebuilt on the related Dockerfile.


## Installation of dependencies

### Docker
The application is running through Docker containers, so we need to install Docker first. Follow the instructions on https://docs.docker.com/engine/install/

For Ubuntu 22 that is:
```
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh ./get-docker.sh --dry-run
```

## kubectl
Next we are installing Kubectl, the command line interface for Kubernetes. Follow the instructions on https://kubernetes.io/docs/tasks/tools/

For Ubuntu 22 that is:
```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

```

## Kind
Kind is a tool that allows us to run Kubernetes locally. It will create a local Kubernetes cluster for us to use as the environment where the application lives.  Follow the instructions on https://kind.sigs.k8s.io/docs/user/quick-start to get it installed

For Ubuntu 22 that is:

```
[ $(uname -m) = x86_64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
```

Other tools could be used instead of Kind, to setup a local cluster. For example, check how you can deploy Kubernetes with the Docker Desktop for Windows/Mac on https://docs.docker.com/desktop/kubernetes/

## Tilt
Tilt is the tool that will handle deployments for us (build containers, deploy to cluster, sync code changes), install following instructions on https://docs.tilt.dev/install.html

For Ubuntu 22 that is:

```
curl -fsSL https://raw.githubusercontent.com/tilt-dev/tilt/master/scripts/install.sh | bash
```


## Development workflow
We need to create a Kubernetes cluster locally, and this is done using the command:

```
kind create cluster --name laravel
```

Then start Tilt

```
tilt up
```
We hit `space` and this is starting the nice dashboard of Tilt on our browser. We click the Details view and choose Laravel, and see how the application is building. After 1-2 minutes the app informs us that it is ready on http://localhost:8000/


We can perform changes to the code, for example edit file `welcome.blade.php`. After a few seconds if we refresh the browser, we will see the changes reflected.

Once we are done with developmnent, we stop Tilt by hitting Ctrl+C and running

```
tilt down
```

Next time we want to start the application, we just need to run `tilt up` again.

## Tilt overview
There are two views, `Table` and `Detail` views.
* `Table` view is useful to see an overview of what resources are running, eg laravel application and MariaDB in our case. Next to the laravel resource, we can see the endpoint, and click on it, this is the endpoint that is getting explosed.
* `Detail` view is more useful when developing, as it shows the logs of the containers, and we can see the changes happening in real time. Also when we make changes to the code, we have the chance to see them getting synced to the running containers. Most important, if errors are happening we can see them here.



## Sample Application details
It's a vanilla Laravel app, inspired by https://hub.docker.com/r/bitnami/laravel/ that starts a Laravel app along with a MariaDB database.


## Deployment notes
* Namespace `dev` is used for all apps on the local deployment. If sandbox or prod environments are used, namespaces used there are sandox and prod relatively
* Different environments (as prod or sandbox) can override the default options that are on `k8s/base` folder, few samples of changes are included on `kustomization.yaml` files for sandbox/prod environments. For example, on sandbox we are using a different image for the application.

## Sealed secrets
To avoid commiting secrets to the repository, we are using sealed secrets. Sealed secrets are encrypted secrets that can be decrypted only by the cluster that created them.


Install `kubeseal` by following https://github.com/bitnami-labs/sealed-secrets#kubeseal


export all variables that will be sealed as ENV variables. Eg

```
export VARIABLE1=VARIABLE1
export VARIABLE2=VARIABLE2
export MARIADB_ROOT_PASSWORD=MARIADB_ROOT_PASSWORD
```

Then install the Sealed Secrets controller (https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.23.0/controller.yaml) on kube-system namespace. Run the following command, to install it on the sandbox namespace (or another namespace as prod if needed)

```
envsubst < k8s/templates/secrets.yaml.template | kubeseal --controller-namespace sandbox --format yaml > k8s/sandbox/secrets.yaml
```

File k8s/sandbox/secrets.yaml now contains the sealed secrets. We can commit it to the repository as secrets are encrypted.