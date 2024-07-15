---
lab:
  title: "Exercice\_1\_: créer une extension de message"
  module: 'LAB 03: Connect Copilot for Microsoft 365 to your external data in real-time with message extension plugins built with .NET and Visual Studio'
---

# Exercice 1 : créer une extension de message

Dans cet exercice, vous créerez une extension de message avec une commande de recherche. Vous créerez d’abord une structure d’un projet à l’aide d’un modèle de projet Teams Toolkit, puis vous le mettrez à jour pour la configurer en utilisant une ressource Azure AI Bot Service pour le développement local. Vous créerez un tunnel de développement pour activer la communication entre le service du bot et votre service web qui s’exécute localement. Vous préparerez ensuite votre application pour provisionner les ressources requises. Enfin, vous exécutez et déboguez votre extension de message et vous la testez dans Microsoft Teams.

:::image type="content" source="../media/2-search-results-nuget.png" alt-text="Capture d’écran des résultats de la recherche retournés par une extension de message basée sur une recherche dans Microsoft Teams." lightbox="../media/2-search-results-nuget.png":::

## Tâche 1 : créer un projet avec Teams Toolkit pour Visual Studio

Commencez par créer un projet.

1. Ouvrez **Visual Studio 2022**.
1. Ouvrez le menu **Fichier**, développez le menu **Nouveau** et sélectionnez **Nouveau projet**.
1. Dans l’écran Créer un projet, développez la liste déroulante **Toutes les plateformes**, puis sélectionnez **Microsoft Teams**. Sélectionnez **Suivant** pour continuer.
1. Dans l’écran Configurer un nouveau projet. Spécifiez les valeurs suivantes :
    1. **Nom du projet** : MsgExtProductSupport.
    1. **Emplacement** : sélectionnez l’emplacement de votre choix.
    1. **Placer la solution et le projet dans le même répertoire** : fait.
1. Structurez le projet en sélectionnant **Créer**.
1. Dans la boîte de dialogue Créer une application Teams, développez la liste déroulante **Tous les types d’applications**, puis sélectionnez **Extension de message**.
1. Dans la liste des modèles, sélectionnez **Résultats de recherche personnalisés**.
1. Structurez l’application en sélectionnant **Créer**.

## Tâche 2 : configurer Azure AI Bot Service

Une ressource de service bot peut être créée dans Azure en tant que ressource ou via dev.botframework.com. Par défaut, le modèle Résultats de recherche personnalisés inscrit un bot à l’aide de dev.botframework.com. Pour le moment, l’inscription du bot avec dev.botframework.com n’est pas compatible avec Copilot pour Microsoft 365.

Pour utiliser Copilot pour Microsoft 365, mettez à jour le projet pour approvisionner une ressource Azure AI Bot Service dans Azure et l’utiliser pour le développement local.

Tout d’abord, nous allons créer une variable d’environnement pour centraliser un nom interne pour l’application que nous pouvons réutiliser dans nos fichiers et utiliser lors de l’approvisionnement de ressources.

Dans Visual Studio :

1. Dans le dossier **env**, ouvrez **.env.local**.
1. Dans le fichier , ajoutez le code suivant :

    ```text
    APP_INTERNAL_NAME=msgext-product-support
    ```

1. Enregistrez vos modifications

Utilisez des expressions de liaison de données, par exemple `${{APP_INTERNAL_NAME}}`, qui vous permettent d’injecter des valeurs de variables d’environnement dans des fichiers lors de l’utilisation de Teams Toolkit pour approvisionner des ressources.

Pour approvisionner une ressource Azure AI Bot Service, une inscription d’application Microsoft Entra est requise. Créez un fichier manifeste d’inscription d’application que Teams Toolkit utilise pour approvisionner l’inscription de l’application.

Continuez dans Visual Studio.

