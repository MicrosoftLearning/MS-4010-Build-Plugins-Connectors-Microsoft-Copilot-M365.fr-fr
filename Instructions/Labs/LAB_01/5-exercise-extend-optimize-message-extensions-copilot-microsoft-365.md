---
lab:
  title: "Exercice\_4\_: étendre et optimiser les extensions de message à utiliser avec Microsoft\_365 Copilot"
  module: 'LAB 01: Connect Microsoft 365 Copilot to your external data in real-time with message extension plugins built with .NET and Visual Studio'
---

# Exercice 4 : étendre et optimiser les extensions de message à utiliser avec Microsoft 365 Copilot

Dans cet exercice, vous étendrez et optimiserez votre extension de message à utiliser avec Microsoft 365 Copilot. Vous ajouterez un nouveau paramètre appelé Public cible et mettrez à jour la logique d’extension de message pour gérer plusieurs paramètres. Enfin, vous exécuterez et déboguerez votre extension de message et vous la testerez dans Copilot intégré à Microsoft Teams.

![Capture d’écran d’une réponse dans Microsoft 365 Copilot qui contient des informations retournées par le plug-in d’extension de message. Une carte adaptative s’affiche avec des informations sur le produit.](../media/5-copilot-answer.png)

> [!NOTE]
> La seule tâche de cet exercice qui nécessite une licence Microsoft 365 Copilot est la tâche 5. Les tâches précédentes doivent être effectuées, que votre locataire possède Copilot ou non.

### Durée de l’exercice

  - **Durée estimée :** 40 minutes

## Tâche 1 : mettre à jour la description de l’application

Il est essentiel de spécifier des descriptions concises et précises dans le manifeste de votre application pour que Copilot sache quand et comment appeler votre plug-in. Mettez à jour les descriptions de l’application, des commandes et des paramètres dans le manifeste de l’application.

Ouvrez Visual Studio. Dans le projet **TeamsApp** :

1. Dans le dossier **appPackage**, ouvrez **manifest.json**.

1. Mettez à jour la **description** de l’objet.

    ```json
    "description": {
        "short": "Product look up tool.",
        "full": "Get real-time product information and share them in a conversation. Search by product name or target audience. ${{APP_DISPLAY_NAME}} works with Microsoft 365 Chat. Find products at Contoso. Find Contoso products called mark8. Find Contoso products named mark8. Find Contoso products related to Mark8. Find Contoso products aimed at individuals. Find Contoso products aimed at businesses. Find Contoso products aimed at individuals with the name mark8. Find Contoso products aimed at businesses with the name mark8."
    },
    ```

## Tâche 2 : ajouter un nouveau paramètre

Ajoutez un nouveau paramètre que Copilot peut utiliser. Ce nouveau paramètre permet aux utilisateurs de rechercher des produits destinés à des publics différents à l’aide de Copilot, tels que des particuliers ou des entreprises.

Poursuivez dans Visual Studio, dans le projet **TeamsApp** :

1. Dans le tableau des paramètres**manifest.json**, dans le tableau **parameters**, ajoutez le paramètre **TargetAudience** après le paramètre **ProductName** :

    ```json
    "parameters": [
        {
            "name": "ProductName",
            "title": "Product name",
            "description": "The name of the product as a keyword",
            "inputType": "text"
        },
        {
            "name": "TargetAudience",
            "title": "Target audience",
            "description": "Audience that the product is aimed at. Consumer products are sold to individuals. Enterprise products are sold to businesses",
            "inputType": "text"
        }
    ]
    ```

1. Enregistrez les changements apportés.

La description du paramètre **TargetAudience** explique ce qu’il est et informe que le paramètre doit accepter les valeurs **Consumer** ou **Enterprise** comme valeurs autorisées.

Ensuite, mettez à jour la description de la commande afin d’inclure le nouveau paramètre.

1. Dans le tableau **commands**, mettez à jour la **description** de la commande à la **ligne 36**.

    ```json
    "description": "Find products by name or by target audience.",
    ```

## Tâche 3 : mettre à jour la logique d’extension de message

Pour prendre en charge le nouveau paramètre et les invites complexes, mettez à jour la méthode **OnTeamsMessagingExtensionQueryAsync** dans le gestionnaire d’activité du bot pour qu’il puisse gérer plusieurs paramètres.

