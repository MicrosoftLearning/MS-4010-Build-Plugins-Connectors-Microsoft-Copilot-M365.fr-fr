---
lab:
  title: "Exercice\_2\_: ajouter l’authentification unique"
  module: 'LAB 03: Connect Copilot for Microsoft 365 to your external data in real-time with message extension plugins built with .NET and Visual Studio'
---

# Exercice 2 : ajouter l’authentification unique

Dans cet exercice, vous mettrez à jour l’extension de message afin que les utilisateurs soient invités à se connecter et à s’authentifier. Vous configurerez l’inscription de l’application Bot Microsoft Entra et le manifeste de l’application pour activer l’authentification unique. Puis vous configurerez une inscription d’application Microsoft Entra pour l’authentification auprès de Microsoft Graph et mettrez à jour la logique d’extension de message pour obtenir un jeton d’accès à l’aide du service de jetons Bot Framework. Ensuite, vous exécuterez et déboguerez votre extension de message pour la tester dans Microsoft Teams.

## Tâche 1 : configurer l’authentification unique

Configurez d’abord l’inscription d’application Bot Microsoft Entra.

Dans Visual Studio :

1. Dans le dossier **infra\entra**, ouvrez le fichier nommé **entra.bot.manifest.json**.
1. Dans le fichier, mettez à jour le tableau **identifierUris** pour définir l’URI de l’ID d’application.

    ```json
    "identifierUris": [
        "api://${{BOT_DOMAIN}}/botid-${{BOT_ID}}"
    ],
    ```

1. Dans le fichier, mettez à jour le tableau **oauth2Permissions** pour créer une étendue qui permettra à Teams d’appeler des API web en tant qu’administrateur ou utilisateur :

    ```json
      "oauth2Permissions": [
        {
          "adminConsentDescription": "Allows Teams to call the app's web APIs as the current user.",
          "adminConsentDisplayName": "Teams can access app's web APIs",
          "id": "${{AAD_APP_ACCESS_AS_USER_PERMISSION_ID}}",
          "isEnabled": true,
          "type": "User",
          "userConsentDescription": "Enable Teams to call this app's web APIs with the same rights that you have",
          "userConsentDisplayName": "Teams can access app's web APIs and make requests on your behalf",
          "value": "access_as_user"
        }
      ]
    ```

1. Dans le fichier, mettez à jour le tableau **preAuthorizedApplications** pour ajouter les clients de Microsoft Teams, Microsoft Outlook et Copilot pour Microsoft 365 à la liste des clients autorisés :

    ```json
      "preAuthorizedApplications": [
        {
          "appId": "1fec8e78-bce4-4aaf-ab1b-5451cc387264",
          "permissionIds": [
            "${{AAD_APP_ACCESS_AS_USER_PERMISSION_ID}}"
          ]
        },
        {
          "appId": "5e3ce6c0-2b1f-4285-8d4b-75ee78787346",
          "permissionIds": [
            "${{AAD_APP_ACCESS_AS_USER_PERMISSION_ID}}"
          ]
        },
        {
          "appId": "4765445b-32c6-49b0-83e6-1d93765276ca",
          "permissionIds": [
            "${{AAD_APP_ACCESS_AS_USER_PERMISSION_ID}}"
          ]
        },
        {
          "appId": "0ec893e0-5785-4de6-99da-4ed124e5296c",
          "permissionIds": [
            "${{AAD_APP_ACCESS_AS_USER_PERMISSION_ID}}"
          ]
        },
        {
          "appId": "d3590ed6-52b3-4102-aeff-aad2292ab01c",
          "permissionIds": [
            "${{AAD_APP_ACCESS_AS_USER_PERMISSION_ID}}"
          ]
        },
        {
          "appId": "bc59ab01-8403-45c6-8796-ac3ef710b3e3",
          "permissionIds": [
            "${{AAD_APP_ACCESS_AS_USER_PERMISSION_ID}}"
          ]
        },
        {
          "appId": "27922004-5251-4030-b22d-91ecd9a37ea4",
          "permissionIds": [
            "${{AAD_APP_ACCESS_AS_USER_PERMISSION_ID}}"
          ]
        }
      ],
    ```

1. Enregistrez vos modifications

Ensuite, mettez à jour le fichier manifeste de l’application pour définir la ressource que le client doit utiliser lors du lancement d’un flux d’authentification unique dans l’application.

Continuez dans Visual Studio.

