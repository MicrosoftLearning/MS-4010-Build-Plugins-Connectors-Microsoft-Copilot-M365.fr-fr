---
lab:
  title: "Exercice\_3\_: retourner les données de produit de l’API protégée par Microsoft Entra"
  module: 'LAB 01: Connect Microsoft 365 Copilot to your external data in real-time with message extension plugins built with .NET and Visual Studio'
---

# Exercice 3 : retourner les données de produit de l’API protégée par Microsoft Entra

Dans cet exercice, vous mettez à jour l’extension de message pour récupérer des données à partir d’une API personnalisée. Vous obtenez des données à partir de l’API personnalisée en fonction de la requête utilisateur et retournez des données dans les résultats de recherche à l’utilisateur.

![Capture d’écran des résultats de la recherche retournés par une extension de message basée sur une recherche dans Microsoft Teams.](../media/3-search-results-api.png)

### Durée de l’exercice

  - **Durée estimée :** 50 minutes

## Tâche 1 : installer et configurer Dev Proxy

Dans cet exercice, vous utilisez Dev Proxy, un outil en ligne de commande qui peut simuler des API. Il est utile lorsque vous souhaitez tester votre application sans avoir à créer une API réelle.

Pour effectuer cet exercice, vous devez installer la [dernière version de Dev Proxy](/microsoft-cloud/dev/dev-proxy/get-started) et télécharger le préréglage Dev Proxy pour ce module.

Le préréglage simule une API CRUD « Create, Read, Update, Delete » (Créer, Lire, Mettre à jour, Supprimer) avec un magasin de données en mémoire, qui est protégé par Microsoft Entra. Cela signifie que vous pouvez tester votre application comme si elle appelait une API réelle nécessitant une authentification.

1. Pour installer Dev Proxy, ouvrez une **nouvelle fenêtre d’invite de commandes en tant qu’administrateur** :

    ```bash
    winget install Microsoft.DevProxy --silent
    ```

1. Exécutez la commande suivante pour télécharger le préréglage :

    ```bash
    devproxy preset get learn-copilot-me-plugin
    ```

1. Laissez la fenêtre de l’invite de commandes ouverte : vous y reviendrez ultérieurement.

## Tâche 2 : obtenir la valeur de requête utilisateur

Créez une méthode qui obtient la valeur de requête utilisateur par le nom du paramètre.

Dans Visual Studio, dans le projet **ProductsPlugin** :

1. Dans le dossier **Helpers**, créez un fichier nommé **MessageExtensionHelpers.cs**.

1. Dans le fichier, remplacez le code par ce qui suit :

   ```csharp
   using Microsoft.Bot.Schema.Teams;
   internal class MessageExtensionHelpers
   {
       internal static string GetQueryParameterValueByName(IList<MessagingExtensionParameter> parameters, string name) => parameters.FirstOrDefault(p => p.Name == name)?.Value as string ?? string.Empty;
   }
   ```

1. Enregistrez les changements apportés.

Ensuite, mettez à jour la méthode **OnTeamsMessagingExtensionQueryAsync** dans la classe SearchApp pour utiliser la nouvelle méthode d’assistance.

1. Dans le dossier **Search**, ouvrez **SearchApp.cs**.

1. Dans la méthode **OnTeamsMessagingExtensionQueryAsync**, remplacez le code ci-dessous :

   ```csharp
   var text = query?.Parameters?[0]?.Value as string ?? string.Empty;
   ```

   par

   ```csharp
   var text = MessageExtensionHelpers.GetQueryParameterValueByName(query.Parameters, "ProductName");
   ```

1. Déplacez le curseur sur la **variable de texte**, utilisez `Ctrl + R`, `Ctrl + R` puis renommez la variable en **name**.

1. Appuyez sur **Entrée** pour renommer la variable pour 3 fichiers.

1. Enregistrez les changements apportés.

La méthode **OnTeamsMessagingExtensionQueryAsync** doit maintenant ressembler à ceci :