Tout d’abord, mettez à jour la classe **ProductService** pour récupérer des produits en fonction des paramètres de nom et d’audience.

Poursuivez dans Visual Studio, dans le projet **ProductPlugin** :

1. Dans le dossier **Services**, ouvrez **ProductsService.cs**.

1. Dans le fichier, créez de nouvelles méthodes appelées **GetProductsByCategoryAsync** et **GetProductsByNameAndCategoryAsync** :

    ```csharp
    internal async Task<Product[]> GetProductsByCategoryAsync(string category)
    {
        var response = await _httpClient.GetAsync($"{_baseUri}products?category={category}");
        response.EnsureSuccessStatusCode();
        var jsonString = await response.Content.ReadAsStringAsync();
        return System.Text.Json.JsonSerializer.Deserialize<Product[]>(jsonString);
    }
    internal async Task<Product[]> GetProductsByNameAndCategoryAsync(string name, string category)
    {
        var response = await _httpClient.GetAsync($"{_baseUri}?name={name}&category={category}");
        response.EnsureSuccessStatusCode();
        var jsonString = await response.Content.ReadAsStringAsync();
        return System.Text.Json.JsonSerializer.Deserialize<Product[]>(jsonString);
    }
    ```

1. Enregistrez les changements apportés.

Ensuite, ajoutez une nouvelle méthode à la classe **MessageExtensionHelper** pour récupérer des produits en fonction des paramètres de nom et d’audience.

1. Dans le dossier **Helpers**, ouvrez **MessageExtensionHelper.cs**.

1. Dans le fichier, créez une méthode appelée **RetrieveProducts** qui récupère les produits en fonction des paramètres de nom et d’audience :

    ```csharp
    internal static async Task<IList<Product>> RetrieveProducts(string name, string audience, ProductsService productsService)
    {
        IList<Product> products;
        if (string.IsNullOrEmpty(name) && !string.IsNullOrEmpty(audience))
        {
            products = await productsService.GetProductsByCategoryAsync(audience);
        }
        else if (!string.IsNullOrEmpty(name) && string.IsNullOrEmpty(audience))
        {
            products = await productsService.GetProductsByNameAsync(name);
        }
        else if (!string.IsNullOrEmpty(name) && !string.IsNullOrEmpty(audience))
        {
            products = await productsService.GetProductsByNameAndCategoryAsync(name, audience);
        }
        else
        {
            products = [];
        }
        return products;
    }
    ```

1. Enregistrez les changements apportés.

La méthode **RetrieveProduct** récupère les produits en fonction des paramètres de nom et d’audience. Si le paramètre de nom est vide et que le paramètre d’audience n’est pas vide, la méthode récupère les produits en fonction du paramètre d’audience. Si le paramètre de nom n’est pas vide et que le paramètre d’audience est vide, la méthode récupère les produits en fonction du paramètre de nom. Si les paramètres de nom et d’audience ne sont pas vides, la méthode récupère les produits en fonction des deux paramètres. Si les deux paramètres sont vides, la méthode retourne une liste vide.

Ensuite, mettez à jour la classe **SearchApp** pour gérer le nouveau paramètre.

1. Dans le dossier **Search**, ouvrez **SearchApp.cs**.

1. Dans la méthode **OnTeamsMessagingExtensionQueryAsync**, remplacez le code suivant à partir de la **ligne 30** :

    ```csharp
    var name = MessageExtensionHelpers.GetQueryParameterValueByName(query.Parameters, "ProductName");
    var productService = new ProductsService(tokenResponse.Token);
    var products = await productService.GetProductsByNameAsync(name);
    ```

    Par :

    ```csharp
    var name = MessageExtensionHelpers.GetQueryParameterValueByName(query.Parameters, "ProductName");
    var audience = MessageExtensionHelpers.GetQueryParameterValueByName(query.Parameters, "TargetAudience");
    var productService = new ProductsService(tokenResponse.Token);
    var products = await MessageExtensionHelpers.RetrieveProducts(name, audience, productService);
    ```

1. Enregistrez les changements apportés.

La méthode **OnTeamsMessagingExtensionQueryAsync** récupère désormais les paramètres de nom et d’audience à partir des paramètres de requête. Elle récupère ensuite les produits en fonction des paramètres de nom et d’audience à l’aide de la méthode **RetrieveProducts**.

