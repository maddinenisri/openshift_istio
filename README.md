# Create Docker Machine
```
docker-machine create --driver virtualbox --virtualbox-memory 6144 --engine-insecure-registry 172.30.0.0/16  --engine-opt dns=8.8.8.8 openshift
```
This essentially ends up setting EXTRA_ARGS in /var/lib/boot2docker/profile
# Install openshift-cli
```
brew update
brew install openshift-cli
```
# Openshift Cluster UP
```
oc cluster up --docker-machine=openshift
```

# Openshift Cluster Down
```
oc cluster down --docker-machine=openshift
```

# Login to OC
```
oc login -u system:admin
```

# Select Project and assign privielages
```
oc new-project istio-system
oc project istio-system
oc adm policy add-scc-to-user anyuid -z default
oc adm policy add-scc-to-user privileged -z default
```

# Clone istio from Github and Apply additional privileages
```
oc adm policy add-scc-to-user anyuid -z istio-ingress-service-account
oc adm policy add-scc-to-user privileged -z istio-ingress-service-account
oc adm policy add-scc-to-user anyuid -z istio-egress-service-account
oc adm policy add-scc-to-user privileged -z istio-egress-service-account
oc adm policy add-scc-to-user anyuid -z istio-pilot-service-account
oc adm policy add-scc-to-user privileged -z istio-pilot-service-account
oc adm policy add-scc-to-user anyuid -z default
oc adm policy add-scc-to-user privileged -z default
oc adm policy add-cluster-role-to-user cluster-admin -z default

curl -L https://git.io/getLatestIstio | sh -
ISTIO=`ls | grep istio`
export PATH="$PATH:~/$ISTIO/bin"
cd $ISTIO
oc apply -f install/kubernetes/istio.yaml

oc create -f install/kubernetes/addons/prometheus.yaml
oc create -f install/kubernetes/addons/grafana.yaml
oc create -f install/kubernetes/addons/servicegraph.yaml
oc create -f install/kubernetes/addons/zipkin.yaml
oc expose svc grafana
oc expose svc servicegraph
oc expose svc zipkin
SERVICEGRAPH=$(oc get route servicegraph -o jsonpath='{.spec.host}{"\n"}')
GRAFANA=$(oc get route grafana -o jsonpath='{.spec.host}{"\n"}')/dotviz
ZIPKIN=$(oc get route zipkin -o jsonpath='{.spec.host}{"\n"}')
```

# Install Sample application
```
oc apply -f <(istioctl kube-inject -f samples/bookinfo/kube/bookinfo.yaml)
oc expose svc productpage

PRODUCTPAGE=$(oc get route productpage -o jsonpath='{.spec.host}{"\n"}')
watch -n 1 curl -o /dev/null -s -w %{http_code}\n $PRODUCTPAGE/productpage
```

ref: https://blog.openshift.com/evaluate-istio-openshift/

