# infr030-P7

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

Allez sur l'url https://console-openshift-console.apps.openshift.kakor.ovh authentifiez-vous avec l'utilisateur ipi-gp-x (le x étant le numéro de groupe) en choisisant "KeystoneIDP".

Une fois authentifié (attention à ne pas recharger la page même si l'affichage prends du temps), allez en haut à droite de la page et sélectionnez l'option "Copy login Command" et réauthentifiez vous. 

Cliquez sur le lien Display Token et copiez dans votre terminal sur VScode la ligne de commande qui à été donné avec oc. 

Pour vérifier que vous êtes authentifiés, lancez la commande ```oc get pod```

## Exo 1) Création de votre premier pod

Via un fichier yaml ou une commande impérative ( oc run <monpod>... ), créez votre premier pod nginx. 

Celui-ci doit utiliser l'image suivante: harbor.kakor.ovh/public/nginx-rootless:latest

Afin de confirmer que le pod existe, utilisez la commande ```oc get pod```

``` oc logs <nomduconteneur> ``` => permet d'identifier les logs du conteneur et ainsi trouver des réponses à des problèmes techniques. 
## Correction 1) 

oc run nginx --image=harbor.kakor.ovh/public/nginx-rootless:latest

oc apply -f .\exo1\pod.yaml

Pour décrire le pod :

oc describe pod nginx

Pour executer des commandes dans le pod

oc exec -it nginx -- /bin/bash

ctrl + d pour sortir du conteneur ou exit

netstat -a pour vérifier les ports d'écoute (ou curl avec de la chance)

## Exo 2) 

Après avoir supprimé le pod de l'exo d'avant. Créez un nouveau pod avec l'image harbor.kakor.ovh/public/nginx:latest

Ce pod ne devrait pas fonctionner, d'après les logs expliquez pourquoi cela ne fonctionne pas. 

Corrigez cette erreur en mettant en place le security context suivant :

securityContext:
  allowPrivilegeEscalation: false