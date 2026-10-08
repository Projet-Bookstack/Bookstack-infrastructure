# Bookstack-infrastructure

Infrastructure as Code du projet **Bookstack** : provisionnement du socle cloud avec **Terraform** et configuration des machines avec **Ansible**, le tout orchestré par une pipeline CI/CD (`.gitlab-ci.yml`).

> Application déployée : [Projet-Bookstack/Bookstack](https://github.com/Projet-Bookstack/Bookstack)

## Sommaire

- [Aperçu](#aperçu)
- [Architecture](#architecture)
- [Structure du dépôt](#structure-du-dépôt)
- [Prérequis](#prérequis)
- [Démarrage rapide](#démarrage-rapide)
- [Configuration](#configuration)
- [Pipeline CI/CD](#pipeline-cicd)
- [Configuration des serveurs avec Ansible](#configuration-des-serveurs-avec-ansible)
- [Sécurité](#sécurité)
- [Contribuer](#contribuer)

## Aperçu

Ce dépôt décrit l'ensemble de l'infrastructure nécessaire pour héberger l'application Bookstack :

- réseau et accès sortant (NAT) ;
- point d'entrée public via un **Application Load Balancer** ;
- accès d'administration via un **bastion** ;
- **cluster** applicatif ;
- base de données managée **RDS** ;
- stockage **S3** ;
- **runner** CI pour exécuter les pipelines ;
- gestion de l'**état Terraform** à distance.

## Architecture

```
Internet
   │
   ▼
[ ALB ]  ──►  [ Cluster applicatif ]  ──►  [ RDS ]
                     │                        
                     └──────────────►  [ S3 ]

Administrateur ──► [ Bastion ] ──► machines privées (via NAT pour l'accès sortant)
```

> ⚠️ Schéma simplifié : à ajuster selon l'architecture réelle (sous-réseaux, zones de disponibilité, groupes de sécurité…).

## Structure du dépôt

| Fichier / dossier | Rôle |
|---|---|
| `main.tf` | Configuration principale (provider, réseau de base) |
| `variables.tf` | Variables d'entrée du projet |
| `state.tf` | Backend / stockage de l'état Terraform |
| `alb.tf` | Application Load Balancer |
| `bastion.tf` | Hôte bastion pour l'administration |
| `cluster.tf` | Cluster qui héberge l'application |
| `nat.tf` | Accès sortant des ressources privées |
| `rds.tf` | Base de données managée |
| `s3.tf` | Buckets de stockage |
| `runner.tf` | Runner CI/CD |
| `ansible/` | Playbooks et rôles de configuration des serveurs |
| `.gitlab-ci.yml` | Définition de la pipeline CI/CD |
| `.terraform.lock.hcl` | Verrouillage des versions de providers |

## Prérequis

- [Terraform](https://developer.hashicorp.com/terraform/install) 
- [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/) 
- Un compte cloud (AWS) avec des identifiants ayant les droits nécessaires
- Une paire de clés SSH pour accéder au bastion

## Démarrage rapide

```bash
# 1. Cloner le dépôt
git clone https://github.com/Projet-Bookstack/Bookstack-infrastructure.git
cd Bookstack-infrastructure

# 2. Configurer les identifiants cloud (exemple AWS)
export AWS_ACCESS_KEY_ID="..."
export AWS_SECRET_ACCESS_KEY="..."
export AWS_DEFAULT_REGION="eu-west-3"

# 3. Initialiser Terraform
terraform init

# 4. Vérifier le plan d'exécution
terraform plan

# 5. Appliquer
terraform apply
```

Pour détruire l'infrastructure :

```bash
terraform destroy
```

## Configuration

Les variables sont déclarées dans `variables.tf`. Pour les surcharger, créez un fichier `terraform.tfvars` (non versionné) :

```hcl
# terraform.tfvars (exemple)
region        = "eu-west-3"
project_name  = "bookstack"
# ...
```

| Variable | Description | Valeur par défaut |
|---|---|---|
| _à compléter_ | _à compléter_ | _à compléter_ |

## Pipeline CI/CD

La pipeline définie dans `.gitlab-ci.yml` automatise les étapes habituelles :

1. validation / formatage (`terraform validate`, `terraform fmt`) ;
2. plan ;
3. application (`terraform apply`) ;
4. configuration des serveurs via Ansible.

> Détaillez ici les stages réels, les variables CI requises et les branches qui déclenchent un déploiement.

## Configuration des serveurs avec Ansible

Le dossier `ansible/` contient la configuration appliquée aux machines une fois provisionnées.

```bash
cd ansible
ansible-playbook -i inventory playbook.yml
```

> Adaptez le nom de l'inventaire et du playbook à ceux présents dans le dossier.

## Sécurité

- Ne **jamais** commiter de secrets (clés d'accès, mots de passe RDS, clés privées SSH).
- Utiliser les variables CI protégées ou un gestionnaire de secrets.
- Restreindre l'accès SSH au bastion (liste d'IP autorisées).
- Le fichier d'état Terraform peut contenir des données sensibles : il doit être stocké dans un backend distant chiffré.

## Contribuer

1. Créer une branche : `git checkout -b feature/ma-modification`
2. Formater le code : `terraform fmt -recursive`
3. Valider : `terraform validate`
4. Ouvrir une Pull/Merge Request décrivant le changement et le résultat de `terraform plan`.

## Licence

À préciser.