1. Dans le dossier **appPackage**, ouvrez le fichier nommé **manifest.json**.
1. Dans le fichier, ajoutez le code suivant après la **description** :

    ```json
    "webApplicationInfo": {
      "id": "${{BOT_ID}}",
      "resource": "api://${{BOT_DOMAIN}}/botid-${{BOT_ID}}"
    },
    ```

1. Enregistrez vos modifications

## Tâche 2 : créer un fichier manifeste d’inscription d’application Microsoft Entra pour Microsoft Graph

Pour vous authentifier auprès de Microsoft Graph, créez un fichier manifeste d’inscription d’application.

Continuez dans Visual Studio.

1. Dans le dossier **infra\entra**, créez un fichier nommé **entra.graph.manifest.json**.
2. Dans le fichier , ajoutez le code suivant :

    ```json
    {
      "id": "${{GRAPH_ENTRA_APP_OBJECT_ID}}",
      "appId": "${{GRAPH_ENTRA_APP_ID}}",
      "name": "${{APP_INTERNAL_NAME}}-graph-${{TEAMSFX_ENV}}",
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
          "resourceAppId": "Microsoft Graph",
          "resourceAccess": [
            {
              "id": "Sites.ReadWrite.All",
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

1. Enregistrez vos modifications

Le tableau **requiredResourceAccess** définit les étendues d’autorisation d’API et le tableau **replyUrlsWithType** définit les URI de redirection sur l’inscription de l’application.

À présent, mettez à jour le fichier projet avec des actions pour créer l’inscription de l’application.

1. Dans le dossier racine du projet, ouvrez **teamsapp.local.yml**.
1. Dans le fichier, recherchez l’étape qui utilise l’action **arm/deploy**.
1. Avant cette étape, ajoutez le code suivant :

    ```yml
      - uses: aadApp/create
        with:
          name: ${{APP_INTERNAL_NAME}}-graph-${{TEAMSFX_ENV}}
          generateClientSecret: true
          signInAudience: AzureADMultipleOrgs
        writeToEnvironmentFile:
          clientId: GRAPH_ENTRA_APP_ID
          clientSecret: SECRET_GRAPH_ENTRA_APP_CLIENT_SECRET
          objectId: GRAPH_ENTRA_APP_OBJECT_ID
          tenantId: GRAPH_ENTRA_APP_TENANT_ID
          authority: GRAPH_ENTRA_APP_OAUTH_AUTHORITY
          authorityHost: GRAPH_ENTRA_APP_OAUTH_AUTHORITY_HOST
    
      - uses: aadApp/update
        with:
          manifestPath: "./infra/entra/entra.graph.manifest.json"
          outputFilePath : "./build/entra.graph.manifest.${{TEAMSFX_ENV}}.json"
    ```

1. Enregistrez vos modifications

## Tâche 3 : créer un paramètre de connexion OAuth

Les paramètres de connexion Azure AI Bot Service sont utilisés pour gérer l’authentification utilisateur dans les bots et les extensions de message.

Tout d’abord, vous devez centraliser le nom du paramètre de connexion OAuth utilisé pour créer le paramètre de connexion en tant que variable d’environnement, puis ajouter du code pour utiliser la valeur de la variable d’environnement au moment de l’exécution.

Continuez dans Visual Studio.  

1. Dans le dossier **env**, ouvrez **.env.local**.
1. Dans le fichier , ajoutez le code suivant :

    ```text
    CONNECTION_NAME=MicrosoftGraph
    ```

1. Dans le dossier racine du projet, ouvrez le fichier nommé **teamsapp.local.yml**.
1. Dans le fichier, recherchez l’étape qui utilise l’action **file/createOrUpdateJsonFile** ciblant les fichiers **./appsettings.Development.json**.
1. Mettez à jour le tableau de contenu, en ajoutant la variable **CONNECTION_NAME**.

    ```yml
      - uses: file/createOrUpdateJsonFile
        with:
          target: ./appsettings.Development.json
          content:
            BOT_ID: ${{BOT_ID}}
            BOT_PASSWORD: ${{SECRET_BOT_PASSWORD}}
            CONNECTION_NAME: ${{CONNECTION_NAME}}
    ```

1. Enregistrez vos modifications
1. Dans le dossier racine du projet, ouvrez **Config.cs**.
1. Dans la classe **ConfigOptions**, ajoutez une nouvelle propriété de chaîne avec le nom **CONNECTION_NAME**.

    ```csharp
    public class ConfigOptions
    {
      public string BOT_ID { get; set; }
      public string BOT_PASSWORD { get; set; }
      public string CONNECTION_NAME { get; set; }
    }
    ```

1. Enregistrez vos modifications
1. Dans le dossier racine du projet, ouvrez le fichier nommé **Program.cs**.
1. Dans le fichier, ajoutez une nouvelle ligne pour ajouter la variable d’environnement **CONNECTION_NAME** en tant que paramètre de configuration d’application :

    ```csharp
    var config = builder.Configuration.Get<ConfigOptions>();
    builder.Configuration["MicrosoftAppType"] = "MultiTenant";
    builder.Configuration["MicrosoftAppId"] = config.BOT_ID;
    builder.Configuration["MicrosoftAppPassword"] = config.BOT_PASSWORD;
    builder.Configuration["CONNECTION_NAME"] = config.CONNECTION_NAME;
    ```

Ensuite, mettez à jour le gestionnaire d’activités du bot pour accéder à la configuration de l’application.

1. Dans le dossier **Search**, ouvrez le fichier nommé **SearchApp.cs**.
1. Dans la classe **SearchApp**, créez une propriété de chaîne en lecture seule avec le nom **connectionName**.

    ```csharp
    public class SearchApp : TeamsActivityHandler
    {
      private readonly string connectionName;
    }
    ```

1. Créez un constructeur, qui injecte la configuration de l’application et définit la valeur de la propriété connectionName.

    ```csharp
    public class SearchApp : TeamsActivityHandler
    {
      private readonly string connectionName;
    
      public SearchApp(IConfiguration configuration)
      {
        connectionName = configuration["CONNECTION_NAME"];
      }  
    }
    ```

1. Enregistrez vos modifications

Vous mettez ensuite à jour les fichiers Bicep pour approvisionner le paramètre de connexion OAuth.

Mettez tout d’abord à jour le fichier de paramètres pour transmettre les informations d’identification de l’inscription de l’application Microsoft Entra Microsoft Graph et le nom du paramètre de connexion.

1. Dans le dossier**infra**, ouvrez le fichier nommé **azure.parameters.local.json**.
1. Dans les paramètres **object**, ajoutez les paramètres **graphEntraAppClientId**, **graphEntraAppClientSecret** et **connectionName**.

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
        "graphEntraAppClientId": {
          "value": "${{GRAPH_ENTRA_APP_ID}}"
        },
        "graphEntraAppClientSecret": {
          "value": "${{SECRET_GRAPH_ENTRA_APP_CLIENT_SECRET}}"
        },
        "connectionName": {
          "value": "${{CONNECTION_NAME}}"
        }
      }
    }
    ```

