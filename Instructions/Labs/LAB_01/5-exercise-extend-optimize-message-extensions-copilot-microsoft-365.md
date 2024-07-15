---
lab:
  title: "Exercice\_4\_: étendre et optimiser les extensions de message à utiliser avec Copilot pour Microsoft\_365"
  module: 'LAB 03: Connect Copilot for Microsoft 365 to your external data in real-time with message extension plugins built with .NET and Visual Studio'
---

# Exercice 4 : étendre et optimiser les extensions de message à utiliser avec Copilot pour Microsoft 365

Dans cet exercice, vous étendrez et optimiserez votre extension de message à utiliser avec Copilot pour Microsoft 365. Vous ajouterez un nouveau paramètre appelé Public cible et mettrez à jour la logique d’extension de message pour gérer plusieurs paramètres. Enfin, vous exécuterez et déboguerez votre extension de message et vous la testerez dans Copilot intégré à Microsoft Teams.

## Tâche 1 : mettre à jour le manifeste d’application

Il est essentiel de spécifier des descriptions concises et précises dans le manifeste de votre application pour que Copilot sache quand et comment appeler votre plug-in. Mettez à jour les descriptions de l’application, des commandes et des paramètres dans le manifeste de l’application.

Ouvrez Visual Studio :

1. Dans le dossier **appPackage**, ouvrez le fichier nommé **manifest.json**.
1. Mettez à jour la **description** de l’objet.

    ```json
    {
        "description": {
            "short": "Product look up tool.",
            "full": "Get real-time product information and share them in a conversation. Search by product name or target audience. ${{APP_DISPLAY_NAME}} works with Microsoft 365 Chat. Find products at Contoso. Find Contoso products called mark8. Find Contoso products named mark8. Find Contoso products related to Mark8. Find Contoso products aimed at individuals. Find Contoso products aimed at businesses. Find Contoso products aimed at individuals with the name mark8. Find Contoso products aimed at businesses with the name mark8."
        },
    }
    ```

    Comme nous allons ajouter un autre paramètre à la commande, mettez également à jour la description de la commande pour inclure le nouveau paramètre.

1. Dans le tableau des **commandes**, mettez à jour la **description** de la commande.

    ```json
    {
        "commands": [
            {
                "id": "Search",
                "type": "query",
                "title": "Products",
                "description": "Find products by name or by target audience",
                "initialRun": true,
                "fetchTask": false,
                "context": [...],
                "parameters": [...]
            }
        ]
    }
    ```

    Ajoutez maintenant un nouveau paramètre que Copilot peut utiliser. Ce nouveau paramètre permet aux utilisateurs de rechercher des produits à l’aide de Copilot destinés à différents publics, tels que des particulier et des entreprises.

1. Dans le tableau des **paramètres**, ajoutez le paramètre **TargetAudience** après le paramètre **ProductName**.

    ```json
    {    
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
    }
    ```

1. Enregistrez vos modifications

La description du paramètre **TargetAudience** explique ce qu’il est et informe que le paramètre doit accepter les valeurs **Consumer** ou **Enterprise** comme valeurs autorisées.

## Tâche 2 : mettre à jour la logique d’extension de message

Pour prendre en charge le nouveau paramètre et les invites complexes, mettez à jour la méthode OnTeamsMessagingExtensionQueryAsync dans le gestionnaire d’activité du bot pour gérer plusieurs paramètres.

Supposons qu’un utilisateur saisisse « Rechercher des produits Contoso dont le nom contient Mark8 destinés aux particuliers ». La description des paramètres, « destinés aux particuliers » se traduit par **Consumer** qui est transmis en tant que valeur au paramètre **TargetAudience**. « Mark8 » est transmis comme valeur du paramètre **ProductName**.

Continuez dans Visual Studio.

1. Dans le dossier **Search**, ouvrez le fichier nommé **SearchApp.cs**.
1. Dans la méthode **OnTeamsMessagingExtensionQueryAsync**, recherchez le bloc de code ci-dessous :

    ```csharp
    var name = GetQueryData(query.Parameters, "ProductName");
    var nameFilter = !string.IsNullOrEmpty(name) ? $"startswith(fields/Title, '{name}')" : string.Empty;
    var filters = new List<string> { nameFilter };
    var filterQuery = filters.Count == 1 ? filters.FirstOrDefault() : string.Join(" and ", filters); 
    ```

1. Mettez à jour le bloc de code pour obtenir la valeur du paramètre **TargetAudience** et créer une requête de filtre à utiliser lors de l’interrogation de la liste SharePoint Online.

    ```csharp
    var name = GetQueryData(query.Parameters, "ProductName");
    var retailCategory = GetQueryData(query.Parameters, "TargetAudience");
    
    var nameFilter = !string.IsNullOrEmpty(name) ? $"startswith(fields/Title, '{name}')" : string.Empty;
    var retailCategoryFilter = !string.IsNullOrEmpty(retailCategory) ? $"fields/RetailCategory eq '{retailCategory}'" : string.Empty;
    var filters = new List<string> { nameFilter };
    filters.RemoveAll(f => string.IsNullOrEmpty(f));
    var filterQuery = filters.Count == 1 ? filters.FirstOrDefault() : string.Join(" and ", filters);
    ```

1. Enregistrez vos modifications

## Tâche 3 : approvisionner les ressources

Exécutez le processus Préparer les dépendances de l’application Teams pour approvisionner des ressources.

Continuez dans Visual Studio.

1. Dans l’**Explorateur de solutions**, cliquez avec le bouton droit sur le projet **MsgExtProductSupport**.
1. Développez le menu **Teams Toolkit** et sélectionnez **Préparer les dépendances de l’application Teams**.
1. Dans la boîte de dialogue **Compte Microsoft 365**, sélectionnez **Continuer**.
1. Dans la boîte de dialogue **Approvisionner**, sélectionnez **Approvisionner**.
1. Dans la boîte de dialogue d’**avertissement de Teams Toolkit**, sélectionnez **Approvisionner**.
1. Dans la boîte de dialogue d’**informations de Teams Toolkit**, **fermez** l’invite.

## Tâche 4 : exécuter et déboguer

Démarrez maintenant le service web et testez l’extension de message dans Copilot pour Microsoft 365.

1. Appuyez sur la touche **F5** pour démarrer une session de débogage et ouvrir une nouvelle fenêtre de navigateur qui accède au client web Microsoft Teams.
1. Entrez vos informations d’identification de compte Microsoft 365 et passez à Microsoft Teams.
1. Dans la boîte de dialogue d’installation de l’application, sélectionnez **Ajouter**.
1. Ouvrez l’application **Copilot** dans Microsoft Teams.
1. Dans la zone de rédaction du message, ouvrez le menu volant **Plug-ins**.
1. Dans la liste des plug-ins, activez le plug-in **Contoso products**.
1. Saisissez **Rechercher des produits Contoso destinés aux particuliers** dans le message et envoyez-le.
1. Dans la réponse Copilot, un bouton de connexion est retourné, sélectionnez le bouton **Se connecter** pour vous authentifier.
1. Une fois l’authentification réussie, saisissez **Rechercher des produits Contoso destinés aux paticuliers** dans le message et envoyez-le.
1. Dans la réponse Copilot, les données retournées dans la réponse du plug-in sont affichées et le plug-in est référencé.
1. Pour afficher la carte adaptative pertinente au résultat, pointez sur les références dans la réponse Copilot.

Fermez le navigateur pour arrêter pour la session de débogage.

[Passez au résumé du labo...](./6-summary.md)