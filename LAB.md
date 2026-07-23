# Déploiement de l'environnement RKE2

## Objectif

L'objectif de ce TP est de vous montrer comment on peut installer simplement un cluster Kubernetes via RKE2 et rancher. 

## Création des VMS :

Provisionnez 2 VMs avec ses prérequis :

- Ubuntu server 2022 ou plus (2024 dans mon cas) https://ubuntu.com/download/server/thank-you?version=22.04.5&architecture=amd64&lts=true
- au moins 2vCPU (4 recommandés)
- au moins 4gb de RAM chacune
- 30 gb de disque **si il n'y a qu'une seule partition (pas de lvm)** ou sinon 127 gb en dynamique. 
- Un accès en ssh depuis votre machine et un accès internet aux vm

Une vm devra s'appeller controlplane-01, et l'autre worker-01

## Options d'installation des VMs selon les panels :

Menu "type of installation" => Ubuntu Server

De préférence utilisez une adresse ip fixe

Dans "Configuration de stockage Guidée" décochez la case Set up this disk as an LVM group

Upgrade to Ubuntu pro => On skip

SSH configuration => On installe

Features server snaps => On laisse vide

## L'installation 

### Prérequis sur les VMs

**A faire sur les deux VM** 

Pour les commandes qui viennent, passez en root et lancez une mise à jour pour avoir la liste des packages :