## Tâche 4 : créer et mettre à jour des ressources

Maintenant que tout est en place, exécutez le processus **Préparer les dépendances de l’application Teams** pour créer de nouvelles ressources et mettre à jour les ressources existantes.

Continuez dans Visual Studio.

1. Dans l’**Explorateur de solutions**, cliquez avec le bouton droit de la souris sur le projet **TeamsApp**.

1. Développez le menu **Teams Toolkit** et sélectionnez **Préparer les dépendances de l’application Teams**.

1. Dans la boîte de dialogue **Compte Microsoft 365**, sélectionnez **Continuer**.

1. Dans la boîte de dialogue **Approvisionner**, sélectionnez **Approvisionner**.

1. Dans la boîte de dialogue d’**avertissement de Teams Toolkit**, sélectionnez **Approvisionner**.

1. Dans la boîte de dialogue d’**informations de Teams Toolkit**, sélectionnez l’icône en forme de croix pour fermer la boîte de dialogue.

## Tâche 5 : exécuter et déboguer

Une fois les ressources approvisionnées, démarrez une session de débogage pour tester l’extension de message.

Commencez par démarrer le **proxy de développement** pour simuler l’API personnalisée.

1. Dans la **fenêtre d’invite de commandes** que vous avez laissée ouverte, exécutez la commande suivante pour démarrer le proxy de développement :

1. Exécutez la commande suivante pour démarrer le proxy de développement :

   ```bash
   devproxy --config-file "~appFolder/presets/learn-copilot-me-plugin/products-api-config.json"
   ```

1. Si vous y êtes invité, acceptez les avertissements de certificat.

> [!NOTE]
> Lorsque le proxy de développement est en cours d’exécution, il agit comme proxy au niveau du système entier.

Ensuite, démarrez une session de débogage dans Visual Studio.

1. Pour démarrer une nouvelle session de débogage, appuyez sur <kbd>F5</kbd> ou sélectionnez **Démarrer** dans la barre d’outils.

1. Attendez qu’une fenêtre de navigateur s’ouvre et que la boîte de dialogue d’installation de l’application s’affiche dans le client web Microsoft Teams. Lorsque vous y êtes invité, saisissez vos informations d’identification de compte Microsoft 365.

1. Dans la boîte de dialogue d’installation de l’application, sélectionnez **Ajouter**.

1. Ouvrez l’application **Copilot** dans Microsoft Teams.

1. Dans la zone de rédaction de message, ouvrez le menu volant **Plug-ins**.

1. Dans la liste des plug-ins, activez le plug-in **Contoso products**.

    ![Capture d’écran de Microsoft 365 Copilot dans Microsoft Teams avec le plug-in de produits Contoso activé.](../media/20-copilot-plugin-enabled.png)

1. Saisissez **Rechercher des produits Contoso destinés aux particuliers** comme message et envoyez-le.

1. Attendez que Copilot réponde :

    ![Capture d’écran de Microsoft 365 Copilot dans Microsoft Teams montrant le message du copilote affiché lors du traitement de la demande de l’utilisateur.](../media/21-copilot-thinking.png)

1. Dans la réponse du copilote, les données retournées dans la réponse du plug-in sont affichées et le plug-in est référencé dans la réponse :

    ![Capture d’écran d’une réponse dans Microsoft 365 Copilot qui contient des informations retournées par le plug-in d’extension de message. Une carte adaptative s’affiche avec des informations sur le produit.](../media/5-copilot-answer.png)

1. Pour afficher la carte adaptative correspondant au résultat, pointez sur les références dans la réponse du copilote :

    ![Capture d’écran de Microsoft 365 Copilot dans Microsoft Teams montrant une carte adaptative affichant des informations de produit. La carte s’affiche lorsque l’utilisateur pointe sur une référence dans la réponse du copilote.](../media/22-copilot-reference.png)

Revenez à Visual Studio et sélectionnez **Arrêter** dans la barre d’outils, ou appuyez sur <kbd>Maj</kbd> + <kbd>F5</kbd> pour arrêter la session de débogage. De plus, arrêtez le proxy de développement en appuyant sur <kbd>Ctrl</kbd> + <kbd>C</kbd>.

[Passez au résumé du labo...](./6-summary.md)