```csharp
protected override async Task<MessagingExtensionResponse> OnTeamsMessagingExtensionQueryAsync(ITurnContext<IInvokeActivity> turnContext, MessagingExtensionQuery query, CancellationToken cancellationToken)
{
    var userTokenClient = turnContext.TurnState.Get<UserTokenClient>();
    var tokenResponse = await AuthHelpers.GetToken(userTokenClient, query.State, turnContext.Activity.From.Id, turnContext.Activity.ChannelId, connectionName, cancellationToken);
    if (!AuthHelpers.HasToken(tokenResponse))
    {
        return await AuthHelpers.CreateAuthResponse(userTokenClient, connectionName, (Activity)turnContext.Activity, cancellationToken);
    }
    var name = MessageExtensionHelpers.GetQueryParameterValueByName(query.Parameters, "ProductName");
    var card = await File.ReadAllTextAsync(Path.Combine(".", "Resources", "card.json"), cancellationToken);
    var template = new AdaptiveCardTemplate(card);
    return new MessagingExtensionResponse
    {
        ComposeExtension = new MessagingExtensionResult
        {
            Type = "result",
            AttachmentLayout = "list",
            Attachments = [
                new MessagingExtensionAttachment
                    {
                        ContentType = AdaptiveCard.ContentType,
                        Content = JsonConvert.DeserializeObject(template.Expand(new { title = name })),
                        Preview = new ThumbnailCard { Title = name }.ToAttachment()
                    }
            ]
        }
    };
}
```

## Tâche 3 : obtenir des données à partir de l’API personnalisée

Pour obtenir des données à partir de l’API personnalisée, vous devez envoyer le jeton d’accès dans l’en-tête d’autorisation de la requête et désérialiser la réponse dans un modèle qui représente les données du produit.

Tout d’abord, créez un modèle qui représente les données de produits retournées par l’API personnalisée.

Dans Visual Studio, dans le projet **ProductsPlugin** :

1. Créez un dossier nommé **Models**.

1. Dans le dossier **Models**, créez un fichier appelé **Product.cs**.

1. Dans le fichier, remplacez le code existant par ce qui suit :

   ```csharp
   using System.Text.Json.Serialization;
   internal class Product
   {
       [JsonPropertyName("productId")]
       public int Id { get; set; }
       [JsonPropertyName("imageUrl")]
       public string ImageUrl { get; set; }
       [JsonPropertyName("name")]
       public string Name { get; set; }
       [JsonPropertyName("category")]
       public string Category { get; set; }
       [JsonPropertyName("callVolume")]
       public int CallVolume { get; set; }
       [JsonPropertyName("releaseDate")]
       public string ReleaseDate { get; set; }
   }
   ```

1. Enregistrez les changements apportés.

Ensuite, créez une classe de service qui récupère les données de produits à partir de l’API personnalisée.

Dans Visual Studio, dans le projet **ProductsPlugin** :

1. Créez un dossier nommé **Services**.

1. Dans le dossier **Services**, créez un fichier nommé **ProductService.cs**.

1. Dans le fichier, remplacez le code existant par ce qui suit :

    ```csharp
    using System.Net.Http.Headers;
    internal class ProductsService
    {
        private readonly HttpClient _httpClient;
        private readonly string _baseUri = "https://api.contoso.com/v1/";
        internal ProductsService(string token)
        {
            _httpClient = new HttpClient();
            _httpClient.DefaultRequestHeaders.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));
            _httpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", token);
        }
        internal async Task<Product[]> GetProductsByNameAsync(string name)
        {
            var response = await _httpClient.GetAsync($"{_baseUri}products?name={name}");
            response.EnsureSuccessStatusCode();
            var jsonString = await response.Content.ReadAsStringAsync();
            return System.Text.Json.JsonSerializer.Deserialize<Product[]>(jsonString);
        }
    }
    ```

1. Enregistrez les changements apportés.

La classe **ProductsService** contient des méthodes pour obtenir les données de produits à partir de l’API personnalisée. Le constructeur de classe prend un jeton d’accès en tant que paramètre et génère une instance **HttpClient** avec le jeton d’accès dans l’en-tête d’autorisation.

Ensuite, mettez à jour la **méthode OnTeamsMessagingExtensionQueryAsync** pour qu’elle utilise la classe **ProductsService** pour obtenir des données de produits à partir de l’API personnalisée.

1. Dans le dossier **Search**, ouvrez **SearchApp.cs**.

1. Dans la **méthode OnTeamsMessagingExtensionQueryAsync**, ajoutez le code suivant après la déclaration de la variable **name** pour obtenir des données de produits à partir de l’API personnalisée :

   ```csharp
   var productService = new ProductsService(tokenResponse.Token);
   var products = await productService.GetProductsByNameAsync(name);
   ```

1. Enregistrez les changements apportés.

## Tâche 4 : créer des résultats de la recherche

