# ingress-nginx-example

`Exact` vs. `Prefix` path matching example.

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
