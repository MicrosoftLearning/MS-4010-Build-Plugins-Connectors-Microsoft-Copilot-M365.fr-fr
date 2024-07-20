---
lab:
  title: "Exercice\_3\_- Garantir l’accès sécurisé avec une liste de contrôle d’accès"
  module: 'LAB 02: Integrate external content with Copilot for Microsoft 365 using Microsoft Graph connectors built with .NET'
---

# Exercice 3 - Garantir l’accès sécurisé avec une liste de contrôle d’accès

Dans cet exercice, vous mettez à jour le code responsable de l’importation de fichiers Markdown locaux avec la configuration d’ACL sur les éléments sélectionnés.

## Avant de commencer

Cet exercice va prendre environ **XX minutes**.

## Tâche 1 - Importer du contenu accessible à tous les membres de l’organisation

Lorsque vous avez implémenté le code pour l’importation de contenu externe dans l’exercice précédent, vous l’avez configuré pour qu’il soit accessible à tous les membres de l’organisation. Voici le code que vous avez utilisé :

```csharp
static IEnumerable<ExternalItem> Transform(IEnumerable<DocsArticle> content)
{
  var baseUrl = new Uri("https://learn.microsoft.com/graph/");

  return content.Select(a =>
  {
    var docId = GetDocId(a.RelativePath ?? "");

    return new ExternalItem
    {
      Id = docId,
      Properties = new()
      {
        AdditionalData = new Dictionary<string, object> {
            { "title", a.Title ?? "" },
            { "description", a.Description ?? "" },
            { "url", new Uri(baseUrl, a.RelativePath!.Replace(".md", "")).ToString() }
        }
      },
      Content = new()
      {
        Value = a.Content ?? "",
        Type = ExternalItemContentType.Html
      },
      Acl = new()
      {
          new()
          {
            Type = AclType.Everyone,
            Value = "everyone",
            AccessType = AccessType.Grant
          }
      }
    };
  });
}
```

Vous avez configuré l’ACL pour accorder l’accès à tout le monde. Nous allons l’ajuster pour les pages Markdown sélectionnées que vous importez.

## Tâche 2 - Importer du contenu accessible à certains utilisateurs

Configurez tout d’abord l’une des pages que vous importez pour qu’elle soit accessible à un utilisateur spécifique seulement.

Dans un navigateur web :

