---
lab:
  title: "Exercice\_1\_: télécharger le projet et créer une carte adaptative"
  module: 'LAB 03: Use Adaptive Cards to show data in API plugins for declarative agents'
---

# Exercice 1 : télécharger le projet et créer une carte adaptative

Commençons par créer des modèles de carte adaptative pour que l’agent affiche les données dans ses réponses. Pour générer le modèle de carte adaptative, vous allez utiliser les extensions Visual Studio Code du générateur d’aperçu de carte adaptative pour afficher facilement un aperçu de votre travail directement dans Visual Studio Code. L’utilisation de l’extension nous permet de créer un modèle de carte adaptative, avec des références aux données. Au moment du runtime, l’agent remplit l’espace réservé avec les données qu’il récupère à partir de l’API.

### Durée de l’exercice

- **Durée estimée :** 10 minutes

## Tâche 1: télécharger le projet de démarrage

Commencez par télécharger l’exemple de projet. Dans un navigateur Web :

1. Accédez à [https://github.com/microsoft/learn-declarative-agent-api-plugin-adaptive-cards-typescript](https://github.com/microsoft/learn-declarative-agent-api-plugin-adaptive-cards-typescript).
  1. Suivez les étapes pour [télécharger le code source du dépôt](https://docs.github.com/repositories/working-with-files/using-files/downloading-source-code-archives#downloading-source-code-archives-from-the-repository-view) sur votre ordinateur.
  1. Extrayez le contenu du fichier zip téléchargé dans votre **dossier Documents**.
  1. Ouvrez le dossier  dans Visual Studio Code.

L’exemple de projet est un projet Teams Toolkit qui inclut un agent déclaratif avec une action générée avec un plug-in d’API. Le plug-in d’API se connecte à une API anonyme s’exécutant sur Azure Functions également incluse dans le projet. L’API appartient à un restaurant italien fictif et vous permet de parcourir le menu d’aujourd’hui et de passer des commandes.

## Tâche 2 : créer une carte adaptative pour un plat

Tout d’abord, créez une carte adaptative qui affiche des informations sur un seul plat.

Dans Visual Studio Code :

1. Dans l’affichage **Explorateur**, créez un dossier nommé **cartes**.
1. Dans le dossier **cartes**, créez un fichier nommé **dish.json**. Collez le contenu suivant qui représente une carte adaptative vide :

  ```json
  {
    "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
    "type": "AdaptiveCard",
    "version": "1.5",
    "body": []
  }
  ```

1. Avant de continuer, à partir de l’onglet **Extensions** de la barre d’activité, recherchez et installez l’extension **Prévisualisation de carte adaptative**, puis créez un fichier de données pour la carte adaptative :
  1. Ouvrez la palette de commandes en appuyant sur <kbd>CTRL</kbd>+<kbd>P</kbd>. Entrez `>Adaptive` pour rechercher des commandes liées à l’utilisation des cartes adaptatives.

    ![Capture d’écran des commandes liées à l’utilisation de cartes adaptatives dans Visual Studio Code.](../media/LAB_03/LAB_03/3-visual-studio-code-adaptive-card-commands.png)

  1. Dans la liste, choisissez **Carte adaptative : nouveau fichier de données**. Visual Studio Code crée un fichier nommé **dish.data.json**.
  1. Remplacez son contenu par une donnée qui représente un plat :

  ```json
  {
    "id": 4,
    "name": "Caprese Salad",
    "description": "Juicy vine-ripened tomatoes, fresh mozzarella, and fragrant basil leaves, drizzled with extra virgin olive oil and a touch of balsamic.",
    "image_url": "https://raw.githubusercontent.com/pnp/copilot-pro-dev-samples/main/samples/da-ristorante-api/assets/caprese_salad.jpeg",
    "price": 10.5,
    "allergens": [
    "dairy"
    ],
    "course": "lunch",
    "type": "dish"
  }
  ```

  1. Enregistrez vos modifications
1. Revenez au fichier **dish.json**.
1. Dans le filtre, sélectionnez **Aperçu de la carte adaptative**.

  ![Capture d’écran de l’aperçu de la carte adaptative dans Visual Studio Code.](../media/LAB_03/LAB_03/3-visual-studio-code-adaptive-card-preview.png)

  Visual Studio Code ouvre un aperçu de la carte à côté. Lorsque vous modifiez la carte, vos modifications sont immédiatement visibles sur le côté.

1. Dans le tableau **corps**, ajoutez un élément **Container** avec une référence à l’URL de l’image stockée dans la propriété **image_url**.

  ```json
  {
    "type": "Container",
    "items": [
    {
      "type": "Image",
      "url": "${image_url}",
      "size": "large"
    }
    ]
  }
  ```

  Notez comment l’aperçu de la carte est automatiquement mis à jour pour afficher votre carte :

  ![Capture d’écran de l’aperçu de la carte adaptative avec une image dans Visual Studio Code.](../media/LAB_03/3-visual-studio-code-adaptive-card-preview-image.png)

1. Ajoutez des références à d’autres propriétés de plat. La carte complète se présente comme suit :

  ```json
  {
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

  ![Capture d’écran de l’aperçu d’une carte adaptative d’un plat dans Visual Studio Code.](../media/LAB_03/3-visual-studio-code-adaptive-card-preview-image-properties.png)

  Notez que pour afficher les allergènes, vous utilisez une fonction pour joindre les allergènes dans une chaîne. Si un plat n’a pas d’allergènes, vous affichez **aucun**. Pour vous assurer que les prix sont correctement mis en forme, vous utilisez la fonction **formatNumber** qui nous permet de spécifier le nombre de décimales à afficher sur la carte.

## Tâche 3 : générer une carte adaptative pour le résumé de la commande

L’exemple d’API permet aux utilisateurs de parcourir le menu et de passer une commande. Nous allons créer une carte adaptative qui affiche le résumé de la commande.

Dans Visual Studio Code :

1. Dans le dossier **cartes**, créez un fichier nommé **order.json**. Collez le contenu suivant qui représente une carte adaptative vide :

  ```json
  {
    "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
    "type": "AdaptiveCard",
    "version": "1.5",
    "body": []
  }
  ```

1. Créez un fichier de données pour la carte adaptative :
  1. Ouvrez la palette de commandes en appuyant sur <kbd>CTRL</kbd>+<kbd>P</kbd> (<kbd>CMD</kbd>+<kbd>P</kbd> sur macOS). Entrez `>Adaptive` pour rechercher des commandes liées à l’utilisation des cartes adaptatives.

    ![Capture d’écran des commandes liées à l’utilisation de cartes adaptatives dans Visual Studio Code.](../media/LAB_03/3-visual-studio-code-adaptive-card-commands.png)

  1. Dans la liste, choisissez **Carte adaptative : nouveau fichier de données**. Visual Studio Code crée un fichier nommé **order.data.json**.
  1. Remplacez son contenu par une donnée qui représente le résumé de la commande :

    ```json
    {
      "order_id": 6210,
      "status": "confirmed",
      "total_price": 25.48
    }
    ```

  1. Enregistrez vos modifications
1. Revenez au fichier **order.json**.
1. Dans le filtre, sélectionnez **Aperçu de la carte adaptative**.
1. Ensuite, remplacez le contenu du fichier **order.json** avec le code suivant :

  ```json
  {
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

  Comme dans la section précédente, vous mappez chaque élément de la carte à une propriété de données.

  ![Capture d’écran de l’aperçu de la carte adaptative d’une commande dans Visual Studio Code.](../media/LAB_03/3-visual-studio-code-adaptive-card-preview-order.png)

  > [!IMPORTANT]
  > Notez l’espace de fin après **${order_id}**. C’est intentionnel, en raison d’un problème connu avec le rendu des nombres par les cartes adaptatives. Pour tester, supprimez l’espace et voyez que le nombre disparaît de l’aperçu.
  >
  > ![Capture d’écran d’un aperçu d’une carte adaptative sans le numéro de commande dans Visual Studio Code.](../media/LAB_03/3-visual-studio-code-adaptive-card-preview-no-number.png)

  Restaurez l’espace de fin afin que votre carte affiche correctement le prix et enregistre vos modifications.