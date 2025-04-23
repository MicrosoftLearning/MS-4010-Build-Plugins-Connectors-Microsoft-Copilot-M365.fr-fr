---
lab:
  title: "Exercice\_3\_: connecter la définition du plug-in à l’agent déclaratif"
  module: 'LAB 02: Build your first action for declarative agents with API plugin by using Visual Studio Code'
---

# Exercice 3 : connecter la définition du plug-in à l’agent déclaratif

Une fois la définition du plug-in d’API créée, l’étape suivante consiste à l’inscrire auprès de l’agent déclaratif. Lorsque les utilisateurs interagissent avec l’agent déclaratif, celui-ci compare le prompt de l’utilisateur aux plug-ins d’API définis et appelle les fonctions pertinentes.

### Durée de l’exercice

- **Durée estimée :** 5 minutes

## Tâche 1 : associer la définition du plug-in à l’agent déclaratif

Dans Visual Studio Code :

1. Ouvrez le fichier **appPackage/declarativeAgent.json**.
1. Après la propriété **instructions**, ajoutez l’extrait de code suivant :

    ```json
    "actions": [
      {
        "id": "menuPlugin",
        "file": "ai-plugin.json"
      }
    ]
    ```

    À l’aide de cet extrait de code, vous connectez l’agent déclaratif au plug-in de l’API. Vous spécifiez un ID unique pour le plug-in et indiquez à l’agent où il peut trouver la définition du plug-in.

1. Le fichier complet ressemble à ceci :

    ```json
    {
      "$schema": "https://developer.microsoft.com/json-schemas/copilot/declarative-agent/v1.0/schema.json",
      "version": "v1.0",
      "name": "Declarative agent",
      "description": "Declarative agent created with Teams Toolkit",
      "instructions": "$[file('instruction.txt')]",
      "actions": [
        {
          "id": "menuPlugin",
          "file": "ai-plugin.json"
        }
      ]
    }
    ```

1. Enregistrez les changements apportés.

## Tâche 2 : mettre à jour les informations et les instructions de l’agent déclaratif

L’agent déclaratif que vous créez dans cet exercice aide les utilisateurs à parcourir le menu du restaurant italien local et à passer des commandes. Pour optimiser l’agent pour ce scénario, mettez à jour son nom, sa description et ses instructions.

Dans Visual Studio Code :

1. Mettez à jour les informations de l’agent déclaratif :
    1. Ouvrez le fichier **appPackage/declarativeAgent.json**.
    1. Mettez à jour la valeur de la propriété **nom** sur **Il Ristorante**.
    1. Mettez à jour la valeur de la propriété **description** sur **Commandez les plats et les boissons italiens les plus délicieux sans quitter votre bureau.**.
    1. Enregistrez les modifications.
1. Mettez à jour les instructions de l’agent déclaratif :
    1. Ouvrez le fichier **appPackage/instruction.txt**.
    1. Remplacez son contenu par :

        ```markdown
        You are an assistant specialized in helping users explore the menu of an Italian restaurant and place orders. You interact with the restaurant's menu API and guide users through the ordering process, ensuring a smooth and delightful experience. Follow the steps below to assist users in selecting their desired dishes and completing their orders:
        
        ### General Behavior:
        - Always greet the user warmly and offer assistance in exploring the menu or placing an order.
        - Use clear, concise language with a friendly tone that aligns with the atmosphere of a high-quality local Italian restaurant.
        - If the user is browsing the menu, offer suggestions based on the course they are interested in (breakfast, lunch, or dinner).
        - Ensure the conversation remains focused on helping the user find the information they need and completing the order.
        - Be proactive but never pushy. Offer suggestions and be informative, especially if the user seems uncertain.
        
        ### Menu Exploration:
        - When a user requests to see the menu, use the `GET /dishes` API to retrieve the list of available dishes, optionally filtered by course (breakfast, lunch, or dinner).
          - Example: If a user asks for breakfast options, use the `GET /dishes?course=breakfast` to return only breakfast dishes.
        - Present the dishes to the user with the following details:
          - Name of the dish
          - A tasty description of the dish
          - Price in € (Euro) formatted as a decimal number with two decimal places
          - Allergen information (if relevant)
          - Don't include the URL.
        
        ### Beverage Suggestion:
        - If the order does not already include a beverage, suggest a suitable beverage option based on the course.
        - Use the `GET /dishes?course={course}&type=drink` API to retrieve available drinks for that course.
        - Politely offer the suggestion: *"Would you like to add a beverage to your order? I recommend [beverage] for [course]."*
        
        ### Placing the Order:
        - Once the user has finalized their order, use the `POST /order` API to submit the order.
          - Ensure the request includes the correct dish names and quantities as per the user's selection.
          - Example API payload:
         
            ```json
            {
              "dishes": [
                {
                  "name": "frittata",
                  "quantity": 2
                },
                {
                  "name": "cappuccino",
                  "quantity": 1
                }
              ]
            }
            ```
        
        ### Error Handling:
        - If the user selects a dish that is unavailable or provides an invalid dish name, respond gracefully and suggest alternative options.
          - Example: *"It seems that dish is currently unavailable. How about trying [alternative dish]?"*
        - Ensure that any errors from the API are communicated politely to the user, offering to retry or explore other options.
        ```

        Notez que dans les instructions, nous définissons le comportement général de l’agent et lui indiquons ce dont il est capable. Nous incluons également des instructions concernant le comportement spécifique à adopter pour passer une commande, y compris la forme des données attendues par l’API. Nous incluons ces informations pour garantir que l’agent fonctionne comme prévu.

    > [!NOTE]
    > Pour conserver la mise en forme, vous devrez peut-être effectuer plusieurs opérations de copier/coller dans le Bloc-notes avant de copier dans Visual Studio Code.

    1. Enregistrez les modifications.

