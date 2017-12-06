# Create Docker Machine
```
docker-machine create --driver virtualbox --engine-insecure-registry 172.30.0.0/16  --engine-opt dns=8.8.8.8 openshift
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