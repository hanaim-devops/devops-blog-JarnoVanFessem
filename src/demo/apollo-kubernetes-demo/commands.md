$ helm install apollo-service-dev ^
    --set configdb.host=10.96.114.26 ^
    --set configdb.userName=root ^
    --set configdb.password=apollo ^
    --set configdb.service.enabled=false ^
    --set configService.replicaCount=1 ^
    --set adminService.replicaCount=1 ^
    -n apollo ^
    apollo/apollo-service

helm uninstall apollo-service-dev -n apollo

$ helm install apollo-portal ^
    --set portaldb.host=10.96.114.26 ^
    --set portaldb.userName=root ^
    --set portaldb.password=apollo ^
    --set portaldb.service.enabled=false ^
    --set config.envs="dev\,pro" ^
    --set config.metaServers.dev=http://apollo-service-dev-apollo-configservice:8080 ^
    --set config.metaServers.pro=http://apollo-service-pro-apollo-configservice:8080 ^
    --set replicaCount=1 ^
    -n apollo ^
    apollo/apollo-portal

helm uninstall apollo-portal -n apollo

kubectl get pods --namespace apollo -l "app=apollo-portal"

kubectl --namespace apollo port-forward <pod-name> 8070:8070

helm install apollo-service-dev ^
    --set configdb.host=my-mysql.default.svc.cluster.local ^
    --set configdb.dbName=ApolloConfigDB ^
    --set configdb.userName=root ^
    --set configdb.password=apollo ^
    --set configdb.service.enabled=false ^
    --set configService.replicaCount=1 ^
    --set adminService.replicaCount=1 ^
    -n apollo ^
    apollo/apollo-service


docker build --no-cache -t jarnovanfessem/apollokubernetes:latest .

docker login

docker tag apollokubernetes:latest jarnovanfessem/apollokubernetes:latest

docker push jarnovanfessem/apollokubernetes:latest

docker run -p 8100:8100 jarnovanfessem/apollokubernetes:latest '<key>'

docker pull jarnovanfessem/apollokubernetes:latest
