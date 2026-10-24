🚀 TP4 — Bases d'Ansible (Inventaire, Playbooks, Clés SSH, Rôles)

Automatisation de l'infrastructure avec Ansible : configuration d'un serveur web, installation de Docker, Portainer, Jenkins et déploiement de conteneurs via des rôles réutilisables.

Étudiante : Narjess Dabbech — Master DevOps & Cloud (M2) — 2026

📑 Sommaire
Objectifs
Architecture
Environnement
Partie 1 : Inventaire et Configuration
Partie 2 : Premier Playbook (Apache)
Partie 3 : Authentification par Clé SSH
Partie 4 & 5 : Rôles Ansible
Difficultés rencontrées
1. 🎯 Objectifs
Décrire l'architecture d'Ansible (nœud de contrôle, nœuds gérés, inventaire, modules).
Écrire un inventaire, un fichier ansible.cfg et un premier playbook idempotent.
Passer d'une authentification SSH par mot de passe à une authentification par clé.
Automatiser l'installation de Jenkins, Docker, Portainer et d'un conteneur applicatif via des rôles.
2. 🏗️ Architecture
Rôle	Machine	IP
Nœud de contrôle	devsecops-vm (Ubuntu 22.04)	192.168.56.10
Nœud géré	master-2 (Ubuntu 22.04)	192.168.56.20
3. 💻 Environnement
Ansible : core 2.17.14 (installé sur devsecops-vm)
Python : 3.10
OS : Ubuntu 22.04 LTS (les deux VM)
Connexion : SSH par clé ED25519 (générée sur le contrôleur)
4. 🔧 Partie 1 : Inventaire et Configuration

Structure du projet demo1/ :

text
demo1/
├── inventory.txt
├── ansible.cfg
├── index.html
├── webserver_install.yml
├── site.yml
├── captures/
└── roles/
    ├── docker_portainer_install/
    ├── jenkins_install/
    └── container_deployment/

inventory.txt :

ini
[webserver]
192.168.56.20

[nodes:children]
webserver

[nodes:vars]
ansible_user=master-2

ansible.cfg :

ini
[defaults]
inventory = ./inventory.txt
remote_user = master-2
host_key_checking = False
deprecation_warnings = False

Test de connexion (ping) :

bash
ansible all -m ping

Ping Ansible

5. 🌐 Partie 2 : Premier Playbook (Apache)

webserver_install.yml installe Apache, crée un dossier, copie un site web et démarre le service. Le port est modifié sur 8081 car le port 80 est déjà utilisé par Coolify/Traefik sur master-2.

Exécution :

bash
ansible-playbook webserver_install.yml -K

Vérification :

bash
curl http://192.168.56.20:8081
Premier passage

Exécution du playbook Apache

Deuxième passage — Idempotence (changed=0)

Idempotence

6. 🔑 Partie 3 : Authentification par Clé SSH

Génération de la clé :

bash
ssh-keygen -t ed25519 -C "ansible-control"

Copie de la clé sur le nœud géré :

bash
ssh-copy-id master-2@192.168.56.20

Une fois la clé en place, le playbook s'exécute sans -k (plus de mot de passe SSH demandé).

7. 🐳 Partie 4 & 5 : Rôles Ansible (Docker, Portainer, Jenkins, Nginx)

site.yml :

yaml
---
- name: Déployer la stack DevOps via des rôles
  hosts: webserver
  become: true
  gather_facts: true

  roles:
    - docker_portainer_install
    - jenkins_install
    - container_deployment

Rôles créés :

Rôle	Description	Accès
docker_portainer_install	Installe Docker et déploie Portainer	http://192.168.56.20:9000
jenkins_install	Déploie Jenkins dans un conteneur Docker	http://192.168.56.20:8085
container_deployment	Déploie un conteneur Nginx	http://192.168.56.20:8082

Exécution :

bash
ansible-playbook site.yml -K
Exécution des rôles

Exécution des rôles

Conteneurs déployés

Conteneurs

8. 🛠️ Difficultés rencontrées
Problème	Cause	Solution
Host key verification failed	La clé SSH de l'hôte a changé (VM recréée)	ssh-keygen -R 192.168.56.20 puis reconnexion manuelle
Conflit de clé GPG Jenkins (NO_PUBKEY)	Deux clés concurrentes dans /etc/apt/keyrings	Nettoyage des fichiers et réinstallation propre de la clé
Jenkins ne démarre pas (Start request repeated too quickly)	Jenkins 2.479.1 exige Java 17, mais le service utilisait Java 11	update-alternatives --set java puis configuration de JAVA_HOME
Port 80 occupé par Coolify	Apache ne peut pas démarrer sur le port 80	Modification du playbook pour utiliser le port 8081
Problème de RAM sur master-2	Coolify + Forgejo + Portainer + Nginx consomment beaucoup	Ajout de 4 Go de swap
Jenkins bloqué sur l'installation des plugins	Timeout réseau	Installation dans un conteneur Docker (jenkins/jenkins:lts-jdk17)
👤 Auteur

Narjess Dabbech — Master DevOps & Cloud — 2026 GitHub : @Narjess805