1. Enregistrez vos modifications

Mettez ensuite à jour le fichier Bicep.

1. Dans le dossier **infra**, ouvrez le fichier nommé **azure.local.bicep**.
1. Dans le fichier, après la déclaration de paramètre **botAppDomain**, ajoutez les déclarations de paramètre **graphEntraAppClientId**, **graphEntraAppClientSecret** et **connectionName**.

    ```bicep
    param graphEntraAppClientId string
    @secure()
    param graphEntraAppClientSecret string
    param connectionName string
    ```

1. Dans le module **azureBotRegistration**, ajoutez les paramètres **graphEntraAppClientId**, **graphEntraAppClientSecret** et **connectionName**.

    ```bicep
    module azureBotRegistration './botRegistration/azurebot.bicep' = {
      name: 'Azure-Bot-registration'
      params: {
        resourceBaseName: resourceBaseName
        botAadAppClientId: botEntraAppClientId
        botAppDomain: botAppDomain
        botDisplayName: botDisplayName
        graphEntraAppClientId: graphEntraAppClientId
        graphEntraAppClientSecret: graphEntraAppClientSecret
        connectionName: connectionName
      }
    }
    ```

1. Enregistrez vos modifications.

Enfin, mettez à jour le fichier Bicep d’inscription du bot.

1. Dans le dossier **infra/botRegistration**, ouvrez le fichier nommé **azurebot.bicep**.
1. Dans le fichier, après la déclaration de paramètre **botAppDomain**, ajoutez les déclarations de paramètre **graphEntraAppClientId**, **graphEntraAppClientSecret** et **connectionName**.

    ```bicep
    param graphEntraAppClientId string
    @secure()
    param graphEntraAppClientSecret string
    param connectionName string
    ```

