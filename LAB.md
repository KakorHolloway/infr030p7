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