# openshift-nodejs-demo

Clone the project and build your docker image locally
```bash
git clone https://github.com/jmgarciac/openshift-nodejs-demo.git
cd openshift-nodejs-demo
docker build -t my-nodejs:v1 .
docker run -p 8080:8080 -d my-nodejs:v1
```

Setup your environment and login to Openshift.
``` bash
export NS="<USERNAME>"
oc login -u <USERNAME>
oc new-project ${NS}
```

Push (upload) your local image to the Openshift private registry
(You'll need to configure your laptop to trust our registry: https://docs.docker.com/registry/insecure/  )
``` bash
docker login -u $(oc whoami) -p $(oc whoami -t) default-route-openshift-image-registry.apps.ocp4.rci-poc.es
docker tag my-nodejs:v1 default-route-openshift-image-registry.apps.ocp4.rci-poc.es/${NS}/my-nodejs:v1
docker push default-route-openshift-image-registry.apps.ocp4.rci-poc.es/${NS}/my-nodejs:v1
```

Deploy your application creating a simple deployment
``` bash
oc -n ${NS} create deployment my-nodejs-deployment --image image-registry.openshift-image-registry.svc:5000/${NS}/my-nodejs:v1 --port 8080 --replicas 2 --dry-run=client -o yaml
oc -n ${NS} create deployment my-nodejs-deployment --image image-registry.openshift-image-registry.svc:5000/${NS}/my-nodejs:v1 --port 8080 --replicas 2
oc -n ${NS} get deployment
oc -n ${NS} get pods
```

Create the service needed to balance the load between the replicas
``` bash
oc -n ${NS} expose deployment my-nodejs-deployment --dry-run -o yaml
oc -n ${NS} expose deployment my-nodejs-deployment
oc -n ${NS} get service
``` 

Publish externally your application using the URL nodejs-${NS}.apps.ocp4.rci-poc.es
``` bash
oc -n ${NS} create route edge my-nodejs-route --service my-nodejs-deployment --hostname nodejs-${NS}.apps.ocp4.rci-poc.es --dry-run=client -o yaml
oc -n ${NS} create route edge my-nodejs-route --service my-nodejs-deployment --hostname nodejs-${NS}.apps.ocp4.rci-poc.es
oc -n ${NS} get route
```
