---
lab:
  title: "Exercice\_3\_- Récupérer des informations sur le produit à partir de SharePoint\_Online"
  module: 'LAB 03: Connect Copilot for Microsoft 365 to your external data in real-time with message extension plugins built with .NET and Visual Studio'
---

# Exercice 3 - Récupérer des informations sur le produit à partir de SharePoint Online

Dans cet exercice, vous approvisionnez et configurez un site SharePoint Online, qui stocke des informations sur le produit en tant qu’éléments d’une liste. Vous mettez à jour le code d’extension de message pour récupérer les éléments de liste à partir de SharePoint Online à l’aide du Kit de développement logiciel (SDK) Microsoft Graph et retourner les données des éléments de la liste dans les résultats de la recherche. Enfin, vous exécutez et déboguez votre extension de message et vous la testez dans Microsoft Teams.

:::image type="content" source="../media/4-search-results-sharepoint-online.png" alt-text="Capture d’écran des résultats de la recherche retournés par une extension de message basée sur une recherche dans Microsoft Teams. Les résultats de la recherche sont retournés à partir de SharePoint Online. Chaque résultat de recherche affiche le nom du produit, la catégorie et l’image du produit." lightbox="../media/4-search-results-sharepoint-online.png":::

## Tâche 1 - Approvisionner et configurer le site SharePoint de marketing produit

Commencez par créer un site SharePoint Online à l’aide du service SharePoint Look Book.

Dans un navigateur Web :

