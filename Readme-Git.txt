SSH
=====================================
ssh inc@m-inc.eastus2.cloudapp.azure.com

=====================================
Az Login
=========================================

az login

--az account list --output table

--------DEV-----------------------------
az account set --subscription "<Your Subscription>"

az aks get-credentials -g aks-showtime-dev -n aks-showtime-dev

az aks get-credentials -g aks-showtime-demo -n aks-showtime-demo

kubectl apply -f helm-rbac.yaml

helm init --service-account tiller

helm repo update

============================================
INSTAL - ISTIO
============================================

https://istio.io/docs/setup/kubernetes/install/helm/

----------PREP----------
kubectl create namespace istio-system

helm template install/kubernetes/helm/istio-init --name istio-init --namespace istio-system | kubectl apply -f -

kubectl get crds | grep 'istio.io\|certmanager.k8s.io' | wc -l

---------INSTAL ISTIO-----------------
helm template install/kubernetes/helm/istio --name istio --namespace istio-system --values install/kubernetes/helm/istio/values-istio-demo.yaml | kubectl apply -f -

--------Validate--------------
kubectl get svc -n istio-system
kubectl get pods -n istio-system


kiali
kubectl -n istio-system port-forward $(kubectl -n istio-system get pod -l app=kiali -o jsonpath='{.items[0].metadata.name}') 20001:20001
kubectl -n istio-system port-forward kiali-5df77dc9b6-2smz7 20001:20001



============================================
INSTAL - Bookinfo
============================================
----- Sidecar inject ------------
kubectl label namespace default istio-injection=enabled

------Sample Application---------------
kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml

----------Gateway-----------------
kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml

kubectl get svc istio-ingressgateway -n istio-system

-------Install default destination rules--------------
kubectl apply -f samples/bookinfo/networking/destination-rule-all.yaml

http://13.66.132.217/productpage

==============================================================
Demo -4 - Route everyone all service to V1
kubectl apply -f samples/bookinfo/networking/virtual-service-all-v1.yaml

Demo -5 - Route user sandeep.sachan to  V2
kubectl apply -f samples/bookinfo/networking/virtual-service-reviews-test-v2.yaml

Demo -6 - Injecting an HTTP delay fault
kubectl apply -f samples/bookinfo/networking/virtual-service-ratings-test-delay.yaml

Demo -7 Apply weight-based routing 50%
kubectl apply -f samples/bookinfo/networking/virtual-service-reviews-50-v3.yaml

Injecting an HTTP abort fault
kubectl -n istio-system get pod -l app=grafana -o jsonpath='{.items[0].metadata.name}'

Demo -8 Enabling Policy Enforcement
1 kubectl -n istio-system get cm istio -o jsonpath="{@.data.mesh}" | grep disablePolicyChecks

2 helm template install/kubernetes/helm/istio --namespace=istio-system -x templates/configmap.yaml --set global.disablePolicyChecks=false | kubectl -n istio-system replace -f -

3 - Play
kubectl apply -f samples/bookinfo/policy/mixer-rule-productpage-ratelimit.yaml

Jaeger
kubectl port-forward -n istio-system istio-tracing-595796cf54-2gcz2 16686:16686  &

kubectl -n istio-system port-forward kiali-5df77dc9b6-2smz7 20001:20001

kubectl port-forward -n istio-system kiali-5df77dc9b6-2smz7 16686:16686  &

kubectl -n istio-system port-forward grafana-749c78bcc5-8qw9k 3000:3000

kubectl -n istio-system port-forward prometheus-7f87866f5f-wl82p 9090:9090

Prometheus
kubectl -n istio-system port-forward $(kubectl -n istio-system get pod -l app=prometheus -o jsonpath='{.items[0].metadata.name}') 9090:9090

Grafana
kubectl -n istio-system port-forward $(kubectl -n istio-system get pod -l app=grafana -o jsonpath='{.items[0].metadata.name}') 3000:3000

Jaeger 
kubectl port-forward -n istio-system $(kubectl get pod -n istio-system -l app=jaeger -o jsonpath='{.items[0].metadata.name}') 16686:16686  &

Kiali
kubectl -n istio-system port-forward $(kubectl -n istio-system get pod -l app=kiali -o jsonpath='{.items[0].metadata.name}') 20001:20001


kubectl -n istio-system port-forward grafana-c49f9df64-5bjrj 3000:3000

kubectl -n istio-system port-forward prometheus-67599bf55b-jx5sb 9090:9090

kubectl -n istio-system port-forward istio-tracing-79db5954f-ftxld 16686:16686

kubectl -n istio-system port-forward kiali-5c4cdbb869-95cfh 20001:20001