---
title: "Ingress Settings"
url: /developerportal/deploy/private-cloud-ingress-settings/
description: "Describes how to set up and configuress various ingresses in Private Cloud"
weight: 10
#To update these screenshots, you can log in with credentials detailed in How to Update Screenshots Using Team Apps.
---

## Introduction

A Kubernetes Ingress is an API object that shows how traffic from the internet should reach internal Kubernetes cluster Services that send requests to groups of Pods. 
In this documentation, we will give a walkthrough on various Ingress types namely kubernetes-ingress, Openshift routes and service-only. This documentation will also provide you information on how to set up various Ingress controllers namely NGINX, AWS Load Balancer Ingress Controller, and Application Gateway Ingress Controller (AGIC) for Azure Kubernetes Service (AKS).

When switching between Ingress, OpenShift Routes, and Service Only, you need to restart the Mendix Operator for the changes to be fully applied.
Additional network options such as Ingress/Service annotations and Service ports are available in advanced network settings.

{{% alert color="warning" %}}
We strongly recommend using the NGINX Ingress Controller, even if other Ingress controllers or OpenShift Routes are available. You may need to check which of the several versions of the NGINX Ingress Controller is installed in your cluster. Mendix recommends the “community version”.

NGINX Ingress can be used to deny access to sensitive URLs, add HTTP headers, enable compression, and cache static content. NGINX Ingress is fully compatible with cert-manager, removing the need to manually manage TLS certificates. In addition, NGINX Ingress can use a Linkerd Service Mesh to encrypt network traffic between the Ingress Controller and the Pod running a Mendix app.

These features will likely be required once your application is ready for production.
{{% /alert %}}

