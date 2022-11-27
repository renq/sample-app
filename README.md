# Tekton

## How to build docker images?

```
docker build -f .docker/app/Dockerfile --build-arg APP_USER_ID=$(id -u) --build-arg APP_GROUP_ID=$(id -g) --tag sample-app:v1 .
docker build -f .docker/nginx/Dockerfile --build-arg APP_IMAGE=sample-app:v1 --tag sample-nginx:v1 .
```


## How to run k8s (kind)

https://github.com/tektoncd/dashboard/blob/release-v0.29.x/docs/walkthrough/walkthrough-kind.md

## How to rum k8s (minikube)

minikube start --driver=virtualbox  --profile tekton-dashboard-tutorial
kubectl apply --filename https://storage.googleapis.com/tekton-releases/pipeline/latest/release.yaml
kubectl apply --filename https://storage.googleapis.com/tekton-releases/dashboard/latest/tekton-dashboard-release.yaml
kubectl port-forward -n tekton-pipelines service/tekton-dashboard 9097:9097


## Access to github

Create key

    ssh-keygen -t ed25519 -C tekton@lipek.net

Copy `cat id_ed25519 | base64 -w 0` to credentials file.



## How to add pipeline?

Add "plugins":
    
    kubectl apply -f https://raw.githubusercontent.com/tektoncd/catalog/main/task/git-clone/0.8/git-clone.yaml

Add pipeline:

    kubectl apply -f ./.tekton/pipeline-2-credentials.yaml
    kubectl apply -f ./.tekton/pipeline-2-task.yaml
    kubectl apply -f ./.tekton/pipeline-2.yaml
    kubectl apply -f ./.tekton/pipeline-2-run.yaml

