---
lab:
  title: "Exercice3\_: intégrer un plug-in d’API à une API sécurisée avec OAuth"
  module: 'LAB 05: Authenticate your API plugin for declarative agents with secured APIs'
---

# Exercice3 : intégrer un plug-in d’API à une API sécurisée avec OAuth

Les plug-ins d’API pour Microsoft 365 Copilot vous permettent d’intégrer des API sécurisées avec OAuth. Vous conservez l’ID client et le secret de l’application qui protège votre API en les inscrivant dans le coffre Teams. Au moment de l’exécution, Microsoft 365 Copilot exécute votre plug-in, récupère les informations du coffre et les utilise pour obtenir un jeton d’accès et appeler l’API. En suivant ce processus, l’ID client et le secret restent sécurisés et ne sont jamais exposés au client.

### Durée de l’exercice

- **Durée estimée :** 10 minutes

## Tâche 1 : ouvrir l’exemple de projet et examiner la configuration de l’authentification

Commencez par télécharger l’exemple de projet.

1. Dans un navigateur web, accédez à [https://aka.ms/learn-da-api-ts-repairs](https://aka.ms/learn-da-api-ts-repairs). Vous obtenez une invite pour télécharger un fichier ZIP avec l’exemple de projet.
1. Enregistrez le fichier zip sur votre ordinateur.
1. Extrayez le contenu du fichier zip.
1. Ouvrez le dossier  dans Visual Studio Code.

L’exemple de projet est un projet Teams Toolkit qui inclut un agent déclaratif, un plug-in d’API et une API sécurisée avec Microsoft Entra ID. L’API s’exécute sur Azure Functions et implémente la sécurité à l’aide des fonctionnalités d’authentification et d’autorisation intégrées d’Azure Functions, parfois appelées Authentification simple.

## Tâche 2 : examiner la configuration d’autorisation OAuth2

Avant de continuer, examinez la configuration d’autorisation OAuth2 dans l’exemple de projet.

### Examiner la définition de l’API

Tout d’abord, examinez la configuration de sécurité de la définition d’API incluse dans le projet.

Dans Visual Studio Code :

1. Ouvrez le fichier **appPackage/apiSpecificationFile/repair.yml**.
1. Dans la section **components.securitySchemes**, notez la propriété **oAuth2AuthCode** :

  ```yml
  components:
    securitySchemes:
    oAuth2AuthCode:
      type: oauth2
      description: OAuth configuration for the repair service
      flows:
      authorizationCode:
        authorizationUrl: https://login.microsoftonline.com/${{AAD_APP_TENANT_ID}}/oauth2/v2.0/authorize
        tokenUrl: https://login.microsoftonline.com/${{AAD_APP_TENANT_ID}}/oauth2/v2.0/token
        scopes:
        api://${{AAD_APP_CLIENT_ID}}/repairs_read: Read repair records 
  ```

  La propriété définit un schéma de sécurité OAuth2 et inclut des informations sur les URL à appeler pour obtenir un jeton d’accès et les étendues que l’API utilise.

  > **IMPORTANT** Notez que l’étendue est entièrement qualifiée avec l’URI de l’ID d’application (**api://...**). Lorsque vous travaillez avec Microsoft Entra, vous devez qualifier entièrement les étendues personnalisées. Lorsque Microsoft Entra voit une étendue non qualifiée, il suppose qu’elle appartient à Microsoft Graph, ce qui entraîne des erreurs de flux d’autorisation.

1. Recherchez la propriété **paths./repairs.get.security**. Notez qu’il fait référence au schéma de sécurité **oAuth2AuthCode** et à l’étendue dont le client a besoin pour effectuer l’opération.

  ```yml
  [...]
  paths:
    /repairs:
    get:
      operationId: listRepairs
      [...]
      security:
      - oAuth2AuthCode:
        - api://${{AAD_APP_CLIENT_ID}}/repairs_read
  [...]
  ```

  > **IMPORTANT** La liste des étendues nécessaires dans la spécification de l’API est purement informative. Lors de l’implémentation de l’API, il vous incombe de valider le jeton et de vérifier qu’il contient les étendues nécessaires.

### Examiner l’implémentation de l’API

Ensuite, examinez l’implémentation de l’API.

Dans Visual Studio Code :

1. Ouvrez le fichier **src/functions/repairs.ts**.
1. Dans la fonction de gestionnaire de **repairs**, recherchez la ligne suivante qui vérifie si la demande contient un jeton d’accès avec les étendues nécessaires :

  ```typescript
  if (!hasRequiredScopes(req, 'repairs_read')) {
    return {
    status: 403,
    body: "Insufficient permissions",
    };
  }
  ```

1. La fonction **hasRequiredScopes** est implémentée plus loin dans le fichier **repairs.ts** :

  ```typescript
  function hasRequiredScopes(req: HttpRequest, requiredScopes: string[] | string): boolean {
    if (typeof requiredScopes === 'string') {
    requiredScopes = [requiredScopes];
    }
  
    const token = req.headers.get("Authorization")?.split(" ");
    if (!token || token[0] !== "Bearer") {
    return false;
    }
  
    try {
    const decodedToken = jwtDecode<JwtPayload & { scp?: string }>(token[1]);
    const scopes = decodedToken.scp?.split(" ") ?? [];
    return requiredScopes.every(scope => scopes.includes(scope));
    }
    catch (error) {
    return false;
    }
  }
  ```

  La fonction commence par extraire le jeton du porteur à partir de l’en-tête de demande d’autorisation. Ensuite, elle utilise le package **jwt-decode** pour décoder le jeton et obtenir la liste des étendues de la revendication **SCP**. Enfin, elle vérifie si la revendication **SCP** contient toutes les étendues requises.

  Notez que la fonction ne valide pas le jeton d’accès. Au lieu de cela, elle vérifie uniquement si le jeton d’accès contient les étendues requises. Dans ce modèle, l’API s’exécute sur Azure Functions et implémente la sécurité à l’aide de l’Authentification simple, qui est chargée de valider le jeton d’accès. Si la demande ne contient pas de jeton d’accès valide, le runtime d’Azure Functions la rejette avant d’atteindre votre code. Bien que l’authentification simple valide le jeton, elle ne vérifie pas les étendues nécessaires, ce que vous devez faire vous-même.

### Examiner la configuration de la tâche du coffre

Dans ce projet, vous utilisez Teams Toolkit pour ajouter les informations OAuth au coffre. Teams Toolkit inscrit les informations OAuth dans le coffre à l’aide d’une tâche spéciale dans la configuration du projet.

Dans Visual Studio Code :

1. Ouvrez le fichier **./teampsapp.local.yml**.
1. Dans la section **Configurer**, recherchez la **tâche oauth/register**.

  ```yml
  - uses: oauth/register
    with:
    name: oAuth2AuthCode
    flow: authorizationCode
    appId: ${{TEAMS_APP_ID}}
    clientId: ${{AAD_APP_CLIENT_ID}}
    clientSecret: ${{SECRET_AAD_APP_CLIENT_SECRET}}
    isPKCEEnabled: true
    # Path to OpenAPI description document
    apiSpecPath: ./appPackage/apiSpecificationFile/repair.yml
    writeToEnvironmentFile:
    configurationId: OAUTH2AUTHCODE_CONFIGURATION_ID
  ```

  La tâche prend les valeurs des variables de projet **TEAMS_APP_ID**, **des AAD_APP_CLIENT_ID** et **SECRET_AAD_APP_CLIENT_SECRET**, stockées dans les fichiers **env/.env.local** et **env/.env.local.user** et les inscrit dans le coffre. Elle active également la clé de preuve pour l’échange de code (PKCE) comme mesure de sécurité supplémentaire. Ensuite, elle prend l’ID d’entrée du coffre et l’écrit dans le fichier d’environnement **env/.env.local**. Le résultat de cette tâche est une variable d’environnement nommée **OAUTH2AUTHCODE_CONFIGURATION_ID**. Teams Toolkit écrit la valeur de cette variable dans le fichier **appPackages/ai-plugin.json** qui contient la définition du plug-in. Au moment du runtime, l’agent déclaratif qui charge le plug-in d’API utilise cet ID pour récupérer les informations OAuth à partir du coffre, et démarrer un flux d’authentification afin d’obtenir un jeton d’accès.

  > **IMPORTANT** La tâche **oauth/register** est uniquement responsable de l’inscription des informations OAuth dans le coffre si elles n’existent pas encore. Si les informations existent déjà, Teams Toolkit ignore l’exécution de cette tâche.

1. Ensuite, recherchez la tâche **oauth/update**.

  ```yml
  - uses: oauth/update
    with:
    name: oAuth2AuthCode
    appId: ${{TEAMS_APP_ID}}
    apiSpecPath: ./appPackage/apiSpecificationFile/repair.yml
    configurationId: ${{OAUTH2AUTHCODE_CONFIGURATION_ID}}
    isPKCEEnabled: true
  ```

  La tâche conserve les informations OAuth dans le coffre synchronisées avec votre projet. Il est nécessaire que votre projet fonctionne correctement. L’une des propriétés de clé est l’URL sur laquelle votre plug-in d’API est disponible. Chaque fois que vous démarrez votre projet, le kit de ressources Teams ouvre un tunnel de développement sur une nouvelle URL. Les informations OAuth dans le coffre doivent référencer cette URL pour que Copilot puisse atteindre votre API.

### Examiner la configuration de l’authentification et de l’autorisation

La partie suivante à explorer est les paramètres d’authentification et d’autorisation d’Azure Functions. L’API de cet exercice utilise les fonctionnalités d’authentification et d’autorisation intégrées d’Azure Functions. Teams Toolkit configure ces fonctionnalités lors de l’approvisionnement d’Azure Functions sur Azure.

Dans Visual Studio Code :

1. Ouvrez le fichier **infra/azure.bicep**.
1. Recherchez la ressource **authSettings** :

  ```bicep
  resource authSettings 'Microsoft.Web/sites/config@2021-02-01' = {
    parent: functionApp
    name: 'authsettingsV2'
    properties: {
    globalValidation: {
      requireAuthentication: true
      unauthenticatedClientAction: 'Return401'
    }
    identityProviders: {
      azureActiveDirectory: {
      enabled: true
      registration: {
        openIdIssuer: oauthAuthority
        clientId: aadAppClientId
      }
      validation: {
        allowedAudiences: [
        aadAppClientId
        aadApplicationIdUri
        ]
      }
      }
    }
    }
  }
  ```

  Cette ressource permet d’activer les fonctionnalités d’authentification et d’autorisation intégrées sur l’application Azure Functions. Tout d’abord, dans la section **globalValidation**, elle définit que l’application autorise uniquement les demandes authentifiées. Si l’application reçoit une demande non authentifiée, elle la rejette avec une erreur HTTP 401. Ensuite, dans la section **identityProviders**, la configuration définit qu’elle utilise Microsoft Entra ID (précédemment appelé Azure Active Directory) pour autoriser les demandes. Elle spécifie l’inscription de l’application Microsoft Entra qu’elle utilise pour sécuriser l’API et indique les audiences autorisées à appeler l’API.

### Examiner l’inscription d’application Microsoft Entra

La dernière partie à examiner est l’inscription de l’application Microsoft Entra que le projet utilise pour sécuriser l’API. Lorsque vous utilisez OAuth, vous sécurisez l’accès aux ressources à l’aide d’une application. L’application définit généralement les informations d’identification nécessaires pour obtenir un jeton d’accès, tel qu’une clé secrète client ou un certificat. Elle spécifie également les différentes autorisations (également appelées étendues) que le client peut demander lorsqu’il appelle l’API. L’inscription d’application Microsoft Entra représente une application dans Microsoft Cloud et définit une application à utiliser avec des flux d’autorisation OAuth.

Dans Visual Studio Code :

1. Ouvrez le fichier **./aad.manifest.json**.
1. Recherchez la propriété **oauth2Permissions**.

  ```json
  "oauth2Permissions": [
    {
    "adminConsentDescription": "Allows Copilot to read repair records on your behalf.",
    "adminConsentDisplayName": "Read repairs",
    "id": "${{AAD_APP_ACCESS_AS_USER_PERMISSION_ID}}",
    "isEnabled": true,
    "type": "User",
    "userConsentDescription": "Allows Copilot to read repair records.",
    "userConsentDisplayName": "Read repairs",
    "value": "repairs_read"
    }
  ],
  ```

  La propriété définit une étendue personnalisée nommée **repairs_read** qui autorise le client à lire les réparations à partir de l’API de réparations.

1. Recherchez la propriété **identifierUris**.

  ```json
  "identifierUris": [
    "api://${{AAD_APP_CLIENT_ID}}"
  ]
  ```

  La propriété **identifierUris** définit un identifiant utilisé pour qualifier entièrement l’étendue.