1. Accédez au portail Azure à l’adresse [https://portal.azure.com](https://portal.azure.com) et connectez-vous avec votre compte professionnel ou scolaire.
1. Dans la barre latérale, sélectionnez **Microsoft Entra ID**.
1. Dans le volet de navigation, sélectionnez **Utilisateurs**.
1. Dans la liste des utilisateurs, ouvrez l’un des utilisateurs en sélectionnant son nom.
1. Copiez la valeur de la propriété **Object ID**.

   :::image type="content" source="../media/8-user.png" alt-text="Capture d’écran du portail Azure avec un profil utilisateur ouvert.":::

Utilisez cette valeur pour définir une nouvelle ACL pour une page Markdown spécifique.

Dans l’éditeur de code :

1. Ouvrez le fichier **ContentService.cs** et recherchez la méthode `Transform`.
1. Dans le délégué `Select`, définissez l’ACL par défaut qui s’applique à tous les éléments importés :

   ```csharp
   var acl = new Acl
   {
     Type = AclType.Everyone,
     Value = "everyone",
     AccessType = AccessType.Grant
   };
   ```

1. Remplacez ensuite l’ACL par défaut pour le fichier Markdown dont le nom se termine par `use-the-api.md` :

   ```csharp
   if (a.RelativePath!.EndsWith("use-the-api.md"))
   {
     acl = new()
     {
       Type = AclType.User,
       // AdeleV
       Value = "6de8ec04-6376-4939-ab47-83a2c85ab5f5",
       AccessType = AccessType.Grant
     };
   }
   ```

1. Enfin, mettez à jour le code qui retourne l’élément externe pour utiliser l’ACL définie :

   ```csharp
   return new ExternalItem
   {
     Id = docId,
     Properties = new()
     {
       AdditionalData = new Dictionary<string, object> {
         { "title", a.Title ?? "" },
         { "description", a.Description ?? "" },
         { "url", new Uri(baseUrl, a.RelativePath!.Replace(".md", "")).   ToString() }
       }
     },
     Content = new()
     {
       Value = a.Content ?? "",
       Type = ExternalItemContentType.Html
     },
     Acl = new()
     {
       acl
     }
   };
   ```

1. La méthode `Transform` mise à jour se présente comme suit :

   ```csharp
   static IEnumerable<ExternalItem> Transform(IEnumerable<DocsArticle>    content)
   {
     var baseUrl = new Uri("https://learn.microsoft.com/graph/");
   
     return content.Select(a =>
     {
       var acl = new Acl
       {
         Type = AclType.Everyone,
         Value = "everyone",
         AccessType = AccessType.Grant
       };
   
       if (a.RelativePath!.EndsWith("use-the-api.md"))
       {
         acl = new()
         {
           Type = AclType.User,
           // AdeleV
           Value = "6de8ec04-6376-4939-ab47-83a2c85ab5f5",
           AccessType = AccessType.Grant
         };
       }
   
       var docId = GetDocId(a.RelativePath ?? "");
   
       return new ExternalItem
       {
         Id = docId,
         Properties = new()
         {
           AdditionalData = new Dictionary<string, object> {
             { "title", a.Title ?? "" },
             { "description", a.Description ?? "" },
             { "url", new Uri(baseUrl, a.RelativePath!.Replace(".md", "")).   ToString() }
           }
         },
         Content = new()
         {
           Value = a.Content ?? "",
           Type = ExternalItemContentType.Html
         },
         Acl = new()
         {
           acl
         }
       };
     });
   }
   ```

1. Enregistrez vos modifications.

## Tâche 3 - Importer du contenu accessible à un groupe donné

Étendons maintenant le code afin qu’une autre page soit accessible à un groupe d’utilisateurs sélectionné seulement.

Dans un navigateur web :

1. Accédez au portail Azure à l’adresse [https://portal.azure.com](https://portal.azure.com) et connectez-vous avec votre compte professionnel ou scolaire.
1. Dans la barre latérale, sélectionnez **Microsoft Entra ID**.
1. Dans le volet de navigation, sélectionnez **Groupes**.
1. Dans la liste des groupes, ouvrez l’un des groupes en sélectionnant son nom.
1. Copiez la valeur de la propriété **Object Id**.

:::image type="content" source="../media/8-group.png" alt-text="Capture d’écran du portail Azure avec la page d’un groupe ouverte.":::

Utilisez cette valeur pour définir une nouvelle ACL pour une page Markdown spécifique.

Dans l’éditeur de code :

1. Ouvrez le fichier **ContentService.cs** et recherchez la méthode `Transform`.
1. Étendez la clause `if`  définie précédemment, avec une condition supplémentaire pour définir l’ACL pour le fichier Markdown dont le nom se termine par `traverse-the-graph.md` :

   ```csharp
   if (a.RelativePath!.EndsWith("use-the-api.md"))
   {
     acl = new()
     {
       Type = AclType.User,
       // AdeleV
       Value = "6de8ec04-6376-4939-ab47-83a2c85ab5f5",
       AccessType = AccessType.Grant
     };
   }
   else if (a.RelativePath.EndsWith("traverse-the-graph.md"))
   {
     acl = new()
     {
       Type = AclType.Group,
       // Sales and marketing
       Value = "a9fd282f-4634-4cba-9dd4-631a2ee83cd3",
       AccessType = AccessType.Grant
     };
   }
   ```

1. La méthode `Transform` mise à jour se présente comme suit :

   ```csharp
   static IEnumerable<ExternalItem> Transform(IEnumerable<DocsArticle>    content)
   {
     var baseUrl = new Uri("https://learn.microsoft.com/graph/");
   
     return content.Select(a =>
     {
       var acl = new Acl
       {
         Type = AclType.Everyone,
         Value = "everyone",
         AccessType = AccessType.Grant
       };
   
       if (a.RelativePath!.EndsWith("use-the-api.md"))
       {
         acl = new()
         {
           Type = AclType.User,
           // AdeleV
           Value = "6de8ec04-6376-4939-ab47-83a2c85ab5f5",
           AccessType = AccessType.Grant
         };
       }
       else if (a.RelativePath.EndsWith("traverse-the-graph.md"))
       {
         acl = new()
         {
           Type = AclType.Group,
           // Sales and marketing
           Value = "a9fd282f-4634-4cba-9dd4-631a2ee83cd3",
           AccessType = AccessType.Grant
         };
       }
   
       var docId = GetDocId(a.RelativePath ?? "");
   
       return new ExternalItem
       {
         Id = docId,
         Properties = new()
         {
           AdditionalData = new Dictionary<string, object> {
               { "title", a.Title ?? "" },
               { "description", a.Description ?? "" },
               { "url", new Uri(baseUrl, a.RelativePath!.Replace(".md",    "")).ToString() }
           }
         },
         Content = new()
         {
           Value = a.Content ?? "",
           Type = ExternalItemContentType.Html
         },
         Acl = new()
         {
             acl
         }
       };
     });
   }
   ```

1. Enregistrez vos modifications.

## Tâche 4 - Appliquer les nouvelles ACL

La dernière suivante consiste à appliquer les ACL nouvellement configurées.

1. Ouvrez un terminal et remplacez le répertoire de travail par votre projet.
1. Générez le projet en exécutant la commande `dotnet build`.
1. Démarrez le chargement du contenu en exécutant la commande `dotnet run -- load-content`.

[Passez à l’exercice suivant…](./5-exercise-enable-inline-results.md)