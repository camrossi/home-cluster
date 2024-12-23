1) Deploy cloudnative-pgvecto.rs I used HELM and didn't have to customize antying. 

2) Create the environment variables/passwords that are needed for your environemtn
```yaml
apiVersion: v1
stringData:
  DB_USERNAME: immich
  DB_DATABASE_NAME: immich
  DB_PASSWORD: immich
  username: immich
  password: immich
kind: Secret
metadata:
  name: immich-postgres-user
type: kubernetes.io/basic-auth
```
3) Bring Up a new postgress instance in parallel to the current one and run an import job. 

```yaml
apiVersion: postgresql.cnpg.io/v1
kind: Cluster
metadata:
  name: immich-postgres
spec:
  # At the time of writing, immich is only compatible with pgvecto.rs <0.4. Latest postgres image with that version is 16.5.
  imageName: ghcr.io/tensorchord/cloudnative-pgvecto.rs:16.5-v0.3.0@sha256:be3f025d79aa1b747817f478e07e71be43236e14d00d8a9eb3914146245035ba
  instances: 1

  postgresql:
    shared_preload_libraries:
      - "vectors.so"

  managed:
    roles:
      - name: immich
        superuser: true
        login: true

  bootstrap:
    initdb:
      database: immich
      owner: immich
      secret:
        name: immich-postgres-user
      import:
        type: microservice
        databases:
          - immich
        source:
          externalCluster: cluster-pg96
      postInitSQL:
        - CREATE EXTENSION IF NOT EXISTS "vectors";
        - CREATE EXTENSION IF NOT EXISTS "cube" CASCADE;
        - CREATE EXTENSION IF NOT EXISTS "earthdistance" CASCADE;

  storage:
    size: 4Gi
    storageClass: longhorn-2-replicas

  externalClusters:
    - name: cluster-pg96
      connectionParameters:
        # Use the correct IP or host name for the source database
        host: immich-postgresql.immich.svc.cluster.local
        user: immich
        dbname: immich
      password:
        name: immich-postgres-user
        key: password
```

Deploy the new DB instance (I use ArgoCD so you might have to do it in otehr ways) eventually your DB should be copied over the new one

```bash
➜  extra git:(main) ✗ kgp
NAME                                       READY   STATUS      RESTARTS   AGE
immich-machine-learning-6f79767cb4-ddjck   1/1     Running     0          5m6s
immich-postgres-1                          0/1     Init:0/1    0          0s
immich-postgres-1-import-hxxf7             0/1     Completed   0          5m5s
immich-postgresql-0                        1/1     Running     0          23d
immich-redis-master-0                      1/1     Running     0          23d
immich-server-7bf45c8c6d-twpjh             1/1     Running     0          5m6s
```

4) Switch to using the new Postgress DB, in my helm filed I did the following: Load the passwords from the secret and set the DB_HOSTNAME env to the service of our new DB
```yaml
envFrom:
  - secretRef:
      name: immich-postgres-user
env:
  DB_HOSTNAME: 'immich-postgres-rw'

```

NOTE: I have not deleted yet the old DB instance becuase I don't wanna loose data in case I have made a mistake. 

If all works fine... Delete the postgress section from the Immich Values files.
