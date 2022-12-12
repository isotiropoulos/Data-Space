# IDS-Testbed deployed using Kubernetes


The original IDS-testbed repository deploys a data space with docker-compose technology, this type of technology is not the best solution to work on a production system. For this reason, in this repository, it has been proposed an alternative IDS-testbed deployed with Kubernetes technology either in a cloud cluster like Rancher or a local cluster like Docker-Desktop or Minikube. The below image shows the transition from a local system (docker-compose) to a cloud system (K8s). 
<img src="pictures/architecture.png" alt="architecture"/>

## Requirements to run in a local machine using K8S. 

This subsection is explained how to deploy a DataSpace using K8S in a local machine. For this task, it has been used a PC with *Kubectl* and *Docker-Desktop*, the software specifications are below in the next subsection. The K8S cluster is deployed using *Docker-Desktop*, this cluster is much like Minikube.

The PC has Windows 10 as OS but to deploy the architecture it has been used WSL V2.0 with Ubuntu 20.04 LTS to work with the clusters. 


### Software-requirements

- [Kubectl](https://kubernetes.io/es/docs/tasks/tools/) version:
  - Client Version: `v1.24.0`
  - Kustomize Version: `v4.5.4`
  - Server Version: `v1.24.0`
- [Kompose](https://kompose.io/)
  - `1.26.0`

- Cluster runs in [Docker-Desktop](https://docs.docker.com/desktop/windows/install/)
  - `v20.10.14`

-  [IDS-Release 1.0](https://github.com/International-Data-Spaces-Association/IDS-testbed)
-   OS Windows 10 Enterprise
    -   WSL2: Ubuntu 20.04 LTS
  
- OpenSSL `1.1.1f`



### Hardware-requirements

The characteristics of the PC used are:

- 16 GB RAM.
- Intel(R) Core(TM) i5-10310U CPU @ 1.70GHz   2.21 GHz.

- 237 GB ROM memory.

## IDS deployment in K8S in a local machine.

In order to work in a cluster using good practices it is essential to have a clean environment, for this reason, it is important to create a namespace for the DataSpace. 

- The easiest way to create a namespace is using the next command. In this case, ids-2 is the name of the namespace. 
  
  `kubectl create namespace ids-2`
  

- It is possible to assign an URL for the localhost IP. In the  `/etc/hosts` file, it is could define that characteristic. 
  
  `127.0.0.1       connectora.localhost`
    
  `127.0.0.1       connectorb.localhost`


- A strength of IDSA is security, for this reason, it is necessary to create a set of certificates.
  
  `openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt`
  
- With these certificates, it will be created a K8S secret. The fastest way to create a secret is using the below sell command.

  `kubectl create secret tls tls-secret --key tls.key --cert tls.crt -n ids-2`

- The [idsa_manifest_local](./idsa_manifest_local/) folder is divided into several folders, each folder corresponding to each IDSA component. Also, there is a folder with the [Traefik](./traefik/)proxy and another with the [Nginx](./Nginx/) ingress. Finally, to create a component in the cluster you can use the above command, being, ".MANIFEST_NAME" the YAML name that describes the component.
    
    `kubectl apply -f .MANIFEST_NAME.yaml -n ids-2`

- It is recommended to use a visualization tool like [K9s](https://k9scli.io/), with this type of software you can check the correct performance of each component. 
    

![figura](./pictures/pods_running_k9s.png)

- Regarding the ingress manifest, in this repository it has beeb deployed two ways with two technologies (Nginx  [<img src="pictures/img-buildkite/nginx.png" width="20" height="20" alt="traefik"/>](https://www.nginx.com/) and Traefik[<img src="pictures/img-buildkite/Traefik.png" width="30" height="30" alt="traefik"/>](https://doc.traefik.io/traefik/)). Nginx is easier to configure when it is worked on a local machine but in a remote cluster is more difficult to configure and more unstable. For this reason, it is proposed another solution, Traefik. This proxy is more flexible and modern than Nginx, in addition, Traefik includes a dashboard that helps us to supervise the DataSpace pods. 

  - To run with Nginx [<img src="pictures/img-buildkite/nginx.png" width="20" height="20" alt="traefik"/>](https://www.nginx.com/), it is necessary to install Nginx driver to be able to make the calls to the cluster from the outside. If the Ingress-Nginx repo is not updated, in this [link](https://kubernetes.github.io/ingress-nginx/deploy/) the official documentation can be found.

    `kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.2.0/deploy/static/provider/cloud/deploy.yaml`

    And finish deploying the ingress manifest with:
      
    `kubectl apply -f ./Ingress/4-ingress-connection-nginx.yaml -n ids-2`

  - On the other hand, the official web defines Traefik as  [<img src="pictures/img-buildkite/Traefik.png" width="30" height="30" alt="traefik"/>](https://doc.traefik.io/traefik/):


    *"Traefik is an open-source Edge Router that makes publishing your services a fun and easy experience. It receives requests on behalf of your system and finds out which components are responsible for handling them."*
  

    Traefik can be deployed with the next manifests (the order to deploy is important).

    `kubectl apply -f ./traefik/010-crd.yaml`

    `kubectl apply -f ./traefik/015-rbac.yaml`
    
    `kubectl apply -f ./traefik/011-middleware.yaml`
        
    `kubectl apply -f ./traefik/020-pvc.yaml`
    
    `kubectl apply -f ./traefik/030-deployment.yaml`
    
    `kubectl apply -f ./traefik/040-service.yaml`

    In addition, Traefik allows deploying a dashboard to track the health of the proxy and their cluster modules connections. Using the next command is deployed the dashboard. 

    `kubectl apply .f ./trafik/Traefik-Dashboard/4-ingress-dashboard-https-local`

        
  The appearance of this Traefik dashboard is shown below.
  ![figura](./pictures/dashboardv2.png)

  Traefik uses ingress manifests similar to Nginx. But also, It is necessary to create a TCP layer transport, being a specific Traefik component. 
  The IngressRouetes folder contains all components to deploy either [local](./traefik/IngressRoutes/4-ingressroute-local.yaml) or in the [cloud](./traefik/IngressRoutes/4-ingressroute-rancher.yaml)
  

    `kubectl apply -f ./IngressRoutes/4-ingressroute-local.yaml`
    
  To remove Traefik manifests.

    `kubectl delete -f ./traefik/`

    `kubectl delete -f ./traefik/IngressRoutes/`

    `kubectl delete -f ./traefik/Traefik-Dashboard/`

Finally, with a tool like [Postman](https://www.postman.com/) it is possible to test our IDS-testbed.  The way to verify the correct deployment is using [ids-certification-testing](TestbedPreconfiguration.postman_collection.json) file. This file is a set of instructions that allow evaluating the correct operation of the system. 

## IDS deployment in K8S in a cloud machine with an external connector

The steps to deploy in a remote cluster are the same as in local cluster. The only thing that needs to be changed is some configuration.

Traefik proxy is deployed the same as in the local example. The only difference is that for the dashboard, it is necessary to deploy the [4-ingress-dashboard-https-rancher.yaml](./traefik/Traefik-Dashboard/4-ingress-dashboard-https-rancher.yaml), to have a public domain. In this case, the domain is linked to the URL of the European FlexiGroBots project. In addition, it is also necessary to deploy the Ingress resource corresponding to the cluster,[4-ingressroute-rancher.yaml](./traefik/IngressRoutes/4-ingressroute-rancher.yaml).

After that, it has to be deployed the manifests that are in the folder [idsa_manifests_rancher](./idsa_manifests_rancher/). This folder contains the modules that compose the DataSpace configured to work remotely. 

 It is necessary to configure two parts in this stage. The first one is to make public the Omejdn and the Broker-Core-Proxy of our data space. And secondly, it is necessary to configure the external connector to link with the DataSpace. 

In order to make public the Omejdn and Broker-core-proxy it is necessary to create an ingress for each component. For the first ingress we use an URL (omejd-idsa.platform.flexigrobots-h2020.eu) to make public the pod, in this case, this ingress has to open the 80 port and it has to attack Omejdn service. Regarding the Broker-proxy is the same, it is necessary to create an ingress manifest with the 443 port open. Also, it has been added a public URL (broker-reverseproxy.platform.flexigrobots-h2020.eu)

After configuring the way to access Omejdn and broker. There are some parameters that it is necessary to configure them. In particular:
- DAPS_VALIDATE_INCOMING: in the Broker-Core, this value changes from True to False.
  
- OMEJDN_ISSUER: in the omejdn_server,  the value changes from http:/omejdn/auth to http://omejd-idsa.platform.flexigrobots-h2020.eu/auth
- OMEJDN_DOMAIN: the next value is in Omejdn deployment. The value changes http://omejdn to http://omejd-idsa.platform.flexigrobots-h2020.eu
- OMEJDN_ISSUER:  the last variable is the Omejdn-UI.  The value changes OIDC_ISSUER to http://omejd-idsa.platform.flexigrobots-h2020.eu

Finally, to deploy the connector, it is necessary to configure some variables of the connector so it can connect well. All the configuration is in the [docker-compose.yaml](./External_connector/docker-compose.yml), where the port, the DAPS_URL, the DAPS_token and the DAPS_KEY_URL are defined, also the Omejdn address must be added to the whitelist of the port for it to work correctly. 