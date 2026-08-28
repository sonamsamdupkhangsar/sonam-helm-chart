# sonam-helm-chart
<img src="https://helm.sh/img/helm.svg" width="100"/>  

This Helm chart will deploy app using a Nginx Ingress controller on a Kubernetes cluster.

Please see the sample-values.yaml file included in this chart for a sample.

This chart supports creating environment variables
and secret environment variables using GITHUB secrets as envrionment variables such as following example:
``` 
--set "secretenvs[0].name=PACT_BROKER_BASIC_AUTH_USERNAME" --set "secretenvs[0].value=${{ secrets.PACT_BROKER_BASIC_AUTH_USERNAME }}" \
            --set "secretenvs[1].name=PACT_BROKER_BASIC_AUTH_PASSWORD" --set "secretenvs[1].value=${{ secrets.PACT_BROKER_BASIC_AUTH_PASSWORD }}" \
```

This chart can also support creation of environment variables for a postgres deployment.


The version 0.1.23 supports referencing multiple secret files using the following in a values.yaml file:
```
secretFiles:
  - file: file1
    keys:
      - key: pgdb_username
        name: PG_USERNAME
      - key: pgdb_password
        name: PG_PASSWORD
  - file: file2abc
    keys:
      - key: pgdb_username
        name: PGDB
      - key: apple
        name: APPLE_DB
```
The `name` field will be set as an environment variable for application to consume.

For users, add the following chart to your environment:

```helm repo add sonam https://sonamsamdupkhangsar.github.io/sonam-helm-chart/```

To deploy:

``` helm install kecha sonam/mychart -f values.yaml```
 

## 0.1.15 version supports Oauth2-proxy
This 0.1.15 now supports OAuth2 proxy setup to secure applications.  See `values-echo-oauth-backend.yaml` for how to 
use this feature.


## The following instructions are for local development and debugging of this Helm chart purposes only.

To remove previous Helm charts under the name 'sonam' `helm repo remove sonam`

Update `Chart.yaml` to bump chart version and application version.

Non-secret application configuration can be supplied as mounted files:

```yaml
configMountPath: /config
configFiles:
  application-policy.yaml: |
    example:
      enabled: true
```

The chart creates a release-specific ConfigMap, mounts it read-only at
`configMountPath`, and changes the pod-template checksum when the file contents change.
Applications must opt in to loading the mounted file; for Spring Boot, for example:

```yaml
envs:
  - name: SPRING_CONFIG_IMPORT
    value: optional:file:/config/application-policy.yaml
```

The following instruction for updating this Helm chart.  Run the following:
```
helm package .
helm repo index --url https://sonamsamdupkhangsar.github.io/sonam-helm-chart/ .
```

install locally
```helm install kecha ../../github/sonam-helm-chart -f values.yaml```   

 dry run to generate yaml
```helm install kecha ../../github/sonam-helm-chart --version 0.1.3 -f values.yaml --dry-run ```

another way to generate yamls for debug
 ```helm template -n stage kechaapp ../../github/sonam-helm-chart --version 0.1.16 -f values.yaml --debug```

 
 ```
helm upgrade --install --timeout 5m0s \
            --set "image.repository=jmalloc/echo-server" \
            --set "image.tag=latest" \
            --set "project=echo" \            
            echo \
            sonam/mychart -f values-echo-oauth-backend.yaml --version 0.1.16 --namespace=backend --dry-run
```

To deploy a docker image from github container registry:
```
export CR_PAT=<YOUR_PERSONAL_ACCESS_TOKEN_FROM_GITHUB>

echo $CR_PAT | docker login ghcr.io -u <USERNAME> --password-stdin

export PROJECT_NAME=discovery-service

helm upgrade --install --timeout 5m0s \
    --set "image.repository=ghcr.io/sonamsamdupkhangsar/$PROJECT_NAME" \
    --set "image.tag=latest" \
    --set "project=$PROJECT_NAME" \
    $PROJECT_NAME \
    sonam/mychart -f <PATH_TO_VALUES>/values-backend.yaml --version 0.1.26 --namespace=backend


or

helm install eds ../../github/sonam-helm-chart -f ../spring-cloud-gateway/discovery-service/values-backend-local.yaml --namespace=backend


```

### Namespace-qualified service values

Environment values support Helm template expressions. This lets an app point to
services in its own release namespace:

```yaml

envs:
  - name: USER_REST_SERVICE
    value: "http://user-rest-service.{{ .Release.Namespace }}.svc.cluster.local"
  - name: AUTH_SERVER
    value: "http://authorization-server.{{ .Release.Namespace }}.svc.cluster.local"
```

Use `.Release.Namespace` for internal service-to-service calls so a release in
`business1` calls services in `business1`, and a release in `free` calls
services in `free`.
