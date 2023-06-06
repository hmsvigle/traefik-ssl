# traefik-ssl

## 1. Deploy Traefikv2

### 1.1 Prepare Traefiv2 Artifacts :

```sh
$ helm repo add dag https://dockerhub.com/chartrepo/dag
"harbor_dag" has been added to your repositories

$ chmod go-r kubeconfig
(Notes: Troubleshoot:WARNING: Kubernetes configuration file is group-readable. This is insecure. Location: /home/.../kubeconfig
WARNING: Kubernetes configuration file is world-readable. This is insecure. Location: /home/.../kubeconfig)

$ helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "harbor_dag" chart repository
Update Complete. ⎈Happy Helming!⎈

$ helm search repo traefik
NAME                    CHART VERSION   APP VERSION     DESCRIPTION
dag/traefik      1.88.0          1.7.20          A Traefik based Kubernetes ingress controller w...
dag/traefikv2    10.3.6          2.5.3           A Traefik based Kubernetes ingress controller


$ helm pull harbor_dag/traefikv2 --untar
```
### 1.2 Deploy Traefikv2 

 * Update [traefik-custom-values.yaml](./traefikv2/custom-values.yaml) with 
   * LB IP, if already IP is available.
   * Respective namespace list, on which traefik should listen to.
 
