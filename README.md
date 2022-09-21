# workshop
openshift-workshop
## From docker-compose to openshift if few steps (this is for MacOSX you can skip docker-machine part if You run Linux)
<sub>
cd docker-compose
docker-machine start default
eval "$(docker-machine env default)"
docker-compose up -d
http://172.16.157.137:8080
docker-compose down
docker volume rm docker-compose_mysql_data
mkdir k8s
cd k8s
kompose convert -f ../docker-compose.yml
</sub>
Edit the liveness and rediness probes on both generated deployments 
Check the storageclasses and attached the correct ones to pvc
oc get storageclasses
Correct the spec.storageClassName in all pvc
Create the k8s objects in following order 
<sub>
oc create -f mysql-data-persistentvolumeclaim.yaml
oc get pvc
oc get pv
oc create -f mariadb-deployment.yaml
oc get deployments
oc get pods
oc logs pod-id
oc create -f mariadb-service.yaml
oc crate -f keycloak-deployment.yaml 
oc get deployments
oc get pods
oc logs pod-id
oc port-forward pod-id 8080
oc create -f keycloak-service.yaml
oc port-forward svc/keycloak 8080
oc expose service keycloak
</sub>
Externalize the application settings
<sub>
oc scale deployment mariadb --replicas=0
oc scale deployment keycloak --replicas=0
oc create secret generic mariadb-data --from-literal=DATABASE=keycloak --from-literal=PASSWORD=password --from-literal=ROOT_PASSWORD=root --from-literal=USER=keycloak
oc set env --from=secret/mariadb-data deployment/mariadb --prefix=MYSQL_
oc set env --from=secret/mariadb-data deployment/keycloak --prefix=DB_
oc edit deployment keycloak
oc create secret generic keycloak-data --from-literal=PASSWORD=Pa55w0rd --from-literal=USER=admin
oc set env --from=secret/keycloak-data deployment/keycloak --prefix=KEYCLOAK_
oc create configmap keycloak-config --from-literal=DB_ADDR=mariadb --from-literal=DB_VENDOR=mariadb --from-literal=JGROUPS_DISCOVERY_PROTOCOL=kubernetes.KUBE_PING --from-literal=JGROUPS_DISCOVERY_PROPERTIES="port_range=0,dump_requests=true,namespace=test-project
oc set env --from configmap/keycloak-config deployment/keycloak
</sub>
# Privilidges of the pod to ask API server and why do we need them
<sub>
oc scale deployment keycloak --replicas=1
oc scale deployment keycalok --replicas=3
oc logs pod-id 
oc scale deployment keycloak --replicas=1
oc get sa
oc create sa keycloak
</sub>
Add ServiceAccount udner the spec
<sub>
oc edit deployment keycloak
</sub>
Create cluster role and role binding for created ServiceAccount 
<sub>
oc create -f ../RBAC/pod-reader-rolebinding.yaml
oc create -f ../RBAC/keycloak-cluster-role-binding.yaml
oc logs pod-id
</sub>
Change the protocol to dns.PING and properties TO dns_query=keycloak-jgroups.test-project.svc.cluster.local and remove the unnedded cluster role and rolebinding
<sub>
oc scale deployment keycloak --replicas=0
oc create -f keycloak-headless.yaml
oc edit cm kaycloak-config
oc delete -f ../RBAC/pod-reader-rolebinding.yaml
oc delete -f ../RBAC/keycloak-cluster-role-binding.yaml
oc scale deployment keycloak --replicas=0
oc logs pod-id
</sub>
# Create an application template, 
The resources have to be exported and cleaned
<sub>
oc get pvc -o yaml > Template/pvc.yaml
oc get cm keycloak-config -o yaml > Template/cm.yaml
oc get secret keycloak-data -o yaml > Template/keycloak-sec.yaml
oc get secret mariadb-data -o yaml > Template/mariadb-sec.yaml
oc get svc -o yaml > Template/svc.yaml
oc get deployment -o yaml > Template/deployment.yaml
oc get route -o yaml > Template/deployment.yaml
</sub>
All the resources have to merged also header and parametres part have to be added
<sub>
oc new-project keycloak
oc create secret generic mariadb-data --from-literal=DATABASE=keycloak --from-literal=PASSWORD=password --from-literal=ROOT_PASSWORD=root --from-literal=USER=keycloak
oc create secret generic keycloak-data --from-literal=PASSWORD=Pa55w0rd --from-literal=USER=admin
oc process -f keycloak-template.yaml -p NAMESPACE=keycloak | oc create -f -
</sub>
# Gitlab adding security context constrain
<sub>
oc new-project gitlab
oc process -f gitlab-template.yaml -p APPLICATION_HOSTNAME=gitlab.itzroks-664003gjth-89y04c-6ccd7f378ae819553d37d5f2ee142bd6-0000.eu-de.containers.appdomain.cloud  | oc create -f -
oc logs -f gitlab-ce-1-pod-id
oc get dc gitlab-ce -o yaml | grep serviceAccount
oc describe scc anyuid
oc adm policy add-scc-to-user anyuid -z gitlab-ce-user
oc rollout latest dc/gitlab-ce
</sub>

