# Oauth2 on k8 Using Nginx Ingress

Recently i was tasked with figuring this out for an internal application using our corp IDP. I usually start these things by using tutorials online to get my feet wet and gain a basic understanding. The majority of the tutorials I found were using GH to create an ouath application to handle the oauth part. What was missing is actually making this all work, there were holes regarding how to handle ingress correctly and the tutorials didnt include issuing certs which turned out to be a tricky combination to get things working. So here I present to you, my step by step to get this working in your own cluster.

I did this in a Windows 10 VM in Azure running Docker Desktop using my own DNS at route53.

## Github
First off, lets create out GH oauth application.

https://github.com/settings/applications/new

For the sake of simplicity, name your app whatever suites your fancy. I am using my root domain for both the "Homepage URL and the "Authorization callback URL" that you need to specify.

For reference, is you ever need to edit your ouath app go here:

https://github.com/settings/developers

It can be hard to find.


## Kubernetes
First and foremost, we need ingress to be installed....

kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/mandatory.yaml

kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/provider/cloud-generic.yaml

Now we need cert manager to handle SSL


Create our cert manager namespace

kubectl create namespace cert-manager

install cert manager

kubectl apply -f .\cert-manager.yaml

create an issuer: 

IM USING PROD LE CERTS, BE WARNED, YOU CAN GO OVER YOUR LIMITS IF YOU ARE USING AGAINST A PRODUCTION TLD

kubectl create -f .\prod-issuer.yaml

Ok that is the prerequisites to get Kubernetes ready for us to deploy our application

Lets build the app as a local container:

docker build -t simplehttp:v1 . 

Let's create a pod and service from our container:

kubectl apply -f .\deployment.yaml

Next you will need to edit ingress.yaml, change each - host: reference to match your domain. You will also need to edit the secrets you retrieved when you created your GH application. For the "OAUTH2_PROXY_COOKIE_SECRET" run this (there are docker containers for this if you do not have python installed, google it):

``` 
OAUTH2_PROXY_COOKIE_SECRET with value of python -c 'import os,base64; print base64.b64encode(os.urandom(16))' 
```

Once you are done, apply the ingress:

kubectl apply -f .\ingress.yaml

thats it, naviageting to your domain will take you to GH sign in to authorize yourself and redirect back to the index page. 