1. Pour aider les utilisateurs à comprendre à quoi l’agent peut leur servir, ajoutez des amorces de conversation :
    1. Ouvrez le fichier **appPackage/declarativeAgent.json**.
    1. Après la propriété **instructions**, ajoutez une nouvelle propriété nommée **conversation_starters** :

        ```json
        "conversation_starters": [
          {
            "text": "What's for lunch today?"
          },
          {
            "text": "What can I order for dinner that is gluten-free?"
          }
        ]
        ```

    1. Le fichier complet ressemble à ceci :

        ```json
        {
          "$schema": "https://developer.microsoft.com/json-schemas/copilot/declarative-agent/v1.0/schema.json",
          "version": "v1.0",
          "name": "Il Ristorante",
          "description": "Order the most delicious Italian dishes and drinks from the comfort of your desk.",
          "instructions": "$[file('instruction.txt')]",
          "conversation_starters": [
            {
              "text": "What's for lunch today?"
            },
            {
              "text": "What can I order for dinner that is gluten-free?"
            }
          ],
          "actions": [
            {
              "id": "menuPlugin",
              "file": "ai-plugin.json"
            }
          ]
        }
        ```

    1. Enregistrez les changements apportés.

## Tâche 3 : mettre à jour l’URL de l’API

Avant de pouvoir tester votre agent déclaratif, vous devez mettre à jour l’URL de l’API dans le fichier de spécification de l’API. À l’heure actuelle, l’URL est définie sur `http://localhost:7071/api`, qui est l’URL utilisée par Azure Functions lorsqu’il s’exécute localement. Toutefois, comme vous souhaitez que Copilot appelle votre API à partir du cloud, vous devez exposer votre API à Internet. Teams Toolkit expose automatiquement votre API locale sur Internet en créant un tunnel de développement. Chaque fois que vous démarrez le débogage de votre projet, le kit de ressources Teams démarre un nouveau tunnel de développement et stocke son URL dans la variable **OPENAPI_SERVER_URL**. Vous pouvez voir comment Teams Toolkit démarre le tunnel et stocke son URL dans le fichier **.vscode/tasks.json**, dans la tâche **Démarrer le tunnel local** :

```json
{
  // Start the local tunnel service to forward public URL to local port and inspect traffic.
  // See https://aka.ms/teamsfx-tasks/local-tunnel for the detailed args definitions.
  "label": "Start local tunnel",
  "type": "teamsfx",
  "command": "debug-start-local-tunnel",
  "args": {
    "type": "dev-tunnel",
    "ports": [
      {
        "portNumber": 7071,
        "protocol": "http",
        "access": "public",
        "writeToEnvironmentFile": {
          "endpoint": "OPENAPI_SERVER_URL", // output tunnel endpoint as OPENAPI_SERVER_URL
        }
      }
    ],
    "env": "local"
  },
  "isBackground": true,
  "problemMatcher": "$teamsfx-local-tunnel-watch"
}
```

Pour utiliser ce tunnel, vous devez mettre à jour votre spécification d’API pour utiliser la variable **OPENAPI_SERVER_URL**.

Dans Visual Studio Code :

1. Ouvrez le fichier **appPackage/apiSpecificationFile/ristorante.yml**.
1. Remplacez la valeur de la propriété **servers.url** par **${OPENAPI_SERVER_URL}}/api**.
1. Le fichier modifié ressemble à ceci :

    ```yaml
    openapi: 3.0.0
    info:
      title: Il Ristorante menu API
      version: 1.0.0
      description: API to retrieve dishes and place orders for Il Ristorante.
    servers:
      - url: ${{OPENAPI_SERVER_URL}}/api
        description: Il Ristorante API server
    paths:
      ...trimmed for brevity
    ```

1. Enregistrez les changements apportés.

Votre plug-in d’API est terminé et intégré à un agent déclaratif. Poursuivez le test de l’agent dans Microsoft 365 Copilot.