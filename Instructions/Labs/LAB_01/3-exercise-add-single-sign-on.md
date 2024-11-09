---
lab:
  title: "Exercice\_2\_: ajouter l’authentification unique"
  module: 'LAB 01: Connect Copilot for Microsoft 365 to your external data in real-time with message extension plugins built with .NET and Visual Studio'
---

# Exercice 2 : ajouter l’authentification unique

Dans cet exercice, vous ajoutez l’authentification unique à l’extension de message pour authentifier les requêtes utilisateur.

![Capture d’écran d’une demande d’authentification dans une extension de message basée sur une recherche.  Un lien de connexion s’affiche.](../media/2-sign-in.png)

### Durée de l’exercice

  - **Durée estimée :** 40 minutes

## Tâche 1 : configurer l’enregistrement d’une application API back-end

Tout d’abord, créez un enregistrement d’application Microsoft Entra pour l’API back-end. Dans le cadre de cet exercice, vous en créez un, mais dans un environnement de production, vous utiliserez une inscription d’application existante.

Dans une fenêtre de navigateur :

1. Accédez au [portail Azure](https://portal.azure.com).

1. Ouvrez le menu du portail, puis sélectionnez **Microsoft Entra ID**.

1. Sélectionnez **Gérer > Inscriptions d’applications**, puis **Nouvelle inscription**.

1. Dans le formulaire Inscrire une application, spécifiez les valeurs suivantes :

    1. **Nom** : API de produits

    1. **Types de comptes pris en charge** : comptes dans tous les répertoires organisationnels (tout locataire - multilocataire Microsoft Entra ID).

1. Sélectionnez **Inscrire** pour l’inscription d’application.

1. Dans le menu de gauche de l’inscription d’application, sélectionnez **Gérer > Exposer une API**.

1. À côté de l’**URI d’ID d’application**, sélectionnez **Ajouter** et **Enregistrer** pour créer un URI d’ID d’application.

1. Dans la section Étendues définies par cette API, sélectionnez **Ajouter une étendue**.

1. Dans le formulaire Ajouter une étendue, spécifiez les valeurs suivantes :

    1. **Nom de l’étendue** : Product.Read

    1. **Qui peut donner son consentement ?**  : administrateurs et utilisateurs

    1. **Nom d’affichage du consentement de l’administrateur** : lire les produits

    1. **Description du consentement de l’administrateur** : permet à l’application de lire les données de produits

    1. **Nom d’affichage du consentement de l’utilisateur** : lire les produits

    1. **Description du consentement de l’utilisateur** : permet à l’application de lire les données de produits

    1. **État** : activé

1. Sélectionnez le bouton **Ajouter une étendue** pour créer l’étendue.

Prenez ensuite note de l’ID d’inscription de l’application et de l’ID d’étendue. Vous allez avoir besoin de ces valeurs pour configurer l’inscription d’application utilisée pour obtenir un jeton d’accès pour l’API back-end.

1. Dans le menu de gauche de l’inscription d’application, sélectionnez **Manifeste**.

1. Copiez la valeur de la propriété **appId** et enregistrez-la pour une utilisation ultérieure.

1. Copiez la valeur de la propriété **oauth2Permissions.id** et enregistrez-la pour une utilisation ultérieure.

Étant donné que nous avons besoin de ces valeurs dans le projet, ajoutez-les au fichier d’environnement.

Dans Visual Studio, dans le projet **TeamsApp** :

1. Dans le dossier **env**, ouvrez **.env.local**.

1. Dans le fichier, ajoutez les variables d’environnement suivantes et définissez les valeurs selon l’**ID d’inscription de l’application** et l’**ID d’étendue** que vous avez enregistrés précédemment :

    ```text
    BACKEND_API_ENTRA_APP_ID=<app-registration-id>
    BACKEND_API_ENTRA_APP_SCOPE_ID=<scope-id>
    ```

1. Enregistrez les changements apportés.

## Tâche 2 : créer un fichier manifeste d’enregistrement pour l’authentification avec l’API back-end

Pour vous authentifier auprès de l’API back-end, vous avez besoin d’une inscription d’application pour obtenir un jeton d’accès avec lequel appeler l’API.

Ensuite, créez un fichier manifeste d’enregistrement d’application. Le manifeste définit les étendues d’autorisation de l’API et l’URI de redirection pour l’inscription de l’application.

Dans Visual Studio, dans le projet **TeamsApp** :

1. Dans le dossier **infra\entra**, créez un fichier (<kbd>Ctrl+Maj+A</kbd>) nommé **entra.products.api.manifest.json**.

1. Dans le fichier , ajoutez le code suivant :

    ```json
    {
      "id": "${{PRODUCTS_API_ENTRA_APP_OBJECT_ID}}",
      "appId": "${{PRODUCTS_API_ENTRA_APP_ID}}",
      "name": "${{APP_INTERNAL_NAME}}-product-api-${{TEAMSFX_ENV}}",
      "accessTokenAcceptedVersion": 2,
      "signInAudience": "AzureADMultipleOrgs",
      "optionalClaims": {
        "idToken": [],
        "accessToken": [
          {
            "name": "idtyp",
            "source": null,
            "essential": false,
            "additionalProperties": []
          }
        ],
        "saml2Token": []
      },
      "requiredResourceAccess": [
        {
          "resourceAppId": "${{BACKEND_API_ENTRA_APP_ID}}",
          "resourceAccess": [
            {
              "id": "${{BACKEND_API_ENTRA_APP_SCOPE_ID}}",
              "type": "Scope"
            }
          ]
        }
      ],
      "oauth2Permissions": [],
      "preAuthorizedApplications": [],
      "identifierUris": [],
      "replyUrlsWithType": [
        {
          "url": "https://token.botframework.com/.auth/web/redirect",
          "type": "Web"
        }
      ]
    }
    ```

1. Enregistrez les changements apportés.

La propriété **requiredResourceAccess** spécifie l’ID d’inscription de l’application et l’ID d’étendue de l’API back-end.

La propriété **replyUrlsWithType** spécifie l’URI de redirection utilisé par le service de jetons Bot Framework pour renvoyer le jeton d’accès au service de jetons après l’authentification de l’utilisateur.

Ensuite, mettez à jour le flux de travail automatisé pour créer et mettre à jour l’inscription de l’application.

Dans le projet **TeamsApp** :

1. Ouvrez **teamsapp.local.yml**.

1. Dans le fichier, recherchez l’étape qui utilise l’action **aadApp/update**.

1. Après l’action, ajoutez les actions **aadApp/create** et **aadApp/update** pour créer et mettre à jour l’inscription de l’application (à partir de **la ligne 31**) :

    ```yml
      - uses: aadApp/create
        with:
            name: ${{APP_INTERNAL_NAME}}-products-api-${{TEAMSFX_ENV}}
            generateClientSecret: true
            signInAudience: AzureADMultipleOrgs
        writeToEnvironmentFile:
            clientId: PRODUCTS_API_ENTRA_APP_ID
            clientSecret: SECRET_PRODUCTS_API_ENTRA_APP_CLIENT_SECRET
            objectId: PRODUCTS_API_ENTRA_APP_OBJECT_ID
            tenantId: PRODUCTS_API_ENTRA_APP_TENANT_ID
            authority: PRODUCTS_API_ENTRA_APP_OAUTH_AUTHORITY
            authorityHost: PRODUCTS_API_ENTRA_APP_OAUTH_AUTHORITY_HOST
    
      - uses: aadApp/update
        with:
            manifestPath: "./infra/entra/entra.products.api.manifest.json"
            outputFilePath : "./infra/entra/build/entra.products.api.${{TEAMSFX_ENV}}.json"
    ```

1. Enregistrez vos modifications

L’action **aadApp/create** crée une inscription d’application avec le nom et l’audience spécifiées et génère une clé secrète client. La **propriété writeToEnvironmentFile** écrit l’ID d’inscription de l’application, la clé secrète client, l’ID d’objet, l’ID de locataire, l’autorité et l’hôte d’autorité dans les fichiers d’environnement. La clé secrète client est chiffrée et stockée de manière sécurisée dans le fichier **env.local.user**. Le nom de la variable d’environnement pour la clé secrète client est préfixé par **SECRET_**, ce qui indique à Teams Toolkit de ne pas écrire la valeur dans les journaux.

L’action **aadApp/update** met à jour l’inscription de l’application avec le fichier manifeste spécifié.

## Tâche 3 : centraliser le nom du paramètre de connexion

Ensuite, centralisez le nom du paramètre de connexion dans le fichier d’environnement et mettez à jour la configuration de l’application pour qu’elle puisse accéder à la valeur de la variable d’environnement lors de l’exécution.

Poursuivez dans Visual Studio, dans le projet **TeamsApp** :

1. Dans le dossier **env**, ouvrez **.env.local**.

1. Dans le fichier , ajoutez le code suivant :

    ```text
    CONNECTION_NAME=ProductsAPI
    ```

1. Ouvrez **teamsapp.local.yml**.

1. Dans le fichier, recherchez l’étape qui utilise l’action **file/createOrUpdateJsonFile** ciblant le fichier **./appsettings.Development.json**. Mettez à jour le tableau de contenu afin d’inclure la variable d’environnement **CONNECTION_NAME** et d’écrire la valeur dans le fichier **appsettings.Development.json** fichier :

    ```yml
      - uses: file/createOrUpdateJsonFile
        with:
          target: ../ProductsPlugin/appsettings.Development.json
          content:
            BOT_ID: ${{BOT_ID}}
            BOT_PASSWORD: ${{SECRET_BOT_PASSWORD}}
            CONNECTION_NAME: ${{CONNECTION_NAME}}
    ```

1. Enregistrez les changements apportés.

Ensuite, mettez à jour la configuration de l’application pour qu’elle accède à la variable d’environnement **CONNECTION_NAME**.

Dans le projet **ProductsPlugin** :

1. Ouvrez **Config.cs**.

1. Dans la classe **ConfigOptions**, ajoutez une nouvelle propriété nommée **CONNECTION_NAME** :

    ```csharp
    public class ConfigOptions
    {
      public string BOT_ID { get; set; }
      public string BOT_PASSWORD { get; set; }
      public string CONNECTION_NAME { get; set; }
    }
    ```

1. Enregistrez les changements apportés.

1. Ouvrez **Program.cs**.

1. Dans le fichier, mettez à jour le code qui lit la configuration de l’application afin d’inclure la propriété **CONNECTION_NAME** :

    ```csharp
    var config = builder.Configuration.Get<ConfigOptions>();
    builder.Configuration["MicrosoftAppType"] = "MultiTenant";
    builder.Configuration["MicrosoftAppId"] = config.BOT_ID;
    builder.Configuration["MicrosoftAppPassword"] = config.BOT_PASSWORD;
    builder.Configuration["ConnectionName"] = config.CONNECTION_NAME;
    ```

1. Enregistrez les changements apportés.

Ensuite, mettez à jour le code du bot pour qu’il utilise le nom du paramètre de connexion au moment de l’exécution.

1. Dans le dossier **Search**, ouvrez **SearchApp.cs**.

1. Au début de la classe **SearchApp**, créez un constructeur qui accepte un objet **IConfiguration** et attribue la valeur de la propriété **CONNECTION_NAME** à un champ privé nommé **connectionName** :

    ```csharp
    private readonly string connectionName;
    public SearchApp(IConfiguration configuration)
    {
      connectionName = configuration["CONNECTION_NAME"];
    }  
    ```

1. Enregistrez les changements apportés.

## Tâche 4 : configurer le paramètre de connexion de l’API Products

Pour vous authentifier auprès de l’API back-end, vous devez configurer un paramètre de connexion dans la ressource Azure Bot.

Poursuivez dans Visual Studio, dans le projet **TeamsApp** :

1. Dans le dossier**infra**, ouvrez le fichier nommé **azure.parameters.local.json**.

1. Dans le fichier, ajoutez les paramètres **backendApiEntraAppClientId**, **productsApiEntraAppClientId**, **productsApiEntraAppClientSecret** et **connectionName** :

    ```json
    {
      "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {
        "resourceBaseName": {
          "value": "bot-${{RESOURCE_SUFFIX}}-${{TEAMSFX_ENV}}"
        },
        "botEntraAppClientId": {
          "value": "${{BOT_ID}}"
        },
        "botDisplayName": {
          "value": "${{APP_DISPLAY_NAME}}"
        },
        "botAppDomain": {
          "value": "${{BOT_DOMAIN}}"
        },
        "backendApiEntraAppClientId": {
          "value": "${{BACKEND_API_ENTRA_APP_ID}}"
        },
        "productsApiEntraAppClientId": {
          "value": "${{PRODUCTS_API_ENTRA_APP_ID}}"
        },
        "productsApiEntraAppClientSecret": {
          "value": "${{SECRET_PRODUCTS_API_ENTRA_APP_CLIENT_SECRET}}"
        },
        "connectionName": {
          "value": "${{CONNECTION_NAME}}"
        }
      }
    }
    ```

1. Enregistrez les changements apportés.

Ensuite, mettez à jour le fichier Bicep afin d’inclure les nouveaux paramètres et les transmettre à la ressource Azure Bot.

1. Dans le dossier **infra**, ouvrez le fichier nommé **azure.local.bicep**.

1. Dans le fichier, après la déclaration de paramètre **botAppDomain**, ajoutez les déclarations de paramètres **backendApiEntraAppClientId**, **productsApiEntraAppClientId**, **productsApiEntraAppClientSecret** et **connectionName** :

    ```bicep
    param backendApiEntraAppClientId string
    param productsApiEntraAppClientId string
    @secure()
    param productsApiEntraAppClientSecret string
    param connectionName string
    ```

1. Dans la déclaration de module **azureBotRegistration**, ajoutez les nouveaux paramètres :

    ```bicep
    module azureBotRegistration './botRegistration/azurebot.bicep' = {
      name: 'Azure-Bot-registration'
      params: {
        resourceBaseName: resourceBaseName
        botEntraAppClientId: botEntraAppClientId
        botAppDomain: botAppDomain
        botDisplayName: botDisplayName
        backendApiEntraAppClientId: backendApiEntraAppClientId
        productsApiEntraAppClientId: productsApiEntraAppClientId
        productsApiEntraAppClientSecret: productsApiEntraAppClientSecret
        connectionName: connectionName
      }
    }
    ```

1. Enregistrez les changements apportés.

Enfin, mettez à jour le fichier Bicep d’inscription du bot afin d’inclure le nouveau paramètre de connexion.

1. Dans le dossier **infra/botRegistration**, ouvrez **azurebot.bicep**.

1. Dans le fichier, après la déclaration de paramètre **botAppDomain**, ajoutez les déclarations de paramètre **backendApiEntraAppClientId**, **productsApiEntraAppClientId**, **productsApiEntraAppClientSecret** et **connectionName** :

    ```bicep
    param backendApiEntraAppClientId string
    param productsApiEntraAppClientId string
    @secure()
    param productsApiEntraAppClientSecret string
    param connectionName string
    ```

1. Dans le fichier, ajoutez une nouvelle ressource nommée **botServicesProductsApiConnection** à la fin du fichier :

    ```bicep
    resource botServicesProductsApiConnection 'Microsoft.BotService/botServices/connections@2022-09-15' = {
      parent: botService
      name: connectionName
      location: 'global'
      properties: {
        serviceProviderDisplayName: 'Azure Active Directory v2'
        serviceProviderId: '30dd229c-58e3-4a48-bdfd-91ec48eb906c'
        clientId: productsApiEntraAppClientId
        clientSecret: productsApiEntraAppClientSecret
        scopes: 'api://${backendApiEntraAppClientId}/Product.Read'
        parameters: [
          {
            key: 'tenantID'
            value: 'common'
          }
          {
            key: 'tokenExchangeUrl'
            value: 'api://${botAppDomain}/botid-${botEntraAppClientId}'
          }
        ]
      }
    }
    ```

1. Enregistrez les changements apportés.

## Tâche 5 : configurer l’authentification dans l’extension de message

Pour authentifier les requêtes des utilisateurs dans l’extension de message, utilisez le kit de développement logiciel (SDK) Bot Framework pour obtenir un jeton d’accès pour l’utilisateur à partir du service de jetons Bot Framework. Le jeton d’accès peut ensuite être utilisé pour accéder aux données à partir d’un service externe.

Pour simplifier le code, créez une classe d’assistance qui gère l’authentification utilisateur.

Poursuivez dans Visual Studio, dans le projet **ProductsPlugin** :

1. Créez un nouveau dossier nommé **Helpers**.

1. Dans le dossier **Helpers**, créez un fichier de classe nommé **AuthHelpers.cs**.

1. Dans le fichier , ajoutez le code suivant :

    ```csharp
    using Microsoft.Bot.Connector.Authentication;
    using Microsoft.Bot.Schema;
    using Microsoft.Bot.Schema.Teams;
    internal static class AuthHelpers
    {
        internal static async Task<MessagingExtensionResponse> CreateAuthResponse(UserTokenClient userTokenClient, string connectionName, Activity activity, CancellationToken cancellationToken)
        {
            var resource = await userTokenClient.GetSignInResourceAsync(connectionName, activity, null, cancellationToken);
            return new MessagingExtensionResponse
            {
                ComposeExtension = new MessagingExtensionResult
                {
                    Type = "auth",
                    SuggestedActions = new MessagingExtensionSuggestedAction
                    {
                        Actions = [
                            new() {
                                Type = ActionTypes.OpenUrl,
                                Value = resource.SignInLink,
                                Title = "Sign In",
                            },
                        ],
                    },
                },
            };
        }
        internal static async Task<TokenResponse> GetToken(UserTokenClient userTokenClient, string state, string userId, string channelId, string connectionName, CancellationToken cancellationToken)
        {
            var magicCode = string.Empty;
            if (!string.IsNullOrEmpty(state))
            {
                if (int.TryParse(state, out var parsed))
                {
                    magicCode = parsed.ToString();
                }
            }
            return await userTokenClient.GetUserTokenAsync(userId, connectionName, channelId, magicCode, cancellationToken);
        }
        internal static bool HasToken(TokenResponse tokenResponse) => tokenResponse != null && !string.IsNullOrEmpty(tokenResponse.Token);
    }
    ```

1. Enregistrez les changements apportés.

Les trois méthodes d’assistance de la classe **AuthHelpers** gèrent l’authentification utilisateur dans l’extension de message.

- La méthode **CreateAuthResponse** construit une réponse qui affiche un lien de connexion dans l’interface utilisateur. Le lien de connexion est récupéré à partir du service de jetons à l’aide de la méthode **GetSignInResourceAsync**.

- La méthode **GetToken** utilise le client du service de jetons pour obtenir un jeton d’accès pour l’utilisateur actuel. La méthode utilise un code magique pour vérifier l’authenticité de la requête.

- La méthode **HasToken** vérifie si la réponse du service de jetons contient un jeton d’accès. Si le jeton n’est pas null ou vide, la méthode retourne true.

Ensuite, mettez à jour le code de l’extension de message pour qu’il utilise les méthodes d’assistance pour authentifier les requêtes des utilisateurs.

1. Dans le dossier **Search**, ouvrez **SearchApp.cs**.

1. Au début du fichier, ajoutez l'instruction using suivante :

    ```csharp
    using Microsoft.Bot.Connector.Authentication;
    ```

1. Dans la méthode **OnTeamsMessagingExtensionQueryAsync**, ajoutez le code suivant au début de la méthode :

    ```csharp
    var userTokenClient = turnContext.TurnState.Get<UserTokenClient>();
    var tokenResponse = await AuthHelpers.GetToken(userTokenClient, query.State, turnContext.Activity.From.Id, turnContext.Activity.ChannelId, connectionName, cancellationToken);
    if (!AuthHelpers.HasToken(tokenResponse))
    {
        return await AuthHelpers.CreateAuthResponse(userTokenClient, connectionName, (Activity)turnContext.Activity, cancellationToken);
    }
    ```

1. Enregistrez les changements apportés.

Ensuite, ajoutez le domaine Service de jetons au fichier manifeste de l’application pour vous assurer que le client peut approuver le domaine lorsqu’il démarre un flux d’authentification unique.

Dans le projet **TeamsApp** :

1. Dans le dossier **appPackage**, ouvrez **manifest.json**.

1. Dans le fichier, mettez à jour le tableau **validDomains** et ajoutez le domaine du service de jetons :

    ```json
    "validDomains": [
        "token.botframework.com",
        "${{BOT_DOMAIN}}"
    ]
    ```

1. Enregistrez les changements apportés.

## Tâche 6 : créer et mettre à jour des ressources

Maintenant que tout est en place, exécutez le processus **Préparer les dépendances de l’application Teams** pour créer de nouvelles ressources et mettre à jour les ressources existantes.

> [!NOTE]
> Si votre approvisionnement ne parvient pas à préparer les dépendances, vérifiez que les valeurs de **BACKEND_API_ENTRA_APP_ID** et **BACKEND_API_ENTRA_APP_SCOPE_ID** dans **env.local** sont correctes.

Continuez dans Visual Studio.

1. Dans l’**Explorateur de solutions**, cliquez avec le bouton droit de la souris sur le projet **TeamsApp**.

1. Développez le menu **Teams Toolkit** et sélectionnez **Préparer les dépendances de l’application Teams**.

1. Dans la boîte de dialogue **Compte Microsoft 365**, sélectionnez **Continuer**.

1. Dans la boîte de dialogue **Approvisionner**, sélectionnez **Approvisionner**.

1. Dans la boîte de dialogue d’**avertissement de Teams Toolkit**, sélectionnez **Approvisionner**.

1. Dans la boîte de dialogue d’**informations de Teams Toolkit**, sélectionnez l’icône en forme de croix pour fermer la boîte de dialogue.

## Tâche 7 : exécuter et déboguer

Une fois les ressources approvisionnées, démarrez une session de débogage pour tester l’extension de message.

1. Pour démarrer une nouvelle session de débogage, appuyez sur <kbd>F5</kbd> ou sélectionnez **Démarrer** dans la barre d’outils.

1. Attendez qu’une fenêtre de navigateur s’ouvre et que la boîte de dialogue d’installation de l’application s’affiche dans le client web Microsoft Teams. Lorsque vous y êtes invité, saisissez vos informations d’identification de compte Microsoft 365.

1. Dans la boîte de dialogue d’installation de l’application, sélectionnez **Ajouter**.

1. Ouvrez une nouvelle conversation Microsoft Teams ou une conversation existante.

1. Dans la zone de rédaction du message, commencez à saisir **/apps** pour ouvrir le sélecteur d’applications.

1. Dans la liste des applications, sélectionnez **Produits Contoso** pour ouvrir l’extension de message.

1. Dans la zone de texte, saisissez **hello**. Vous devrez peut-être saisir votre recherche plusieurs fois.

1. Le message **Vous devez vous connecter pour utiliser cette application** s’affiche :

    ![Capture d’écran d’une demande d’authentification dans une extension de message basée sur une recherche.  Un lien de connexion s’affiche.](../media/2-sign-in.png)

1. Suivez le **lien de connexion** pour démarrer le flux d’authentification.

1. Donnez votre consentement pour les autorisations demandées et revenez à Microsoft Teams :

    ![Capture d’écran de la boîte de dialogue de consentement des autorisations d’API Microsoft Entra.](../media/18-api-permission-consent.png)

1. Attendez que la recherche se termine et que les résultats s’affichent.

1. Dans la liste des résultats, sélectionnez **hello** pour incorporer une carte dans la zone de rédaction de message.

Revenez à Visual Studio et sélectionnez **Arrêter** dans la barre d’outils ou appuyez sur <kbd>Maj</kbd> + <kbd>F5</kbd> pour arrêter la session de débogage.

[Passez à l’exercice suivant…](./4-exercise-return-data-api.md)