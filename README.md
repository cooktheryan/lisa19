# This directory is supplemental things to help setup the environment for LISA 19

# Add the repo
argocd repo add https://github.com/cooktheryan/federation-dev.git

# Add clusters
argocd cluster add east1
argocd cluster add east2
argocd cluster add west2

# Define simple app
argocd app create --project default --name simple-app \
--repo https://github.com/cooktheryan/federation-dev.git \
--path lisa19/simple-app \
--dest-server https://api.lisa-east1.sysdeseng.com:6443 \
--dest-namespace simple-app  \
--revision master \
--sync-policy automated

# List the app and view
argocd app get simple-app
oc get all -n simple-app

# Mongo

# Pacman
argocd app create --project default --name pacman-east1 --repo https://github.com/cooktheryan/federation-dev.git --path lisa19/pacman/overlays/cluster1 --dest-server https://api.lisa-east1.sysdeseng.com:6443 --dest-namespace pacman  --revision master --sync-policy automated

# Play the game

# Extend to east2 
argocd app create --project default --name pacman-east2 --repo https://github.com/cooktheryan/federation-dev.git --path lisa19/pacman/overlays/cluster2 --dest-server https://api.lisa-east2.sysdeseng.com:6443 --dest-namespace pacman  --revision master --sync-policy automated

# Set replicas to 0
vi overlays/cluster1/pacman-deployment.yaml
git commit -am 'resizing'
git push origin master

# Play the game

# Extend to west2
argocd app create --project default --name pacman-west2 --repo https://github.com/cooktheryan/federation-dev.git --path lisa19/pacman/overlays/cluster3 --dest-server https://api.lisa-west2.sysdeseng.com:6443 --dest-namespace pacman  --revision master --sync-policy automated
vi overlays/cluster2/pacman-deployment.yaml
git commit -am 'resizing'
git push origin master

