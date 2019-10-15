[![PKS Logo][pks-image]][pks-docs]



# Current ENV

This document is a high level overview of the environment and the technologies used.
Setting up and use of the tools is documented in detail in [ArrigoCI.md][arrigo-ci]
All Titles link to more docs on the subject. 

- [Pivotal Container Service][#PKS]
- [Kubernetes](#Kubernetes)
  - [Harbor](#Harbor)
  - [Docker](#Docker)
- [TeamCity](#TeamCity)
  - [Git](#Git)
  - [Build](#Build)



## [PKS][pks-docs]

VMware's take on Clusters using Kubernetes -> Ask Magnus




## [Kubernetes][k8s-docs]

The main focus here is explaining the [structure][k8s-img] of K8s and the specification we adopted in our usage.
The aim is that further details will be more easily understood during usage of K8s.

#### ENV Structure

##### Namespaces

Kubernetes uses namespaces to organise cluster specific resources.
When a deployment or other resource is created, it's always created under a specific or default namespace.
This simplifies naming schemes and allows for more organisation as well as security with RBAC.
It does however require extra caution when managing them since deleting one deletes all the resources belonging to a namespace.

##### Deployments

Our main interest in K8s when managing containers will be in deployments.
A deployment is effectively an automatic manager of another resource **Pods**
The deployments are defined using YAML and the definition require you to specify how pods should be managed, update behaviour, scaling of pods, labelling, starting new containers if one fails and so on.

##### Pods

These are the the basics K8s, they can be described as simple as a container, or a few containers dependent on each other.
Imagine if ArrigoAPI needed a Redis cache in front of it, an easy way to make sure it always had would be to tell a deployment that a pod with the ArrigoAPI also needs a Redis container.
A big part of clusters is labelling your resources adequately, the reason is that other resources in K8s use labels to distinguish between the resources.

##### Services

When we want to access our containers, we need to tell K8s how that is supposed to happen, this is where Services are used. Define which labels should have traffic routed to them from this service, whether that traffic should be load balanced internally or have direct routes using ports managed by K8s, [svc docs][pks-svc-docs]

##### Secrets

This is the last highly relevant resource for managing K8s, this is basically encrypted Key-Value-Pairs that you can use in containers. 
Currently Arrigo uses 2 secrets to supply ENV variables to the container when it's starting up, 1 being the DB connection string and the the other being the salt for JWT.

### Harbor

Rather than using the public Dockerhub we use the more secure and locally hosted **Harbor** for storing and serving images. 
In order to use Harbor we need to change how we tag containers.
The tag is structured so that Docker can find where to push the container and allow Harbor to host it in the right repository and project. 

Example: `FQDN/Project/Repository:Version`
Current:  `Harbor.rssoftware.se/arrigo/arrigo-api-test:1.0.XX`

When pushing the container, make sure that the Harbor package registry is registered in docker as described in [ArrigoCI.md][arrigo-ci] 


### [Docker][docker]

It's now only used to build and push the images used by K8s.
Read [Everything Docker.md][docker] if necessary.




## [TeamCity

It's the CI of choice in RSS.
The build configuration for ArrigoAPI was made with "Fail fast and fail often" in mind. 
This basically means that the slowest operations are behind as many checks as possible so that if there is an error, it's discovered before a 2 minute long docker container build process.

The goal of the TeamCity configuration is to make is easily configurable as possible and utilising the parameters that are supplied by TeamCity.

###  Git

Any build is triggered by changes on a specific branch in the repository.
The branches that trigger builds should be prefixed with **CI/**
The suffix should be **-test** /**-prod** /**-beta** So as to avoid confusion to which one is being updated. 

###  Build 


The build consists of 9 steps:
1. Find and replace the version in the *package.json* with the build number.
2. Set the build version to the version in the *package.json*
3. Install production only dependencies in *prod_modules*
4. Install all dependencies in node_modules
5. "Compile": check for Syntax errors and Transpile to JS.
6. Build the Docker container according to the above [specs](#Harbor).
7. Login to Harbor.
8. Push the container to Harbor.
9. Tell Kubernetes to update the deployment with the new image.



[k8s-docs]: <https://docs.pivotal.io/pks/1-5/create-cluster.html>
[k8s-img]: <K8s-Capture.png>
[arrigo-ci]: <ArrigoCI.md>
[docker]: <Everything-Docker.md>
[pks-image]: <https://assets.brandfolder.com/pj2oej-6w2u14-1lospv/view@2x.png?v=1543692135>
[pks-docs]: <https://docs.pivotal.io/pks/1-5/index.html>
[pks-svc-docs]: <https://docs.pivotal.io/pks/1-5/about-lb.html>