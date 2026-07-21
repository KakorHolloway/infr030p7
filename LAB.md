# Déploiement de l'environnement RKE2

## Objectif

L'objectif de ce TP est de vous montrer comment on peut installer simplement un cluster Kubernetes via RKE2 et rancher. 

## Création des VMS :

Provisionnez 2 VMs avec ses prérequis :

- Ubuntu server 2022 ou plus (2024 dans mon cas) https://ubuntu.com/download/server/thank-you?version=22.04.5&architecture=amd64&lts=true
- au moins 2vCPU (4 recommandés)
- au moins 4gb de RAM chacune
- 20 gb de disque **si il n'y a qu'une seule partition (pas de lvm)** ou sinon 127 gb en dynamique. 
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


