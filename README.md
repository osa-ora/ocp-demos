# Demo For Service Mesh (TBC)


Traditional Way of Deploying Application and Switch Version ... (To be completed)
```
oc new-project dev
oc new-app --name=loyalty-v1 java~https://github.com/osa-ora/ocp-demos--context-dir=backend -e APP_VERSION=1.2
oc new-app --name=loyalty-v2 java~https://github.com/osa-ora/ocp-demos --context-dir=backend -e APP_VERSION=2.1
oc new-app --name=front-app java~https://github.com/osa-ora/ocp-demos --context-dir=frontend -e END_POINT=http://loyalty-v1:8080/loyalty/balance/
oc label deployment/loyalty-v2 app.kubernetes.io/part-of=my-application
oc label deployment/loyalty-v1 app.kubernetes.io/part-of=my-application
oc label deployment/front-app app.kubernetes.io/part-of=my-application
oc expose svc/front-app
//wait for app deployment completed ... then test it using curl ...
curl $(oc get route front-app -o jsonpath='{.spec.host}')/front/test/1999
// outcome: {"Response:":"{\"account\":1999,\"balance\": 3000, \", app-version: 1.2}","Welcome":" guest"}%
oc set env deployment/front-app END_POINT=http://loyalty-v2:8080/loyalty/balance/
//wait for the new version deployment
curl $(oc get route front-app -o jsonpath='{.spec.host}')/front/test/1999
// outcome: {"Response:":"{\"account\":1999,\"balance\": 3000, \", app-version: 2.1}","Welcome":" guest"}
```

# Demo for Skupper (TBC)
```
oc login //first cluster 
oc new-project dev-local
oc new-app --name=loyalty-v1 java~https://github.com/osa-ora/ocp-demos --context-dir=backend -e APP_VERSION=LOCAL1
oc new-app --name=front-app java~https://github.com/osa-ora/ocp-demos --context-dir=frontend -e END_POINT=http://loyalty-v1:8080/loyalty/balance/
oc label deployment/loyalty-v1 app.kubernetes.io/part-of=my-application
oc label deployment/front-app app.kubernetes.io/part-of=my-application
oc expose svc/front-app
//skupper init --enable-console --enable-flow-collector --console-auth unsecured
//skupper init --enable-console --enable-flow-collector --console-auth internal --console-user <username> --console-password <password> 
skupper init --enable-console --enable-flow-collector --console-auth openshift
skupper token create secret_connect.token
skupper status

oc login //second cluster 
oc new-project dev-remote
oc new-app --name=loyalty-v2 java~https://github.com/osa-ora/ocp-demos --context-dir=backend -e APP_VERSION=REMOTE2
oc label deployment/loyalty-v2 app.kubernetes.io/part-of=my-application

skupper init --ingress none
skupper status
skupper link create secret_connect.token --name first-to-second-link
//to delete this link: skupper link delete first-to-second-link
skupper expose host host.containers.internal --address loyalty-v2 --port 8080 --protocol http

oc login //first cluster 
oc project dev-local
skupper service create loyalty-v2 8080 --protocol http



//wait for app deployment completed ... then test it using curl ...
curl $(oc get route front-app -o jsonpath='{.spec.host}')/front/test/1999
// outcome: {"Response:":"{\"account\":1999,\"balance\": 3000, \", app-version: LOCAL1}","Welcome":" guest"}%
oc set env deployment/front-app END_POINT=http://loyalty-v2:8080/loyalty/balance/
//wait for the new version deployment
curl $(oc get route front-app -o jsonpath='{.spec.host}')/front/test/1999
// outcome: {"Response:":"{\"account\":1999,\"balance\": 3000, \", app-version: REMOTE2}","Welcome":" guest"}
//end of demo
//skupper delete on both sides

```

