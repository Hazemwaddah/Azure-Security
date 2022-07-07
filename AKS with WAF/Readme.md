# Securing Kubernetes cluster with WAF.


This article shows you how to secure your Azure Kubernetes cluster [AKS] by implementing a WAF to manage traffic to AKS. This is an excellent practice to implement.


### **Create an application gateway.**

The WAF type should be of tier WAF_V2 not Standard_V2 because this is how we get to create the WAF policy to filter legitimate traffic from malicious one.

If you want to know how to create WAF Tier WAF_V2 through Azure CLI, you can go to this link.
	



### **Enable the AGIC.**

Enabling AGIC [Application Gateway Ingress Controller] add-on in the Kubernetes cluster tells the AKS to, automatically, control the WAF, create routing rules, listeners for each service, and get the IP address of pods after new pods are created. You can enable AGIC by typing this command in PS terminal:
	
	appgwId=$(az network application-gateway show -n waf-aks -g rg-privateendpoint-uae -o tsv --query "id") 
	az aks enable-addons -n aks-privateendpoint-tst-uae -g rg-privateendpoint-uae -a ingress-appgw --appgw-id $appgwId

or go to **Networking** under Kubernetes cluster and check enable Application Gateway Ingress Controller as in the picture below:

![This is an image](https://github.com/Hazemwaddah/Azure_Security/blob/main/AKS%20with%20WAF/AGIC.png)


#### Note:-
- If the application gateway is NOT in the same virtual network, then peering between the two virtual networks must be created before enabling step two, or else the application gateway will NOT appear to the AGIC.


- When we enable the AGIC, any configuration existed before in the application gateway such as [Routing rules, Listeners, backend pools] will be lost, and the only source for the WAF configuration will be automatically configured in the application gateway and it cannot be changed manually by the user.



### **Build a YAML file for the application.**

In here, I'll build a simple Angular application to deploy on Kubernetes cluster.




Important notes to consider creating YAML file:-

**i.** Deploy the type of services [ClusterIP] instead of [Load-Balancer] for two reasons:-

 - First, it'll be more secure because this type of service does not have public IP address exposed to the Internet.
 - Secondly, Cost of ClusterIP is cheaper than Load-balancer since no reserved IP address is required.

**ii.** Access to the services will be given through creating an Ingress, which in this case will be the IP address of the WAF. That way all traffic will go through WAF, be tested, and judged whether to be forwarded or to be dropped. And that's the goal of the whole process.

**iii.** It's important to add the correct annotations in the YAML file, in this case **"Kubernetes.io/ingress.class: azure/application-gateway"** as shown in the following picture, otherwise it'll NOT work correctly.

![This is an image](https://github.com/Hazemwaddah/Azure_Security/blob/main/AKS%20with%20WAF/Angular_ingress.png)


### **Deploy the YAML files using Kubectl apply.**

For this example, there are three files that need deployment:

i. First file is for creating configmap that contains the configuration of Nginx .conf file [configure HTTPS, TLS version, .crt, .key files].

ii. Second file contains the deployment and service configuration file.

iii. Finally, the third file contains the deployment of the ingress for external IP.

All these files can be deployed in one line command by adding -f before each file: -

	kubectl apply -f nginx-conf.yml -f angular.yml -f angular_ingress.yml
	

### **It works.** 

After a small bit of time, the configuration of the WAF will be automatically created and controlled by AGIC, and hence, every change manually done will be overwritten by AGIC
	
As shown in the figure below, the backend health is healthy, and it returns back code 200 which is ok. It is also important to bring to attention, the IP address of the backend server [AKS] is private, which means traffic between WAF & AKS is going through Microsoft backbone network without being exposed to the Internet. And that's the best practice for securing Kubernetes clusters in Azure.

![This is an image](https://github.com/Hazemwaddah/Azure_Security/blob/main/AKS%20with%20WAF/WAF_backend_health.png)



#### **Final Notes.**
- monitoring the backend health after configuration is complete will show us the backend pools change the IP address, automatically. Since pods crash in Kubernetes clusters all the time, and new pods created with new IP addresses, the AGIC will reflect that in the WAF, without user intervention.
- 
