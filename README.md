# Project GitOps with Flux y Kind

This project guides you through setting up a GitOps environment using [Flux](https://fluxcd.io) and [Kind](https://kind.sigs.k8s.io/) (Kubernetes in Docker).

## Prerequisites

Make sure you have the following prerequisites installed:

- [Docker](https://www.docker.com/get-started)
- [Kind](https://kind.sigs.k8s.io/)
- [Kubectl](https://kubernetes.io/docs/tasks/tools/)
- [Flux CLI](https://fluxcd.io/flux/installation/#install-the-flux-cli)

### 1. Create a Kubernetes Cluster using Kind

> kind create cluster --name flux-cluster

This command will create a Kubernetes cluster named `flux-cluster` using Kind.

### 2. Installing Flux on the Cluster

First, authenticate with your Git repository (on GitHub, for example) where the cluster configuration will be stored:

    export GITHUB_TOKEN=<tu-token-github>
    export GITHUB_USER=<tu-usuario-github>

Next, initialize Flux on the cluster and connect it to your Git repository:

    flux bootstrap github \
      --owner=$GITHUB_USER \
      --repository=<nombre-del-repositorio> \
      --branch=main \
      --path=./clusters/my-cluster \
      --personal

This command will:

- Install Flux on the cluster.
- Connect the cluster to your Git repository at the path ./clusters/my-cluster.
- Create an initial commit to the repository with the basic Flux configuration.

### 3. Managing Infrastructure with GitOps

Now, you can start managing your infrastructure through GitOps. Any configuration changes you make in the Git repository will be automatically applied to your Kubernetes cluster.

### 4. Desplegar Recursos con Flux
Create a Kubernetes manifest file in your repository, for example podinfo.yaml, at the specified path (e.g., ./clusters/my-cluster/podinfo.yaml):

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: podinfo
      namespace: default
    spec:
      replicas: 2
      selector:
        matchLabels:
          app: podinfo
      template:
        metadata:
          labels:
            app: podinfo
        spec:
          containers:
            - name: podinfo
              image: ghcr.io/stefanprodan/podinfo:6.1.4
              ports:
                - containerPort: 9898
                  protocol: TCP
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: podinfo
      namespace: default
    spec:
      type: ClusterIP
      selector:
        app: podinfo
      ports:
        - port: 80
          targetPort: 9898

Add, commit, and push changes to your repository:

> git add .

> git commit -m "Deploy podinfo app"

> git push origin main

Flux will automatically apply these changes to your Kubernetes cluster.

### 5. Check Deployment

Verify that Flux has correctly deployed the resources:

> flux get all

This command will show all the resources managed by Flux in your cluster.

### 6. Clean the Environment

If you want to delete the Kind cluster when you are done:

> kind delete cluster --name flux-cluster

## Additional Resources

- [Flux Documentation](https://fluxcd.io/flux/)
- [Kind User Guide](https://kind.sigs.k8s.io/docs/user/quick-start/)
- [Kubernetes Documentation](https://kubernetes.io/docs/home/)