```
sudo su
apt-get update && apt-get upgrade
````

Modifiez si ce n'est pas le cas le fichier /etc/hostname pour qu'il contienne le nom de votre VM. 

Enfin éditez le fichier /etc/hosts pour mettre les ip et nom de vos vm. Par exemple dans mon adressage 

````
127.0.0.1 localhost
127.0.1.1 controlplane-01

172.29.111.10 controlplane-01
172.29.111.11 worker-01

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
````

Vous pouvez lancer les pings pour vérifier que tout fonctionne :

````
ping controlplane-01
ping worker-01
````

### Installation binaire RKE2

Sur les deux VM lancez les commandes suivante pour installer les binaires rke2 (attention aucun service ne va se lancer) :

````
curl -sfL https://get.rke2.io | sudo sh -
````

### Mise en place du fichier config 

Sur la machine controlplane-01 : 

Créez et éditez le fichier config.yaml

````
mkdir -p /etc/rancher/rke2 
vi /etc/rancher/rke2/config.yaml
````

Dans le fichier mettez en place les éléments suivants :

````yaml
tls-san:
  - "controlplane-01"
ingress-controller:
- traefik
kubelet-arg:
- "kube-reserved=cpu=100m,memory=50Mi,ephemeral-storage=1Gi"
````

Une fois que le fichier est configuré, lancez les commandes suivantes pour démarrer le service rke2-server et procéder ainsi à l'installation :

````
systemctl enable rke2-server.service
systemctl start rke2-server.service
````

### Vérification de l'installation (toujours sur le controlplane-01)

Lancez la commande suivante :

````
wget https://github.com/okd-project/okd/releases/download/4.22.0-okd-scos.7/openshift-client-linux-4.22.0-okd-scos.7.tar.gz

tar -xvf openshift-client-linux-4.22.0-okd-scos.7.tar.gz

cp kubectl /usr/bin
````

Afin de récupérer les identifiants lancez les commandes suivantes :

````
mkdir ~/.kube
cp /etc/rancher/rke2/rke2.yaml ~/.kube/config
````

Ensuite la commande suivante devrait fonctionner :

````
kubectl get pod -A 
````

### Ajout d'un worker (ou agent)

Sur le controlplane, récupérez le token d'authentification d'un noeud au cluster :

````
cat /var/lib/rancher/rke2/server/node-token
````

Editez le fichier config.yaml avec ce token et l'adresse du controlplane-01:

````
mkdir -p /etc/rancher/rke2 
vi /etc/rancher/rke2/config.yaml
````

````
server: https://controlplane-01:9345
token: <token from server node>
tls-san:
  - "worker-01"
kubelet-arg:
- "kube-reserved=cpu=100m,memory=50Mi,ephemeral-storage=1Gi"
````

Enfin installez l'agent :

````
systemctl enable rke2-agent.service
systemctl start rke2-agent.service
````

## Installation de rancher manager 

Sur le controlplane-01 :

Dans un premier temps on va installer helm 

````
curl -fsSL https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
````
Installation de cert-manager en prérequis 

````
helm repo add jetstack https://charts.jetstack.io --force-update
helm repo update
helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --set crds.enabled=true
````

Installation rancher manager

````
helm repo add rancher-stable https://releases.rancher.com/server-charts/stable

helm repo update
````

````
helm install rancher rancher-stable/rancher \
  --namespace cattle-system \
  --create-namespace \
  --set hostname=rancher.local \
  --set bootstrapPassword=<votremdp> \
  --set replicas=1
````

Pour vous connecter à l'url rancher.local, modifiez le ficher hosts de votre machine hôte. Attention ouvrez le en tant qu'administrateur. (Moi par exemple j'ouvre notepadd++ en tant qu'admin et j'ouvre le fichier à partir de l'onglet "ouvrir")

C:/Windows/System32/drivers/etc/hosts

## Installation d'ArgoCD 

Dans un premier temps nous devons créer un fichier de valeur pour initialiser notre ArgoCD : 

````
vi argocd-values.yaml
````

Vous pouvez mettre les valeurs suivantes :

````yaml
global:
  domain: argocd.local

configs:
  params:
    server.insecure: true
    redis.compression: none

server:
  ingress:
    enabled: true
    ingressClassName: nginx
````

On peut désormais installer la solution (attention ça sera dans le namespace local):

````
helm install argo-cd oci://ghcr.io/argoproj/argo-helm/argo-cd --version 10.1.4 -f argocd-values.yaml --create-namespace -n argocd
````

# EXO RBAC 

## Pour ceux qui n'ont pas d'environement fonctionnel :

## Connection au cluster Openshift
Pour vous connecter au cluster Openshift mis à disposition :

### Pour trouver les sources rendez-vous dans la release suivante : https://github.com/okd-project/okd/releases/tag/4.19.0-okd-scos.19

### Pour Windows x86

Téléchargez sur votre machine le paquet OKD suivant : 
https://github.com/okd-project/okd/releases/download/4.19.0-okd-scos.19/openshift-client-windows-4.19.0-okd-scos.19.zip

Copiez le fichier oc.exe dans C:\Windows\System32

### Pour Linux x86
Téléchargez sur votre machine le paquet OKD suivant : 
https://github.com/okd-project/okd/releases/download/4.19.0-okd-scos.19/openshift-client-linux-4.19.0-okd-scos.19.tar.gz

Dézippez et copiez le fichier oc dans /urs/bin
```
tar -xvf openshift-client-linux-4.19.0-okd-scos.19.tar.gz
cp oc /usr/bin/
```

### Pour MAC

Télécharger le client suivant 

https://github.com/okd-project/okd/releases/download/4.19.0-okd-scos.19/openshift-install-mac-arm64-4.19.0-okd-scos.19.tar.gz pour ARM

https://github.com/okd-project/okd/releases/download/4.19.0-okd-scos.19/openshift-install-mac-4.19.0-okd-scos.19.tar.gz pour x86

Mettez le fichier oc dans le path.

Ou alors :
```
brew install openshift-cli
```


## Connexion au cluster (pour tous)

Allez sur l'url https://console-openshift-console.apps.openshift.kakor.ovh authentifiez-vous avec l'utilisateur ipi-gp-1 en choisisant "KeystoneIDP".

Une fois authentifié (attention à ne pas recharger la page même si l'affichage prends du temps), allez en haut à droite de la page et sélectionnez l'option "Copy login Command" et réauthentifiez vous. 

Cliquez sur le lien Display Token et copiez dans votre terminal sur VScode la ligne de commande qui à été donné avec oc. 

Pour vérifier que vous êtes authentifiés, lancez la commande ```oc get pod```

Créez votre propre namespace pour l'exercice via la commande ````oc new-project <votrenom>````

##Pour tout le monde

Dans un premier temps créez un conteneur avec une image qui va contenir bitnami/kubectl (avec la policy ifnotpresent sur mon cluster). Attention cette image contient uniquement l'invite de commande kubectl, donc il va falloir la laisser tourner via la commande sleep 3600 par exemple au démarrage. 

Lancez la commande ````kubectl get pod```` et confirmez que vous n'avez pas les droits pour lister les pods du namespace. 

Créez un role et un rolebinding qui permettrons au serviceaccount monté dans le conteneur de lister les pods (get, watch, list) 

Vérifiez le bon fonctionnement 

De même faites en sorte que cet utilisateur puisse lister les nodes (le clusterrole devra avoir un nom unique si vous êtes dans mon cluster)

Vérifiez le bon fonctionnement. 

Supprimer les roles binding et cluster rolebinding précédents. 

Ajoutez les droits de get, watch et list des pods dans le clusterrole, ajoutez ensuite un role binding et vérifier que vous avez accès uniquement aux pods du namespace. 