1. Accédez à **SharePoint Look Book** à l’adresse [https://lookbook.microsoft.com](https://lookbook.microsoft.com).
1. Dans la navigation supérieure, développez **Afficher les conceptions**.
1. Dans le menu **Afficher les conceptions**, développez **Équipe** et sélectionnez **Support technique**.
1. Sélectionnez **Ajouter votre locataire**.
1. Si vous y êtes invité, connectez-vous à votre compte locataire.
1. Dans l’écran de consentement des autorisations, passez en revue les autorisations requises et sélectionnez **Accepter** pour revenir au service SharePoint Look Book.
1. Dans le formulaire, acceptez les valeurs par défaut et sélectionnez **Approvisionner**.

Un e-mail est envoyé à votre adresse e-mail pour vous avertir lorsque l’approvisionnement du site est terminé. L’exécution de ce processus peut prendre plusieurs minutes.

:::image type="content" source="../media/1-sharepoint-online-product-support-site.png" alt-text="Capture d’écran de la page d’accueil du site d’équipe SharePoint Online du support produit. Une liste de produits récemment publiés s’affiche." lightbox="../media/1-sharepoint-online-product-support-site.png":::

Pour activer le filtrage sur les colonnes Titre et Catégorie de vente au détail lors de l’interrogation de la liste à l’aide de l’API Microsoft Graph, créez des index sur la liste.

Continuez dans le navigateur web :

1. Accédez au site de **Support technique** à l’adresse **<https://tenant.sharepoint.com/sites/productmarketing>**, et remplacez le **locataire** par le nom de votre instance SharePoint Online.
1. Dans la **barre de la suite Microsoft 365**, sélectionnez la **roue dentée des paramètres** pour ouvrir le panneau latéral Paramètres.
1. Sous le titre **SharePoint**, sélectionnez **Contenu du site**.
1. Dans la liste des listes et des bibliothèques, pointez sur la liste **Produits**, sélectionnez l’icône des **trois points verticaux** pour développer le menu **Afficher les actions**, puis sélectionnez **Paramètres**.
1. Dans la section **Colonnes**, sous la liste des colonnes, sélectionnez **Colonnes indexées**.
1. Sélectionnez **Créer un index**.

## Tâche 2 - Ajouter des variables d’environnement de nom d’hôte et d’URL de site SharePoint

Nous centralisons ensuite le nom d’hôte de votre instance SharePoint Online et l’URL du site de support technique en tant que variables d’environnement. Vous exposez ensuite les valeurs en tant que variable d’environnement à utiliser au moment de l’exécution et mettez à jour le code pour lire la valeur.

Ouvrez Visual Studio :

1. Dans le dossier **env**, ouvrez le fichier nommé **.env.local**.
1. Dans le fichier, ajoutez les variables d’environnement **SPO_HOSTNAME** et **SPO_SITE_URL**, en remplaçant **locataire** par le nom de votre instance SharePoint Online :

    ```text
    SPO_HOSTNAME=tenant.sharepoint.com
    SPO_SITE_URL=sites/productmarketing
    ```

1. Enregistrez vos modifications

Ensuite, mettez à jour l’action pour écrire les variables d’environnement dans le fichier de paramètres de l’application.

1. Dans le dossier racine du projet, ouvrez le fichier nommé **teamsapp.local.yml**.
1. Recherchez l’étape qui utilise l’action **file/createOrUpdateJsonFile**, qui cible le fichier **./appsettings.Development.json**.
1. Dans le fichier, mettez à jour le tableau **content**, en ajoutant les variables **SPO_HOSTNAME** et **SPO_SITE_URL** :

    ```yml
      - uses: file/createOrUpdateJsonFile
        with:
          target: ./appsettings.Development.json
          content:
            BOT_ID: ${{BOT_ID}}
            BOT_PASSWORD: ${{SECRET_BOT_PASSWORD}}
            CONNECTION_NAME: ${{CONNECTION_NAME}}
            SPO_HOSTNAME: ${{SPO_HOSTNAME}}
            SPO_SITE_URL: ${{SPO_SITE_URL}}
    ```

1. Enregistrez vos modifications

À présent, mettez à jour la classe ConfigOptions pour inclure les nouvelles variables d’environnement.

1. Dans le dossier racine du projet, ouvrez Config.cs.
1. Dans la classe ConfigOptions, ajoutez de nouvelles propriétés de chaîne nommées SPO_HOSTNAME et SPO_SITE_URL.

    ```csharp
    public class ConfigOptions
    {
      public string BOT_ID { get; set; }
      public string BOT_PASSWORD { get; set; }
      public string CONNECTION_NAME { get; set; }
      public string SPO_HOSTNAME { get; set; }
      public string SPO_SITE_URL { get; set; }
    }
    ```

1. Enregistrez vos modifications

Ensuite, mettez à jour la configuration de l’application avec les deux variables d’environnement.

1. Dans le dossier racine du projet, ouvrez Program.cs.
1. Ajoutez de nouvelles lignes pour ajouter les variables d’environnement SPO_HOSTNAME et SPO_SITE_URL en tant que paramètres de configuration de l’application.

    ```csharp
    var config = builder.Configuration.Get<ConfigOptions>();
    builder.Configuration["MicrosoftAppType"] = "MultiTenant";
    builder.Configuration["MicrosoftAppId"] = config.BOT_ID;
    builder.Configuration["MicrosoftAppPassword"] = config.BOT_PASSWORD;
    builder.Configuration["CONNECTION_NAME"] = config.CONNECTION_NAME;
    builder.Configuration["SPO_HOSTNAME"] = config.SPO_HOSTNAME;
    builder.Configuration["SPO_SITE_URL"] = config.SPO_SITE_URL;
    ```

1. Enregistrez vos modifications

La dernière étape consiste à mettre à jour le gestionnaire d’activités du bot pour lire les valeurs de la configuration de l’application et stocker la valeur dans des propriétés en lecture seule.

1. Dans le dossier de recherche, ouvrez le fichier nommé SearchApp.cs.
1. Dans la classe SearchApp, créez des propriétés de chaîne en lecture seule nommées spoHostname et spoSiteUrl.

    ```csharp
    public class SearchApp : TeamsActivityHandler
    {
      private readonly string connectionName;
      private readonly string spoHostname;
      private readonly string spoSiteUrl;
    }
    ```

1. Mettez à jour le constructeur pour définir les valeurs de propriété à l’aide de la configuration de l’application injectée :

    ```csharp
    public SearchApp(IConfiguration configuration)
    {
        connectionName = configuration["CONNECTION_NAME"];
        spoHostname = configuration["SPO_HOSTNAME"];
        spoSiteUrl = configuration["SPO_SITE_URL"];
    } 
    ```

1. Enregistrez vos modifications.

## Tâche 3 - Mettre à jour la commande de recherche

Lorsque l’extension de message retourne des informations sur le produit, mettez à jour le titre et la description de la commande de recherche, et mettez également à jour le nom du paramètre et sa description.

Continuez dans Visual Studio.

1. Dans le dossier **appPackage**, ouvrez le fichier nommé **manifest.json**.
1. Dans le tableau **composeExtensions**, mettez à jour l’objet de commande avec :

    ```json
    "composeExtensions": [
      {
        "botId": "${{BOT_ID}}",
        "commands": [
          {
            "id": "Search",
            "type": "query",
            "title": "Products",
            "description": "Find products by name",
            "initialRun": false,
            "fetchTask": false,
            "context": [
              "commandBox",
              "compose",
              "message"
            ],
            "parameters": [
              {
                "name": "ProductName",
                "title": "Product name",
                "description": "The name of the product as a keyword",
                "inputType": "text"
              }
            ]
          }
        ]
      }
    ],
    ```

1. Enregistrez vos modifications

## Tâche 4 - Obtenir la valeur de requête de l’utilisateur

Lorsque la méthode OnTeamsMessagingExtensionQueryAsync est exécutée, la première chose à faire est de comprendre ce que l’utilisateur a entré dans la zone de recherche.

Nous allons tout d’abord supprimer le code existant.

Continuez dans Visual Studio.

1. Dans le dossier **Search**, ouvrez le fichier nommé **SearchApp.cs**.
1. Dans la méthode **OnTeamsMessagingExtensionQueryAsync**, supprimez tout le code **après** l’instruction **if** qui recherche un jeton d’accès.
1. Dans la classe **SearchApp**, supprimez la méthode **FindPackages** et la propriété **_adaptiveCardFilePath**.

Après avoir supprimé le code existante, votre classe **SearchApp** doit ressembler à l’extrait de code suivant :

```csharp
public class SearchApp : TeamsActivityHandler
{
    private readonly string connectionName;
    private readonly string spoHostname;
    private readonly string spoSiteUrl;

    public SearchApp(IConfiguration configuration)
    {
        connectionName = configuration["CONNECTION_NAME"];
        spoHostname = configuration["SPO_HOSTNAME"];
        spoSiteUrl = configuration["SPO_SITE_URL"];
    }

    protected override async Task<MessagingExtensionResponse> OnTeamsMessagingExtensionQueryAsync(ITurnContext<IInvokeActivity> turnContext, MessagingExtensionQuery query, CancellationToken cancellationToken)
    {
        var userTokenClient = turnContext.TurnState.Get<UserTokenClient>();
        var tokenResponse = await GetToken(userTokenClient, query.State, turnContext.Activity.From.Id, turnContext.Activity.ChannelId, connectionName, cancellationToken);

        if (!HasToken(tokenResponse))
        {
            return await CreateAuthResponse(userTokenClient, connectionName, (Activity)turnContext.Activity, cancellationToken);
        }
    }
}
```

Nous allons maintenant écrire le code pour obtenir la valeur du paramètre **ProductName**.

1. Dans la méthode **OnTeamsMessagingExtensionQueryAsync**, ajoutez du code pour récupérer la valeur du paramètre **ProductName** à partir du tableau **Parameters** dans l’objet **MessagingExtensionQuery**.

    ```csharp
    var name = GetQueryData(query.Parameters, "ProductName");
    ```

1. Dans la classe **SearchApp**, implémentez la méthode **GetQueryData**.

    ```csharp
    private static string GetQueryData(IList<MessagingExtensionParameter> parameters, string key)
    {
      if (parameters.Any() != true)
      {
        return string.Empty;
      }
    
      var foundPair = parameters.FirstOrDefault(pair => pair.Name == key);
      return foundPair?.Value?.ToString() ?? string.Empty;
    }
    ```

1. Enregistrez vos modifications

La méthode **GetQueryData** est utilisée pour récupérer la valeur associée à une clé spécifique à partir d’une liste d’objets **MessagingExtensionParameter**. Elle offre un moyen pratique d’extraire des données du tableau de paramètres dans l’objet **MessagingExtensionQuery**.

## Tâche 5 - Créer un filtre OData de requête de liste SharePoint

Maintenant que nous avons la valeur transmise par l’utilisateur, utilisez cette valeur pour créer un filtre de requête OData. Le filtre est utilisé pour interroger la liste SharePoint Online sur la colonne Title, qui contient le nom du produit.

Continuez dans Visual Studio.

1. Dans la méthode **OnTeamsMessagingExtensionQueryAsync**, ajoutez du code pour créer la variable **filterQuery**.

    ```csharp
    var nameFilter = !string.IsNullOrEmpty(name) ? $"startswith(fields/Title, '{name}')" : string.Empty;
    var filters = new List<string> { nameFilter };
    var filterQuery = filters.Count == 1 ? filters.FirstOrDefault() : string.Join(" and ", filters);
    ```

1. Enregistrez vos modifications

Le code construit une requête de filtre basée sur le paramètre **name**. Si le paramètre de nom est fourni, il crée une expression de filtre pour rechercher des éléments avec un champ **Title** commençant par le nom fourni. Si le paramètre de nom n’est pas fourni, il affecte une chaîne vide en tant que requête de filtre. La requête de filtre obtenue est utilisée plus loin dans le code pour récupérer des éléments filtrés à partir d’un site SharePoint.

## Tâche 6 - Installer et configurer le Kit de développement logiciel (SDK) Microsoft Graph

Pour exécuter des demandes authentifiées auprès de Microsoft Graph, vous utilisez le **Kit de développement logiciel (SDK) Microsoft Graph**.

Installez le package du Kit de développement logiciel (SDK) Microsoft Graph à partir de NuGet, puis créez une classe **TokenProvider**, qui vous permet d’utiliser le jeton d’accès obtenu à partir du service de jetons, puis d’initialiser un nouveau **GraphServiceClient**.

Continuez dans Visual Studio.

1. Dans l’Explorateur de solutions, cliquez avec le bouton droit sur le projet **MsgExtProductSupport**.
1. Sélectionnez **Gérer les packages NuGet…**
1. Sélectionnez l’onglet **Parcourir** et recherchez **Microsoft.Graph**.
1. Dans la liste des résultats, sélectionnez **Microsoft Graph**.
1. Dans la liste déroulante **Version**, sélectionnez **5.42.0**.
1. Sélectionnez **Installer**
1. Dans la boîte de dialogue **Acceptation de la licence**, sélectionnez **J’accepte** pour installer le Kit de développement logiciel (SDK).

Après avoir installé le package, créez un fournisseur de jetons pour le Kit de développement logiciel (SDK) Microsoft Graph.

1. Dans le dossier **Search**, créez un fichier nommé **TokenProvider.cs**.
1. Dans le fichier , ajoutez le code suivant :

    ```csharp
    using Microsoft.Kiota.Abstractions.Authentication;
    
    namespace MsgExtProductSupport.Search
    {
       public class TokenProvider : IAccessTokenProvider
        {
            public string Token { get; set; }
            public AllowedHostsValidator AllowedHostsValidator => throw new NotImplementedException();
    
            public Task<string> GetAuthorizationTokenAsync(Uri uri, Dictionary<string, object>? additionalAuthenticationContext = null, CancellationToken cancellationToken = default)
            {
                return Task.FromResult(Token);
            }
        }
    }
    ```

1. Enregistrez vos modifications

Développez maintenant une méthode pour créer une instance **GraphServiceClient**.

1. Dans le dossier **Search**, ouvrez le fichier nommé **SearchApp.cs**.
1. Dans le fichier, importez les espaces de noms dont vous avez besoin :

    ```csharp
    using Microsoft.Graph;
    using Microsoft.Kiota.Abstractions.Authentication;
    ```

1. Dans la méthode **OnTeamsMessagingExtensionQueryAsync**, ajoutez du code pour créer un client Graph que vous utiliserez pour envoyer des requêtes à Microsoft Graph.

    ```csharp
    var graphClient = CreateGraphClient(tokenResponse);
    ```

1. Dans la classe **SearchApp**, implémentez la méthode **CreateGraphClient**.

    ```csharp
    private static GraphServiceClient CreateGraphClient(TokenResponse tokenResponse)
    {
      TokenProvider provider = new() { Token = tokenResponse.Token };
      var authenticationProvider = new BaseBearerTokenAuthenticationProvider(provider);
      var graphClient = new GraphServiceClient(authenticationProvider);
      return graphClient;
    }
    ```

1. Enregistrez vos modifications

Ce code configure le fournisseur d’authentification et crée un objet client qui peut être utilisé pour interagir avec l’API Microsoft Graph à l’aide du jeton d’accès fourni.

## Tâche 7 - Interroger la liste des produits

Pour interroger les éléments de la liste des produits et créer ultérieurement les résultats de la recherche, vous utilisez GraphServiceClient pour envoyer des demandes dans le but d’obtenir des données de produit à partir de SharePoint Online.

Continuez dans Visual Studio.

Dans la méthode **OnTeamsMessagingExtensionQueryAsync**, ajoutez du code pour obtenir les données SharePoint :

  ```csharp
  var site = await GetSharePointSite(graphClient, spoHostname, spoSiteUrl, cancellationToken);
  var drive = await GetSharePointDrive(graphClient, site.SharepointIds.SiteId, "Product Imagery", cancellationToken);
  var items = await GetProducts(graphClient, site.SharepointIds.SiteId, filterQuery, cancellationToken);
  ```

Ce code :

- **Obtenir le site de marketing produit**, l’objet de site contient l’ID du site SharePoint, qui est utilisé pour retourner et interroger des objets dans le site.
- **Obtenir le lecteur d’image de produit**, le lecteur représente la bibliothèque de documents qui contient les images de produit. Vous utilisez le lecteur ultérieurement pour obtenir les images de produit que vous affichez dans les résultats de la recherche.
- **Obtenir les produits**, que vous utilisez pour interroger la liste des produits en fonction de la requête d’utilisateur.

Implémentez les trois méthodes de la classe **SearchApp**.

- Implémenter la **méthode GetSharePointSite**

    ```csharp
    private static async Task<Site> GetSharePointSite(GraphServiceClient graphClient, string hostName, string siteUrl, CancellationToken cancellationToken)
    {
        return await graphClient.Sites[$"{hostName}:/{siteUrl}"].GetAsync(r => r.QueryParameters.Select = new string[] { "sharePointIds" }, cancellationToken);
    }
    ```

Cette méthode utilise GraphServiceClient pour envoyer une demande pour que Microsoft Graph retourne l’objet de site à l’aide d’un chemin. Le chemin  est créé en combinant le nom d’hôte SharePoint Online et l’URL de site. Étant donné que vous n’avez besoin que des valeurs de propriété sharePointIds, le paramètre de requête Select est configuré pour ne renvoyer que propriété dans la réponse.

- Implémenter la méthode **GetSharePointDrive**

    ```csharp
    private static async Task<Drive> GetSharePointDrive(GraphServiceClient graphClient, string siteId, string name, CancellationToken cancellationToken)
    {
        var drives = await graphClient.Sites[siteId].Drives.GetAsync(r => r.QueryParameters.Select = new string[] { "id", "name" }, cancellationToken);
        var drive = drives.Value.Find(d => d.Name == name);
        return drive;
    }
    ```

Cette méthode utilise GraphServiceClient et l’ID de site pour retourner une collection de bibliothèques de documents à partir du site, en retournant les propriétés d’ID et de nom de chaque bibliothèque de documents. La collection de bibliothèques est ensuite filtrée pour retourner le lecteur, qui a le même nom que le paramètre de méthode name.

- Implémenter la méthode **GetProducts**

    ```csharp
    private static async Task<SiteCollectionResponse> GetProducts(GraphServiceClient graphClient, string siteId, string filterQuery, CancellationToken cancellationToken)
    {
        var fields = new string[]
        {
            "fields/Id",
            "fields/Title",
            "fields/RetailCategory",
            "fields/PhotoSubmission",
            "fields/CustomerRating",
            "fields/ReleaseDate"
        };
    
        var request = graphClient.Sites.WithUrl($"https://graph.microsoft.com/v1.0/sites/{siteId}/lists/Products/items?expand={string.Join(",", fields)}&$filter={filterQuery}");
        return await request.GetAsync(null, cancellationToken);
    }
    ```

Cette méthode utilise GraphServiceClient pour retourner les éléments de liste filtrés de la liste des produits à l’aide de la requête de filtre transmise et retourne les données d’élément de liste définies dans le tableau de champs.

## Tâche 8 - Créer des résultats de la recherche

Après avoir obtenu les produits à partir de SharePoint, vous allez créer les résultats de la recherche, qui seront retournés à l’utilisateur.

La création des résultats de la recherche consiste à effectuer une itération sur le tableau d’éléments. Pour chaque élément, vous créez un MessagingExtensionAttachment, qui contient les cartes d’aperçu et de contenu.

Avant d’effectuer une itération sur les éléments, créez un modèle de carte adaptative que vous utiliserez dans votre boucle.

Continuez dans Visual Studio.

1. Dans le dossier **Resources**, créez un fichier nommé **Product.json**.

    ```json
    {
      "type": "AdaptiveCard",
      "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
      "version": "1.6",
      "body": [
        {
          "type": "TextBlock",
          "text": "${Product.Title}",
          "wrap": true,
          "style": "heading"
        },
        {
          "type": "TextBlock",
          "text": "${Product.RetailCategory}",
          "wrap": true
        },
        {
          "type": "Container",
          "items": [
            {
              "type": "Image",
              "url": "${ProductImage}",
              "altText": "${Product.Title}"
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
              "value": "${formatNumber(Product.CustomerRating,0)}"
            },
            {
              "title": "Release Date",
              "value": "${formatDateTime(Product.ReleaseDate,'dd/MM/yyyy')}"
            }
          ]
        },
        {
          "type": "ActionSet",
          "actions": [
            {
              "type": "Action.OpenUrl",
              "title": "View",
              "url": "https://${SPOHostname}/${SPOSiteUrl}/Lists/Products/DispForm.aspx?ID=${Product.Id}"
            }
          ]
        }
      ]
    }
    ```

Le modèle utilise des expressions de liaison de données, qui sont remplacées par des valeurs réelles lors du rendu de la carte adaptative. Lorsque la carte est affichée, elle contient des informations sur le produit, une image de produit et un bouton d’action. Le bouton d’action ouvre un navigateur qui accède au formulaire d’affichage de liste SharePoint de l’élément de produit.

Ajoutez ensuite le code pour transformer le fichier JSON en modèle de carte adaptative.

1. Dans le dossier **Search**, ouvrez **SearchApp.cs**.
1. Dans le fichier, importez les espaces de noms dont vous avez besoin :

    ```csharp
    using AdaptiveCards.Templating;
    ```

1. Dans la méthode **OnTeamsMessagingExtensionQueryAsync**, ajoutez le code pour lire le contenu du fichier JSON et créer un objet **AdaptiveCardTemplate**.

    ```csharp
    var card = File.ReadAllText(@"Resources\Product.json");
    var template = new AdaptiveCardTemplate(card);
    ```

1. Enregistrez vos modifications

Vous créez ensuite une boucle pour itérer sur les éléments de liste. Chaque itération permettra de :

- Désérialiser les données d’élément actuelles dans un objet Product
- Obtenir la miniature de l’image de produit
- Créer une carte de contenu
- Créer une carte d’aperçu
- Créer un MessagingExtensionAttachment combinant les cartes de contenu et d’aperçu
- Ajouter MessagingExtensionAttachment à la liste

Une fois la boucle terminée ; vous disposez d’une liste de pièces jointes qui peuvent être retournées à l’utilisateur.

1. Dans le dossier **Search**, ouvrez **SearchApp.cs**.
1. Dans la méthode **OnTeamsMessagingExtensionQueryAsync**, ajoutez du code pour créer une liste, dans laquelle stocker des objets **MessagingExtensionAttachment**.

    ```csharp
    var attachments = new List<MessagingExtensionAttachment>();
    ```

1. Créez une boucle foreach pour itérer sur les éléments de liste.

    ```csharp
    foreach (var item in items.Value) { 
            
    }
    ```

1. Ajoutez le code suivant à la boucle foreach.

    ```csharp
    var product = JsonConvert.DeserializeObject<Product>(item.AdditionalData["fields"].ToString());
    product.Id = item.Id;
    
    var thumbnails = await GetThumbnails(graphClient, drive.Id, product.PhotoSubmission, cancellationToken);
    
    var resultCard = template.Expand(new
    {
      Product = product,
      ProductImage = thumbnails.Large.Url,
      SPOHostname = spoHostname,
      SPOSiteUrl = spoSiteUrl,
    });
    
    var previewcard = new ThumbnailCard
    {
      Title = product.Title,
      Subtitle = product.RetailCategory,
      Images = new List<CardImage> { new() { Url = thumbnails.Small.Url } }
    }.ToAttachment();
    
    var attachment = new MessagingExtensionAttachment
    {
      Content = JsonConvert.DeserializeObject(resultCard),
      ContentType = AdaptiveCard.ContentType,
      Preview = previewcard
    };
    
    attachments.Add(attachment);
    ```

1. Enregistrez vos modifications

Pour effectuer un typage fort des données d’élément de liste, créez un modèle qui représente le **produit**.

1. Dans le dossier racine du projet, créez un dossier nommé **Models**.
1. Dans le dossier **Models**, créez un fichier appelé **Product.cs**.

    ```csharp
    namespace MsgExtProductSupport.Models
    {
        public class Product
        {
            public string Title { get; set; }
            public string RetailCategory { get; set; }
            public Link Specguide { get; set; }
            public string PhotoSubmission { get; set; }
            public double CustomerRating { get; set; }
            public DateTime ReleaseDate { get; set; }
            public string Id { get; set; }
            public string ContentType { get; set; }
            public DateTime Modified { get; set; }
            public DateTime Created { get; set; }
        }
    
        public class Link
        {
            public string Description { get; set; }
            public string Url { get; set; }
        }
    }
    ```

1. Enregistrez vos modifications

Implémentez ensuite la méthode **GetThumbnails** pour récupérer des images miniatures de Microsoft Graph pour un produit.

1. Dans le dossier **Search**, ouvrez le fichier nommé **SearchApp.cs**.
1. Dans la classe **SearchApp**, créez la méthode **GetThumbnails**.

    ```csharp
    private static async Task<ThumbnailSet> GetThumbnails(GraphServiceClient graphClient, string driveId, string photoUrl, CancellationToken cancellationToken)
    {
        var fileName = photoUrl.Split('/').Last();
        var driveItem = await graphClient.Drives[driveId].Root.ItemWithPath(fileName).GetAsync(null, cancellationToken);
        var thumbnails = await graphClient.Drives[driveId].Items[driveItem.Id].Thumbnails["0"].GetAsync(r => r.QueryParameters.Select = new string[] { "small", "large" }, cancellationToken);
        return thumbnails;
    }
    ```

1. Enregistrez vos modifications

La méthode **GetThumbnails** utilise le point de terminaison Miniatures dans l’API Microsoft Graph pour retourner des miniatures de petite et grande taille de l’image de produit stockée dans SharePoint.

## Tâche 9 - Retourner les résultats de la recherche

Maintenant que nous avons une collection d’objets MessagingExtensionResult, nous pouvons les retourner à l’utilisateur en tant que résultats de la recherche.

- Dans la méthode **OnTeamsMessagingExtensionQueryAsync**, ajoutez du code pour retourner les résultats de la recherche en tant que réponse d’extension de message.

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

## Tâche 10 - Approvisionner des ressources

Exécutez le processus Préparer les dépendances de l’application Teams pour approvisionner des ressources.

Continuez dans Visual Studio.

1. Dans l’**Explorateur de solutions**, cliquez avec le bouton droit sur le projet **MsgExtProductSupport**.
1. Développez le menu **Teams Toolkit** et sélectionnez **Préparer les dépendances de l’application Teams**.
1. Dans la boîte de dialogue **Compte Microsoft 365**, sélectionnez **Continuer**.
1. Dans la boîte de dialogue **Approvisionner**, sélectionnez **Approvisionner**.
1. Dans la boîte de dialogue d’**avertissement de Teams Toolkit**, sélectionnez **Approvisionner**.
1. Dans la boîte de dialogue d’**informations de Teams Toolkit**, **fermez** l’invite.

## Tâche 11 - Exécuter et déboguer

Démarrez maintenant le service web et testez l’extension de message dans Microsoft Teams.

Continuez dans Visual Studio.

1. Appuyez sur la touche **F5** pour démarrer une session de débogage et ouvrir une nouvelle fenêtre de navigateur qui accède au client web Microsoft Teams.
1. Entrez vos informations d’identification de compte Microsoft 365 et passez à Microsoft Teams.
1. Dans la boîte de dialogue d’installation de l’application, sélectionnez **Ajouter**.
1. Ouvrez une conversation Microsoft Teams, nouvelle ou existante.
1. Dans la zone de rédaction du message, sélectionnez **...** pour ouvrir le menu volant de l’application.
1. Dans la liste des applications, sélectionnez **Produits Contoso** pour ouvrir l’extension de message.
1. Entrez **Mark8** dans la zone de texte. Deux résultats sont affichés, **Mark8** et **Contrôleur Mark8**.
1. Sélectionnez **Mark8** pour incorporer une carte dans la zone de rédaction du message.
1. **Envoyez le message** contenant la carte.
1. Dans la carte envoyée, sélectionnez la touche **Affichage** pour afficher l’élément de liste SharePoint du produit dans la liste de produits dans un nouvel onglet.

Fermez le navigateur pour arrêter pour la session de débogage.

[Passez à l’exercice suivant…](./5-exercise-extend-optimize-message-extensions-copilot-microsoft-365.md)