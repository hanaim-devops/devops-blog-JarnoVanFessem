# Stappenplan voor apollo draaien met kubernetes

## Stappen van te voren

volledige stappen van apollo zelf: <https://www.apolloconfig.com/#/en/deployment/distributed-deployment-guide?id=i-preparation>

instaleer mysql(version 5.6.5+)

installeer docker

instaleer kind(Kubernetes IN Docker)

maak een cluster aan

```bash
kind create cluster
```

## Step 1: Setup MySQL Database

1. **Add Bitnami Repository**:

   ```bash
   helm repo add bitnami https://charts.bitnami.com/bitnami
   ```

### Install MySQL:

```bash
helm install my-mysql bitnami/mysql --set auth.rootPassword=apollo,auth.database=ApolloConfigDB
```

### Check MySQL Service:

```bash
kubectl get svc --namespace default my-mysql
```

### Create Apollo Databases:

Connect to MySQL using port-forwarding:

```bash

kubectl port-forward service/my-mysql 3306:3306 --namespace default
```

Run the following command to connect to MySQL:

```bash
mysql -h 127.0.0.1 -u root -p
```

Create the databases:

```sql
-- Create ApolloPortalDB
CREATE DATABASE ApolloPortalDB;

-- Create ApolloConfigDB
CREATE DATABASE ApolloConfigDB;
```

## Step 2: Import the SQL Scripts

## For ApolloPortalDB:

Exit the MySQL terminal if you are currently in it by typing:

```sql
exit;
```

Use the following command to import the SQL file for the Apollo Portal database. Run this command in your terminal (not in MySQL):

```bash
mysql -h 127.0.0.1 -P 3306 -u root -p ApolloPortalDB < C:\Users\jarno\Downloads\apolloportaldb.sql
```

### For ApolloConfigDB:

Import the SQL file for the Apollo Config database with the following command:

```bash
mysql -h 127.0.0.1 -P 3306 -u root -p ApolloConfigDB < C:\Users\jarno\Downloads\apolloconfigdb.sql
```

## Step 3: Verify Data Import

### For ApolloPortalDB:
To verify the import for ApolloPortalDB, log back into MySQL:

```bash
mysql -h 127.0.0.1 -P 3306 -u root -p ApolloPortalDB
```

Run the following query:

```sql
SELECT `Id`, `Key`, `Value`, `Comment` FROM `ServerConfig` LIMIT 1;
```

### For ApolloConfigDB:

Repeat the same for ApolloConfigDB:

```bash
mysql -h 127.0.0.1 -P 3306 -u root -p ApolloConfigDB
```

Then run:

```sql
SELECT `Id`, `Key`, `Value`, `Comment` FROM `ServerConfig` LIMIT 1;
```

## Step 4: Install Apollo Services

### Install Apollo Configuration Service:

```bash
helm install apollo-service-dev ^
    --set configdb.host=10.96.114.26 ^
    --set configdb.userName=root ^
    --set configdb.password=apollo ^
    --set configdb.service.enabled=false ^
    --set configService.replicaCount=1 ^
    --set adminService.replicaCount=1 ^
    -n apollo ^
    apollo/apollo-service
```

### Port Forward for Apollo Services:

Port forward for Apollo Config Service:

```bash
kubectl port-forward service/apollo-service-apollo-configservice 8080:8080 --namespace apollo
```

Port forward for Apollo Admin Service:

```bash
kubectl port-forward service/apollo-service-apollo-adminservice 8090:8090 --namespace apollo
```

## Step 5: Install Apollo Portal

### Install Apollo Portal:

```bash
helm install apollo-portal ^
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
```

### Port Forward to Access Apollo Portal:

```bash
kubectl --namespace apollo port-forward <POD_NAME> 8070:8070
```

Replace <POD_NAME> with the actual name of the Apollo Portal pod, which you can find using:

```bash
kubectl get pods --namespace apollo
```

### Open the Apollo Portal: Access via <http://127.0.0.1:8070>.
