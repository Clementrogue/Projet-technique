# CloudShop – Projet Kubernetes

## Description

CloudShop est une application e-commerce simplifiée déployée sur Kubernetes.  
Elle est composée de trois services principaux :

- Frontend (Nginx)
- Backend (API REST)
- Base de données PostgreSQL

L’objectif du projet est de démontrer la mise en place complète d’une architecture Kubernetes sécurisée et conforme aux bonnes pratiques.

---

## Architecture

L’architecture de l’application repose sur le flux suivant :

Internet → Ingress NGINX → Frontend → Backend → PostgreSQL (PVC)

- L’Ingress NGINX expose l’application en HTTPS
- Le Frontend sert l’interface web
- Le Backend gère la logique applicative
- La base PostgreSQL stocke les données de manière persistante

---

## Namespace

L’ensemble des ressources Kubernetes est isolé dans un namespace dédié :

cloudshop

---

## Déploiement du projet

### Prérequis

- Docker
- Minikube
- kubectl
- Linux / WSL / macOS

### Démarrage du cluster

minikube start --driver=docker  
minikube addons enable ingress

### Création du namespace

kubectl create namespace cloudshop  
kubectl config set-context --current --namespace=cloudshop

### Déploiement de l’application

Depuis la racine du projet :

kubectl apply -f .

---

## Accès à l’application

### Configuration DNS locale

Ajouter l’entrée suivante dans le fichier hosts :

<IP_MINIKUBE> cloudshop.local

Exemple :

192.168.49.2 cloudshop.local

### URLs

Frontend :  
https://cloudshop.local

Backend API :  
https://cloudshop.local/api

Le certificat TLS est auto-signé (avertissement navigateur normal).

---

## Sécurité réseau – NetworkPolicies

Une politique Zero Trust est appliquée.

Tout le trafic est bloqué par défaut, seuls les flux nécessaires sont autorisés.

NetworkPolicies en place :

- deny-all
- allow-frontend-to-backend
- allow-backend-to-db
- allow-ingress-nginx-to-frontend
- allow-ingress-nginx-to-backend
- allow-dns-egress

Vérification :

kubectl get networkpolicy

---

## Sécurité Kubernetes – RBAC

Deux ServiceAccounts sont utilisés :

- frontend-sa
- backend-sa

Règles appliquées :

- Le backend peut lire uniquement les ConfigMaps et Secrets nécessaires
- Le frontend ne dispose d’aucun accès sensible
- Le principe du moindre privilège est respecté

Test RBAC :

kubectl auth can-i get secret/test-secret --as=system:serviceaccount:cloudshop:backend-sa

---

## Persistance des données

La base PostgreSQL utilise un PersistentVolumeClaim.

kubectl get pvc  
kubectl describe pvc db-pvc

Les données sont conservées même après redémarrage du pod.

---

## Haute disponibilité

- Frontend : 2 replicas
- Backend : 2 replicas
- Répartition de charge assurée par les Services Kubernetes

---

## Tests internes

Frontend vers Backend :

kubectl run fe-test -n cloudshop -it --rm --image=curlimages/curl --restart=Never -- curl http://backend-svc:5000/api

Backend vers base de données :

kubectl run be-test -n cloudshop -it --rm --image=busybox:1.36 --restart=Never -- nc -zv db-svc 5432

---

## Rolling Update

kubectl rollout restart deployment/backend  
kubectl rollout status deployment/backend

---

## Taints et Tolerations

- Le node hébergeant la base de données est tainté
- Seul le pod PostgreSQL possède la toleration correspondante
- La base ne peut pas être planifiée sur un autre node

---

## Nettoyage

kubectl delete namespace cloudshop  
minikube stop

---

## Objectifs pédagogiques atteints

- Architecture microservices Kubernetes
- Sécurité réseau avec NetworkPolicies
- Sécurité des accès avec RBAC
- Persistance des données
- Exposition HTTPS via Ingress
- Respect des bonnes pratiques Kubernetes

---