Below are the Ingress types currently supported in Private Cloud:
1. [kubernetes-ingress](/developerportal/deploy/private-cloud-ingress-settings/#ingress-controllers) will configure ingress according to the additional domain name you supply. This option allows you to configure the ingress path and custom ingress class (dependent on the Ingress controller) and enable or disable TLS.
2. [openshift-routes](/developerportal/deploy/private-cloud-ingress-settings/#openshift-routes) will configure an OpenShift Route. This can only be used for OpenShift clusters. This option allows you to enable or disable TLS.
3. [service-only](/developerportal/deploy/private-cloud-ingress-settings/#service-only) will create just a Kubernetes Service, without an Ingress or OpenShift route. This option enables you to use a Load Balancer without an Ingress, or to manually create and manage the Ingress object (an Ingress that is not managed by Mendix for Private Cloud).

{{% alert color="info" %}}
When switching between Ingress, OpenShift Routes, and Service Only, you need to restart the Mendix Operator for the changes to be fully applied.
Additional network options such as Ingress/Service annotations and Service ports are available in advanced network settings.
{{% /alert %}}

## Ingress Controllers{#ingress-controllers}

Kubernetes Ingress resource is a configuration request for the ingress controller that allows the user to define how external clients are routed to a cluster’s internal Services. When you deploy an Ingress resource, the Ingress controller watches for these resources and configures the underlying proxy or load balancer accordingly.
For example, if you're using the NGINX Ingress Controller, it will update the NGINX configuration to reflect the rules defined in your Ingress resource.

An ingress controller acts as a reverse proxy and load balancer. It implements a Kubernetes Ingress. The ingress controller adds a layer of abstraction to traffic routing, accepting traffic from outside the Kubernetes platform and load balancing it to Pods running inside the platform. It converts configurations from Ingress resources into routing rules that reverse proxies can recognize and implement.

Below are the various Ingress Controller supported in Private Cloud:

* [NGINX Ingress Controller](https://kubernetes.github.io/ingress-nginx/)
* [Traefik](https://traefik.io/traefik/)
* [AWS Application Load Balancer](https://docs.aws.amazon.com/eks/latest/userguide/alb-ingress.html)
* [Ingress for External Application Load Balancer](https://cloud.google.com/kubernetes-engine/docs/concepts/ingress-xlb)
* [Azure Application Gateway Ingress Controller](https://learn.microsoft.com/en-us/azure/application-gateway/ingress-controller-overview)

For ingress, below functionalities are currently supported:

1. Turn TLS on and off
2. Add ingress annotations
3. Add service annotations
4. Specify the ingress class, path and path type
5. Provide the name of an existing TLS secret to use
6. Provide a domain name (for example, mendix.example.com)

{{% alert color="info" %}}
k8s ingress controllers don’t accept URLs, the URL needs to be split in multiple fragments (domain, path, path suffix) and the HTTP/HTTPS paths need to be configured completely separately
{{% /alert %}}

For each environment, the URL will be automatically generated based on the domain name. For example, if the domain name is set to mendix.example.com, then apps will have URLs such as myapp1-dev.mendix.example.com, myapp1-prod.mendix.example.com and so on.

The DNS server should be configured to route all subdomains (the * subdomain, for example, *.mendix.example.com)*. to the ingress/load balancer.

It is also possible to provide a custom TLS configuration for individual environments, overriding the default configuration (only available in Standalone Mendix Operator installations):

* Turn TLS on and off
* Specify the name of an existing TLS certificate secret to use
* Provide the TLS Certificate and Private Key values directly in the environment specification
* There are multiple ways of managing TLS certificates:

* The Ingress controller can have a default certificate with a wildcard domain (for example, *.mendix.example.com)*.. For Ingress controllers which support for [Let’s Encrypt](https://letsencrypt.org/), the Ingress controller can also request and manage TLS certificates automatically.
* Providing a TLS certificate secret for each environment.
* Using [cert-manager](https://cert-manager.io/) or a similar solution by using Ingress annotations. This service can be used to automatically request TLS certificates and create secrets for the Ingress controller.

Starting from Mendix Operator v1.11.0, Mendix app environments can use a [Linkerd](https://linkerd.io/) Service Mesh. Linkerd can be used to monitor and re-encrypt HTTP (or HTTPs) traffic between the Ingress Controller and the Pod running a Mendix app.


### NGINX Ingress Controller Overview

The NGINX Ingress Controller is an open-source solution that uses NGINX as a reverse proxy and load balancer to manage ingress resources in Kubernetes. It provides advanced features like URL routing, SSL termination, and WebSocket support.

NGINX ingress controller is compatible with all the Cloud Providers and K8s flavours Mx4PC supports.

For NGINX Ingress, it’s possible to set headers by providing a default configuration snippet in [OperatorConfiguration](https://docs.mendix.com/developerportal/deploy/private-cloud-cluster/#advanced-network-settings)(e.g. globally for all apps in a namespace), or specifying the nginx.ingress.kubernetes.io/configuration-snippet annotation in Portunus.

#### Installing NGINX

Helm is the recommended way to installing nginx. You can download it from [here](https://helm.sh/docs/intro/install/)

Official procedure to install nginx with manifest is published [here](https://docs.nginx.com/nginx-ingress-controller/installation/installation-with-manifests/) and with helm [here](https://docs.nginx.com/nginx-ingress-controller/installation/installation-with-helm/).

To install it in one go, in any K8s cluster,  just copy and paste below command:

```bash
helm repo add nginx-stable https://helm.nginx.com/stable
helm repo update
helm install nginx nginx-stable/nginx-ingress
```

#### Configuring NGINX in mxpc-cli

1. Select Ingress Type as kubernetes-ingress. **kubernetes-ingress** will configure ingress according to the additional domain name you supply. 

2. Ingress Domain Name - provide the domain name which you want to set for the Ingress resource file

3. Ingress Path -  its optional, which can be used to specify the Ingress path; default value is **/** 

{{% alert color="info" %}}
For Operator version 2.19.0 and Mendix version 10.3.0 onwards, NGINX path based routing is supported. A new option /(.*) in the ingress path is provided which sets the path prefix to support this feature. To support this feature, NGINX Ingress uses nginx.ingress.kubernetes.io/rewrite-target/(.*)
{{% /alert %}}

4. Enable TLS - allows you to enable or disable TLS for the Mendix App’s Ingress

5. Custom Ingress Class - enable this option for providing the ingress class name.

6. Ingress Class Name - provide **nginx** as the ingress class name 

7. Set Ingress Class as Annotation - Unselect this option. This option adds the kubernetes.io/ingress.class annotation to set the ingress class.

{{< figure src="/attachments/deployment/private-cloud/private-cloud-ingress/nginx-configuration.png" class="no-border" >}}


### AWS Load Balancer Ingress Controller Overview

The AWS Load Balancer Ingress Controller integrates with Amazon’s Application Load Balancer (ALB) or Network Load Balancer (NLB) to provide ingress capabilities. It is designed specifically for EKS but can be configured for any Kubernetes cluster running in AWS. 

It satisfies [Kubernetes Ingress resources](https://kubernetes.io/docs/concepts/services-networking/ingress/) by provisioning [Application Load Balancers](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html).

{{% alert color="info" %}}
[AWS Load Balancer Controller](https://kubernetes-sigs.github.io/aws-load-balancer-controller/v2.2/) is the AWS-recommended way to provide ingress capability on EKS. 
{{% /alert %}}

{{% alert color="warning" %}}
Its not possible to set any headers in ALB. This can only be done by putting [cloud front](https://aws.amazon.com/cloudfront/) in front of ALB.
{{% /alert %}}


#### Install the aws-load-balancer-controller:

{{% alert color="info" %}}
To run ALB there are some prerequisites, like AWS Load Balancer Controller deployed on your cluster and at least two subnets in different Availability Zones (more details [here](https://docs.aws.amazon.com/eks/latest/userguide/alb-ingress.html#_prerequisites)).
{{% /alert %}}


1. Deploy using Helm{#alb-helmchart}

```bash
helm repo add eks https://aws.github.io/eks-charts;kubectl apply -k "github.com/aws/eks-charts/stable/aws-load-balancer-controller//crds?ref=master"
helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=<cluster-name>
```

#### Configuring ALB Configuration in mxpc-cli

1. Select Ingress Type as kubernetes-ingress. **kubernetes-ingress** will configure ingress according to the additional domain name you supply. 

2. Ingress Domain Name - provide the domain name which was registered for ALB

3. Ingress Path -  Set it to **/***

4. Enable TLS - allows you to enable or disable TLS for the Mendix App’s ingress.

5. Custom Ingress Class - enable this option for providing the ingress class name.

6. Ingress Class Name - provide **alb** as the ingress class name 

7. Set Ingress Class as Annotation - Select this option. This option adds the kubernetes.io/ingress.class annotation to set the ingress class.

{{< figure src="/attachments/deployment/private-cloud/private-cloud-ingress/alb-configuration.png" class="no-border" >}}

8. Update the Operator configuration and set below ALB Specific annotations under Ingress section.

```bash
kubernetes.io/ingress.class: alb 
alb.ingress.kubernetes.io/scheme: internet-facing 
alb.ingress.kubernetes.io/target-type: ip # 'ip' mode will route traffic directly to the pod IP; 'instance' mode will route traffic to all ec2 instances within cluster on NodePort opened for your service. 
alb.ingress.kubernetes.io/subnets: subnet-value1, subnet-value 2 # optional field, if you added tags to the subnets (details here - https://docs.aws.amazon.com/eks/latest/userguide/alb-ingress.html). This subnets for des-non-prod cluster.
alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:eu-west-1:908971519767:certificate/5e9fbc51-9614-4ec9-90a1-f58dbab1b4e4 # enables tls; sets certificate from ACM.
```

Below is an example yaml for the Operator Configuration when using AWS Load balancer

**ALB Ingress Example:**

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/subnets: subnet-0aagceel11d431b269, subnet-078993d64425e96767
    alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:eu-west-1:908971519767:certificate/5e9fbc51-9614-4ec9-90a1-f58dbab1b4e4
  generation: 1
  labels:
    app: aws
  name: aws
  namespace: mxpcns
  resourceVersion: "218900618"
  selfLink: /apis/extensions/v1beta1/namespaces/anton-aws/ingresses/aws
  uid: 8a67352e-c656-444f-bb66-323463e79199
spec:
  rules:
  - http:
      paths:
      - backend:
          serviceName: aws
          servicePort: mendix-app
        path: /*
```

### AKS Application Gateway Ingress Controller (AGIC) Overview


The AKS Application Gateway Ingress Controller (AGIC) is a specialized ingress controller for managing HTTP and HTTPS traffic in Azure Kubernetes Service (AKS) environments. It leverages Azure Application Gateway, a Layer 7 load balancer, to route traffic to services running in an AKS cluster. AGIC monitors the Kubernetes cluster it is hosted on and continuously updates an Application Gateway, so that selected services are exposed to the Internet. This integration offers advanced features and seamless Azure-native capabilities for managing ingress in Kubernetes.

The Ingress Controller runs in its own pod on the customer’s AKS. AGIC monitors a subset of Kubernetes Resources for changes. The state of the AKS cluster is translated to Application Gateway specific configuration and applied to the Azure Resource Manager (ARM).

{{% alert color="info" %}}
Azure Gateway Ingress Controller needs up to 90 seconds to remove a pod from its routing table. Stopping an app pod immediately would still send traffic to the pod for a few minutes, causing random 502 errors to appear in the client web browser. Hence, its recommended to add [runtimeTerminationDelaySeconds](/developerportal/deploy/private-cloud-cluster/#termination-delay) value to the OperatorConfiguration CR.
{{% /alert %}}

#### Install the AKS Application Gateway Ingress Controller:

Refer [this](https://learn.microsoft.com/en-us/azure/application-gateway/tutorial-ingress-controller-add-on-existing#enable-the-agic-add-on-in-existing-aks-cluster-through-azure-cli) document to install AKS Application Gateway Ingress Controller

#### Configuring AGIC Configuration in mxpc-cli

1. Select Ingress Type as kubernetes-ingress. **kubernetes-ingress** will configure ingress according to the additional domain name you supply. 

2. Ingress Domain Name - provide the domain name which was registered for AGIC

3. Ingress Path -  Set it to **/**

4. Enable TLS - allows you to enable or disable TLS for the Mendix App’s ingress.

5. Custom Ingress Class - enable this option for providing the ingress class name.

6. Ingress Class Name - provide **azure/application-gateway** as the ingress class name 

7. Set Ingress Class as Annotation - Select this option. This option adds the kubernetes.io/ingress.class annotation to set the ingress class.

{{< figure src="/attachments/deployment/private-cloud/private-cloud-ingress/agic-configuration.png" class="no-border" >}}

To complete AGIC ingress configuration, set additional ingress annotations:

```bash
NAMESPACE=<operator namespace>
kubectl patch operatorconfiguration mendix-operator-configuration -n $NAMESPACE --type=merge -p '{"spec":{"endpoint":{"ingress":{"annotations":{"appgw.ingress.kubernetes.io/appgw-ssl-certificate":"agic-tls","appgw.ingress.kubernetes.io/ssl-redirect":"true"}}}}}'
```

If you would also like to set up SSL certificates, kindly follow [SSL certificate](https://azure.github.io/application-gateway-kubernetes-ingress/features/appgw-ssl-certificate/) documentation which explains how to make AGIC load a cert from KeyVault.

### Traefik Ingress Controller Overview

Traefik is a cloud-native reverse proxy and a load balancer. When deployed as an ingress controller in Kubernetes, it manages HTTP and HTTPS traffic to services running within the cluster. It automatically discovers services using Kubernetes' native APIs, based on Kubernetes Ingress resources and other configurations. One of the main advantages of using Traefik is its built in [Let's Encrypt support](https://doc.traefik.io/traefik/https/acme/)

Official documentation to install is provided [here](https://doc.traefik.io/traefik/providers/kubernetes-ingress/)

{{% alert color="warning" %}}
Traefik uses 2 types of providers: CRDs or Kubernetes Ingress. Please make sure you install Kubernetes Ingress one as it’s the only one supported by Mx4PC.
{{% /alert %}}


#### Configuring Traefik Ingress Controller in mxpc-cli

1. Select Ingress Type as kubernetes-ingress. **kubernetes-ingress** will configure ingress according to the additional domain name you supply. 

2. Ingress Domain Name - provide the domain name which was registered for ALB

3. Ingress Path -  Set it to **/**

4. Enable TLS - allows you to enable or disable TLS for the Mendix App’s ingress.

5. Custom Ingress Class - enable this option for providing the ingress class name.

6. Ingress Class Name - provide **traefik** as the ingress class name 

7. Set Ingress Class as Annotation - Select this option. This option adds the kubernetes.io/ingress.class annotation to set the ingress class.


## Openshift Route{#openshift-routes}

OpenShift supports both Routes and Ingress. The OpenShift IngressController acts as a bridge, managing both Routes and Ingress resources. This gives flexibility for using either approach based on specific requirements or familiarity.

Openshift routes are only supported in Openshift.

The only configuration option currently supported is turning TLS on or off. When TLS is turned on, Edge termination (where TLS termination occurs at the router, before the traffic gets routed to the pods) will be used, with automatic redirection from HTTP to HTTPS.

The following configuration options are available in OpenShift:

* Turn TLS on and off
* Add route annotations
* Provide the name of an existing TLS certificate secret to use instead of the default router certificate
* Provide a custom domain name (for example, mendix.example.com) to use instead of the default OpenShift route domain

It is also possible to provide a custom TLS configuration for individual environments, overriding the default configuration (only available in Standalone Mendix Operator installations):

* Turn TLS on and off
* Specify the name of an existing TLS certificate secret to use
* Provide the TLS Certificate and Private Key values directly in the environment specification

To use Ingress on OpenShift:

1. Ensure that the OpenShift IngressController is deployed.
2. Define your Ingress resources as per Kubernetes standards.
3. Configure annotations specific to OpenShift (if needed) for enhanced behavior.

{{% alert color="info" %}}
Its advisable to use OpenShift Routes when working exclusively within OpenShift and leveraging its built-in router features, and use Kubernetes Ingress when portability or specific controller features are required.
{{% /alert %}}

{{% alert color="info" %}}
For Operator version 2.19.0 and Mendix version 10.3.0 onwards, NGINX path based routing is supported. A new option /(.*) in the ingress path is provided which sets the path prefix to support this feature. To support this feature, OpenShift route uses haproxy.router.openshift.io/rewrite-target./(.*)
{{% /alert %}}

The OperatorConfiguration contains user-editable options for Openshift routes for network endpoints.

Below is an example yaml file when using OpenShift Routes for Network Endpoints:

```yaml
apiVersion: privatecloud.mendix.com/v1alpha1
kind: OperatorConfiguration
spec:
  # Endpoint (Network) configuration
  endpoint:
    # Endpoint type: ingress, openshiftRoute, or service
    type: openshiftRoute
    # OpenShift Route configuration: used only when type is set to openshiftRoute
    openshiftRoute:
      # Optional, can be omitted: annotations which should be applied to all Ingress Resources
      annotations:
        haproxy.router.openshift.io/hsts_header: max-age=31536000;includeSubDomains;preload
      # Optional: App URLs will be generated for subdomains of this domain, unless an app is using a custom appURL
      domain: mendix.example.com
      # Enable or disable TLS
      enableTLS: true
      # Optional: name of a kubernetes.io/tls secret containing the TLS certificate
      tlsSecretName: 'mendixapps-tls'
```    


## Service Only{#service-only}

Mendix for Private Cloud can create Services without an Ingress. In this way, the Ingress objects can be managed separately from Mendix for Private Cloud.

Mendix for Private Cloud can create Services that are compatible with:

* [Istio](https://istio.io/)
* [Linkerd](https://linkerd.io/)

The OperatorConfiguration contains user-editable options for Services for network endpoints.

Below is an example yaml file when using Services for Network Endpoints:

```yaml

apiVersion: privatecloud.mendix.com/v1alpha1
kind: OperatorConfiguration
spec:
  # Endpoint (Network) configuration
  endpoint:
    # Endpoint type: ingress, openshiftRoute, or service
    type: service
    # Optional, can be omitted: the Service type
    serviceType: LoadBalancer
    # Optional, can be omitted: Service annotations
    serviceAnnotations:
      # example: annotations required for AWS NLB
      service.beta.kubernetes.io/aws-load-balancer-type: external
      service.beta.kubernetes.io/aws-load-balancer-nlb-target-type: ip
      service.beta.kubernetes.io/aws-load-balancer-scheme: internet-facing
    # Optional, can be omitted: Service ports
    servicePorts:
      - 80
      - 443
```

## Istio Service Mesh and Linkerd

Mendix only supports unencrypted HTTP between the ingress controller and the app. However, there is no higher level of security with service-to-service encryption and policy controls. In such situation, Integrating Ingress controllers with Istio Service Mesh/Linkerd can help you manage both external traffic entering your Kubernetes cluster (via Ingress Controller) and internal traffic between services (handled by Istio/linkerd). 

Istio service Mesh/Linkerd: helps to manage service-to-service communication within a Kubernetes cluster. It provides features such as traffic management (e.g., canary releases), service discovery, load balancing, security (e.g., mutual TLS), and observability (e.g., metrics and tracing).

In an Istio/linkerd enabled Kubernetes cluster, Ingress controller can be used as the external entry point for HTTP(S) traffic. Once the traffic reaches the Ingress controller, it can be forwarded to the Istio Ingress Gateway, which acts as the entry into the Istio service mesh. In case of linkerd, if Linkerd is enabled, each service is "sidecar injected" with a Linkerd proxy (a lightweight data plane proxy running alongside the application container in the pod).


{{% alert color="info" %}}
If you are facing Network error with using Service mesh, check [documentation](/developerportal/deploy/private-cloud-deploy/#network-errors-when-using-an-istio-service-mesh)
{{% /alert %}}

### Istio Service Mesh integration with Ingress Controller

1. Follow [documentation](https://istio.io/latest/docs/setup/install/helm/) to install istio

2. Once the installation is done, enable istio ingress for the namespace where application is deployed.

```bash
kubectl label namespace <name> istio-injection=enabled --overwrite
```
3. Once the Service mesh is installed, deploy the Ingress Controller of your choice.

4. Following this, Istio Ingress Gateway needs to be deployed. Istio's Ingress Gateway handles incoming traffic and applies service mesh rules. Enable the Istio Ingress Gateway by default during installation or deploy it separately.

5. Configure Ingress Controller to Forward to Istio Ingress Gateway

6. Create Gateway and VirtualService in Istio

6.1. Create Gateway: Configure a Gateway resource to allow traffic through the ingress gateway.

6.2. Create VirtualService: Define a VirtualService to route traffic from the gateway to a service in the mesh.

#### Configure Istio Service mesh in mxpc-cli

1. Select Ingress Type as kubernetes-ingress. **kubernetes-ingress** will configure ingress according to the additional domain name you supply. 

2. Ingress Domain Name - provide the domain name which was configured for Istio

3. Ingress Path -  Set it to **/***

4. Enable TLS - allows you to enable or disable TLS for the Mendix App’s ingress.

5. Custom Ingress Class - enable this option for providing the ingress class name.

6. Ingress Class Name - provide **istio** as the ingress class name

{{% alert color="info" %}}
AWS ALB and AGIC only work with Istio.
{{% /alert %}}

### Linkerd Installation

1. Follow [documentation](https://linkerd.io/2.17/getting-started/) to install linkerd

2. Exclude the NGINX Ingress Controller namespace from Linkerd injection by labeling it like below:

```bash
kubectl label namespace ingress-nginx linkerd.io/inject=disabled
```

3. Now annotate the namespace where your application is deployed with below command:

```bash
kubectl annotate namespace linkerd.io/inject=enabled
```

#### Configure linkerd  ingress in mxpc-cli

1. Select Ingress Type as kubernetes-ingress. **kubernetes-ingress** will configure ingress according to the additional domain name you supply. 

2. Ingress Domain Name - provide the domain name which was configured for linkerd

3. Ingress Path -  Set it to **/**

4. Enable TLS - allows you to enable or disable TLS for the Mendix App’s ingress.

5. Custom Ingress Class - enable this option for providing the ingress class name.

6. Ingress Class Name - provide **nginx** as the ingress class name


## Known Issues:

1. ALB doesn't work properly with HTTP2 WebSockets: https://forums.aws.amazon.com/thread.jspa?threadID=332847

Workaround: use HTTP1 as the ingress backend protocol: alb.ingress.kubernetes.io/backend-protocol-version: HTTP1.

Some ALB firewall rules can block file uploads or other Mx app features.

2. Linkerd does not work with AWS ALB and AGIC.


## Conclusion
Each ingress controller offers unique features suitable for specific environments and requirements. Choosing the right one depends on your cloud provider, workload needs, and operational goals. In private cloud setups, ensure all resources are internal-facing and securely configured for compliance with your organization’s standards.
