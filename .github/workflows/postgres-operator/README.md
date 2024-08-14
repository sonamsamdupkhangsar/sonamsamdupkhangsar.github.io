# Intro
This is for connecting to a database in a K8 cluster that is deployed using the postgres-operator

# Connecting to database
[from Zalando's Connect to PostgreSQL] (https://github.com/zalando/postgres-operator/blob/master/docs/user.md#connect-to-postgresql)

## get name of master pod of acid-minimal-cluster
export CLUSTER=role-minimal-cluster
export NS=backend

export PGMASTER=$(kubectl get pods -o jsonpath={.items..metadata.name} -l application=spilo,cluster-name=$CLUSTER,spilo-role=master -n $NS)

## set up port forward
kubectl port-forward $PGMASTER 6432:5432 -n $NS

## Open terminal to connect to pgsql
Open a new terminal.  Type the following

```
export PGPASSWORD=$(kubectl get secret postgres.$CLUSTER.credentials.postgresql.acid.zalan.do -o 'jsonpath={.data.password}' -n $NS | base64 -d)
export PGSSLMODE=require
psql -U postgres -h localhost -p 6432

psql (14.9 (Homebrew), server 15.2 (Ubuntu 15.2-1.pgdg22.04+1))
WARNING: psql major version 14, server major version 15.
         Some psql features might not work.
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, bits: 256, compression: off)
Type "help" for help.

postgres=# 
```

## Connect to database called `authorization`
 \c authorization
 postgres=# \c authorization
psql (14.9 (Homebrew), server 15.2 (Ubuntu 15.2-1.pgdg22.04+1))
WARNING: psql major version 14, server major version 15.
         Some psql features might not work.
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, bits: 256, compression: off)
You are now connected to database "authorization" as user "postgres".

## Listing tables
`> \dt`

## show data in tables
`> select * from "authorization" `

## drop table - remove table
> `drop table "authorization";`

