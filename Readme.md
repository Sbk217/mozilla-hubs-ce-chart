# Welcome to the Mozilla Hubs Community Edition Helm Chart version for TU Ilmenau Research Project S4_VWDS6 done in SS2024 !
# In our setup 2 bare metal on-premise university servers were given 
# Master server - Server 1
# Worker server - Server 2

1. To start you need a kubernetes Rancher kubernetes engine 2 (RKE2) cluster setup of which is described in the Project Report. 
2. Your worker node/nodes should have these ports open as done via UFW in the project:
   * TCP: 80, 443, 4443, 5349 //For Mozilla Hubs CE only
   * UDP: 35000 - 60000 //For Mozilla Hubs CE only
   * Additonal ports required for the RKE2 cluster and pod networking are described in the Project Report.
  
3. A Load balancer is required for Networking load balancing (which are by default present in cloud clusters) since we are setting up the cluster on bare metal servers.
   This is done through the use of MetalLb, steps of which are described in the Project Report.


## Setup SSL Certs
4. We need a few ssl certs and cert manager allows us to automate the request but also the renewal of the certs. lets install it!
### Install cert-manager 
See https://cert-manager.io/docs/installation/helm/ for more information

```
    kubectl create ns cert-manager
    helm repo add jetstack https://charts.jetstack.io
    helm repo update
    helm install cert-manager jetstack/cert-manager \
     --namespace cert-manager \
     --set ingressShim.defaultIssuerName=letsencrypt-issuer \
     --set ingressShim.defaultIssuerKind=ClusterIssuer \
     --set installCRDs=true
```

### Create cluster-issuer.yaml for Let's Encrypt
```
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-issuer
spec:
  acme:
    # You must replace this email address with your own.
    # Let's Encrypt will use this to contact you about expiring
    # certificates, and issues related to your account.
    email: '{YOUR_EMAIL_ADDRESS}'
    server: https://acme-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      # Secret resource that will be used to store the account's private key.
      name: letsencrypt-issuer
    # Add a single challenge solver, HTTP01 using nginx
    solvers:
      - http01:
          ingress:
            class: haproxy
```

Then run
```
kubectl apply -f 'PATH_TO/cluster-issuer.yaml'
```

## Deployment
### Install this helm chart

Copy and paste the configs.data output of render_helm.sh into the values.yaml file
```
./render_helm.sh {YOUR_HUBS_DOMAIN} {ADMIN_EMAIL_ADDRESS}
```
> You need the default cert to allow cert manager to request certs. 
> It will create a new file `config.yaml` 

Modify the values.yaml with your domain and email, replace whats inside the string.
```
global:
  domain: &HUBS_DOMAIN "{YOUR_HUBS_DOMAIN}"
  adminEmail: &ADMINEMAIL "{ADMIN_EMAIL_ADDRESS}"
```

To finish up the install run:
```
git clone git@github.com:hubs-community/mozilla-hubs-ce-chart.git
cd mozilla-hubs-ce-chart

kubectl create ns {YOUR_NAMESPACE}

helm install moz . --namespace={YOUR_NAMESPACE} --debug --dry-run
```
> remove --dry-run to fully install

> [AWS Deployment Notes](./Readme.aws.md)

## Update this helm chart
Update what you need to, ie, values.yaml or template files. Remove --dry-run to upgrade
```
helm upgrade moz . --namespace={YOUR_NAMESPACE} --debug --dry-run
```

## Delete this helm chart
This will remove everything installed by this chart. Remove --dry-run to delete
```
helm delete moz --namespace={YOUR_NAMESPACE} --dry-run
```


