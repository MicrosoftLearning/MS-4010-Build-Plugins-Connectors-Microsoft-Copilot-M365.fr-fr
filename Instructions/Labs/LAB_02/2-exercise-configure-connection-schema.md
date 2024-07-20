---
lab:
  title: "Exercice\_1\_- Configurer une connexion externe et déployer un schéma"
  module: 'LAB 02: Integrate external content with Copilot for Microsoft 365 using Microsoft Graph connectors built with .NET'
---

# Exercice - Configurer une connexion externe et déployer un schéma

Dans cet exercice, vous créez un connecteur Microsoft Graph personnalisé en tant qu’application console. Vous enregistrez une nouvelle inscription d’application Microsoft Entra et ajoutez le code pour créer une connexion externe et déployer son schéma.

## Avant de commencer

Cet exercice va prendre environ **XX minutes**.

## Tâche 1 - Enregistrer une nouvelle inscription d’application Microsoft Entra

Commencez enregistrer inscrire une nouvelle inscription d’application Entra, que le connecteur Graph personnalisé utilise pour s’authentifier auprès de Microsoft 365.

Dans un navigateur Web :

1. Accédez au **portail Azure** à l’adresse [https://portal.azure.com](https://portal.azure.com).
1. Dans le volet de navigation, sélectionnez **Microsoft Entra ID**.
1. Dans le volet de navigation latéral, sélectionnez **Inscriptions d’applications**.
1. Dans le volet de navigation supérieur, sélectionnez **Nouvelle inscription**.
1. Spécifiez les valeurs suivantes :
   1. **Nom :** connecteur MSGraph docs Graph
   1. **Types de compte pris en charge :** comptes dans cet annuaire d’organisation uniquement (locataire unique)
1. Sélectionnez **Enregistrer** pour confirmer votre entrée.
1. Dans l’écran de vue d’ensemble, copiez les valeurs des propriétés **Application ID** et **Directory (tenant) ID**. Vous en aurez besoin plus tard.

## Tâche 2 - Créer des informations d’identification

Étant donné que ce connecteur Graph personnalisé s’exécute sans interaction utilisateur, vous devez le configurer pour s’authentifier automatiquement. Par souci de simplicité, créez un secret.

Continuez dans le navigateur web :

1. Dans le volet de navigation latéral, sélectionnez **Certificats et secrets**.
1. Activez l’onglet **Secrets client** et cliquez sur le bouton **Nouveau secret client**.
1. Créez le secret en sélectionnant **Ajouter**.
1. Copiez la **Valeur** du secret nouvellement créé. Vous en aurez besoin ultérieurement.

## Tâche 3 - Accorder des autorisations d’API

La dernière étape de la configuration de l’inscription d’application Entra consiste à lui accorder des autorisations d’API afin qu’elle puisse créer la connexion externe et le schéma.

Continuez dans le navigateur web :

1. Dans le volet de navigation latéral, sélectionnez **Autorisations d’API**.
1. Sélectionnez **Ajouter une autorisation**.
1. Dans la liste des API, sélectionnez **Microsoft Graph**.
1. Sélectionnez ensuite **Autorisations d’application**.
1. Dans la zone de texte de filtre, entrez **externe**.
1. Développez la section **ExternalConnection** et sélectionnez l’autorisation **ExternalConnection.ReadWrite.OwnedBy**.
1. Développez la section **ExternalItem** et sélectionnez l’autorisation **ExternalItem.ReadWrite.OwnedBy**.
1. Pour confirmer votre choix, cliquez sur le bouton **Ajouter des autorisations**.
1. Pour terminer la configuration, octroyez le consentement administrateur en cliquant sur le bouton **Accorder le consentement administrateur pour (locataire)**.
1. Confirmez la boîte de dialogue en sélectionnant **Oui**.

## Tâche 4 - Créer une application console et installer des dépendances

Après avoir configuré l’inscription de l’application Entra, l’étape suivante consiste à créer une application console, où vous allez implémenter le code du connecteur Graph.

Dans un terminal :

1. Créez un dossier et remplacez le répertoire de travail par celui-ci.
1. Créez une application console en exécutant `dotnet new console`.
1. Ajoutez des dépendances, dont vous avez besoin pour générer le connecteur :
   1. Pour ajouter la bibliothèque nécessaire à l’authentification auprès de Microsoft 365, exécutez `dotnet add package Azure.Identity`.
   1. Pour ajouter la bibliothèque de client pour communiquer avec les API Graph, exécutez `dotnet add package Microsoft.Graph`.
   1. Pour ajouter la bibliothèque nécessaire pour utiliser les secrets utilisateur, que vous allez configurer à l’étape suivante, exécutez `dotnet add package Microsoft.Extensions.Configuration.UserSecrets`.
   1. Vous implémentez le connecteur Graph en tant qu’application console avec deux commandes : une pour créer la connexion externe et déployer le schéma, et une autre pour importer le contenu. Pour prendre en charge la définition de commandes dans votre application, exécutez `dotnet add package System.CommandLine --prerelease`.

## Tâche 5 - Stocker en toute sécurité les informations d’inscription de l’application Entra

Après avoir créé l’inscription de l’application Entra, vous avez noté ses informations telles que l’ID d’application et de locataire, ainsi que le secret. Vous utilisez ces informations dans votre connecteur pour vous authentifier auprès de Microsoft 365. Pour les rendre disponible dans votre code, vous les stockez en toute sécurité avec votre projet en tant que secrets utilisateur.

Dans un terminal :

1. Vérifiez que votre répertoire de travail est défini sur votre application console nouvellement créée.
1. Pour commencer les secrets utilisateur, exécutez `dotnet user-secrets init`.
1. Pour stocker en toute sécurité les informations relatives à l’inscription de l’application, remplacez les jetons par les valeurs réelles que vous avez copiées précédemment et exécutez :

   ```dotnetcli
   dotnet user-secrets set "EntraId:ClientId" "[application id]"
   dotnet user-secrets set "EntraId:ClientSecret" "[secret value]"
   dotnet user-secrets set "EntraId:TenantId" "[directory (tenant) id]"
   ```

## Tâche 6 - Créer un client Microsoft Graph

Les connecteurs Graph personnalisés utilisent l’API Microsoft Graph pour gérer leur connexion externe et les éléments. Commencez par créer une instance de la classe `GraphServiceClient` à partir du package NuGet **Microsoft Graph** que vous avez installé dans le projet.

1. Ouvrez votre projet dans votre éditeur de code.
1. Dans votre projet, ajoutez un nouveau fichier de code nommé **GraphService.cs**.
1. Dans le fichier, commencez par ajouter des références aux espaces de noms que vous allez utiliser, en ajoutant :

   ```csharp
   using Azure.Identity;
   using Microsoft.Graph;
   using Microsoft.Extensions.Configuration;
   ```

1. Définissez ensuite une nouvelle classe nommée GraphService :

   ```csharp
   class GraphService
   {
   }
   ```

1. Dans la classe `GraphService`, définissez un singleton pour stocker une instance de `GraphServiceClient` afin de communiquer avec les API Microsoft Graph :

   ```csharp
   class GraphService
   {
     static GraphServiceClient? _client;

     public static GraphServiceClient Client
     {
       get
       {
         // TODO: implement
       }
     }
   }
   ```

1. Dans le singleton `Client`, implémentez le getter afin qu’il crée une nouvelle instance de `GraphServiceClient` s’il n’en existe pas encore.

   ```csharp
   public static GraphServiceClient Client
   {
     get
     {
       if (_client is null)
       {
         // TODO: implement
       }
       return _client;
     }
   }
   ```

1. Créez une instance de `GraphServiceClient`, à l’aide d’informations d’identification sur l’inscription de l’application Entra que vous avez stockée précédemment :

   ```csharp
   var builder = new ConfigurationBuilder().AddUserSecrets<GraphService>();
   var config = builder.Build();

   var clientId = config["EntraId:ClientId"];
   var clientSecret = config["EntraId:ClientSecret"];
   var tenantId = config["EntraId:TenantId"];

   var credential = new ClientSecretCredential(tenantId, clientId, clientSecret);
   _client = new GraphServiceClient(credential);
   ```

   Vous commencez par créer un générateur de configuration pour accéder aux informations relatives à l’inscription de l’application Entra stockée dans les secrets utilisateur. Vous utilisez ensuite le générateur pour récupérer les informations d’inscription de l’application. Puis vous créez des informations d’identification de clé secrète client en transmettant l’ID de locataire et l’ID client, ainsi que la clé secrète client. Enfin, vous créez une instance de `GraphServiceClient` transmettant les informations d’identification nouvellement créées.

1. Le code complet se présente comme suit :

   ```csharp
   using Azure.Identity;
   using Microsoft.Graph;
   using Microsoft.Extensions.Configuration;
   
   class GraphService
   {
     static GraphServiceClient? _client;
   
     public static GraphServiceClient Client
     {
       get
       {
         if (_client is null)
         {
           var builder = new ConfigurationBuilder().   AddUserSecrets<GraphService>();
           var config = builder.Build();
     
           var clientId = config["EntraId:ClientId"];
           var clientSecret = config["EntraId:ClientSecret"];
           var tenantId = config["EntraId:TenantId"];
           
           var credential = new ClientSecretCredential(tenantId, clientId,    clientSecret);
           _client = new GraphServiceClient(credential);
         }
   
         return _client;
       }
     }
   }
   ```

1. Enregistrez vos modifications

## Tâche 7  - Définir une connexion externe et une configuration de schéma

L’étape suivante consiste à définir la connexion externe et le schéma que le connecteur Graph doit utiliser. Étant donné que le code du connecteur doit accéder à l’ID de la connexion externe à plusieurs emplacements, stockez-le dans un emplacement central dans votre code.

Dans l’éditeur de code :

1. Créez un fichier nommé **ConnectionConfiguration.cs**.
1. Ajoutez une référence à l’espace de noms avec des modèles Microsoft Graph :

   ```csharp
   using Microsoft.Graph.Models.ExternalConnectors;
   ```

1. Ensuite, dans le même fichier, définissez une nouvelle classe statique nommée `ConnectionConfiguration` :

   ```csharp
   static class ConnectionConfiguration
   {
   
   }
   ```

1. Dans la classe `ConnectionConfiguration`, ajoutez une nouvelle propriété nommée `ExternalConnection`. Implémentez-la pour retourner une instance du modèle Microsoft Graph `ExternalConnection` :

   ```csharp
   public static ExternalConnection ExternalConnection
   {
     get
     {
       return new ExternalConnection
       {
         Id = "msgraphdocs",
         Name = "Microsoft Graph documentation",
         Description = "Documentation for Microsoft Graph API which    explains what Microsoft Graph is and how to use it."
       };
     }
   }
   ```

1. Ajoutez ensuite une autre propriété nommée `Schema`. Implémentez-le pour retourner une instance du modèle Graph `Schema` :

   ```csharp
   public static Schema Schema
   {
     get
     {
       return new Schema
       {
         BaseType = "microsoft.graph.externalItem",
         Properties = new()
         {
           // TODO: implement
         }
       };
     }
   }
   ```

1. Dans le schéma, définissez les propriétés que vous suivez pour chaque élément externe que vous ingérez à l’aide du connecteur :

   ```csharp
   new Property
   {
     Name = "title",
     Type = PropertyType.String,
     IsQueryable = true,
     IsSearchable = true,
     IsRetrievable = true,
     Labels = new() { Label.Title }
   },
   new Property
   {
     Name = "description",
     Type = PropertyType.String,
     IsQueryable = true,
     IsSearchable = true,
     IsRetrievable = true
   },
   new Property
   {
     Name = "iconUrl",
     Type = PropertyType.String,
     IsRetrievable = true,
     Labels = new() { Label.IconUrl }
   },
   new Property
   {
     Name = "url",
     Type = PropertyType.String,
     IsRetrievable = true,
     Labels = new() { Label.Url }
   }
   ```

   Vous commencez par définir la propriété **title**, qui stocke le titre de l’élément externe importé dans Microsoft 365. Le titre de l’élément fait partie de l’index de recherche en texte intégral (`IsSearchable = true`). Les utilisateurs peuvent également interroger explicitement son contenu dans les requêtes de mots clés (`IsQueryable = true`). Le titre peut également être récupéré et affiché dans les résultats de la recherche (`IsRetrievable = true`). La propriété **title** représente le titre de l’élément, que vous indiquez à l’aide de l’étiquettes émantique `Title`.

   Vous définissez ensuite la propriété **description**, qui stocke le résumé du contenu de l’élément externe. Sa définition est similaire au titre. Il n’existe toutefois aucune étiquette sémantique pour la description, c’est pourquoi vous ne la définissez pas.

   Vous définissez ensuite une propriété pour stocker l’URL de l’icône de chaque élément. Copilot pour Microsoft 365 requiert cette propriété et elle doit être mappée à l’aide de l’étiquette sémantique `IconUrl`.

   Enfin, vous définissez la propriété **url**, qui stocke l’URL d’origine de l’élément externe. Les utilisateurs se servent de cette URL pour accéder à l’élément externe à partir des résultats de la recherche ou de Copilot à partir de Microsoft 365. L’URL est l’une des propriétés requises par Copilot pour Microsoft 365, c’est pourquoi vous devez la mapper à l’aide de l’étiquette sémantique `Url`.

1. Le code complet se présente comme suit :

   ```csharp
   using Microsoft.Graph.Models.ExternalConnectors;
   static class ConnectionConfiguration
   {
     public static ExternalConnection ExternalConnection
     {
       get
       {
         return new ExternalConnection
         {
           Id = "msgraphdocs",
           Name = "Microsoft Graph documentation",
           Description = "Documentation for Microsoft Graph API which    explains what Microsoft Graph is and how to use it."
         };
       }
     }
     public static Schema Schema
     {
       get
       {
         return new Schema
         {
           BaseType = "microsoft.graph.externalItem",
           Properties = new()
           {
             new Property
             {
               Name = "title",
               Type = PropertyType.String,
               IsQueryable = true,
               IsSearchable = true,
               IsRetrievable = true,
               Labels = new() { Label.Title }
             },
             new Property
             {
               Name = "description",
               Type = PropertyType.String,
               IsQueryable = true,
               IsSearchable = true,
               IsRetrievable = true
             },
             new Property
             {
               Name = "iconUrl",
               Type = PropertyType.String,
               IsRetrievable = true,
               Labels = new() { Label.IconUrl }
             },
             new Property
             {
               Name = "url",
               Type = PropertyType.String,
               IsRetrievable = true,
               Labels = new() { Label.Url }
             }
           }
         };
       }
     }
   }
   ```

1. Enregistrez vos modifications

## Tâche 8 : créer une connexion externe

Poursuivez en ajoutant du code qui utilise les informations sur la connexion externe que vous avez définie dans la section précédente pour créer la connexion externe dans Microsoft 365.

Dans l’éditeur de code :

1. Créez un fichier nommé **ConnectionService.cs**.
1. Dans le fichier, commencez par ajouter des références aux espaces de noms :

   ```csharp
   using Microsoft.Graph.Models.ExternalConnectors;
   ```

1. Ensuite, dans le même fichier, définissez une nouvelle classe statique nommée `ConnectionService` :

   ```csharp
   static class ConnectionService
   {
   
   }
   ```

1. Dans la classe `ConnectionService`, ajoutez une nouvelle méthode nommée `CreateConnection` :

   ```csharp
   async static Task CreateConnection()
   {
   }
   ```

1. Dans la méthode `CreateConnection`, utilisez l’instance du client Microsoft Graph pour appeler les API Microsoft Graph et créer la connexion externe à l’aide des informations de connexion précédemment définies :

   ```csharp
   Console.Write("Creating connection...");
   
   await GraphService.Client.External.Connections
     .PostAsync(ConnectionConfiguration.ExternalConnection);
   
   Console.WriteLine("DONE");
   ```

1. Le code complet se présente comme suit :

   ```csharp
   using Microsoft.Graph.Models.ExternalConnectors;
   static class ConnectionService
   {
     async static Task CreateConnection()
     {
       Console.Write("Creating connection...");
   
       await GraphService.Client.External.Connections
         .PostAsync(ConnectionConfiguration.ExternalConnection);
   
       Console.WriteLine("DONE");
     }
   }
   ```

1. Enregistrez vos modifications.

Avant de tester ce code, ajoutez le code pour créer le schéma. De cette façon, vous pouvez tester le flux complet de création et de configuration de la connexion externe.

## Tâche 9 : créer un schéma de connexion externe

La dernière partie de la création d’une connexion externe consiste à faire son schéma.

Dans l’éditeur de code :

1. Ouvrez le fichier **ConnectionService.cs**.
1. Dans la classe ConnectionService, ajoutez une nouvelle méthode nommée `CreateSchema` :

   ```csharp
   async static Task CreateSchema()
   {
   }
   ```

1. Dans la méthode `CreateSchema`, utilisez l’instance du client Microsoft Graph pour appeler l’API Microsoft Graph afin de créer le schéma. Ensuite, attendez sa création.

   ```csharp
   Console.WriteLine("Creating schema...");
   
   await GraphService.Client.External
     .Connections[ConnectionConfiguration.ExternalConnection.Id]
     .Schema
     .PatchAsync(ConnectionConfiguration.Schema);
   
   do
   {
     var externalConnection = await GraphService.Client.External
       .Connections[ConnectionConfiguration.ExternalConnection.Id]
       .GetAsync();
   
     Console.Write($"State: {externalConnection?.State.ToString()}");
   
     if (externalConnection?.State != ConnectionState.Draft)
     {
       Console.WriteLine();
       break;
     }
   
     Console.WriteLine($". Waiting 60s...");
   
     await Task.Delay(60_000);
   }
   while (true);
   
   Console.WriteLine("DONE");
   ```

1. Dans le même fichier, ajoutez une nouvelle méthode nommée `ProvisionConnection`. Dans son code, appelez les méthodes `CreateConnection` et `CreateSchema` définies précédemment :

   ```csharp
   public static async Task ProvisionConnection()
   {
     try
     {
       await CreateConnection();
       await CreateSchema();
     }
     catch (Exception ex)
     {
       Console.WriteLine(ex.Message);
     }
   }
   ```

1. Le code complet se présente comme suit :

   ```csharp
   using Microsoft.Graph.Models.ExternalConnectors;
   
   static class ConnectionService
   {
     async static Task CreateConnection()
     {
       Console.Write("Creating connection...");
   
       await GraphService.Client.External.Connections
         .PostAsync(ConnectionConfiguration.ExternalConnection);
   
       Console.WriteLine("DONE");
     }
   
     async static Task CreateSchema()
     {
       Console.WriteLine("Creating schema...");
   
       await GraphService.Client.External
         .Connections[ConnectionConfiguration.ExternalConnection.Id]
         .Schema
         .PatchAsync(ConnectionConfiguration.Schema);
   
       do
       {
         var externalConnection = await GraphService.Client.External
           .Connections[ConnectionConfiguration.ExternalConnection.Id]
           .GetAsync();
   
         Console.Write($"State: {externalConnection?.State.ToString()}");
   
         if (externalConnection?.State != ConnectionState.Draft)
         {
           Console.WriteLine();
           break;
         }
   
         Console.WriteLine($". Waiting 60s...");
   
         await Task.Delay(60_000);
       }
       while (true);
   
       Console.WriteLine("DONE");
     }
   
     public static async Task ProvisionConnection()
     {
       try
       {
         await CreateConnection();
         await CreateSchema();
       }
       catch (Exception ex)
       {
         Console.WriteLine(ex.Message);
       }
     }
   }
   ```

1. Enregistrez vos modifications.

## Tâche 10 : tester le code

La dernière étape consiste à créer un point d’entrée dans l’application, qui créera la connexion et son schéma. Pour ce faire, créez une commande que vous appelez en démarrant l’application à partir de la ligne de commande.

Dans l’éditeur de code :

1. Ouvrez le fichier **Program.cs**.
1. Remplacez le contenu par le code suivant :

   ```csharp
   using System.CommandLine;
   
   var provisionConnectionCommand = new Command("provision-connection",    "Provisions external connection");
   provisionConnectionCommand.SetHandler(ConnectionService.   ProvisionConnection);
   
   var rootCommand = new RootCommand();
   rootCommand.AddCommand(provisionConnectionCommand);
   Environment.Exit(await rootCommand.InvokeAsync(args));
   ```

   Vous commencerez par définir une commande nommée `provision-connection.`. Cette commande appelle la méthode `ConnectionService.ProvisionConnection` définie précédemment. Enfin, vous inscriverez la commande auprès du processeur de ligne de commande et démarrerez les arguments de surveillance de l’application dans la ligne de commande.

1. Enregistrez vos modifications

Pour tester l'application :

1. Ouvrez un terminal.
1. Changez le répertoire de travail vers le dossier du projet.
1. Exécutez `dotnet build` pour générer le projet.
1. Démarrez l’application en exécutant `dotnet run -- provision-connection`.
1. Patientez plusieurs minutes pour que la connexion et le schéma soient créés.

[Passez à l’exercice suivant…](./3-exercise-import-external-content.md)