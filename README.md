# k8s-immich

A collection of kubectl yaml files to deploy immich onto kubernetes. Customisation is provided via kustomize.

# Requirements

- Kustomize
- Postgres
- Redis
- Kubernetes Cluster (I'd recommend RKE2 from Rancher)

These are not necessary but useful if you want to store your data outside the cluster.

- TrueNAS
- Democratic-csi (For auto creating NFS/ISCSI shares on TrueNAS)

## Kustomize

Pretty simple to install. It does come with kubectl but if you want the latest one you can use docker.

`docker pull k8s.gcr.io/kustomize/kustomize:v3.8.7`

Then running a command via it:

`docker run k8s.gcr.io/kustomize/kustomize:v3.8.7 version`


## Database

You will need a postgres database. I have not included the postgres docker container that immich uses by default. You will probably
want to run database for many services. I recommend [Crunchydata postgres operator](https://access.crunchydata.com/documentation/postgres-operator/v5/quickstart/).

Once it is deployed then you need to [add a user](https://access.crunchydata.com/documentation/postgres-operator/5.3.0/architecture/user-management/). This will create a K8s secret for that user which you can refer to in the example-env.yaml.

```yaml
            - name: BASE_DB_URL
              valueFrom:
                secretKeyRef:
                  key: uri
                  name: example-db-pguser-immich-user
```

## Redis

You will also need a redis database. For something similar to the database operator above you could use [spotahomes operator](https://github.com/spotahome/redis-operator). I just needed a single redis pod so I went for the [bitnami redis chart](https://github.com/bitnami/charts/tree/main/bitnami/redis/#installing-the-chart).


# Install/Deployment

Copy the example overlays folder so you can customise to your liking. I have provided examples of changing the most common things,
but you can change most parts via kustomize.

As of 26th Jan 2023 you will need to use the latest image tag to get [pull request](https://github.com/immich-app/immich/pull/1256).

## Via kubectl

Build the templates to see what they look like without applying.

`kubectl kustomize overlays/custom`

Deploy to the cluster.

`kubectl apply -k overlays/custom`

## Via docker kustomize

Build the templates to see what they look like without applying.

`docker run -v  absolute_path_to_checkout:/k8s k8s.gcr.io/kustomize/kustomize:v3.8.7 build /k8s/k8s-immich/overlays/custom`

Deploy to the cluster.

`docker run -v  absolute_path_to_checkout:/k8s k8s.gcr.io/kustomize/kustomize:v3.8.7 build /k8s/k8s-immich/overlays/custom  | kubectl apply -f -`

More info: [Kustomize](https://github.com/kubernetes-sigs/kustomize)

# Why not helm?

Personal choice. For the past few months I have been migrating traditional applications to the K8s world and I like using kubectl directly.

- More familiar with the yaml files
- Yaml works nicely in my IDE (PyCharm)
- Can deploy directly from my IDE
- I find it more readable
