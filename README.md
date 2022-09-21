## Phase II

**Deploy container image built in above step(Phase I) on local/any kubernetes cluster.**

I'll be deploying it locally using `minikube`.

- Configure [minikube](https://minikube.sigs.k8s.io/docs/start/)

`curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube_latest_amd64.deb`
`sudo dpkg -i minikube_latest_amd64.deb`
`minikube start`

- Create a Deployment

`kubectl create deployment guestbook-go --image=hub.docker.com/zubayrrr/guestbook-go:latest`
`kubectl get deployments`

- Create a Service

`kubectl expose deployment guestbook-go --type=LoadBalancer --port=3000`
`minikube service guestbook-go`

![image](https://user-images.githubusercontent.com/31969517/191456701-a2a5da6b-d067-4073-8274-ed586bb53427.png)
