# How to Build Your Own BXFinance 

## Prerequisites
- Docker
- docker hub account
- kustomize
- kubernetes cluster
- envsubst

## Build the BXFinance App: 

1. go to a temporary directory and clone [BXFinance-Apps](https://github.com/Technical-Enablement-PingIdentity/BXFinance-Apps)
  ```
  cd /tmp
  git clone https://github.com/Technical-Enablement-PingIdentity/BXFinance-Apps
  cd BXFinance-Apps/bxfinance-scenario1
  ```

2. edit 2 variables in `.env`
  REACT_APP_HOST=bxfinance-<yourname>.ping-devops.com
  REACT_APP_ENV=bxfinance-<yourname>

3. edit the `server-name` line in `nginx.conf`
example:
```
    server {
        listen 5000;
        server_name demo.bxfinance.xyz bxfinance-qa.ping-devops.com bxfinance-staging.ping-devops.com demo.bxfinance.org;
```
>
```
    server {
        listen 5000;
        server_name bxfinance-<yourname>.ping-devops.com
```

4. Build your very own docker image of BXFinance app

  ```
  docker build -t <your-dockerhub-id>/bxfinance-app:latest .
  ```

5. 'push' your image to docker hub
  ```
  docker push <your-dockerhub-id>/bxfinance-app:latest
  ```

## Deploy the BXFinance Demo

1. set up your shell environment
  ```
  export BXFINANCE_EXTERNAL_HOST=bxfinance-<yourname>.ping-devops.com
  export BXFINANCE_APP_IMAGE_NAME=<your-dockerhub-id>/bxfinance-app:latest
  ```

2. make sure you have:
  - access to a kubernetes cluster and namespace without any conflicting deployments running. (an empty namespace is a good idea. running `kubectl get all` returns nothing). 
  - make sure you have a devops-secret to get licenses
    If you don't see it when you do a `kubectl get secrets` you can run 
    ```
    ping-devops generate devops-secret | k apply -f -
    ```

3. deploy
```
kustomize build 

2. it is a private repo so you will need to add a user/token to the github URL for server-profiles.
The .yamls expect this and are looking for a secret called 'server-profile-url'

to create this secret, run: 
```
kubectl create secret generic server-profile-url --from-literal=SERVER_PROFILE_URL='https://<user>:<token>@<repo_url>'
```

  Note: your devops user and key should also be stored in a secret called `devops-secret`. 


3. The files have some variables that will need to be filled when deploying. 
  - in the `shared-vars` file - replace domains as necessary. 
  - the ingresses will be set up based on value of var: PING_IDENTITY_DEVOPS_DNS_ZONE
    so, export this with:
    ```
    export PING_IDENTITY_DEVOPS_DNS_ZONE=your-base-domain.com
    ```

4. look at what you've done. run the following command to see the files that will be applied to the cluster. 
```
kustomize build . | envsubst '${PING_IDENTITY_DEVOPS_DNS_ZONE}'
```

## Deploy

```
kustomize build . | envsubst '${PING_IDENTITY_DEVOPS_DNS_ZONE}' | kubectl apply -f -
```

## View


```
kubectl get pods,svc,deploy,rs,job,ing,pvc
```
or `kall`

Once the ingresses show an address, you can go to that host in a browser