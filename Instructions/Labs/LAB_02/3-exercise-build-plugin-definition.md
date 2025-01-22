---
lab:
  title: "Exercice\_2\_: créer une définition de plug-in d’API"
  module: 'LAB 02: Build your first action for declarative agents with API plugin by using Visual Studio Code'
---

# Exercice 2 : créer une définition de plug-in d’API

L’étape suivante consiste à ajouter la définition du plug-in au projet. La définition du plug-in contient les informations suivantes :

- Quelles sont les actions que le plug-in peut effectuer ?
- Quelle est la forme des données attendues et retournées ?
- Comment l’agent déclaratif doit-il appeler l’API sous-jacente ?

### Durée de l’exercice

- **Durée estimée :** 10 minutes

## Tâche 1 : ajouter la structure de définition de plug-in de base

Dans Visual Studio Code :

1. Dans le dossier **appPackage**, ajoutez un nouveau fichier nommé **ai-plugin.json**.
1. Collez le contenu suivant :

    ```json
    {
      "$schema": "https://developer.microsoft.com/json-schemas/copilot/plugin/v2.1/schema.json",
      "schema_version": "v2.1",
      "namespace": "ilristorante",
      "name_for_human": "Il Ristorante",
      "description_for_human": "See the today's menu and place orders",
      "description_for_model": "Plugin for getting the today's menu, optionally filtered by course and allergens, and placing orders",
      "functions": [ 
      ],
      "runtimes": [
      ],
      "capabilities": {
        "localization": {},
        "conversation_starters": []
      }
    }
    ```

    Le fichier contient une structure de base pour un plug-in d’API avec une description de l’humain et du modèle. La **description_for_model** inclut des informations détaillées sur ce que le plug-in peut faire pour aider l’agent à comprendre quand il doit envisager de l’appeler.
1. Enregistrez les changements apportés.

## Tâche 2 : définir des fonctions

Un plug-in d’API définit une ou plusieurs fonctions qui correspondent aux opérations d’API définies dans la spécification de l’API. Chaque fonction se compose d’un nom, d’une description et d’une définition de réponse qui indique à l’agent comment afficher les données aux utilisateurs.

### Définir une fonction pour récupérer le menu

Commencez par définir une fonction pour récupérer les informations sur le menu d’aujourd’hui.

Dans Visual Studio Code :

1. Ouvrez le fichier **appPackage/ai-plugin.json**.
1. Dans le tableau **fonctions**, ajoutez l’extrait de code suivant :

    ```json
    {
      "name": "getDishes",
      "description": "Returns information about the dishes on the menu. Can filter by course (breakfast, lunch or dinner), name, allergens, or type (dish, drink).",
      "capabilities": {
        "response_semantics": {
          "data_path": "$.dishes",
          "properties": {
            "title": "$.name",
            "subtitle": "$.description"
          }
        }
      }
    }
    ```

    Commencez par définir une fonction qui appelle l’opération **getDishes** de la spécification de l’API. Ensuite, vous fournissez une description de fonction. Cette description est importante, car Copilot l’utilise pour décider de la fonction à appeler pour l’invite d’un utilisateur.

    Dans la propriété response_semantics, vous spécifiez comment Copilot doit afficher les données qu’il reçoit de l’API. Étant donné que l’API renvoie les informations sur les plats du menu dans la propriété **dishes**, vous définissez la propriété **data_path** sur l’expression JSONPath `$.dishes`.

    Ensuite, dans la section **propriétés**, vous mappez les propriétés de la réponse de l’API qui représentent le titre, la description et l’URL. Étant donné que dans ce cas, les plats n’ont pas d’URL, vous mappez uniquement le **titre** et la **description**.

1. L’extrait de code complet se présente ainsi :

    ```json
    {
      "$schema": "https://developer.microsoft.com/json-schemas/copilot/plugin/v2.1/schema.json",
      "schema_version": "v2.1",
      "namespace": "ilristorante",
      "name_for_human": "Il Ristorante",
      "description_for_human": "See the today's menu and place orders",
      "description_for_model": "Plugin for getting the today's menu, optionally filtered by course and allergens, and placing orders",
      "functions": [
        {
          "name": "getDishes",
          "description": "Returns information about the dishes on the menu. Can filter by course (breakfast, lunch or dinner), name, allergens, or type (dish, drink).",
          "capabilities": {
            "response_semantics": {
              "data_path": "$.dishes",
              "properties": {
                "title": "$.name",
                "subtitle": "$.description"
              }
            }
          }
        }
      ],
      "runtimes": [
      ],
      "capabilities": {
        "localization": {},
        "conversation_starters": []
      }
    }
    ```

1. Enregistrez les changements apportés.

