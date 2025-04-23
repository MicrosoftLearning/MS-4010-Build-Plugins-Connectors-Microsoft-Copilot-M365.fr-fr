---
lab:
  title: "Exercice\_2\_: créer un agent déclaratif et intégrer le connecteur Graph"
  module: 'LAB 04: Add custom knowledge to declarative agents using Microsoft Graph connectors and Visual Studio Code'
---

# Exercice 2 : créer un agent déclaratif et intégrer le connecteur Graph

Dans cet exercice, vous allez créer un agent déclaratif à partir de zéro et le configurer pour utiliser la connexion externe comme source de base.

### Durée de l’exercice

- **Durée estimée :** 10 minutes

## Tâche 1 : créer un agent déclaratif

Une façon de créer un agent déclaratif consiste à utiliser Teams Toolkit. Le kit de ressources Teams propose un projet de modèle pour la création d’agents déclaratifs, ce qui vous donne un excellent point de départ pour configurer les paramètres de votre agent et inclure des fonctionnalités supplémentaires.

Pour créer un agent déclaratif, ouvrez Visual Studio Code.

Dans Visual Studio Code :

1. Dans la **barre d’activité** (barre latérale), ouvrez l’extension **Teams Toolkit**.
1. Dans le panneau Teams Toolkit, sélectionnez le bouton **Créer une application**.
1. Dans la boîte de dialogue **Nouveau projet**, choisissez l’option **Agent**.
1. Dans la boîte de dialogue suivante, choisissez l’option **Agent déclaratif**.
1. N’ajoutez pas de plug-in en sélectionnant l’option **Aucun plug-in**.
1. Sélectionnez un dossier dans lequel vous souhaitez stocker le projet sur votre ordinateur.
1. Nommez votre projet `da-it-policies`.

## Tâche 2 : configurer les instructions de l’agent déclaratif

Teams Toolkit crée un projet d’agent déclaratif. Pour l’adapter à votre scénario, mettez à jour la description et les instructions de l’agent.

Dans Visual Studio Code :

1. Ouvrez le fichier **appPackage/declarativeAgent.json**.
1. Mettez à jour la valeur de la propriété **name** sur `Contoso IT policies`.
1. Mettez à jour la valeur de la propriété **description** sur `Assistant specialized in Contoso IT policies`.
1. Enregistrez les changements apportés.
1. Le contenu du fichier mise à jour se présente comme suit :

    ```json
    {
      "$schema": "https://aka.ms/json-schemas/copilot/declarative-agent/v1.0/schema.json",
      "version": "v1.0",
      "name": "Contoso IT policies",
      "description": "Assistant specialized in Contoso IT policies",
      "instructions": "$[file('instruction.txt')]",
    }
    ```

1. Ouvrez le fichier **appPackage/instruction.txt**.
1. Mettez à jour le contenu pour :

    ```text
    You are a helpful assistant that can answer questions from users in a friendly manner. You should take your time to respond. Your responses should be accurate and helpful. If you don't have the information, you should say so in your response. When answering follow-up questions, you should review the information you gathered from external sources, if you don't already have the information to give an accurate answer, you should search for more information. Only answer when you have the information to give an accurate response.
    ```

1. Enregistrez les changements apportés.

## Tâche 3 : intégrer le connecteur Microsoft Graph à un agent déclaratif

Après avoir créé un agent déclaratif, l’étape suivante consiste à l’intégrer à un connecteur Microsoft Graph afin qu’il puisse accéder aux données externes.

Dans Visual Studio Code :

1. Ouvrez le fichier **appPackage/declarativeAgent.json**.
1. Après la propriété **instructions**, ajoutez une nouvelle propriété nommée **fonctionnalités** avec le code suivant :

    ```json
    { 
      "$schema": "https://aka.ms/json-schemas/copilot/declarative-agent/v1.0/schema.json",
      "version": "v1.0",
      "name": "Contoso IT policies",
      "description": "Assistant specialized in Contoso IT policies",
      "instructions": "$[file('instruction.txt')]",
      "capabilities": [
        {
          "name": "GraphConnectors",
          "connections": [ 
            {
              "connection_id": ""
            }
          ]
        }
      ]
    } 
    ```

1. Dans la propriété **connection_id**, spécifiez `policieslocal` en tant qu’ID de la connexion externe. `policieslocal` est l’ID de la connexion externe que le connecteur Graph a créé aux étapes précédentes.
1. Le contenu du fichier mise à jour se présente comme suit :

    ```json
    { 
      "$schema": "https://aka.ms/json-schemas/copilot/declarative-agent/v1.0/schema.json",
      "version": "v1.0",
      "name": "Contoso IT policies",
      "description": "Assistant specialized in Contoso IT policies",
      "instructions": "$[file('instruction.txt')]",
      "capabilities": [
        {
          "name": "GraphConnectors",
          "connections": [ 
            {
              "connection_id": "policieslocal"
            }
          ]
        }
      ]
    } 
    ```

1. Enregistrez les changements apportés.
