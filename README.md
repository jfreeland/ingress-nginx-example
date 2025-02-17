# ingress-nginx-example

## `Exact` vs `Prefix`

```bash
kind create cluster -n ine --kubeconfig ~/.kube/config
# go install sigs.k8s.io/cloud-provider-kind@latest
# or
# brew install cloud-provider-kind
sudo cloud-provider-kind

helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm install ingress-nginx ingress-nginx/ingress-nginx -n ingress-nginx --create-namespace

export INGRESS=$(kubectl -n ingress-nginx get svc ingress-nginx-controller -o jsonpath='{.status.loadBalancer.ingress[0].ip}')

curl http://$INGRESS
# should return a default ingress-nginx 404

kubectl apply -f exact.yaml
kubectl apply -f prefix.yaml

curl -H "Host: foo.com" http://$INGRESS
curl -H "Host: foo.com" http://$INGRESS/
# should return a 200 and default nginx page coming from the `nginx` pod from `exact.yaml`

curl -v -H "Host: foo.com" http://$INGRESS/bar
# should return a 200 hello-world from the `prefix` pod

kind delete cluster -n ine
```

## overlapping

```bash
kind create cluster -n ine --kubeconfig ~/.kube/config
sudo cloud-provider-kind

helm install ingress-nginx ingress-nginx/ingress-nginx -n ingress-nginx --create-namespace

export INGRESS=$(kubectl -n ingress-nginx get svc ingress-nginx-controller -o jsonpath='{.status.loadBalancer.ingress[0].ip}')

curl http://$INGRESS
# should return a default ingress-nginx 404

kubectl apply -f overlapping.yaml

kind delete cluster -n ine
```

Should get you:

```bash
Error from server (BadRequest): error when creating "overlapping.yaml": admission webhook "validate.nginx.ingress.kubernetes.io" denied the request:
-------------------------------------------------------------------------------
Error: exit status 1
2025/02/17 21:32:01 [emerg] 370#370: duplicate location "/" in /tmp/nginx/nginx-cfg2520639532:538
nginx: [emerg] duplicate location "/" in /tmp/nginx/nginx-cfg2520639532:538
nginx: configuration file /tmp/nginx/nginx-cfg2520639532 test failed

-------------------------------------------------------------------------------
```

Or:

```bash
Error from server (BadRequest): error when creating "overlapping.yaml": admission webhook "validate.nginx.ingress.kubernetes.io" denied the request: host "foo.com" and path "/" is already defined in ingress exact/exact
```