Maintenant que vous disposez des données du produit, vous pouvez les inclure dans les résultats de la recherche retournés à l’utilisateur.

Tout d’abord, nous allons mettre à jour le modèle de carte adaptative existant pour qu’il affiche les informations des produits.

Poursuivez dans Visual Studio et dans le projet **ProductsPlugin** :

1. Dans le dossier **Resources**, renommez **card.json** en **Product.json**.

1. Dans le dossier **Resources**, créez un fichier nommé **Product.data.json**. Ce fichier contient des exemples de données que Visual Studio utilise pour générer une prévisualisation du modèle de carte adaptative.

1. Ajoutez le code JSON suivant au fichier :

    ```json
    {
      "callVolume": 36,
      "category": "Enterprise",
      "imageUrl": "https://raw.githubusercontent.com/SharePoint/sp-dev-provisioning-templates/master/tenant/productsupport/source/Product%20Imagery/Contoso4.png",
      "name": "Contoso Quad",
      "productId": 1,
      "releaseDate": "2019-02-09"
    }
    ```

1. Enregistrez les changements apportés.

1. Dans le dossier **Resources**, ouvrez **Product.json**.

1. Remplacez le contenu du fichier par le code JSON suivant :

    ```json
    {
      "type": "AdaptiveCard",
      "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
      "version": "1.5",
      "body": [
        {
          "type": "TextBlock",
          "text": "${name}",
          "wrap": true,
          "style": "heading"
        },
        {
          "type": "TextBlock",
          "text": "${category}",
          "wrap": true
        },
        {
          "type": "Container",
          "items": [
            {
              "type": "Image",
              "url": "${imageUrl}",
              "altText": "${name}"
            }
          ],
          "minHeight": "350px",
          "verticalContentAlignment": "Center",
          "horizontalAlignment": "Center"
        },
        {
          "type": "FactSet",
          "facts": [
            {
              "title": "Call Volume",
              "value": "${formatNumber(callVolume,0)}"
            },
            {
              "title": "Release Date",
              "value": "${formatDateTime(releaseDate,'dd/MM/yyyy')}"
            }
          ]
        }
      ]
    }
    ```

1. Enregistrez les changements apportés.

Le modèle de carte adaptative utilise des expressions de liaison pour afficher les informations des produits. Les expressions **\$\{name\}**, **\$\{category\}**, **\$\{imageUrl\}**, **\$\{callVolume\}**, et **\$\{releaseDate\}** sont remplacées par les valeurs correspondantes issues des données de produits. Les fonctions de modèle **formatNumber** et **formatDateTime** sont utilisées pour mettre en forme les valeurs **callVolume** et **releaseDate**, respectivement sous forme de nombre et de date.

Prenez un moment pour explorer la prévisualisation de la carte adaptative dans Visual Studio. La prévisualisation montre à quoi le modèle de carte adaptative ressemble lorsque les données de produits sont liées au modèle. Il utilise les exemples de données du fichier **Product.data.json** pour générer la prévisualisation.

Ensuite, mettez à jour la propriété **validDomains** dans le manifeste de l’application afin d’inclure le domaine **raw.githubusercontent.com**, pour que les images du modèle de carte adaptative puissent être affichées dans Microsoft Teams.

Dans le projet **TeamsApp** :

1. Dans le dossier **appPackage**, ouvrez **manifest.json**.

1. Dans le fichier, ajoutez le domaine GitHub à la propriété **validDomains** :

    ```json
      "validDomains": [
        "token.botframework.com",
        "raw.githubusercontent.com",
        "${{BOT_DOMAIN}}"
      ],
    ```

1. Enregistrez les changements apportés.

Ensuite, mettez à jour la méthode **OnTeamsMessagingExtensionQueryAsync** afin de créer une liste de pièces jointes qui contiennent les informations des produits.

Dans le projet **ProductsPlugin** :

1. Dans le dossier **Search**, ouvrez **SearchApp.cs**.

1. Mettez à jour **card.json** vers **Product.json**, afin de refléter la modification du nom du fichier. À la ligne 34, remplacez le code suivant :

   ```csharp
   var card = await File.ReadAllTextAsync(Path.Combine(".", "Resources", "card.json"), cancellationToken);
   ```

   par

   ```csharp
   var card = await File.ReadAllTextAsync(Path.Combine(".", "Resources", "Product.json"), cancellationToken);
   ```