### Définir une fonction pour passer la commande

Ensuite, définissez une fonction pour passer la commande.

Dans Visual Studio Code :

1. Ouvrez le fichier **appPackage/ai-plugin.json**.
1. À la fin du tableau **fonctions**, ajoutez l’extrait code suivant :

    ```json
    {
      "name": "placeOrder",
      "description": "Places an order and returns the order details",
      "capabilities": {
        "response_semantics": {
          "data_path": "$",
          "properties": {
            "title": "$.order_id",
            "subtitle": "$.total_price"
          }
        }
      }
    }
    ```

    Vous commencez par faire référence à l’opération d’API avec l’ID **placeOrder**. Ensuite, vous fournissez une description que Copilot utilise pour faire correspondre cette fonction à l’invite d’un utilisateur. Ensuite, vous indiquez à Copilot comment retourner les données. Voici les données retournées par l’API après avoir passé une commande :

    ```json
    {
      "order_id": 6532,
      "status": "confirmed",
      "total_price": 21.97
    }
    ```

    Étant donné que les données que vous souhaitez afficher se trouvent directement à la racine de l’objet réponse, vous définissez la propriété **data_path** sur **$**, laquelle indique le nœud supérieur de l’objet JSON. Vous définissez le titre pour qu’il affiche le numéro de la commande, et son sous-titre pour qu’il affiche le prix.

1. Le fichier complet ressemble à ceci :

    ```json
    {
      "$schema": "https://developer.microsoft.com/json-schemas/copilot/plugin/v2.1/schema.json",
      "schema_version": "v2.1",
      "namespace": "ilristorante",
      "name_for_human": "Il Ristorante",
      "description_for_human": "See the today's menu and place orders",
      "description_for_model": "Plugin for getting the today's menu, optionally filtered by course and allergens, and placing orders",
      "functions": [
        {
          "name": "getDishes",
          "description": "Returns information about the dishes on the menu. Can filter by course (breakfast, lunch or dinner), name, allergens, or type (dish, drink).",
          ...trimmed for brevity
        },
        {
          "name": "placeOrder",
          "description": "Places an order and returns the order details",
          "capabilities": {
            "response_semantics": {
              "data_path": "$",
              "properties": {
                "title": "$.order_id",
                "subtitle": "$.total_price"
              }
            }
          }
        }
      ],
      "runtimes": [
      ],
      "capabilities": {
        "localization": {},
        "conversation_starters": []
      }
    }
    ```

1. Enregistrez les changements apportés.

## Tâche 3 : définir des runtimes

Après avoir défini des fonctions que Copilot peut appeler, l’étape suivante consiste à lui indiquer comment il doit les appeler. Pour ce faire, accédez à la section **runtimes** de la définition du plug-in.

Dans Visual Studio Code :

1. Ouvrez le fichier **appPackage/ai-plugin.json**.
1. Dans le tableau runtimes, ajoutez le code suivant :

    ```json
    {
      "type": "OpenApi",
      "auth": {
        "type": "None"
      },
      "spec": {
        "url": "apiSpecificationFile/ristorante.yml"
      },
      "run_for_functions": [
        "getDishes",
        "placeOrder"
      ]
    }
    ```

    Vous commencez par indiquer à Copilot que vous lui fournissez des informations OpenAPI sur l’API (**type: OpenApi**) à appeler et qu’elle est anonyme (**auth.type: None**). Ensuite, dans la section **spec**, vous spécifiez le chemin d’accès relatif à la spécification de l’API située dans votre projet. Enfin, dans la propriété **run_for_functions**, vous répertoriez toutes les fonctions qui appartiennent à cette API.

1. Le fichier complet ressemble à ceci :

    ```json
    {
      "$schema": "https://developer.microsoft.com/json-schemas/copilot/plugin/v2.1/schema.json",
      "schema_version": "v2.1",
      "namespace": "ilristorante",
      "name_for_human": "Il Ristorante",
      "description_for_human": "See the today's menu and place orders",
      "description_for_model": "Plugin for getting the today's menu, optionally filtered by course and allergens, and placing orders",
      "functions": [
        {
          "name": "getDishes",
          ...trimmed for brevity
        },
        {
          "name": "placeOrder",
          ...trimmed for brevity
        }
      ],
      "runtimes": [
        {
          "type": "OpenApi",
          "auth": {
            "type": "None"
          },
          "spec": {
            "url": "apiSpecificationFile/ristorante.yml"
          },
          "run_for_functions": [
            "getDishes",
            "placeOrder"
          ]
        }
      ],
      "capabilities": {
        "localization": {},
        "conversation_starters": []
      }
    }
    ```

1. Enregistrez vos modifications.

