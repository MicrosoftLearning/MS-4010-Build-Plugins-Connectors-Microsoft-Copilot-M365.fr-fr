---
lab:
  title: "Exercice\_3\_: ajouter des démarrages de conversation à votre agent déclaratif"
  module: 'LAB 01: Build a declarative agent for Microsoft 365 Copilot using Visual Studio Code'
---

# Exercice 3 : ajouter des démarrages de conversation à votre agent déclaratif

Dans cet exercice, vous allez mettre à jour l’agent déclaratif pour inclure des démarrages de conversation qui fournissent aux utilisateurs des exemples de prompts pour les aider à comprendre les types de questions qu’ils peuvent poser.

### Durée de l’exercice

- **Durée estimée :** 5 minutes

## Tâche 1 : ajouter des démarrages de conversation

Dans Visual Studio Code :

1. Dans le dossier **appPackage**, ouvrez le fichier **declarativeAgent.json**.
1. Ajoutez l’extrait de code suivant au fichier :

   ```json
   "conversation_starters": [
       {
           "title": "Microsoft 365",
           "text": "Tell me about Microsoft 365"
       },
       {
           "title": "Licensing",
           "text": "What licenses are available for Microsoft 365?"
       },
       {
           "title": "Product information",
           "text": "Can you provide information on a specific product?"
       },
       {
           "title": "Product troubleshooting",
           "text": "I'm having trouble with a product. Can you help me troubleshoot the issue?"
       },
       {
           "title": "Contact support",
           "text": "How can I contact support for help?"
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
    ],
  "conversation_starters": [
       {
           "title": "Microsoft 365",
           "text": "Tell me about Microsoft 365"
       },
       {
           "title": "Licensing",
           "text": "What licenses are available for Microsoft 365?"
       },
       {
           "title": "Product information",
           "text": "Can you provide information on a specific product?"
       },
       {
           "title": "Product troubleshooting",
           "text": "I'm having trouble with a product. Can you help me troubleshoot the issue?"
       },
       {
           "title": "Contact support",
           "text": "How can I contact support for help?"
       }
  ]
}
```

## Tâche 2 : tester l’agent déclaratif dans Microsoft 365 Copilot Chat

Ensuite, chargez vos modifications et démarrez une session de débogage.

Dans Visual Studio Code :

1. Dans la **barre d’activité**, ouvrez l’extension **Teams Toolkit**.
1. Dans la section **Cycle de vie**, sélectionnez **Configurer**, puis **Publie**r.
1. **Confirmez** que vous souhaitez envoyer une mise à jour pour le catalogue d’applications.
1. Attendez la fin du chargement.

Continuez dans le navigateur web :

1. Dans **Microsoft 365 Copilot**, sélectionnez l’icône en haut à droite pour **développer le panneau latéral de Copilot**.
1. Recherchez **Support technique** dans la liste des agents et sélectionnez-le pour entrer dans l’expérience immersive afin de discuter directement avec l’agent. Notez que les amorces de conversation que vous avez définies dans le manifeste s’affichent dans l’interface utilisateur.

![Capture d’écran de Microsoft Edge montrant l’agent déclaratif expert en Microsoft 365 dans l’expérience immersive avec des amorces de conversation personnalisées.](../media/LAB_01/test-conversation-starters.png)

Fermez le navigateur pour arrêter la session de débogage dans Visual Studio Code.