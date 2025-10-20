# Istio-ServiceMesh
# What is Service Mesh
Service mesh help you with the traffic management of your Kubernetes cluster, especially east–west (internal cluster traffic) traffic management of your Kubernetes cluster.

<img width="1303" height="636" alt="Image" src="https://github.com/user-attachments/assets/48dbeeeb-9235-4808-9262-95e0504b8bdb" />

# Challenge in Microservices
## communication configuration
when we move to microservice we introduce a new challenge that we did not have in monolith architecture. 
Let's say we have an e-commerce application which is made up with several microservices. we have a web server that get request, payment application, shopping cart, Database and more service. and we are deploying microservice applcation into kubernetes cluster
What does our microservices application setup need to run successfully or what are some of the required configurations for the apps. this microservices need to talk each other such as : when user put stuff in shoping cart request is receive by ``webserver`` which hand it over to the ``shopping cart`` then it will talk to ``database`` to persist the data. so How does services know how to talk each other what is the end point of this service, all the service endpoints that web server talks to must be configured for web server. 
so when we add a new service we need to add the endpoint of that new service to all the microservices that need to talk to it. so we have that information as part of application deployment code. 

## Security
now how about security in our microservice application setup. generally a common environment in many project will look like this: 
you have firewall rules setup for kubernetes cluster might be you have proxy as an entry point that get request first so cluster can't be access directly. so you have security around cluster. 
However once request inside cluster in communication is insecure, microservices talk to each other through http or insecure protocol. also microservice talk each other freely, there are no restrictions on that, this case in security perspective when attackers gets inside cluster it can do anything because we don't have addtional security inside.
again additional configuration inside each application is needed to secure communication within the cluster.

## retry logic
you also need retry logic in each microservice to make the whole application more robust. if one microservice unreachable or you lose connectivity for a bit so you want to retry the connection, developer will add this retry logic also to the services 

## observability
you want to be able to monitor how this services out performing, what's http error are you getting, how many request are your microservices are receiving. so development team may add monitoring logic for prometheus, and tracing data using jaeger 

as you see teams of developers of each microservice need to add all this logic to each service and may be to configure some additional stuff in the cluster to handle all this important challenges in microservice application. and this means developer are not working on the actual service logic but are busy adding network logic for metric and security, etc. for each microservice which also add complexity to the services instead of keeping them simple and lightweight.

# Solution
Istio can solve this problem by adding a ``sidecar`` inside this sidecar container has a envoy proxy. just like proxy server and this proxy can handle the traffic management of your pods, that means any request that is coming to your pods and any request that is going outside your kubernetes pods will go through this ``sidecar``. the sidecar container is installed in each an every pods in kubernetes cluster.
example e-commerce app:
when catalog service try to initiate API calls to the payment service, request will be taken by sidecar container and it will add a certificate or it will initiate a tls. so the API call will be intercepted by sidecar so it goes to sc then from there it will go to sc of payment then this sc will add and the sc of payment intercept the request then it goes to payment app and it will try to verity the certificate from catalog along with verifying the certificate it will also display its tls certificate to the catalog service, this is the ``istio`` and ``mTLS`` concept both catalog and payment trust each other only if they have the valid certificate. so both of inbound and outbound traffic are intercept by sidecar container.

this sidecar container implement mTLS, canary deployment, circuit braker, and observability.

and note that you don't have to add this sidecart configuration to your microservice deployment.yaml, because Service Mesh has a controll plan that automatically inject this proxy in every microservice's pod. and now this microservice can talk each other through those proxies (envoy).

# How does the communication between Istio and API server?
for that Istio use a concept called as ``admission controller`` which is called dynamic admission controll or admission webhook, example concept of admission cotroll: there's a users who try to create a pod then request will go to API server, basically there's a component in this API server which will basically verify the user is authenticated or authorized to perform this actio or not. if the API has verified this, then it will persist in the object in ETCD.
Admission controller come in the second step where it can intercept before the request go to ETCD.

<img width="639" height="638" alt="Image" src="https://github.com/user-attachments/assets/458cede4-b91d-4d21-b003-7a8221643c13" />

so before API server create those object in ETCD, admission controller can mutate (modify) or validate (verify few thing in pod or any resources thas is getting created).

for example: you want to create a ``pvc ``(persistent volume claim) and you didn't add a ``storage class`` in the pvc resource, so the request go to API you have authentication and authorization, then API server tries to add the resource to ETCD, before it added, there is an admission controller that is called storage class admission controller It will see if the ``pvc`` has the storage field or not. If it doesn't has it will mutate the object, in the object it will add the field to your ``pvc`` creation request, mutation admission controller will add a new field and this field called as storage class then the object persisted in ETCD

there are 30+ admission controller and you don't need to install them because they are pre-compiled into the API server. for all 30+ admission controller you can just enable or disable them