1. Après la ressource **botServiceM365ExtensionsChannel**, ajoutez une nouvelle ressource pour la connexion Microsoft Graph.

    ```bicep
    resource botServicesMicrosoftGraphConnection 'Microsoft.BotService/botServices/connections@2022-09-15' = {
      parent: botService
      name: connectionName
      location: 'global'
      properties: {
        serviceProviderDisplayName: 'Azure Active Directory v2'
        serviceProviderId: '30dd229c-58e3-4a48-bdfd-91ec48eb906c'
        clientId: graphEntraAppClientId
        clientSecret: graphEntraAppClientSecret
        scopes: 'email offline_access openid profile Sites.ReadWrite.All'
        parameters: [
          {
            key: 'tenantID'
            value: 'common'
          }
          {
            key: 'tokenExchangeUrl'
            value: 'api://${botAppDomain}/botid-${ botAadAppClientId}'
          }
        ]
      }
    }
    ```

1. Enregistrez vos modifications

## Tâche 4 - Authentifier les requêtes des utilisateurs

Vous ajoutez ensuite du code pour authentifier les utilisateurs lorsqu’ils lancent une recherche à l’aide de l’extension de message.

Continuez dans Visual Studio.

1. Dans le dossier **Search**, ouvrez le fichier nommé **SearchApp.cs**.
1. Dans le fichier, commencez par ajouter l’espace de noms **Authentification** à partir du Kit de développement logiciel (SDK) Bot Framework.

    ```csharp
    using Microsoft.Bot.Connector.Authentication;
    ```

1. Dans la méthode **OnTeamsMessagingExtensionQueryAsync**, ajoutez le code suivant au début de la méthode :

    ```csharp
    var userTokenClient = turnContext.TurnState.Get<UserTokenClient>();
    var tokenResponse = await GetToken(userTokenClient, query.State, turnContext.Activity.From.Id, turnContext.Activity.ChannelId, connectionName, cancellationToken);
    
    if (!HasToken(tokenResponse))
    {
        return await CreateAuthResponse(userTokenClient, connectionName, (Activity)turnContext.Activity, cancellationToken);
    }
    ```

Le bloc de code ci-dessus utilise trois méthodes qui gèrent l’authentification de l’utilisateur.

- **GetToken** utilise le client du service de jetons pour obtenir un jeton d’accès pour l’utilisateur actuel.
- **HasToken** vérifie si la réponse du service de jetons contient un jeton d’accès.
- **CreateAuthResponse** est appelé si aucun jeton n’est retourné et retourne une réponse, qui affiche un lien de connexion dans l’interface utilisateur.

Créez maintenant les méthodes dans la classe **SearchApp**.

- Créez la méthode **GetToken** à l’aide du code suivant :

```csharp
private static async Task<TokenResponse> GetToken(UserTokenClient userTokenClient, string state, string userId, string channelId, string connectionName, CancellationToken cancellationToken)
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
```

Vous vérifiez tout d’abord si le paramètre d’état n’est pas nul ou vide. Le code tente de l’analyser en tant qu’entier à l’aide de la méthode int.TryParse. Si l’analyse réussit, la valeur analysée est affectée à la variable magicCode sous forme de chaîne. Le magicCode est ensuite transmis en tant qu’argument à la méthode GetUserTokenAsync, ainsi que d’autres paramètres. La méthode GetUserTokenAsync utilise le magicCode pour vérifier l’authenticité de la demande. Elle garantit que le jeton d’utilisateur est demandé par la même entité que celle qui a lancé le processus d’authentification.

- Créez la méthode HasToken à l’aide du code suivant :

```csharp
private static bool HasToken(TokenResponse tokenResponse)
{
    return tokenResponse != null && !string.IsNullOrEmpty(tokenResponse.Token);
}
```

Vous vérifiez si un jeton valide est obtenu du service de jetons en vérifiant si la réponse du jeton et la propriété Token sur la réponse ne sont pas vides ou nuls.

- Créez la méthode **CreateAuthResponse** à l’aide du code suivant :

```csharp
private static async Task<MessagingExtensionResponse> CreateAuthResponse(UserTokenClient userTokenClient, string connectionName, Activity activity, CancellationToken cancellationToken)
{
    var resource = await userTokenClient.GetSignInResourceAsync(connectionName, activity, null, cancellationToken);

    return new MessagingExtensionResponse
    {
        ComposeExtension = new MessagingExtensionResult
        {
            Type = "auth",
            SuggestedActions = new MessagingExtensionSuggestedAction
            {
                Actions = new List<CardAction>
                {
                    new() {
                        Type = ActionTypes.OpenUrl,
                        Value = resource.SignInLink,
                        Title = "Sign In",
                    },
                },
            },
        },
    };
}
```

