---
lab:
  title: "Exercice\_1\_: télécharger le projet et examiner les fichiers"
  module: 'LAB 02: Build your first action for declarative agents with API plugin by using Visual Studio Code'
---

# Exercice 1 : télécharger le projet et examiner les fichiers

L’extension d’un agent déclaratif avec des actions lui permet de récupérer et de mettre à jour les données stockées dans des systèmes externes en temps réel. À l’aide de plug-ins d’API, vous pouvez vous connecter à des systèmes externes via leurs API pour récupérer et mettre à jour des informations.

### Durée de l’exercice

- **Durée estimée :** 10 minutes

## Tâche 1: télécharger le projet de démarrage

Commencez par télécharger l’exemple de projet. Dans un navigateur Web :

1. Accédez à [https://github.com/microsoft/learn-declarative-agent-api-plugin-typescript](https://github.com/microsoft/learn-declarative-agent-api-plugin-typescript).
    1. Suivez les étapes pour [télécharger le code source du dépôt](https://docs.github.com/repositories/working-with-files/using-files/downloading-source-code-archives#downloading-source-code-archives-from-the-repository-view) sur votre ordinateur.
    1. Extrayez le contenu du fichier zip téléchargé dans votre **dossier Documents**.
    1. Ouvrez le dossier  dans Visual Studio Code.

L’exemple de projet est un projet Teams Toolkit qui inclut un agent déclaratif et une API anonyme s’exécutant sur Azure Functions. L’agent déclaratif est identique à un agent déclaratif nouvellement créé à l’aide de Teams Toolkit. L’API appartient à un restaurant italien fictif et vous permet de parcourir le menu d’aujourd’hui et de passer des commandes.

## Tâche 2 : examiner la définition de l’API

Tout d’abord, examinez la définition de l’API du restaurant italien.

Dans Visual Studio Code :

1. Dans l’affichage **Explorateur**, ouvrez le fichier **appPackage/apiSpecificationFile/ristorante.yml**. Le fichier est une spécification OpenAPI qui décrit l’API du restaurant italien.
1. Recherchez la propriété **servers.url**.

    ```yaml
    servers:
      - url: http://localhost:7071/api
        description: Il Ristorante API server
    ```

    Notez qu’il pointe vers une URL locale qui correspond à l’URL standard lors de l’exécution locale d’Azure Functions.

1. Localisez la propriété **chemins**, qui contient deux opérations : **/plats** pour récupérer le menu d’aujourd’hui et **/orders** pour passer une commande.

    > [!IMPORTANT]
    > Notez que chaque opération contient la propriété **operationId** qui identifie de façon unique l’opération dans la spécification de l’API. Copilot exige que chaque opération ait un identifiant unique afin de savoir quelle API il doit appeler pour les invites spécifiques de l’utilisateur.

## Tâche 3 : examiner l’implémentation de l’API

Examinez ensuite l’exemple d’API que vous utilisez dans cet exercice.

Dans Visual Studio Code :

1. Dans l’affichage **Explorateur**, ouvrez le fichier **src/data.json**. Le fichier contient un élément de menu fictif pour notre restaurant italien. Chaque plat se compose des éléments suivants :

    - un nom,
    - une description,
    - un lien vers une image,
    - un prix,
    - s’il s’agit d’une entrée, d’un plat principal ou d’un dessert,
    - un type (plat ou boisson)
    - et éventuellement une liste d’allergènes.

    Dans cet exercice, les API utilisent ce fichier comme source de données.
1. Ensuite, développez le dossier **src/functions**. Notez deux fichiers nommés **dishes.ts** et **placeOrder.ts**. Ces fichiers contiennent l’implémentation des deux opérations définies dans la spécification de l’API.
1. Ouvrez le fichier **src/functions/dishes.ts**. Prenez un moment pour examiner le fonctionnement de l’API. Elle commence par charger les exemples de données à partir du fichier **src/functions/data.json**.

    ```typescript
    import data from "../data.json";
    ```

    Ensuite, elle recherche dans les différents paramètres de la chaîne de requête les filtres éventuels que le client appelant l’API pourrait passer.

    ```typescript
    const course = req.query.get('course');
    const allergensString = req.query.get('allergens');
    const allergens: string[] = allergensString ? allergensString.split(",") : [];
    const type = req.query.get('type');
    const name = req.query.get('name');
    ```

    En fonction des filtres spécifiés sur la requête, l’API filtre le jeu de données et retourne une réponse.

1. Ensuite, examinez l’API pour passer des commandes définies dans le fichier **src/functions/placeOrder.ts**. L’API commence par référencer les exemples de données. Ensuite, elle définit la forme de la commande que le client envoie dans le corps de la demande.

    ```typescript
    interface OrderedDish {
      name?: string;
      quantity?: number;
    }
    interface Order {
      dishes: OrderedDish[];
    }
    ```

    Lorsque l’API traite la demande, elle vérifie d’abord si la demande contient un corps et si elle a la forme correcte. Si ce n’est pas le cas, elle rejette la demande avec une erreur 400 Requête incorrecte.

    ```typescript
    let order: Order | undefined;
    try {
      order = await req.json() as Order | undefined;
    }
    catch (error) {
      return {
        status: 400,
        jsonBody: { message: "Invalid JSON format" },
      } as HttpResponseInit;
    }
    if (!order.dishes || !Array.isArray(order.dishes)) {
      return {
        status: 400,
        jsonBody: { message: "Invalid order format" }
      } as HttpResponseInit;
    }
    ```

    Ensuite, l’API résout la demande en fonction des plats du menu et calcule le prix total.

    ```typescript
    let totalPrice = 0;
    const orderDetails = order.dishes.map(orderedDish => {
      const dish = data.find(d => d.name.toLowerCase().includes(orderedDish.name.toLowerCase()));
      if (dish) {
        totalPrice += dish.price * orderedDish.quantity;
        return {
          name: dish.name,
          quantity: orderedDish.quantity,
          price: dish.price,
        };
      }
      else {
        context.error(`Invalid dish: ${orderedDish.name}`);
        return null;
      }
    });
    ```

    > [!IMPORTANT]
    > Notez comment l’API s’attend à ce que le client spécifie le plat par une partie de son nom plutôt que son ID. C'est volontaire, car les grands modèles de langage fonctionnent mieux avec les mots qu’avec les chiffres. En outre, avant d’appeler l’API pour passer la commande, Copilot a le nom du plat facilement disponible dans le cadre de l’invite de l’utilisateur. Si Copilot devait faire référence à un plat par son ID, il devrait d’abord le récupérer, ce qui nécessite des requêtes API supplémentaires, ce que Copilot ne peut pas faire actuellement.

    Lorsque l’API est prête, elle retourne une réponse avec un prix total, un ID de commande et un statut.

    ```typescript
    const orderId = Math.floor(Math.random() * 10000);
    return {
      status: 201,
      jsonBody: {
        order_id: orderId,
        status: "confirmed",
        total_price: totalPrice,
      }
    } as HttpResponseInit;
    ```