1. Ajoutez le code suivant après la déclaration de la variable **template** pour créer une liste de pièces jointes :

   ```csharp
    var attachments = products.Select(product =>
    {
        var content = template.Expand(product);
        return new MessagingExtensionAttachment
        {
            ContentType = AdaptiveCard.ContentType,
            Content = JsonConvert.DeserializeObject(content),
            Preview = new ThumbnailCard
            {
                Title = product.Name,
                Subtitle = product.Category,
                Images = [new() { Url = product.ImageUrl }]
            }.ToAttachment()
        };
    }).ToList();
   ```

1. Mettez à jour l’instruction **return** afin d’inclure la variable **attachments** :

   ```csharp
   return new MessagingExtensionResponse
   {
       ComposeExtension = new MessagingExtensionResult
       {
           Type = "result",
           AttachmentLayout = "list",
           Attachments = attachments
       }
   };
   ```

1. Enregistrer les modifications

## Tâche 5 : créer et mettre à jour des ressources

Maintenant que tout est en place, exécutez le processus **Préparer les dépendances de l’application Teams** pour créer de nouvelles ressources et mettre à jour les ressources existantes.

Continuez dans Visual Studio.

1. Dans l’**Explorateur de solutions**, cliquez avec le bouton droit de la souris sur le projet **TeamsApp**.

1. Développez le menu **Teams Toolkit** et sélectionnez **Préparer les dépendances de l’application Teams**.

1. Dans la boîte de dialogue **Compte Microsoft 365**, sélectionnez **Continuer**.

1. Dans la boîte de dialogue **Approvisionner**, sélectionnez **Approvisionner**.

1. Dans la boîte de dialogue d’**avertissement de Teams Toolkit**, sélectionnez **Approvisionner**.

1. Dans la boîte de dialogue d’**informations de Teams Toolkit**, sélectionnez l’icône en forme de croix pour fermer la boîte de dialogue.

## Tâche 6 - Exécuter et déboguer

Une fois les ressources approvisionnées, démarrez une session de débogage pour tester l’extension de message.

Tout d’abord, démarrez le proxy de développement pour simuler l’API personnalisée.

1. Dans la **fenêtre d’invite de commandes** que vous avez laissée ouverte, exécutez la commande suivante pour démarrer le proxy de développement :

   ```bash
   devproxy --config-file "~appFolder/presets/learn-copilot-me-plugin/products-api-config.json"
   ```

1. Si vous y êtes invité, acceptez tout avertissement de certificat éventuel.

> [!NOTE]
> Lorsque le proxy de développement est en cours d’exécution, il agit comme proxy au niveau du système entier.

Ensuite, démarrez une session de débogage dans Visual Studio.

1. Pour démarrer une nouvelle session de débogage, appuyez sur <kbd>F5</kbd> ou sélectionnez **Démarrer** dans la barre d’outils.

1. Attendez qu’une fenêtre de navigateur s’ouvre et que la boîte de dialogue d’installation de l’application s’affiche dans le client web Microsoft Teams. Lorsque vous y êtes invité, saisissez vos informations d’identification de compte Microsoft 365.

1. Dans la boîte de dialogue d’installation de l’application, sélectionnez **Ajouter**.

1. Ouvrez une nouvelle conversation Microsoft Teams ou une conversation existante.

1. Dans la zone de rédaction du message, commencez à taper **/apps** pour ouvrir le sélecteur d’applications.

1. Dans la liste des applications, sélectionnez **Produits Contoso** pour ouvrir l’extension de message.

1. Saisissez **mark8** dans la zone de texte. Vous devrez peut-être saisir votre requête de recherche plusieurs fois.

1. Attendez que la recherche se termine et que les résultats s’affichent.

    ![Capture d’écran des résultats de la recherche retournés par une extension de message basée sur une recherche dans Microsoft Teams.](../media/3-search-results-api.png)

1. Dans la liste des résultats, sélectionnez un résultat de recherche pour incorporer une carte dans la zone de rédaction de message.

Revenez à Visual Studio et sélectionnez **Arrêter** dans la barre d’outils, ou appuyez sur <kbd>Maj</kbd> + <kbd>F5</kbd> pour arrêter la session de débogage. De plus, arrêtez le proxy de développement en appuyant sur <kbd>Ctrl</kbd> + <kbd>C</kbd>.

[Passez à l’exercice suivant…](./5-exercise-extend-optimize-message-extensions-copilot-microsoft-365.md)