# ArgoCD Installation using ArgoCD Autopilot on K3s Cluster

This guide walks you through installing ArgoCD using ArgoCD Autopilot on a K3s cluster. ArgoCD is a declarative, GitOps continuous delivery tool for Kubernetes. ArgoCD Autopilot simplifies the process of setting up ArgoCD by managing repository bootstrapping.

## Prerequisites
- A running K3s cluster. Ensure the cluster is installed and working correctly.
- `kubectl` installed and configured to interact with your K3s cluster.
- A GitHub account to host the GitOps repository.
- A valid GitHub token with access to create repositories and push changes.

## Step 1: Install ArgoCD Autopilot
Open a terminal on your VM or localhost that has access to your K3s cluster.

Run the following commands to install the latest version of `argocd-autopilot`:

```bash
VERSION=$(curl --silent "https://api.github.com/repos/argoproj-labs/argocd-autopilot/releases/latest" | grep '"tag_name"' | sed -E 's/.*"([^"]+)".*/\1/')
curl -L --output - https://github.com/argoproj-labs/argocd-autopilot/releases/download/"$VERSION"/argocd-autopilot-linux-amd64.tar.gz | tar zx
sudo mv ./argocd-autopilot-* /usr/local/bin/argocd-autopilot
```

**Verify the installation by checking the version:**

```bash
argocd-autopilot version
```

The version should print out, indicating a successful installation.

## Step 2: Set Up GitHub Repository and Token
Obtain a GitHub Token by following [this guide](https://docs.github.com/en/github/authenticating-to-github/creating-a-personal-access-token) if you haven't already.

Export your GitHub token and set up the Git repository you will use for ArgoCD:

```bash
export GIT_TOKEN=your-github-token
export GIT_REPO=your-repo-link
```

Replace `GIT_TOKEN` with your actual GitHub token and `GIT_REPO` with the URL of your Git repository.

## Step 3: Bootstrap the GitOps Repository
With the repository and Git token set up, bootstrap ArgoCD into your cluster by running the following command:

```bash
argocd-autopilot repo bootstrap
```

This will deploy ArgoCD to your K3s cluster and initialize your GitOps repository.

Once the process completes successfully, you will see two important outputs:

- **Admin Password for ArgoCD.**
- **Port Forwarding Command to access the ArgoCD UI.**

Save these details for later use.

## Step 4: Exposing ArgoCD Service Using NodePort
By default, ArgoCD is exposed using a `ClusterIP` service. To expose ArgoCD via a NodePort for external access, follow these steps:

Patch the ArgoCD server service to use NodePort:

```bash
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort"}}'
```

Get the NodePort assigned to the ArgoCD service:

```bash
kubectl get svc argocd-server -n argocd
```

The output will show a NodePort under the `PORT(S)` column, typically in the range 30000–32767. For example:

```bash
NAME            TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)             AGE
argocd-server   NodePort   10.43.52.15    <none>        80:32000/TCP        2m
```

**Access the ArgoCD UI by navigating to:**

```bash
http://<NODE_IP>:<NODE_PORT>
```

Replace `<NODE_IP>` with the IP address of your K3s node and `<NODE_PORT>` with the actual NodePort (e.g., 32000).

## Step 5: Log in to ArgoCD
Log in using the following credentials:

- **Username**: `admin`
- **Password**: (The password provided during installation)

## Step 6: Verify GitOps Repository Updates
After logging into the ArgoCD UI, check your GitHub repository to verify that ArgoCD has successfully pushed the initial GitOps configuration files.

Ensure that all necessary ArgoCD resources (projects, applications, etc.) are visible on the ArgoCD dashboard.
```
