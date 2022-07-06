# Securing Kubernetes cluster with WAF


This article shows you how to secure your Azure Kubernetes cluster [AKS] by implementing a WAF to manage traffic to AKS. This is an excellent practice to implement.


**1- Create an application gateway with a Tier of WAF_V2 Not Standard_V2**

If you want to know how to create WAF Tier WAF_V2 through Azure CLI, you can go to this link.
	



**2- Enable the AGIC [Application Gateway Ingress Controller] add-on in the Kubernetes cluster. This tells the AKS to control the WAF.**
	
	appgwId=$(az network application-gateway show -n waf-aks -g rg-privateendpoint-uae -o tsv --query "id") 
	az aks enable-addons -n aks-privateendpoint-tst-uae -g rg-privateendpoint-uae -a ingress-appgw --appgw-id $appgwId

Note:-
- If the application gateway is NOT in the same virtual network, then peering between the two virtual networks must be created before enabling step two, or else the application gateway will NOT appear to the AGIC.




**Note:-**

When we enable the AGIC, any configuration existed before in the application gateway such as [Routing rules, Listeners, backend pools] will be lost, and the only source for the WAF configuration will be automatically configured in the application gateway and it cannot be changed manually by the user.



**4- Build a YAML file for the application:-**

In here, I'll build a simple Angular application to deploy on Kubernetes cluster.




Important notes to consider creating YAML file:-

**i.** Deploy the type of services [ClusterIP] instead of [Load-Balancer] for two reasons:-

 - First, it'll be more secure because this type of service does not have public IP address exposed to the Internet.
 - Secondly, Cost of ClusterIP is cheaper than Load-balancer since no reserved IP address is required.

**ii.** Access to the services will be given through creating an Ingress, which in this case will be the IP address of the WAF. That way all traffic will go through WAF, be tested, and judged whether to be forwarded or to be dropped. And that's the goal of the whole process.

**iii.** It's important to add the correct annotations in the YAML file, in this case **"Kubernetes.io/ingress.class: azure/application-gateway"** as shown in the following picture, otherwise it'll NOT work correctly.

![This is an image](https://myoctocat.com/assets/images/base-octocat.svg)


**5- Deploy the YAML files [Deployment, Service, Ingress, ConfigMap, Secrets, … etc] created in the previous step to the Kubernetes cluster using Kubectl apply.**


**6- After a small bit of time, the configuration of the WAF will be automatically created and controlled by AGIC, and hence, every change manually done will be overwritten by AGIC.**
	
	
**7- As shown in the figure before, the backend health is healthy, and it returns back code 200 which is ok. It is also important to bring to attention, the IP address of the backend server [AKS] is private, which means traffic between WAF & AKS is going through Microsoft backbone network without being exposed to the Internet. And that's the best practice for securing Kubernetes clusters in Azure.


