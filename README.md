## Phase I

```yaml
name: Docker Image CI

on:
  push:
    branches: ["main"]
  pull_request:
    branches: ["main"]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Login to Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKER_USER }}
          password: ${{ secrets.DOCKER_TOKEN }}
      - uses: actions/checkout@v3
      - name: Docker Build
        run: docker build . --file Dockerfile --tag ${{secrets.DOCKER_USER}}/guestbook-go:latest
      - name: Docker Push
        run: docker push ${{secrets.DOCKER_USER}}/guestbook-go
```

Don't forget to create the secrets `DOCKER_USER` & `DOCKER_TOKEN`.

## Phase II

**Deploy container image built in above step(Phase I) on local/any kubernetes cluster.**

I'll be deploying it locally using `minikube`.

### Configure [minikube](https://minikube.sigs.k8s.io/docs/start/)

- `curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube_latest_amd64.deb`
- `sudo dpkg -i minikube_latest_amd64.deb`
- `minikube start`

### Create a Deployment

- `kubectl create deployment guestbook-go --image=hub.docker.com/zubayrrr/guestbook-go:latest`
- `kubectl get deployments`

[zubayrrr/guestbook-go](https://hub.docker.com/repository/docker/zubayrrr/guestbook-go)

### Create a Service

- `kubectl expose deployment guestbook-go --type=LoadBalancer --port=3000`
- `minikube service guestbook-go`

![image](https://user-images.githubusercontent.com/31969517/191456701-a2a5da6b-d067-4073-8274-ed586bb53427.png)

## Phase III

**Prevent merging anything in main branch without review.**

We'll be using Github feature - [Branch protection rules](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/defining-the-mergeability-of-pull-requests/managing-a-branch-protection-rule) to accomplish this task.

![image](https://user-images.githubusercontent.com/31969517/191463754-857a6b79-10fe-46dd-8ad8-8f34936bc43a.png)

You'll get "Rule is invalid" if your protection rule name is invalid. See [About branch protection rules - GitHub Docs](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/defining-the-mergeability-of-pull-requests/managing-a-branch-protection-rule#about-branch-protection-rules)

---

**Build container image only when one of the below conditions is true:**

1. **When PR get merged in main/master branch from any other branch.**
2. **When commit message contains `BUILD_CONTAINER_IMAGE` string**

```yaml
name: Docker Image CI

on:
  pull_request_review:
    types:
      - submitted

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Login to Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKER_USER }}
          password: ${{ secrets.DOCKER_TOKEN }}
      - uses: actions/checkout@v3
      - name: Docker Build
        if: contains(github.event.head_commit.message, 'BUILD_CONTAINER_IMAGE')
        run: docker build . --file Dockerfile --tag ${{secrets.DOCKER_USER}}/guestbook-go:latest
      - name: Docker Push
        run: docker push ${{secrets.DOCKER_USER}}/guestbook-go
```
