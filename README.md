# Kubernetes Homework Commands

## 1. Start Docker Desktop

Docker Desktop must be running before kind can create a local Kubernetes cluster.

```powershell
docker ps
```

## 2. Create the kind cluster

The cluster configuration is stored in kind-config.yaml.

```powershell
kind create cluster --config .\kind-config.yaml
```

Check clusters:

```powershell
kind get clusters
```

Check current kubectl context:

```powershell
kubectl config current-context
```

Expected context:

```text
kind-cubix
```

Check nodes:

```powershell
kubectl get nodes
```

Expected state:

```text
cubix-control-plane   Ready
```

## 3. Install ingress-nginx controller

Label the kind node:

```powershell
kubectl label node cubix-control-plane ingress-ready=true
```

Install ingress-nginx controller:

```powershell
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
```

Wait for the controller:

```powershell
kubectl wait --namespace ingress-nginx --for=condition=ready pod --selector=app.kubernetes.io/component=controller --timeout=180s
```

Check ingress-nginx:

```powershell
kubectl get pods -n ingress-nginx
```

Expected state:

```text
ingress-nginx-controller-...   1/1   Running
```

## 4. Deploy backapp

Apply backapp Service:

```powershell
kubectl apply -f .\k8s\backapp-service.yaml
```

Apply backapp Deployment:

```powershell
kubectl apply -f .\k8s\backapp-deployment.yaml
```

Check backapp:

```powershell
kubectl get deploy
kubectl get pods
kubectl get svc
kubectl logs deploy/backapp
```

Expected result:

```text
backapp Deployment is 1/1 Ready
backapp Pod is Running
backapp Service exists
backapp logs show Tomcat started on port 8080
Spring Boot banner is disabled
```

## 5. Deploy frontapp

Apply frontapp Deployment:

```powershell
kubectl apply -f .\k8s\frontapp-deployment.yaml
```

Apply frontapp Service:

```powershell
kubectl apply -f .\k8s\frontapp-service.yaml
```

Apply frontapp Ingress:

```powershell
kubectl apply -f .\k8s\frontapp-ingress.yaml
```

Check frontapp:

```powershell
kubectl get deploy
kubectl get pods
kubectl get svc
kubectl get ingress
kubectl logs deploy/frontapp
```

Expected result:

```text
frontapp Deployment is 1/1 Ready
frontapp Pod is Running
frontapp Service exists
frontapp Ingress exists
Ingress ADDRESS is localhost
```

## 6. Test endpoints

Root path:

```powershell
curl.exe http://localhost:8080
```

Expected:

```text
404 Not Found
```

This is acceptable because the application has no root endpoint.

Frontapp endpoint:

```powershell
curl.exe http://localhost:8080/frontapp
```

Expected:

```text
frontapp calls backapp
backappMessage is returned
frontappHostAddress is present
backappHostAddress is present
doExtraImageDataMatch is true
```

Frontapp endpoint with message:

```powershell
curl.exe "http://localhost:8080/frontapp?message=Hello"
```

Expected:

```text
backappMessage is Hello
```

Local frontapp endpoint:

```powershell
curl.exe http://localhost:8080/frontapp/local
```

Expected:

```text
frontapp responds locally
backappHostAddress is null
```

Local frontapp endpoint with message:

```powershell
curl.exe "http://localhost:8080/frontapp/local?message=Hello"
```

Expected:

```text
frontapp responds locally with Hello
backappHostAddress is null
```