```sh
# Save the traefik template file for future reference. 
$ helm upgrade traefikv2 traefikv2/ \
  --install   --namespace ingress-dev \
  -f traefikv2/custom-values.yaml --dry-run  > traefikv2/traefikv2-template.yaml

# Deploy Traefik with custom values.
$ helm upgrade traefikv2 traefikv2/ \
  --install   --namespace ingress-dev \
  -f traefikv2/custom-values.yaml

```

  ### 1.3 Deploy nginx, sample app to test routing with non-ssl
   * Deploy [nginx & ClusterIP Service](nginx/nginx.yaml) , [ingressroute](nginx/nginx-ingressroute-pathprefix-no-ssl.yaml).

  ### 1.4 Test nginx dashboard with non-ssl
  * Nginx is hosted at `/`
      *  **`PathPrefix`** & **`Path`** will work with out any middleware.
      *  **`AddPrefix`** : It will not work as nginx has to be hosted at a different path for this to work.
      *  **`PathPrefixStrip`** : It also should work

  * **`PathPrefix`**: [ingressroute](nginx/nginx-ingressroute-pathprefix-no-ssl.yaml)

  ![image](https://media.git.daimler.com/user/3406/files/9e57b380-37ec-11ec-9e00-8f69e635fcd4)

  * **`Path`**: [ingressroute](nginx/nginx-ingressroute-path-no-ssl.yaml)
  
  ![image](https://media.git.daimler.com/user/3406/files/3b1e4f00-37f5-11ec-96c9-2434d6bb75bb)

  * **`StripPrefix`**: [ingressroute](nginx/nginx-ingressroute-stripPrefix-no-ssl.yaml)
  
  ![image](https://media.git.daimler.com/user/3406/files/630eb200-37f7-11ec-965c-126ba845cfd2)


  ### 1.5 Traefik-dashboard non-ssl
   * Deploy [ingressroute](traefikv2/traefik-dashboard-ingressroute-no-ssl.yaml) for traefik-dashboard with no-ssl.
   
  
  ![image](https://media.git.daimler.com/user/3406/files/58501f00-37ef-11ec-98da-18b33ac9d96f)

  
  
## 2. Cert-Manager:

  ### 2.1 Pull Cert-Manager Repo
  
```sh
$ helm repo add jetstack https://charts.jetstack.io
"jetstack" has been added to your repositories
$ helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "jetstack" chart repository
Update Complete. ⎈Happy Helming!⎈

$ helm pull jetstack/cert-manager --version v1.5.3  --untar

```

  ### 2.2 Deploy Secret 
  
  * Create cert-manager namespace :- `kubectl create ns cert-manager`
  * Deploy Secret with Daimler CA root Certificate with [01-secret.yaml](./cert-manager/01-secret.yaml) : `kubectl apply -f ./cert-manager/01-secret.yaml -n cert-manager`
  
  ### 2.3 Deploy Cert-Manager
  
  * Deploy Cert-manager with Daimler CA integrated Secret in it. Use [Custom-Values.yaml](./cert-manager/02-custom-values.yaml)
  
```sh
$ 
$ helm template cert-manager cert-manager/  \
--namespace cert-manager  --set installCRDs=true \
-f ./cert-manager/02-custom-values.yaml --version v1.5.3  \
| kubectl apply -f - -n cert-manager

```

<details><summary>Output Logs</summary>
<p>
serviceaccount/cert-manager-cainjector created
serviceaccount/cert-manager created
serviceaccount/cert-manager-webhook created
customresourcedefinition.apiextensions.k8s.io/challenges.acme.cert-manager.io created
customresourcedefinition.apiextensions.k8s.io/certificaterequests.cert-manager.io created
customresourcedefinition.apiextensions.k8s.io/orders.acme.cert-manager.io created
customresourcedefinition.apiextensions.k8s.io/certificates.cert-manager.io created
customresourcedefinition.apiextensions.k8s.io/issuers.cert-manager.io created
customresourcedefinition.apiextensions.k8s.io/clusterissuers.cert-manager.io created
clusterrole.rbac.authorization.k8s.io/cert-manager-cainjector created
clusterrole.rbac.authorization.k8s.io/cert-manager-controller-orders created
clusterrole.rbac.authorization.k8s.io/cert-manager-edit created
clusterrole.rbac.authorization.k8s.io/cert-manager-controller-certificatesigningrequests created
clusterrole.rbac.authorization.k8s.io/cert-manager-controller-clusterissuers created
clusterrole.rbac.authorization.k8s.io/cert-manager-controller-certificates created
clusterrole.rbac.authorization.k8s.io/cert-manager-view created
clusterrole.rbac.authorization.k8s.io/cert-manager-controller-issuers created
clusterrole.rbac.authorization.k8s.io/cert-manager-controller-ingress-shim created
clusterrole.rbac.authorization.k8s.io/cert-manager-controller-challenges created
clusterrole.rbac.authorization.k8s.io/cert-manager-controller-approve:cert-manager-io created
clusterrole.rbac.authorization.k8s.io/cert-manager-webhook:subjectaccessreviews created
clusterrolebinding.rbac.authorization.k8s.io/cert-manager-cainjector created
clusterrolebinding.rbac.authorization.k8s.io/cert-manager-controller-orders created
clusterrolebinding.rbac.authorization.k8s.io/cert-manager-controller-certificates created
clusterrolebinding.rbac.authorization.k8s.io/cert-manager-controller-clusterissuers created
clusterrolebinding.rbac.authorization.k8s.io/cert-manager-controller-ingress-shim created
clusterrolebinding.rbac.authorization.k8s.io/cert-manager-controller-certificatesigningrequests created
clusterrolebinding.rbac.authorization.k8s.io/cert-manager-controller-approve:cert-manager-io created
clusterrolebinding.rbac.authorization.k8s.io/cert-manager-controller-issuers created
clusterrolebinding.rbac.authorization.k8s.io/cert-manager-controller-challenges created
clusterrolebinding.rbac.authorization.k8s.io/cert-manager-webhook:subjectaccessreviews created
role.rbac.authorization.k8s.io/cert-manager-cainjector:leaderelection created
role.rbac.authorization.k8s.io/cert-manager:leaderelection created
role.rbac.authorization.k8s.io/cert-manager-webhook:dynamic-serving created
rolebinding.rbac.authorization.k8s.io/cert-manager-cainjector:leaderelection created
rolebinding.rbac.authorization.k8s.io/cert-manager:leaderelection created
rolebinding.rbac.authorization.k8s.io/cert-manager-webhook:dynamic-serving created
service/cert-manager created
service/cert-manager-webhook created
deployment.apps/cert-manager-cainjector created
deployment.apps/cert-manager created
deployment.apps/cert-manager-webhook created
mutatingwebhookconfiguration.admissionregistration.k8s.io/cert-manager-webhook created
validatingwebhookconfiguration.admissionregistration.k8s.io/cert-manager-webhook created

</p>
</details>

  ### 2.4 Deploy ClusterIssuer into Default namespace:
   * ClusterIssuer will be same for all components. We can define only one class name as input. Cant confugure array of classes.
   * ClusterIssuer = Daimler Acme service + Traefik Class name + email
   * Update email-id before applying it.
   * Deploy with [03-ClusterIssuer.yaml](./cert-manager/03-cluster-issuer.yaml)
   
   ```sh
   
    $ kubectl apply -f ./cert-manager/03-cluster-issuer.yaml 
    
    $ kubectl get clusterissuer
      NAME           READY   AGE
      acme   True    2d

   ```
   ### 2.5 Deploy NetworkPolicy to namespace where traefik is deployed:
   
   * Deploy Network Policy into traefik namespace.
   * Make sure to update NamespaceSelector value to traefik namespace
   * The traefik label should be correct as per the [04-acme-netpol.yaml](./cert-manager/04-acme-networkpolicy.yaml)

   ```sh
   $ kubectl apply -n ingress-dev -f ./cert-manager/04-acme-networkpolicy.yaml
   ```
   
   * After deployment of acme netpol, we can find acme pod created in ingress-dev namespace (the namespace where traefik is deployed)
  
  ```sh
  $ k get po -n ingress-dev
  NAME                             READY   STATUS    RESTARTS   AGE
  cm-acme-http-solver-jldhl        1/1     Running   0          6s
  traefikv2-dev-68478d55c7-sg4cr   1/1     Running   0          29m
  ```
  ### 2.6 Deploy Certificate & Traefik-dashboard-ingressroute
  
  * Deply Certificate & ingressroute to traefik Dashboard using [05-cert-traefik-dashboard-ingr.yaml](./cert-manager/05-cert-traefik-dashboard-ingr.yaml)
  * Deploy certificate in namespace, where traefik is deployed i.e ingress-dev
  * Update DNS name at all the places carefully.
  * We can use the same certificate for all application routing
  

  ```sh
  $ kubectl apply -f  ./cert-manager/05-cert-traefik-dashboard-ingr.yaml -n ingress-dev
  ```
  
  * After Deployng certificate & ingressroute (+ tls & secret details), output:
  
  ```sh
  $ kubectl get po -n ingress-dev
  NAME                         READY   STATUS    RESTARTS   AGE
  cm-acme-http-solver-ml8xg    1/1     Running   0          51s
  traefikv2-7f4f88df66-gnllz   1/1     Running   0          93m
  
  $ kubectl get orders,challenges,certificaterequest -n ingress-dev
  NAME                                                           STATE     AGE
  order.acme.cert-manager.io/traefik-dashboard-5dtj6-778711773   Valid   72s

  NAME                                                                          STATE     DOMAIN                                   AGE
  challenge.acme.cert-manager.io/traefik-dashboard-5dtj6-778711773-3645429168   pending   iaas-c02-53-6-21-162.dhc.corpintra.net   71s
  
  NAME                                                         APPROVED   DENIED   READY   ISSUER         REQUESTOR                                         AGE
  certificaterequest.cert-manager.io/traefik-dashboard-5dtj6   True                True   daimler-acme   system:serviceaccount:cert-manager:cert-manager   72s
  
  ```

  * Verify traefik-dashboard url
  
  
  
  * Deploy nginx with [ingressroute-ssl.yaml](./nginx/nginx-ingressroute-ssl.yaml) & verify in browser
  


  
