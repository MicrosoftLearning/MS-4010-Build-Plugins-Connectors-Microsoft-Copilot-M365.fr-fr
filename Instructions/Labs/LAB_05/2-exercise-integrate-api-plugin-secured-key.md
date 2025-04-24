---
lab:
  title: "Exercice\_1\_: intégrer un plug-in d’API à une API sécurisée avec une clé"
  module: 'LAB 05: Authenticate your API plugin for declarative agents with secured APIs'
---

# Exercice 1 : intégrer un plug-in d’API à une API sécurisée avec une clé

Les plug-ins d’API pour Microsoft 365 Copilot vous permettent d’intégrer des API sécurisées avec une clé. Vous sécurisez la clé API en l’inscrivant dans le coffre Teams. Au moment du runtime, Microsoft 365 Copilot exécute votre plug-in, récupère la clé API du coffre et l’utilise pour appeler l’API. En suivant ce processus, la clé API reste sécurisée et n’est jamais exposée au client.

Dans cet exercice, vous créez un agent déclaratif avec un plug-in d’API qui s’authentifie avec une clé API que vous générez.

### Durée de l’exercice

- **Durée estimée :** 10 minutes

## Tâche 1 : créer un projet

Commencez par créer un plug-in d’API pour Microsoft 365 Copilot. Ouvrez Visual Studio Code.

Dans Visual Studio Code :

1. Dans la **barre d’activité** (barre latérale), ouvrez l’extension Teams Toolkit.
1. Dans le panneau d’extension **Kit de ressources Teams**, choisissez **Créer une application**.
1. Dans la liste des modèles de projet, choisissez **Assistant Copilot**.
1. Dans la liste des fonctionnalités d’application, choisissez **Agent déclaratif**.
1. Choisissez l’option **Ajouter un plug-in**.
1. Choisissez l’option **Démarrer avec une nouvelle option d’API**.
1. Dans la liste des types d’authentification, choisissez **Clé API (Authentification du jeton du porteur)**.
1. Pour le langage de programmation, sélectionnez **TypeScript**.
1. Choisissez un dossier pour stocker votre projet.
1. Nommez votre projet **da-repairs-key**.

Teams Toolkit crée un projet qui inclut un agent déclaratif, un plug-in d’API et une API sécurisée avec une clé.

## Tâche 2 : examiner la configuration de l’authentification par clé API

Avant de continuer, examinez la configuration de l’authentification par clé API dans le projet généré.

### Examiner la définition de l’API

Tout d’abord, examinez comment l’authentification par clé API est définie dans la définition de l’API.

Dans Visual Studio Code :

1. Ouvrez le fichier **appPackage/apiSpecificationFile/repair.yml**. Ce fichier contient la définition OpenAPI de l’API.
1. Dans la section **components.securitySchemes**, notez la propriété **apiKey** :

  ```yml
  components:
    securitySchemes:
    apiKey:
      type: http
      scheme: bearer
  ```

  La propriété définit un schéma de sécurité qui utilise la clé API comme jeton du porteur dans l’en-tête de demande d’autorisation.

1. Recherchez la propriété **paths./repairs.get.security**. Notez qu’elle fait référence au schéma de sécurité **apiKey**.

  ```yml
  [...]
  paths:
    /repairs:
    get:
      operationId: listRepairs
      [...]
      security:
      - apiKey: []
  [...] 
  ```

### Examiner l’implémentation de l’API

Découvrez ensuite comment l’API valide la clé API sur chaque requête.

Dans Visual Studio Code :

1. Ouvrez le fichier **src/functions/repairs.ts**.
1. Dans la fonction de gestionnaire **repairs**, recherchez la ligne suivante qui rejette toutes les requêtes non autorisées :

  ```typescript
  if (!isApiKeyValid(req)) {
    // Return 401 Unauthorized response.
    return {
    status: 401,
    };
  } 
  ```

1. La fonction **isApiKeyValid** est implémentée plus loin dans le fichier repairs.ts :

  ```typescript
  function isApiKeyValid(req: HttpRequest): boolean {
    const apiKey = req.headers.get("Authorization")?.replace("Bearer ", "").trim();
    return apiKey === process.env.API_KEY;
  }
  ```

  La fonction vérifie si l’en-tête d’autorisation contient un jeton du porteur et le compare à la clé API définie dans la variable d’environnement **API_KEY**.

Ce code montre une implémentation simpliste de la sécurité des clés API, mais il illustre le fonctionnement pratique de la sécurité des clés API.

### Examiner la configuration de la tâche du coffre

Dans ce projet, vous utilisez Teams Toolkit pour ajouter la clé API au coffre. Teams Toolkit inscrit la clé API dans le coffre à l’aide d’une tâche spéciale dans la configuration du projet.

Dans Visual Studio Code :

1. Ouvrez le fichier **./teampsapp.local.yml**.
1. Dans la section **Configurer**, recherchez la tâche **apiKey/register**.

  ```yml
  # Register API KEY
  - uses: apiKey/register
    with:
    # Name of the API Key
    name: apiKey
    # Value of the API Key
    primaryClientSecret: ${{SECRET_API_KEY}}
    # Teams app ID
    appId: ${{TEAMS_APP_ID}}
    # Path to OpenAPI description document
    apiSpecPath: ./appPackage/apiSpecificationFile/repair.yml
    # Write the registration information of API Key into environment file for
    # the specified environment variable(s).
    writeToEnvironmentFile:
    registrationId: APIKEY_REGISTRATION_ID
  ```

  La tâche prend la valeur de la variable de projet **SECRET_API_KEY**, stockée dans le fichier **env/.env.local.user** et l’inscrit dans le coffre. Ensuite, elle prend l’ID d’entrée du coffre et l’écrit dans le fichier d’environnement **env/.env.local**. Le résultat de cette tâche est une variable d’environnement nommée **APIKEY_REGISTRATION_ID**. Teams Toolkit écrit la valeur de cette variable dans le fichier **appPackages/ai-plugin.json** qui contient la définition du plug-in. Au moment de l’exécution, l’agent déclaratif qui charge le plug-in d’API utilise cet ID pour récupérer la clé API à partir du coffre et appeler l’API en toute sécurité.

## Tâche 3 : configurer la clé API pour le développement local

Avant de pouvoir tester le projet, vous devez définir une clé API pour votre API. Ensuite, stockez la clé API dans le coffre et inscrivez l’ID d’entrée du coffre dans votre plug-in d’API. Pour le développement local, stockez la clé API dans votre projet et utilisez le kit de ressources Teams pour l’inscrire dans le coffre.

Dans Visual Studio Code :

1. Ouvrez le volet **Terminal** (Ctrl + `).
1. Dans une ligne de commande :
  1. Restaurez les dépendances du projet en exécutant `npm install`.
  1. Générez une nouvelle clé API en exécutant : `npm run keygen`.
  1. Copiez la clé générée dans le presse-papiers.
1. Ouvrez le fichier **env/.env.local.user**.
1. Mettez à jour la propriété **SECRET_API_KEY** vers la clé API nouvellement générée. La propriété mise à jour se présente comme suit :

  ```text
  SECRET_API_KEY='your_key'
  ```

1. Enregistrez les changements apportés.

Chaque fois que vous générez le projet, le kit de ressources Teams met automatiquement à jour la clé API dans le coffre et met à jour votre projet avec l’ID d’entrée du coffre.