------------------------------------------------------------------------------------------------------
ATELIER API-DRIVEN INFRASTRUCTURE
------------------------------------------------------------------------------------------------------
L’idée en 30 secondes : **Orchestration de services AWS via API Gateway et Lambda dans un environnement émulé**.  
Cet atelier propose de concevoir une architecture **API-driven** dans laquelle une requête HTTP déclenche, via **API Gateway** et une **fonction Lambda**, des actions d’infrastructure sur des **instances EC2**, le tout dans un **environnement AWS simulé avec LocalStack** et exécuté dans **GitHub Codespaces**. L’objectif est de comprendre comment des services cloud serverless peuvent piloter dynamiquement des ressources d’infrastructure, indépendamment de toute console graphique.Cet atelier propose de concevoir une architecture API-driven dans laquelle une requête HTTP déclenche, via API Gateway et une fonction Lambda, des actions d’infrastructure sur des instances EC2, le tout dans un environnement AWS simulé avec LocalStack et exécuté dans GitHub Codespaces. L’objectif est de comprendre comment des services cloud serverless peuvent piloter dynamiquement des ressources d’infrastructure, indépendamment de toute console graphique.
  
-------------------------------------------------------------------------------------------------------
Séquence 1 : Codespace de Github
-------------------------------------------------------------------------------------------------------
Objectif : Création d'un Codespace Github  
Difficulté : Très facile (~5 minutes)
-------------------------------------------------------------------------------------------------------
RDV sur Codespace de Github : <a href="https://github.com/features/codespaces" target="_blank">Codespace</a> **(click droit ouvrir dans un nouvel onglet)** puis créer un nouveau Codespace qui sera connecté à votre Repository API-Driven.
  
---------------------------------------------------
Séquence 2 : Création de l'environnement AWS (LocalStack)
---------------------------------------------------
Objectif : Créer l'environnement AWS simulé avec LocalStack  
Difficulté : Simple (~5 minutes)
---------------------------------------------------

Dans le terminal du Codespace copier/coller les codes ci-dessous etape par étape :  

**Installation de l'émulateur LocalStack**  
```
sudo -i mkdir rep_localstack
```
```
sudo -i python3 -m venv ./rep_localstack
```
```
sudo -i pip install --upgrade pip && python3 -m pip install localstack && export S3_SKIP_SIGNATURE_VALIDATION=0
```
```
localstack start -d
```
**vérification des services disponibles**  
```
localstack status services
```
**Réccupération de l'API AWS Localstack** 
Votre environnement AWS (LocalStack) est prêt. Pour obtenir votre AWS_ENDPOINT cliquez sur l'onglet **[PORTS]** dans votre Codespace et rendez public votre port **4566** (Visibilité du port).
Réccupérer l'URL de ce port dans votre navigateur qui sera votre ENDPOINT AWS (c'est à dire votre environnement AWS).
Conservez bien cette URL car vous en aurez besoin par la suite.  

Pour information : IL n'y a rien dans votre navigateur et c'est normal car il s'agit d'une API AWS (Pas un développement Web type UX).

---------------------------------------------------
Séquence 3 : Exercice
---------------------------------------------------
Objectif : Piloter une instance EC2 via API Gateway
Difficulté : Moyen/Difficile (~2h)
---------------------------------------------------  
🚀 Atelier : API-Driven Infrastructure (LocalStack)
📋 Description du Projet
L'objectif de cet atelier est de concevoir une architecture Serverless capable de piloter dynamiquement des ressources d'infrastructure (instances EC2) via des appels API, le tout dans un environnement émulé avec LocalStack au sein de GitHub Codespaces.

L'idée centrale est de s'affranchir de toute console graphique (AWS Console) pour interagir avec le Cloud uniquement par le code et les requêtes HTTP.

🏗️ Architecture Cible
Le flux de données suit ce parcours :

Client : Envoie une requête HTTP POST avec une instruction JSON.

Endpoint API : Reçoit la requête sur le port 4566.

AWS Lambda : Fonction Python (boto3) qui traite l'instruction.

AWS EC2 : L'instance change d'état (start ou stop) selon l'ordre reçu.

🛠️ Configuration de l'Environnement
1. Initialisation de LocalStack
Dans le terminal du Codespace, exécutez les commandes suivantes pour démarrer l'émulateur avec les fonctionnalités avancées :


# Activation du Token LocalStack
```
export LOCALSTACK_AUTH_TOKEN="ls-vaCOfAxo-ciZu-koWe-3102-nOza5540ad00"
```
💻 Étape 2 : Création de l'Infrastructure Cible
1. Lancement de l'instance EC2
```
awslocal ec2 run-instances --image-id ami-03cf127a --count 1 --instance-type t2.micro
```
L'ID généré pour ce projet est : i-0824c42ba26583dab

🐍 Étape 3 : Développement de la Logique (Lambda & Boto3)
Fichier : lambda_function.py
Ce code utilise la bibliothèque Boto3 pour envoyer des ordres au SDK AWS simulé.

