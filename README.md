Cart Service
=========

This is the quarkus version for the coolstore cart service. 
It uses infinispan, openapi and rest extenstions.

To deploy the service to a running single-node OpenShift cluster:

   ```
$ oc login -u developer -p developer

$ oc new-project MY_PROJECT_NAME

   ```

== More Information

Following ConfigMap needs to exist which is mounted to /deployments/config/app-config-properties
NAME: cart-service
Key: application.properties

   ```
quarkus.http.port=8081
quarkus.infinispan-client.server-list=datagrid-service:11222
quarkus.swagger-ui.always-include=true

   ```

Installing Datagrid-service and configmap
You might want to import the streams if already not done
 ```
for resource in cache-service-template.yaml \
  datagrid-service-template.yaml
do
  oc create -n openshift -f \
  https://raw.githubusercontent.com/jboss-container-images/jboss-datagrid-7-openshift-image/7.3-v1.2/services/${resource}
done
 ```


in the scripts directory createResources.sh does the same as follows, which will create the data-grid service and also create a configmap for the app.
 ```
   oc new-app datagrid-service -p APPLICATION_USER=demo -p APPLICATION_PASSWORD=demo -p NUMBER_OF_INSTANCES=3 -e AB_PROMETHEUS_ENABLE=true
   oc expose svc/datagrid-service-ping
   oc expose svc/datagrid-service
   
   oc create -f configmap.yml
   
   ```
   
Finally run the following

oc new-app jboss/infinispan-server:10.0.0.Beta3 --name=datagrid-service
oc create -f config/configmap.yml

./mvnw package -Pnative
./mvnw package -Pnative -Dnative-image.docker-build=true
docker build -f src/main/docker/Dockerfile.native -t cart-service .

oc new-build --binary --name=cart-service -l app=cart-service
oc patch bc/cart-service -p '{"spec":{"strategy":{"dockerStrategy":{"dockerfilePath":"src/main/docker/Dockerfile.native"}}}}'
oc start-build cart-service --from-dir=. --follow
oc new-app --image-stream=cart-service:latest
oc expose svc/cart-service


oc start-build cart-service --from-file target/*-runner.jar --follow

oc apply -f cr_minimal.yaml

sysh@apple /o/g/s/c/config> oc create -f service.yml 
service/cart-service created
sysh@apple /o/g/s/c/config> oc expose svc/cart-service




./mvnw package
docker build -f src/main/docker/Dockerfile.jvm -t cart-service .

oc new-build --binary --name=cart-service -l app=cart-service
oc patch bc/cart-service -p '{"spec":{"strategy":{"dockerStrategy":{"dockerfilePath":"src/main/docker/Dockerfile.jvm"}}}}'
oc start-build cart-service --from-file=. --follow
oc new-app --image-stream=cart-service:latest
oc expose svc/cart-service


