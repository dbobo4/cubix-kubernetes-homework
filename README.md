# Cubix Kubernetes Homework - Focused Submission (Adam Kovacs)

## Required installation / deployment commands

```powershell
kind create cluster --config .\kind-config.yaml
kubectl label node cubix-control-plane ingress-ready=true
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
kubectl wait --namespace ingress-nginx --for=condition=ready pod --selector=app.kubernetes.io/component=controller --timeout=180s
kubectl apply -f .\k8s\backapp-service.yaml
kubectl apply -f .\k8s\backapp-deployment.yaml
kubectl apply -f .\k8s\frontapp-deployment.yaml
kubectl apply -f .\k8s\frontapp-service.yaml
kubectl apply -f .\k8s\frontapp-ingress.yaml
```

## Command for checking the logs of the backapp application

```powershell
kubectl logs deploy/backapp
```

Expected note: the Spring Boot banner is disabled for the backapp because JAVA_OPTS is set to -Dspring.main.banner-mode=off.

## One kubectl command for listing all frontapp related resources

```powershell
kubectl get all,ingress -l app=frontapp,homework=frontapp,training=block3
```

## One kubectl command for deleting both applications resources

```powershell
kubectl delete all,ingress -l "training=block3,homework in (frontapp,backapp)"
```

## Public endpoint of the frontapp

```powershell
curl.exe http://localhost:8080/frontapp
```

Expected behavior: the frontapp is reachable from localhost and calls the backapp through the internal Kubernetes Service.