Vous utilisez tout d’abord la méthode **GetSignInResourceAsync** pour récupérer le lien de connexion à partir du service de jetons. Le lien de connexion est utilisé pour construire un objet **MessagingExtensionResponse**. Vous créez un objet et définissez la propriété **ComposeExtension** de la réponse sur un nouvel objet **MessagingExtensionResult**. La propriété type du résultat est définie sur « auth », indiquant que le résultat est une réponse d’authentification. La propriété **SuggestedActions** du résultat est définie sur un nouvel objet **MessagingExtensionSuggestedAction**. La propriété Actions des actions suggérées est définie sur une liste contenant un seul objet **CardAction**. Cet objet **CardAction** représente une action qui peut être effectuée par l’utilisateur. La propriété Type de **CardAction** est définie sur **ActionTypes.OpenUrl**, indiquant qu’il s’agit d’une action qui ouvre une URL. La propriété Value est définie sur le lien de connexion récupéré à partir de la ressource. La propriété Title est définie sur « Sign In », qui spécifie le titre de l’action. Enfin, la réponse construite est retournée à partir de la méthode.

Lorsqu’un utilisateur suit le lien de connexion, il est dirigé vers une ressource hébergée sur un domaine externe. Le domaine doit être inclus dans le fichier manifeste de l’application. Ajoutez le domaine Service de jetons Bot Framework au manifeste de l’application.

Continuez dans Visual Studio.

1. Dans le dossier **appPackage**, ouvrez **manifest.json**.
1. Dans le fichier, mettez à jour le tableau **validDomains** et ajoutez le domaine du service de jetons :

    ```json
    "validDomains": [
        "token.botframework.com",
        "${{BOT_DOMAIN}}"
    ]    
    ```

1. Enregistrez vos modifications

## Tâche 5 : Approvisionner des ressources

Tout étant maintenant en place, exécutez le processus Préparer les dépendances de l’application Teams pour approvisionner les ressources requises.

Continuez dans Visual Studio.

1. Dans l’**Explorateur de solutions**, cliquez avec le bouton droit de la souris sur le projet **TeamsApp**.
1. Développez le menu **Teams Toolkit** et sélectionnez **Préparer les dépendances de l’application Teams**.
1. Dans la boîte de dialogue **Compte Microsoft 365**, sélectionnez **Continuer**.
1. Dans la boîte de dialogue **Approvisionner**, sélectionnez **Approvisionner**.
1. Dans la boîte de dialogue d’**avertissement de Teams Toolkit**, sélectionnez **Approvisionner**.
1. Dans la boîte de dialogue d’**informations de Teams Toolkit**, sélectionnez **Afficher les ressources approvisionnées** pour ouvrir une nouvelle fenêtre de navigateur.

Prenez un moment pour explorer les ressources qui sont créées et mises à jour dans Azure.

## Tâche 6 - Exécuter et déboguer

Démarrez maintenant le service web et testez l’extension de message dans Microsoft Teams.

Continuez dans Visual Studio.

1. Appuyez sur la touche **F5** pour démarrer une session de débogage et ouvrir une nouvelle fenêtre de navigateur qui accède au client web Microsoft Teams.
1. Dans le navigateur, et si nécessaire, entrez vos informations d’identification de compte Microsoft 365 et passez à Microsoft Teams.
1. Dans la boîte de dialogue d’installation de l’application, sélectionnez **Ajouter**.
1. Ouvrez une conversation Microsoft Teams, nouvelle ou existante.
1. Dans la zone de rédaction du message, sélectionnez **...** pour ouvrir le menu volant de l’application.
1. Dans la liste des applications, sélectionnez **Produits Contoso** pour ouvrir l’extension de message.
1. Dans la zone de texte, entrez **Bot Builder** pour démarrer une recherche.
1. Le message **You'll need to sign in to use this app** s’affiche.
1. Sélectionnez le **lien de connexion** pour ouvrir un nouvel onglet et démarrer le flux d’authentification.
1. Dans la page de consentement des autorisations, examinez les autorisations demandées.
1. Sélectionnez **Accepter** pour fermer l’onglet et revenir à Microsoft Teams.
1. Dans la liste des résultats, **sélectionnez un résultat** pour incorporer une carte dans la zone de rédaction du message et l’envoyer.

Fermez le navigateur pour arrêter pour la session de débogage.

[Passez à l’exercice suivant…](./4-exercise-retrieve-product-information-from-sharepoint-online.md)