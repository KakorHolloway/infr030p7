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

```yaml
securityContext:
  allowPrivilegeEscalation: true
```

oc replace --force -f .\exo2\pod.yaml

## Exemple de création de yaml rapide

oc run nginx --image=nginx --port 80 **--dry-run=client -o yaml** > fichierpod.yaml

## Exo 3) Mise en place des services

Vous allez devoir créer 2 pods : 

Un premier pod sera un pod nginx avec l'image harbor.kakor.ovh/public/nginx:latest

Le second sera un pod avec l'image harbor.kakor.ovh/public/curl:latest. Ce pod aura pour objectif de vérifier si vous pouvez tester d'accéder au serveur nginx du premier pod. Attention, à la création de ce mod une commande (regardez l'option command dans la documentation Kubernetes) devra être lancée pour que le pod ne s'arrête pas automatiquement. 

Ensuite, créez le service nommé nginx qui va permettre au pod curl de lancer la comande de test pour accéder au pod nginx. 

Vérifiez que tout fonctionne correctement. 

## Correction 3) 

Création du service :

oc create service clusterip nginx --tcp:80:80 --dry-run -o yaml

Aller sur le pod curl :

oc exec -it curl -- /bin/sh

## Exo 4) Exposition de votre application 

En gardant les éléments créés lors de l'exercice 3, ajoutez un ingress en réadaptant l'exemple ci-dessous pour que ce dernier puisse vous permettre d'accéder en https à l'application nginx :

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: frontend
spec:
  rules:
    - host: <nomquevoussouhaitez>.apps.openshift.kakor.ovh
      http:
        paths:
        - path: ''
          pathType: ImplementationSpecific
          backend:
            service:
              name: frontend
              port:
                number: 443
  tls:
  - {}
```

## Exo 5) 

Supprimez les différents objets Kubernetes que vous avez créé jusqu'à présent. 

Créez un nouveau pod basé sur l'image mariadb suivante : harbor.kakor.ovh/public/mariadb:latest

Ce dernier ne fonctionnera pas correctement, via les logs identifiez l'erreur. 

Corrigez l'erreur sur l'image mariadb en mettant les bonnes variables d'environnement. 

