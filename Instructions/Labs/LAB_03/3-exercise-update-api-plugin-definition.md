---
lab:
  title: "Exercice\_2\_: mettre à jour la définition du plug-in d’API"
  module: 'LAB 03: Use Adaptive Cards to show data in API plugins for declarative agents'
---

# Exercice 2 : mettre à jour la définition du plug-in d’API

L’étape suivante consiste à mettre à jour la définition du plug-in d’API avec des cartes adaptatives que Copilot doit utiliser pour afficher les données de l’API aux utilisateurs.

### Durée de l’exercice

- **Durée estimée :** 10 minutes

## Tâche 1 : ajouter une carte adaptative pour afficher un plat

Dans Visual Studio Code :

1. Ouvrez le fichier **cards/dish.json** et copiez son contenu.
1. Ouvrez le fichier **appPackage/ai-plugin.json**.
1. Dans la propriété **functions.getDishes.capabilities.response_semantics**, ajoutez une nouvelle propriété nommée **static_template** et définissez la valeur du **corps** sur le contenu de **dish.json**.
1. L’extrait de code complet se présente ainsi :

    ```json
    "static_template": {
      "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
      "type": "AdaptiveCard",
      "version": "1.5",
      "body": [
          {
            "type": "Container",
            "items": [
              {
                "type": "Image",
                "url": "${image_url}",
                "size": "large"
              },
              {
                "type": "TextBlock",
                "text": "${name}",
                "weight": "Bolder"
              },
              {
                "type": "TextBlock",
                "text": "${description}",
                "wrap": true
              },
              {
                "type": "TextBlock",
                "text": "Allergens: ${if(count(allergens) > 0, join(allergens, ', '), 'none')}",
                "weight": "Lighter"
              },
              {
                "type": "TextBlock",
                "text": "**Price:** €${formatNumber(price, 2)}",
                "weight": "Lighter",
                "spacing": "None"
              }
            ]
          }
      ]
    }
    ```

1. Enregistrez les changements apportés.

## Tâche 2 : ajouter un modèle de carte adaptative pour afficher le résumé de la commande

Dans Visual Studio Code :

1. Ouvrez le fichier **cards/order.json** et copiez son contenu.
1. Ouvrez le fichier **appPackage/ai-plugin.json**.
1. Dans la propriété **functions.placeOrder.capabilities.response_semantics**, ajoutez une nouvelle propriété nommée **static_template** et définissez son contenu sur la carte adaptative.
1. Le fichier complet ressemble à ceci :

    ```json
    "static_template": {
      "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
      "type": "AdaptiveCard",
      "version": "1.5",
      "body": [
          {
            "type": "TextBlock",
            "text": "Order Confirmation 🤌",
            "size": "Large",
            "weight": "Bolder",
            "horizontalAlignment": "Center"
          },
          {
            "type": "Container",
            "items": [
              {
                "type": "TextBlock",
                "text": "Your order has been successfully placed!",
                "weight": "Bolder",
                "spacing": "Small"
              },
              {
                "type": "FactSet",
                "facts": [
                  {
                    "title": "Order ID:",
                    "value": "${order_id} "
                  },
                  {
                    "title": "Status:",
                    "value": "${status}"
                  },
                  {
                    "title": "Total Price:",
                    "value": "€${formatNumber(total_price, 2)}"
                  }
                ]
              }
            ]
          }
        ]
    }
    ```

1. Enregistrez vos modifications.