Python
import boto3
import os
import json

def lambda_handler(event, context):
    # Identifiant de notre instance cible
    INSTANCE_ID = "i-0824c42ba26583dab"
    
    # Configuration du client Boto3 pour pointer vers LocalStack
    endpoint = f"http://{os.environ['LOCALSTACK_HOSTNAME']}:4566"
    ec2 = boto3.client('ec2', endpoint_url=endpoint)
    
    # Extraction de l'action depuis le corps de la requête HTTP
    try:
        body = json.loads(event.get('body', '{}'))
        action = body.get('action', 'status')
    except:
        action = 'status'

    # Logique de pilotage
    if action == 'start':
        ec2.start_instances(InstanceIds=[INSTANCE_ID])
        res = f"Instance {INSTANCE_ID} démarrée avec succès."
    elif action == 'stop':
        ec2.stop_instances(InstanceIds=[INSTANCE_ID])
        res = f"Instance {INSTANCE_ID} arrêtée avec succès."
    elif action == 'status':
        data = ec2.describe_instances(InstanceIds=[INSTANCE_ID])
        etat = data['Reservations'][0]['Instances'][0]['State']['Name']
        res = f"L'état actuel de l'instance est : {etat}"
    else:
        res = "Erreur : Action non reconnue (utilisez start, stop ou status)."

    return {
        "statusCode": 200,
        "headers": {"Content-Type": "application/json"},
        "body": json.dumps({"message": res})
    }
🚀 Étape 4 : Déploiement et Pilotage API
1. Déployer la Lambda
Bash
zip function.zip lambda_function.py
```
awslocal lambda create-function \
    --function-name ControlEC2 \
    --runtime python3.9 \
    --zip-file fileb://function.zip \
    --handler lambda_function.lambda_handler \
    --role arn:aws:iam::000000000000:role/admin
 ```   
# Démarrage du service
localstack start -d
2. Définition de l'Endpoint
Pour faciliter les tests, nous utilisons une variable d'environnement pour l'URL de l'API :

```
export ENDPOINT="https://expert-space-funicular-7vww4jjxvgq4hqxw-4566.app.github.dev"
```
(Note : Assurez-vous que le port 4566 est réglé sur "Public" dans l'onglet PORTS).

🚀 Guide d'Utilisation (Pilotage par API)
Voici les commandes curl pour interagir avec l'infrastructure de manière programmable.

🟢 Démarrer l'instance EC2
Envoie l'ordre de démarrage à la Lambda :

```
curl -X POST $ENDPOINT/ -H "Content-Type: application/json" -d '{"action": "start"}'
```
🔴 Stopper l'instance EC2
Envoie l'ordre d'arrêt à la Lambda :

```
curl -X POST $ENDPOINT/ -H "Content-Type: application/json" -d '{"action": "stop"}'
```
🔍 Vérifier le Statut
Demande l'état actuel via l'API :

```
curl -X POST $ENDPOINT/ -H "Content-Type: application/json" -d '{"action": "status"}'
```
🧪 Vérification Technique (Preuve de Concept)
Pour valider que l'API a bien modifié l'infrastructure réelle, utilisez la commande suivante pour interroger directement le service EC2 de LocalStack :

```
awslocal ec2 describe-instances --instance-ids i-0824c42ba26583dab --query "Reservations[*].Instances[*].State.Name"
```
🧠 Défis Techniques Résolus
Gestion du format binaire : Utilisation de --cli-binary-format raw-in-base64-out pour permettre l'invocation correcte de la Lambda via l'AWS CLI.

Communication Inter-Services : Configuration de l' endpoint_url dans le script Python pour permettre à la Lambda de communiquer avec EC2 à l'intérieur du conteneur LocalStack.

Exposition Publique : Tunneling du port 4566 via GitHub Codespaces pour rendre l'API accessible à distance.  
  
![Screenshot Actions](API_Driven.png)   
  
---------------------------------------------------  
## Processus de travail (résumé)

1. Installation de l'environnement Localstack (Séquence 2)
2. Création de l'instance EC2
3. Création des API (+ fonction Lambda)
4. Ouverture des ports et vérification du fonctionnement

---------------------------------------------------
Séquence 4 : Documentation  
Difficulté : Facile (~30 minutes)
---------------------------------------------------
**Complétez et documentez ce fichier README.md** pour nous expliquer comment utiliser votre solution.  
Faites preuve de pédagogie et soyez clair dans vos expliquations et processus de travail.  
   
---------------------------------------------------
Evaluation
---------------------------------------------------
Cet atelier, **noté sur 20 points**, est évalué sur la base du barème suivant :  
- Repository exécutable sans erreur majeure (4 points)
- Fonctionnement conforme au scénario annoncé (4 points)
- Degré d'automatisation du projet (utilisation de Makefile ? script ? ...) (4 points)
- Qualité du Readme (lisibilité, erreur, ...) (4 points)
- Processus travail (quantité de commits, cohérence globale, interventions externes, ...) (4 points) 
