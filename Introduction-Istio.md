# Istio-ServiceMesh
# What is Service Mesh
Service mesh help you with the traffic management of your Kubernetes cluster, especially east–west (internal cluster traffic) traffic management of your Kubernetes cluster.

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
now wouldn't it make more sense to extract all non business logic out of the microservices and into small ``sidecart`` application that handles all these logic and acs as proxy, and this small application is a third party application the cluster operators can easily configure through an API without worried about how the logic is implemented. and developers can focus on actual business logic.
and note that you don't have to add this sidecart configuration to your microservice deployment.yaml, because Service Mesh has a controll plan that automatically inject this proxy in every microservice's pod. and now this microservice can talk each other through those proxies (envoy).
and the network layer for service to service communication consisting of controll plane and the proxy is a Service Mesh.

