#####################
Once we create the create the cluster
Check the cluster status by using following commands:
kubectl get nodes
#####################
Then update the kubeconfig to connect to the cluster using command line
- aws eks update-kubeconfig --name demo-cluster-1 --region ap-south-1


Create the alias so that we dont have to use kubectl always by command: alias k=kubectl

k apply -f namespace.yaml

k apply -f deployment.yaml

k apply -f service.yaml

Now if see our svc by then our service is nodeport mode means within cluster we can access the application but from outside we cannot
But as we have deployed the ingress but it wont create the load balancer because there is no ingress controller 


# Now create the load balancer controller using following command:

But first we need to associate iam oidc provider
eksctl utils associate-iam-oidc-provider --cluster demo-cluster --approve

Run following commands to install the nginx ingress controller:

helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx

kubectl create namespace nginx-ingress-sample

helm install my-nginx ingress-nginx/ingress-nginx \
--namespace nginx-ingress-sample \
--set controller.metrics.enabled=true \
--set-string controller.metrics.service.annotations."prometheus\.io/port"="10254" \
--set-string controller.metrics.service.annotations."prometheus\.io/scrape"="true"

kubectl get service -n nginx-ingress-sample
EXTERNAL_IP=your-nginx-controller-external-ip

After this our load balancer controller has been deployed to check:
k get po -n nginx-ingress-sample

All the pods should be in running state

##############################################

Now we need to create the self signed certificate and attach it so that we can access our domain using https:
Follow following steps to achieve that,
sudo apt-get install openssl

Generate the Self-Signed Certificate:
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout demo.local.key \
    -out demo.local.crt \
    -subj "/CN=demo.local"
This command will the 
demo.local.key: This is the private key.
demo.local.crt: This is the self-signed certificate.

After this create the secret :
kubectl create secret tls demo-local-tls \
    --cert=demo.local.crt \
    --key=demo.local.key \
    -n game-2048
# Then Apply the ingress file
k apply -f ingress.yaml

Now we we see our ingress resource by command:
k get ing -n game-2048 > we can see that we have got the load balancer attach to our ingress

Now wait till the load balancer is active

Then run following command:

nslookup a0c6bf6e9752845c48eb0b99060bd0f8-1831895503.ap-south-1.elb.amazonaws.com

We will get 2 ips mark then like follow in the /etc/host file in the local system
3.7.192.252 demo.local
52.66.75.97 demo.local

After this if we try to acceess the url https://demo.local we will get the msg that it is insecure still proceed then
we can access the application

![image](https://github.com/user-attachments/assets/f6fd9f7e-1ab0-4ae0-8967-1493b040f26a)

