 #####################################################      Securing AKS with WAF      ########################################################################

# Today I've finished implementing one of the most complex topics in securing Azure Kubernetes Service, and I'm going to explain how I have done it, in this article, 
# for the hope it benefits someone.

# 1- Create an application gateway with a Tier of WAF_V2 Not Standard_V2
	
# 2- Enable the AGIC [Application Gateway Ingress Controller] add-on in the Kubernetes cluster. This tells the AKS to control the WAF.
#  - If the application gateway is in the same virtual network as the Kubernetes cluster, then complete to the next step.
#  - If the application gateway is NOT in the same virtual network, then peering between the two virtual networks must be created before enabling step two, 
#    or else the application gateway will NOT appear to the Kubernetes cluster.

# 3- When we enable the AGIC, any configuration existed before in the application gateway will be lost, and the only source for configuration will be 
#    automatically configured in the application gateway and it will be unchangeable by manual users.
	
# 4- Create a YAML file for the application such as Angular, backend apps, … etc.
#  - It's important to add the correct annotations, in this case "Kubernetes.io/ingress.class: azure/application-gateway" as shown in the figure above, 
#    otherwise it'll NOT work correctly.
#  - Important notes to consider creating YAML file:-

#   i. Deploy the default type of services [ClusterIP] which will be more secure by not having public IP address like [Load-Balancer] type which 
#      is exposed to the Internet.	
#   ii. Access to the services will be given through creating an Ingress, which in this case will be the IP address of the WAF. 
#       That way all traffic will go through WAF, be tested, and judged whether to be forwarded or to be dropped. And that's the goal of the whole process.

# 5- Deploy the YAML files [Deployment, Service, Ingress, ConfigMap, Secrets, … etc] created in the previous step to the Kubernetes cluster using Kubectl apply.

# 6- After a small bit of time, the configuration of the WAF will be automatically created and controlled by AGIC, and hence, every change manually done 
# will be overwritten by AGIC.
	
# 7- As shown in the figure before, the backend health is healthy, and it returns back code 200 which is ok. 
# It is also important to bring to attention, the IP address of the backend server [AKS] is private, which means traffic between WAF & AKS is going 
# through Microsoft backbone network without being exposed to the Internet. And that's the best practice for securing Kubernetes clusters in Azure.
