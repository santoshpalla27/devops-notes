# Argo CD Notes

Git is single point of truth with ArgoCD

Kubernetes manifest files are present in GitHub and they are ultimate

Argo CD maintains desired state and current state

GitHub acts as desired state and EKS acts as current state

If someone makes change in EKS and it doesn't match with GitHub state then it will roll back to the desired state (GitHub state)

ArgoCD integrates with GitHub and takes GitHub state as primary state

## Installation

AWS configure should be done

And EKS cluster should be operational

```bash
aws eks update-kubeconfig --region <region> --name <cluster-name>  # to configure the eks cluster or get into the eks cluster

curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

kubectl create namespace argocd

helm repo add argo-cd https://argoproj.github.io/argo-helm

helm repo update

helm install argocd argo-cd/argo-cd --namespace argocd

kubectl get svc -n argocd

kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'

kubectl get svc argocd-server -n argocd
```

There you will find the DNS of load balancer (ArgoCD) loading may take time be patient

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
```

Default username: admin
Password format: twAhFDiPdhll1zDu (pass will be in this format)

## Setup an Application

1. First click on new app
2. Give an application name
3. Project name select default
4. Sync policy select automatic (how GitHub is sync with argocd) under automatic select selfheal and purse (this will maintain GitHub state)
5. Scroll down and copy paste the repo url
6. And path give the deployment folder path you can go to GitHub and go to the folder above you can copy the path
7. And in destination
8. Cluster url just select the our or available cluster you can choose as option by click in the space
9. Give a namespace
10. Rest leave everything and click on create button at top

Create a namespace in the cluster if not exist

To sync again click on sync