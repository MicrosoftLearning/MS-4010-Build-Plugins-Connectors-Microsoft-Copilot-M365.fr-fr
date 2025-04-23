---
lab:
  title: "Exercice\_2\_: configurer des connaissances personnalisées"
  module: 'LAB 01: Build a declarative agent for Microsoft 365 Copilot using Visual Studio Code'
---

# Exercice 2 : configurer des connaissances personnalisées

Dans cet exercice, vous allez utiliser Microsoft Learn comme source d’ancrage pour votre agent. Votre agent deviendra un expert sur Microsoft 365.

### Durée de l’exercice

- **Durée estimée :** 10 minutes

## Tâche 1 : configurer les données d’ancrage

Configurez le dossier OneDrive comme source de données de base dans le manifeste de l’agent déclaratif.

Dans Visual Studio Code :

1. Dans le dossier **appPackage**, ouvrez le fichier **declarativeAgent.json**.
1. Ajoutez l’extrait de code suivant au fichier après la définition **« instructions »**, en remplaçant **{URL}** par l’URL directe vers la page de destination de Microsoft 365 sur Microsoft Learn :

    ```json
    "capabilities": [
        {
            "name": "WebSearch",
            "sites": [
                {
                    "url": "{URL}"
                }
            ]
        }
    ]
    ```

1. Enregistrez les changements apportés.

Le fichier **declarativeAgent.json** doit être similaire à ceci :

```json
{
    "$schema": "https://developer.microsoft.com/json-schemas/copilot/declarative-agent/v1.0/schema.json",
    "version": "v1.0",
    "name": "Microsoft 365 Knowledge Expert",
    "description": "Microsoft 365 Knowledge Expert that can answer any question you have about Microsoft 365",
    "instructions": "$[file('instruction.txt')]",
    "capabilities": [
        {
            "name": "WebSearch",
            "sites": [
                {
                    "url": "https://learn.microsoft.com/microsoft-365/"
                }
            ]
        }
    ]
}
```

## Tâche 3 : mettre à jour les instructions personnalisées

Mettez à jour les instructions dans le manifeste de l’agent déclaratif pour donner à notre agent un contexte supplémentaire et de l’aider lorsqu’il répond aux questions des clients.

Dans Visual Studio Code :

1. Ouvrez le fichier **appPackage/instruction.txt** et mettez à jour le contenu avec :

    ```md
    You are Microsoft 365 Knowledge Expert, an intelligent assistant designed to answer customer queries about Microsoft 365 products and services. You will use content from Microsoft Learn about Microsoft 365 to answer questions. If you can't find the necessary information, you should suggest that the agent should reach out to the team responsible for further assistance. Your responses should be concise and always include a cited source.
    ```

1. Enregistrez les changements apportés.

## Tâche 4 : charger l’agent déclaratif dans Microsoft 365

Chargez vos modifications dans Microsoft 365.

Dans Visual Studio Code :

1. Dans la **barre d’activité**, ouvrez l’extension **Teams Toolkit**.
1. Dans la section **Cycle de vie**, sélectionnez **Configurer**, puis **Publie**r.
1. **Confirmez** que vous souhaitez envoyer une mise à jour pour le catalogue d’applications.
1. Attendez la fin de la publication des tâches.

## Tâche 5 : tester l’agent déclaratif dans Microsoft 365 Copilot

Testez votre agent déclaratif dans Microsoft 365 Copilot et valider les résultats. En continuant dans le navigateur de l’exercice précédent, actualisez la fenêtre (**F5**).

Tout d’abord, testons les instructions :

1. Dans **Microsoft 365 Copilot**, sélectionnez l’icône en haut à droite pour **développer le panneau latéral de Copilot**.
1. Recherchez **Expert en Microsoft 365** dans la liste des agents et sélectionnez-le pour entrer dans l’expérience immersive afin de discuter directement avec l’agent.
1. Demandez à l’agent de support technique **Que peux-tu faire ?** et envoyez l’invite.
1. Attendez la réponse. Notez comment la réponse est différente des instructions précédentes et reflète les nouvelles instructions.

    ![Capture d’écran de Microsoft 365 Copilot dans Microsoft Edge. Une réponse de l’agent expert en Microsoft 365 s’affiche pour expliquer ses fonctionnalités.](../media/LAB_01/test-m365-knowledge-expert.png)

Ensuite, testons les données de base.

1. Dans la zone de message, entrez **Parle-moi de la protection des données**, puis envoyez le message.
1. Attendez la réponse. Notez que la réponse contient des informations sur la protection des données. La réponse contient des citations et des références au site web spécifique utilisé pour générer la réponse.

    ![Capture d’écran de Microsoft 365 Copilot dans Microsoft Edge. Une réponse de l’agent expert en Microsoft 365 s’affiche avec des informations sur la protection des données dans Microsoft 365.](../media/LAB_01/test-m365-knowledge-expert-1.png)

Essayons quelques invites supplémentaires :

1. Dans la zone de message, entrez **Recommande un produit adapté à une communication en temps réel**.
1. Dans la boîte de message, entrez **Parle-moi des options de support pour Microsoft 365**.

Fermez le navigateur pour arrêter la session de débogage dans Visual Studio Code.
