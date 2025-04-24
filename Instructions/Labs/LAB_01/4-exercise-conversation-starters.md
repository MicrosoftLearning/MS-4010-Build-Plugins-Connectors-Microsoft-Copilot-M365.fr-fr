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
           "title": "Product information",
           "text": "Tell me about Eagle Air"
       },
       {
           "title": "Returns policy",
           "text": "What is the returns policy?"
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
           "title": "Repair information",
           "text": "Can you provide information on how to get a product repaired?"
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
  "name": "Product support",
  "description": "Product support agent that can help answer customer queries about Contoso Electronics products",
  "instructions": "$[file('instruction.txt')]",
  "capabilities": [
    {
      "name": "OneDriveAndSharePoint",
      "items_by_url": [
        {
          "url": "https://{tenant}-my.sharepoint.com/personal/{user}/Documents/Products"
        }
      ]
    }
  ],
  "conversation_starters": [
    {
      "title": "Product information",
      "text": "Tell me about Eagle Air"
    },
    {
      "title": "Returns policy",
      "text": "What is the returns policy?"
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
      "title": "Repair information",
      "text": "Can you provide information on how to get a product repaired?"
    },
    {
      "title": "Contact support",
      "text": "How can I contact support for help?"
    }
  ]
}
```

## Tâche 2 : tester l’agent déclaratif dans Microsoft 365 Copilot

Ensuite, chargez vos modifications et démarrez une session de débogage.

Dans Visual Studio Code :

1. Dans la **barre d’activité**, ouvrez l’extension **Teams Toolkit**.
1. Dans la section **Cycle de vie**, sélectionnez **Approvisionner**.
1. Attendez la fin du chargement.
1. Dans la **barre d’activité**, basculez vers la vue **Exécuter et Déboguer**.
1. Sélectionnez le bouton **Démarrer le débogage** en regard de la liste déroulante de la configuration, ou appuyez sur <kbd>F5</kbd>. Une nouvelle fenêtre de navigateur s’ouvre et accède à Microsoft 365 Copilot.

Ensuite, testez votre agent déclaratif dans Microsoft 365 Copilot et validez les résultats.

Continuez dans le navigateur web :

1. Dans **Microsoft 365 Copilot**, sélectionnez l’icône en haut à droite pour **développer le panneau latéral de Copilot**.
1. Recherchez **Support technique** dans la liste des agents et sélectionnez-le pour entrer dans l’expérience immersive afin de discuter directement avec l’agent. Notez que les amorces de conversation que vous avez définies dans le manifeste s’affichent dans l’interface utilisateur.

![Capture d’écran de l’agent déclaratif de support technique dans l’expérience immersive avec des amorces de conversation personnalisées dans Microsoft Edge.](../media/LAB_01/test-conversation-starters.png)

Fermez le navigateur pour arrêter la session de débogage dans Visual Studio Code.