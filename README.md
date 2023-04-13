# Overview : argocd-dex-onenldap
this repo provide the solotion to secure access to the Argocd using Oauth2-proxy, Dex and an OpenLDAP server - without requiring code changes to the ArgoCD app itself. 


# Prerequisites #

In order to complete this project, you will need an environment with the following prerequisites.


* [(macOS Only) Homebrew](#homebrew) - Package manager used to install prereqs
* [(Windows Only) Chocolatey](#chocolatey) - Package manager used to install prereqs
* [git](#git) - Used to clone the Pac-Man application
* [docker](#docker) - Container runtime
* [EKS](#EKS) - Running a cluster Kubernetes on aws
* [kubectl](#kubectl) - Kubernetes command-line tool
* [helm](#helm) - Kubernetes package manager
* [openldap](#openldap) - Used to populate OpenLDAP instance with user/group data




Apply complete! Resources: 58 added, 0 changed, 0 destroyed.

Outputs:

cluster_endpoint = "https://64D372A4AAF75516AEFFD0C9628ED692.gr7.eu-west-2.eks.amazonaws.com"
cluster_name = "education-eks-nFWFo235"
cluster_security_group_id = "sg-0596b405366e797c8"
region = "eu-west-2"






aws eks update-kubeconfig --name education-eks-nFWFo235 --region eu-west-2

kubectl port-forward service/argocd-server   9090:80


cd ../oauth2-proxy
kubectl create ns argocd
kubectl create -f oauth2-proxy-deployment.yaml -n argocd
kubectl create -f oauth2-proxy-service.yaml -n argocd

kubectl patch svc argocd-server -n argocd --type='json' -p='[{"op": "replace", "path": "/spec/ports/0/targetPort", "value":4180}]'
kubectl patch svc argocd-server  -n argocd --type='json' -p='[{"op": "replace", "path": "/spec/selector", "value":{"k8s-app": "oauth2-proxy"}}]'


kubectl port-forward service/argocd-server -n argocd 9090:80

kubectl create secret generic openldap --from-literal=adminpassword=adminpassword --from-literal=users=productionadmin,productionbasic,productionconfig --from-literal=passwords=testpasswordadmin,testpasswordbasic,testpasswordconfig -n openldap


(base) ghazal.naderi@contino.io@Ghazals-MacBook-Pro:/Users/ghazal.naderi@contino.io/projects/2023/helm/second-cluster$ k -n dex logs  deployments/dex

k -n argocd logs deployments/oauth2-proxy

kubectl port-forward service/openldap -n openldap 1389:1389   
kubectl port-forward service/dex -n dex 5556:5556
kubectl port-forward service/oauth2-proxy -n argocd 4180:4180
kubectl port-forward service/argocd-server -n argocd 9090:80
kubectl port-forward service/argocd-server-actual -n argocd 8080:8080
