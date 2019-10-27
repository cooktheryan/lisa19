# This directory is supplemental things to help setup the environment for LISA 19

# Add the repo
argocd repo add https://github.com/cooktheryan/lisa19.git

# Add clusters
argocd cluster add east1
argocd cluster add east2
argocd cluster add west2

# Define simple app
argocd app create --project default --name simple-app --repo https://github.com/cooktheryan/lisa19.git --path simple-app --dest-server https://api.lisa-east1.sysdeseng.com:6443 --dest-namespace simple-app  --revision master --sync-policy automated --self-heal

# List the app and view
argocd app get simple-app
oc get all -n simple-app

# Mongo
argocd app create --project default --name cluster1-mongo \
  --repo https://github.com/cooktheryan/lisa19.git \
  --path mongo/overlays/cluster1 \
  --dest-server https://api.lisa-east1.sysdeseng.com:6443 \
  --dest-namespace mongo --revision master --sync-policy automated

argocd app create --project default --name cluster2-mongo \
  --repo https://github.com/cooktheryan/lisa19.git \
  --path mongo/overlays/cluster2 \
  --dest-server https://api.lisa-east2.sysdeseng.com:6443 \
  --dest-namespace mongo --revision master --sync-policy automated

argocd app create --project default --name cluster3-mongo  --repo https://github.com/cooktheryan/lisa19.git \
  --path mongo/overlays/cluster3 \
  --dest-server https://api.lisa-west2.sysdeseng.com:6443 \
  --dest-namespace mongo --revision master --sync-policy automated


# Labeling of pods
for cluster in east1 east2 west2; do ./utility/wait-for-deployment $cluster mongo mongo; done

# Label pod
# Select Primary MongoDB pod
MONGO_POD=$(oc --context=east1 -n mongo get pod --selector="name=mongo" --output=jsonpath='{.items..metadata.name}')

# Label primary pod
oc --context=east1 -n mongo label pod $MONGO_POD replicaset=primary

# Validate successful labeling
./utility/wait-for-mongo-replicaset east1 mongo 3

# Pacman
argocd app create --project default --name pacman-east1 --repo https://github.com/cooktheryan/lisa19.git --path pacman/overlays/cluster1 --dest-server https://api.lisa-east1.sysdeseng.com:6443 --dest-namespace pacman  --revision master --sync-policy automated

# Play the game
https://pacman.demo-sysdeseng.com

# Extend to east2 
argocd app create --project default --name pacman-east2 --repo https://github.com/cooktheryan/lisa19.git --path lisa19/pacman/overlays/cluster2 --dest-server https://api.lisa-east2.sysdeseng.com:6443 --dest-namespace pacman  --revision master --sync-policy automated

# Set replicas to 0
vi overlays/cluster1/pacman-deployment.yaml
git commit -am 'resizing'
git push origin master

# Play the game

# Extend to west2
argocd app create --project default --name pacman-west2 --repo https://github.com/cooktheryan/lisa19.git --path lisa19/pacman/overlays/cluster3 --dest-server https://api.lisa-west2.sysdeseng.com:6443 --dest-namespace pacman  --revision master --sync-policy automated

# Remove east2 
vi overlays/cluster2/pacman-deployment.yaml
git commit -am 'resizing'
git push origin master