1. Dans le dossier **infra**, créez un dossier appelé **entra**.
1. Dans le dossier, créez un fichier nommé **entra.bot.manifest.json**.
1. Dans le fichier , ajoutez le code suivant :

    ```json
    {
      "id": "${{BOT_ENTRA_APP_OBJECT_ID}}",
      "appId": "${{BOT_ID}}",
      "name": "${{APP_INTERNAL_NAME}}-bot-${{TEAMSFX_ENV}}",
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
      "requiredResourceAccess": [],
      "oauth2Permissions": [],
      "preAuthorizedApplications": [],
      "identifierUris": [],
      "replyUrlsWithType": []
    }
    ```

1. Enregistrez vos modifications.

Teams Toolkit utilise des fichiers Bicep pour approvisionner et configurer des ressources dans Azure. Créez premièrement un fichier de paramètres. Le fichier de paramètres sert à passer des variables d’environnement dans un modèle Bicep.

Continuez dans Visual Studio.

1. Dans le dossier **infra**, créez un fichier nommé **azure.parameters.local.json**.
1. Dans le fichier , ajoutez le code suivant :

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
        }
      }
    }
    ```

1. Enregistrez vos modifications.

À présent, créez un fichier Bicep utilisé avec le fichier de paramètres.

1. Dans le dossier **infra**, créez un fichier appelé **azure.local.bicep**.
1. Dans le fichier , ajoutez le code suivant :

    ```bicep
    @maxLength(20)
    @minLength(4)
    @description('Used to generate names for all resources in this file')
    param resourceBaseName string
    
    @description('Required when create Azure Bot service')
    param botEntraAppClientId string
    @maxLength(42)
    param botDisplayName string
    param botAppDomain string
    
    module azureBotRegistration './botRegistration/azurebot.bicep' = {
      name: 'Azure-Bot-registration'
      params: {
        resourceBaseName: resourceBaseName
        botAadAppClientId: botEntraAppClientId
        botAppDomain: botAppDomain
        botDisplayName: botDisplayName
      }
    }
    ```

1. Enregistrez vos modifications.

La dernière étape consiste à mettre à jour le fichier projet de Teams Toolkit. Remplacez les étapes qui utilisent les actions Bot Framework pour provisionner l’inscription de l’application Bot Microsoft Entra à l’aide du fichier manifeste et de la ressource Azure AI Bot Service avec le fichier Bicep.

Continuez dans Visual Studio.

1. Dans le dossier racine du projet, ouvrez **teamsapp.local.yml**.
1. Dans le fichier, recherchez l’étape qui utilise l’action **botAadApp/create** et remplacez-la par :

    ```yml
      - uses: aadApp/create
        with:
          name: ${{APP_INTERNAL_NAME}}-bot-${{TEAMSFX_ENV}}
          generateClientSecret: true
          signInAudience: AzureADMultipleOrgs
        writeToEnvironmentFile:
          clientId: BOT_ID
          clientSecret: SECRET_BOT_PASSWORD
          objectId: BOT_ENTRA_APP_OBJECT_ID
          tenantId: BOT_ENTRA_APP_TENANT_ID
          authority: BOT_ENTRA_APP_OAUTH_AUTHORITY
          authorityHost: BOT_ENTRA_APP_OAUTH_AUTHORITY_HOST
    
      - uses: aadApp/update
        with:
          manifestPath: "./infra/entra/entra.bot.manifest.json"
          outputFilePath : "./build/entra.bot.manifest.${{TEAMSFX_ENV}}.json"
    
      - uses: arm/deploy
        with:
          subscriptionId: ${{AZURE_SUBSCRIPTION_ID}}
          resourceGroupName: ${{AZURE_RESOURCE_GROUP_NAME}}
          templates:
            - path: ./infra/azure.local.bicep
              parameters: ./infra/azure.parameters.local.json
              deploymentName: Create-resources-for-${{APP_INTERNAL_NAME}}-${{TEAMSFX_ENV}}
          bicepCliVersion: v0.9.1
    ```

1. Dans le fichier, supprimez l’étape qui utilise l’action **botFramework/create**.
1. Enregistrez vos modifications.

L’inscription d’application est configurée en deux étapes, tout d’abord, l’action **aadApp/create** crée une inscription d’application multi-locataire avec une clé secrète client, en écrivant ses sorties dans le fichier **.env.local** en tant que variables d’environnement. Ensuite, l’action **aadApp/update** utilise le fichier **entra.bot.manifest.json** pour mettre à jour l’inscription de l’application.

La dernière étape utilise l’action **arm/deploy** pour approvisionner la ressource Azure AI Bot Service au groupe de ressources à l’aide du fichier **azure.parameters.local.json** et du fichier **azure.local.bicep**.

## Tâche 3 : créer un tunnel de développement

Lorsque l’utilisateur interagit avec votre extension de message, le service bot envoie des requêtes au service web. Pendant le développement, votre service web s’exécute localement sur votre ordinateur. Pour permettre au service bot d’atteindre votre service web, vous devez l’exposer au-delà de votre ordinateur à l’aide d’un tunnel de développement.

:::image type="content" source="../media/18-select-dev-tunnel.png" alt-text="Capture d’écran du menu Tunnels dev développé dans Visual Studio." lightbox="../media/18-select-dev-tunnel.png":::

Continuez dans Visual Studio.

1. Dans la barre d’outils, développez le menu du profil de débogage **en sélectionnant la liste déroulante en regard du bouton Microsoft Teams (navigateur)**.
1. Développez le menu **Tunnels dev (aucun tunnel actif)** et sélectionnez **Créer un tunnel…**.
1. Dans la boîte de dialogue, spécifiez les valeurs suivantes :
    1. **Compte** : sélectionnez le compte de votre choix.
    1. **Nom** : MsgExtProductSupport.
    1. **Type de tunnel** : temporaire.
    1. **Accès** : public.
1. Créez le tunnel en sélectionnant **OK**. Une invite s’affiche indiquant que le nouveau tunnel est maintenant le tunnel actif.
1. Fermez l’invite en sélectionnant **OK**.

## Tâche 4 : mettre à jour le manifeste d’application

Le manifeste de l’application décrit les fonctionnalités et caractéristiques de l’application. Mettez à jour les propriétés dans le manifeste de l’application pour mieux décrire les caractéristiques de l’application et ses fonctionnalités.

Tout d’abord, téléchargez les icônes de l’application et ajoutez-les au projet.

:::image type="content" source="../media/app/color-local.png" alt-text="Icône de couleur utilisée pour le développement local." lightbox="../media/app/color-local.png":::

:::image type="content" source="../media/app/color-dev.png" alt-text="Icône de couleur utilisée pour le développement à distance." lightbox="../media/app/color-dev.png":::

1. Téléchargez **color-local.png** et **color-dev.png**.
1. Dans le dossier **appPackage**, ajoutez **color-local.png** et **color-dev.png**.
1. Dans le dossier, supprimez le fichier nommé **color.png**.

À mesure que le nom de l’application est répliqué à différents endroits dans le projet, créez une variable d’environnement pour stocker cette valeur de manière centralisée.

Continuez dans Visual Studio.

1. Dans le dossier **env**, ouvrez le fichier nommé **.env.local**.
1. Dans le fichier , ajoutez le code suivant :

    ```text
    APP_DISPLAY_NAME=Contoso products
    ```

1. Enregistrez vos modifications

Enfin, mettez à jour les objets des icônes, du nom et de la description dans le fichier manifeste de l’application.

1. Dans le dossier **appPackage**, ouvrez le fichier nommé **manifest.json**.
1. Dans le fichier, mettez à jour les objets **icônes**, **nom et **description** avec :

    ```json
    {
        "icons": {
            "color": "color-${{TEAMSFX_ENV}}.png",
            "outline": "outline.png"
        },
        "name": {
            "short": "${{APP_DISPLAY_NAME}}",
            "full": "${{APP_DISPLAY_NAME}}"
        },
        "description": {
            "short": "Product look up tool.",
            "full": "Get real-time product information and share them in a conversation."
        }
    }
    ```

1. Enregistrez vos modifications

## Tâche 6 : approvisionner les ressources

Tout étant maintenant en place, utilisez Teams Toolkit et exécutez le processus Préparer les dépendances de l’application Teams pour approvisionner les ressources requises.

:::image type="content" source="../media/19-prepare-teams-app-dependencies.png" alt-text="Capture d’écran du menu Teams Toolkit développé dans Visual Studio." lightbox="../media/19-prepare-teams-app-dependencies.png":::

Le processus Préparer les dépendances de l’application Teams met à jour les variables d’environnement **BOT_ENDPOINT** et **BOT_DOMAIN** dans le fichier .env.local à l’aide de l’URL du tunnel de développement actif et exécute les actions décrites dans le fichier **teamsapp.local.yml**.

Continuez dans Visual Studio.

1. Dans l’Explorateur de solutions, cliquez avec le bouton droit sur le projet **MsgExtProductSupport**.
1. Développez le menu **Teams Toolkit** et sélectionnez **Préparer les dépendances de l’application Teams**.
1. Dans la boîte de dialogue **Compte Microsoft 365**, sélectionnez le compte de votre locataire développeur, puis sélectionnez **Continuer**
1. Dans la boîte de dialogue **Approvisionner**, sélectionnez le compte à utiliser pour déployer des ressources sur Azure et spécifiez les valeurs suivantes :
    1. **Nom de l’abonnement** : sélectionnez l’abonnement à utiliser dans la liste déroulante.
    1. **Groupe de ressources** : sélectionnez Nouveau… pour ouvrir une boîte de dialogue, saisissez **rg-msgext-product-support-local, puis sélectionnez **OK**.
    1. **Région** : sélectionnez la région la plus proche de vous dans la liste déroulante.
1. Approvisionnez les ressources dans Azure en sélectionnant **Approvisionner**.
1. Dans l’invite d’avertissement de Teams Toolkit, sélectionnez **Approvisionner**.
1. Dans l’invite d’informations de Teams Toolkit, sélectionnez **Afficher les ressources approvisionnées** pour ouvrir une nouvelle fenêtre de navigateur.

Prenez un moment pour explorer les ressources créées dans Azure.

## Tâche 7 : exécuter et déboguer  

Démarrez maintenant le service web et testez l’extension de message. Utilisez Teams Toolkit pour charger votre manifeste d’application et tester votre extension de message dans Microsoft Teams.

Continuez dans Visual Studio.

1. Appuyez sur la touche F5 pour démarrer une session de débogage et ouvrir une nouvelle fenêtre de navigateur qui accède au client web Microsoft Teams.
1. À l’invite, saisissez vos informations d’identification de compte Microsoft 365.

  > [!IMPORTANT]
  > Si une boîte de dialogue s’affiche dans Microsoft Teams avec le message « Cette application est introuvable », suivez les étapes ci-dessous pour charger manuellement le package de l’application :
  >
  >  1. Fermez la boîte de dialogue.
  >  2. Sur la barre latérale, accédez à **Applications**.
  >  3. Dans le menu de gauche, sélectionnez **Gérer vos applications**.
  >  4. Dans la barre de commandes, sélectionnez **Charger une application**.
  >  5. Dans la boîte de dialogue, sélectionnez **Charger une application personnalisée**.
  >  6. Dans l’Explorateur de fichiers, accédez au dossier de solution, ouvrez le dossier **appPackage\build**, puis sélectionnez **appPackage.local.zip**, et enfin **Ajouter**.

Continuez pour installer l’application :

1. Dans la boîte de dialogue d’installation de l’application, sélectionnez **Ajouter**.
1. Ouvrez une conversation Microsoft Teams, nouvelle ou existante.
1. Dans la zone de rédaction du message, sélectionnez **...** pour ouvrir le menu volant de l’application.
1. Dans la liste des applications, sélectionnez **Produits Contoso** pour ouvrir l’extension de message.
1. Dans la zone de texte, entrez **Bot Builder** pour démarrer une recherche.
1. Dans la liste des résultats, sélectionnez un résultat pour incorporer une carte dans la zone de rédaction du message.

Fermez le navigateur pour arrêter pour la session de débogage.

[Passez à l’exercice suivant…](./3-exercise-add-single-sign-on.md)