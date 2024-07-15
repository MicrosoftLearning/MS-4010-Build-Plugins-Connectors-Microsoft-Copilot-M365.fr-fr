---
lab:
  title: "Exercice\_2\_- Importer du contenu externe"
  module: 'LAB 02: Integrate external content with Copilot for Microsoft 365 using Microsoft Graph connectors built with .NET'
---

# Exercice - Importer du contenu externe

Dans cet exercice, vous étendez le connecteur Microsoft Graph personnalisé avec le code pour importer des fichiers Markdown locaux dans Microsoft 365.

## Avant de commencer

Cet exercice va prendre environ **XX minutes**.

## Tâche 1 - Télécharger du contenu externe

Pour suivre cet exercice, copiez les exemples de fichiers de contenu utilisés dans cet exercice à partir de [GitHub](https://pnp.github.io/download-partial/?url=https://github.com/pnp/graph-connectors-samples/tree/main/samples/dotnet-csharp-graphdocs/content) et stockez-les dans votre projet, dans un dossier nommé **contenu**.

:::image type="content" source="../media/6-content-files.png" alt-text="Capture d’écran d’un éditeur de code montrant les fichiers de contenu utilisés dans cet exercice.":::

Pour que le code fonctionne correctement, le dossier **contenu** et son contenu doivent être copiés dans le dossier de sortie de build.

Dans l’éditeur de code :

1. Ouvrez le fichier index **.csproj** et ajoutez le code suivant avant la balise `</Project>` :

   ```xml
   <ItemGroup>
     <ContentFiles Include="content\**"    CopyToOutputDirectory="PreserveNewest" />
   </ItemGroup>
   
   <Target Name="CopyContentFolder" AfterTargets="Build">
     <Copy SourceFiles="@(ContentFiles)" DestinationFiles="@   (ContentFiles->'$(OutputPath)\content\%(RecursiveDir)%(Filename)%   (Extension)')" />
   </Target>
   ```

1. Enregistrez vos modifications.

## Tâche 2 - Ajouter des bibliothèques pour analyser Markdown et YAML

Le connecteur Microsoft Graph que vous créez importe les fichiers Markdown locaux dans Microsoft 365. Chacun de ces fichiers contient un en-tête avec des métadonnées au format YAML, également appelé frontmatter. En outre, le contenu de chaque fichier est écrit en Markdown. Pour extraire les métadonnées et convertir le corps en HTML, vous utilisez des bibliothèques personnalisées :

1. Ouvrez un terminal et remplacez le répertoire de travail par votre projet.
1. Pour ajouter la bibliothèque de traitement Markdown, exécutez la commande suivante : `dotnet add package Markdig`.
1. Pour ajouter la bibliothèque de traitement YAML, exécutez la commande suivante : `dotnet add package YamlDotNet`.

## Tâche 3 - Définir la classe pour représenter le fichier importé

Pour simplifier l’utilisation des fichiers Markdown importés et de leur contenu, nous allons définir une classe avec les propriétés nécessaires.

Dans l’éditeur de code :

1. Créez un fichier nommé **ContentService.cs**.
1. Ajoutez le code suivant :

   ```csharp
   using YamlDotNet.Serialization;
   
   public interface IMarkdown
   {
     string? Markdown { get; set; }
   }
   
   class DocsArticle : IMarkdown
   {
     [YamlMember(Alias = "title")]
     public string? Title { get; set; }
     [YamlMember(Alias = "description")]
     public string? Description { get; set; }
     public string? Markdown { get; set; }
     public string? Content { get; set; }
     public string? RelativePath { get; set; }
   }
   ```

   L’interface `IMarkdown` représente le contenu du fichier Markdown local. Elle doit être définie séparément pour prendre en charge la désérialisation du contenu du fichier. La classe `DocsArticle` représente le document final avec des propriétés YAML et du contenu HTML analysés. Les attributs `YamlMember` mappent les propriétés aux métadonnées dans l’en-tête de chaque document.

1. Enregistrez vos modifications.

## Tâche 4 - Définir la classe `ContentService`

Générez ensuite une classe qui contient le code permettant de charger des fichiers Markdown locaux, de les transformer en éléments externes et de les charger dans Microsoft 365.

Dans l’éditeur de code :

1. Assurez-vous de modifier le fichier **ContentService.cs**.
1. Au début du fichier, ajoutez l’instruction using suivante :

   ```csharp
   using Microsoft.Graph.Models.ExternalConnectors;
   ```

1. Puis, à la fin du fichier, ajoutez le code suivant :

   ```csharp
   static class ContentService
   {
     static IEnumerable<DocsArticle> Extract()
     {}
   
     static IEnumerable<ExternalItem> Transform(IEnumerable<DocsArticle>    content)
     {}
   
     static async Task Load(IEnumerable<ExternalItem> items)
     {}
   
     public static async Task LoadContent()
     {
       var content = Extract();
       var transformed = Transform(content);
       await Load(transformed);
     }
   }
   ```

   La classe `ContentService` définit trois méthodes qui représentent le processus de gestion du contenu :

   1. `Extract`, qui charge les fichiers Markdown locaux et les analyse en instances de la classe `DocsArticle` pour faciliter la gestion.
   1. `Transform`, qui convertit les objets `DocsArticle` en instances de la classe `ExternalItems`, qui fait partie du Kit de développement logiciel (SDK) .NET Microsoft Graph et qui représente les éléments externes à charger dans Microsoft 365.
   1. `Load`, qui charge les éléments externes dans Microsoft 365 à l’aide de l’API Microsoft Graph.

   Ces méthodes sont appelées dans cet ordre spécifique à partir de la méthode `LoadContent`.

1. Enregistrez vos modifications.

## Tâche 5 - Configurer le traitement Markdown

Commençons par extraire le contenu des fichiers Markdown locaux.

Ajoutez tout d’abord des méthodes d’assistance pour utiliser facilement les bibliothèques et `Markdig` et `YamlDotNet`.

Dans l’éditeur de code :

1. Créez un fichier nommé **MarkdownExtensions.cs**.
1. Dans le fichier , ajoutez le code suivant :

   ```csharp
   // from: https://khalidabuhakmeh.com/parse-markdown-front-matter-with-csharp
   using Markdig;
   using Markdig.Extensions.Yaml;
   using Markdig.Syntax;
   using YamlDotNet.Serialization;
   
   public static class MarkdownExtensions
   {
     private static readonly IDeserializer YamlDeserializer =
         new DeserializerBuilder()
         .IgnoreUnmatchedProperties()
         .Build();
         
     private static readonly MarkdownPipeline Pipeline
         = new MarkdownPipelineBuilder()
         .UseYamlFrontMatter()
         .Build();
   }
   ```

   La propriété `YamlDeserializer` définit un nouveau désérialiseur pour le bloc YAML dans chacun des fichiers Markdown que vous extrayez. Vous configurez le désérialiseur pour ignorer toutes les propriétés qui ne font pas partie de la classe dans laquelle le fichier est désérialisé.

   La propriété `Pipeline` définit un pipeline de traitement pour l’analyseur Markdown. Vous pouvez la configurer pour analyser l’en-tête YAML. Sans cette configuration, les informations de l’en-tête sont ignorées.

1. Étendez la classe `MarkdownExtensions` avec le code suivant :

   ```csharp
   public static T GetContents<T>(this string markdown) where T :    IMarkdown, new()
   {
     var document = Markdown.Parse(markdown, Pipeline);
     var block = document
         .Descendants<YamlFrontMatterBlock>()
         .FirstOrDefault();
   
     if (block == null)
       return new T { Markdown = markdown };
   
     var yaml =
         block
         // this is not a mistake
         // we have to call .Lines 2x
         .Lines // StringLineGroup[]
         .Lines // StringLine[]
         .OrderByDescending(x => x.Line)
         .Select(x => $"{x}\n")
         .ToList()
         .Select(x => x.Replace("---", string.Empty))
         .Where(x => !string.IsNullOrWhiteSpace(x))
         .Aggregate((s, agg) => agg + s);
   
     var t = YamlDeserializer.Deserialize<T>(yaml);
     t.Markdown = markdown.Substring(block.Span.End + 1);
     return t;
   }
   ```

   La méthode `GetContents` convertit une chaîne Markdown avec des métadonnées YAML dans l’en-tête dans le type spécifié, qui implémente l’interface `IMarkdown` . À partir de la chaîne Markdown, elle extrait l’en-tête YAML et le désérialise dans le type spécifié. Ensuite, elle extrait le corps de l’article et le définit sur la propriété `Markdown` pour un traitement ultérieur.

1. Enregistrez vos modifications.

## Tâche 6 : extraire le contenu Markdown et YAML

Avec les méthodes d’assistance en place, implémentez la méthode Extract pour charger les fichiers Markdown locaux et extraire des informations à partir de ces derniers.

Dans l’éditeur de code :

1. Ouvrez le fichier **ContentService.cs**.
1. Au début du fichier, ajoutez l’instruction using suivante :

   ```csharp
   using Markdig;
   ```

1. Ensuite, implémentez la méthode `Extract` dans la classe `ContentService` en utilisant le code suivant :

   ```csharp
   static IEnumerable<DocsArticle> Extract()
   {
     var docs = new List<DocsArticle>();
   
     var contentFolder = "content";
     var contentFolderPath = Path.Combine(Directory.GetCurrentDirectory(),    contentFolder);
     var files = Directory.GetFiles(contentFolder, "*.md", SearchOption.   AllDirectories);
   
     foreach (var file in files)
     {
       try
       {
         var contents = File.ReadAllText(file);
         var doc = contents.GetContents<DocsArticle>();
         doc.Content = Markdown.ToHtml(doc.Markdown ?? "");
         doc.RelativePath = Path.GetRelativePath(contentFolderPath, file);
         docs.Add(doc);
       }
       catch (Exception ex)
       {
         Console.WriteLine(ex.Message);
       }
     }
   
     return docs;
   }
   ```

   La méthode commence par le chargement de fichiers Markdown à partir du dossier de **contenu**. Pour chaque fichier, elle charge son contenu sous forme de chaîne. Elle convertit la chaîne en objet avec les métadonnées et le contenu stockés dans des propriétés distinctes à l’aide de la méthode d’extension `GetContents` définie précédemment dans la classe `MarkdownExtensions`. Ensuite, elle convertit la chaîne Markdown en HTML. Enfin, elle stocke le chemin d’accès relatif au fichier et ajoute l’objet à une collection pour un traitement ultérieur.

1. Enregistrez vos modifications.

## Tâche :7 : transformer du contenu en éléments externes

Une fois que vous avez lu le contenu externe, l’étape suivante consiste à la transformer en éléments externes, qui seront chargés dans Microsoft 365.

Pour commencer, elle ajoute une méthode d’assistance qui génère un ID unique pour chaque élément externe en fonction de son chemin d’accès de fichier relatif.

Dans l’éditeur de code :

1. Vérifiez que vous modifiez le fichier **ContentService.cs**.
1. Dans la classe `ContentService`, ajoutez la méthode suivante :

   ```csharp
   static string GetDocId(string relativePath)
   {
     var id = relativePath.Replace(Path.DirectorySeparatorChar.ToString(),    "__").Replace(".md", "");
     return id;
   }
   ```

   La méthode `GetDocId` prend le chemin du fichier relatif et remplace tous les séparateurs de répertoires par un trait de soulignement double. Cela est nécessaire, car les caractères de séparateur de chemin d’accès ne peuvent pas être utilisés dans un ID d’élément externe.

1. Enregistrez vos modifications.

À présent, implémentez la méthode `Transform`, qui convertit les objets représentant les fichiers Markdown locaux en éléments externes à partir de Microsoft Graph.

Dans l’éditeur de code :

1. Vérifiez que vous êtes bien dans le fichier **ContentService.cs**.
1. Implémentez la méthode `Transform` avec le code suivant :

   ```csharp
   static IEnumerable<ExternalItem> Transform(IEnumerable<DocsArticle> content)
   {
     var baseUrl = new Uri("https://learn.microsoft.com/graph/");
   
     return content.Select(a =>
     {
       var docId = GetDocId(a.RelativePath ?? '');
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

   Tout d’abord, définissez une URL de base. Utilisez cette URL pour générer une URL complète pour chaque élément, afin que l’élément permette l’accès à l’élément d’origine aux utilisateurs et aux utilisatrices. Ensuite, transformez chaque élément d’un `DocsArticle` en un `ExternalItem`. Récupérez premièrement un ID unique pour chaque élément en fonction de son chemin d’accès de fichier relatif. Ensuite, créez une nouvelle instance de `ExternalItem` et remplissez ses propriétés avec des informations à partir du `DocsArticle`. Ensuite, définissez le contenu de l’élément par rapport au contenu HTML extrait du fichier local et définissez le type de contenu de l’élément selon le fichier HTML. Enfin, configurez l’autorisation de l’élément afin qu’il soit visible par toutes les personnes membres de l’organisation.

1. Enregistrez vos modifications.

## Tâche 8 : charger des éléments externes dans Microsoft 365

La dernière étape du traitement du contenu consiste à charger les éléments externes transformés dans Microsoft 365.

Dans l’éditeur de code :

1. Assurez-vous de modifier le fichier **ContentService.cs**.
1. Implémentez la méthode `Load` dans la classe `ContentService` en utilisant le code suivant :

   ```csharp
   static async Task Load(IEnumerable<ExternalItem> items)
   {
     foreach (var item in items)
     {
       Console.Write(string.Format("Loading item {0}...", item.Id));
   
       try
       {
         await GraphService.Client.External
           .Connections[Uri.EscapeDataString(ConnectionConfiguration.   ExternalConnection.Id!)]
           .Items[item.Id]
           .PutAsync(item);
   
         Console.WriteLine("DONE");
       }
       catch (Exception ex)
       {
         Console.WriteLine("ERROR");
         Console.WriteLine(ex.Message);
       }
     }
   }
   ```

   Pour chaque élément externe, utilisez le Kit de développement logiciel (SDK) Microsoft Graph .NET pour appeler l’API Microsoft Graph et charger l’élément. Dans la requête, spécifiez l’ID de la connexion externe créée précédemment, l’ID de l’élément à charger et le contenu complet de l’élément.

1. Enregistrez vos modifications.

## Tâche 9 : ajouter la commande de chargement de contenu

Avant de pouvoir tester le code, vous devez étendre l’application console avec une commande qui appelle la logique de chargement du contenu.

Dans l’éditeur de code :

1. Ouvrez le fichier **Program.cs**.
1. Ajoutez une nouvelle commande pour charger le contenu avec le code suivant :

    ```csharp
    var loadContentCommand = new Command("load-content", "Loads content   into the external connection");
    loadContentCommand.SetHandler(ContentService.LoadContent);
    ```

1. Inscrivez la commande nouvellement définie avec la commande racine afin qu’elle puisse être appelée à l’aide du code suivant :

     ```csharp
     rootCommand.AddCommand(loadContentCommand);
     ```

1. Enregistrez vos modifications.

## Tâche 10 : tester le code

Il ne reste plus qu’à tester si le connecteur Microsoft Graph importe correctement du contenu externe.

1. Ouvrez un terminal.
1. Modifiez le répertoire de travail de votre projet.
1. Générez le projet en exécutant la commande `dotnet build`.
1. Démarrez le chargement du contenu en exécutant la commande `dotnet run -- load-content`.
1. Attendez que la commande se termine et charge le contenu.

[Passez à l’exercice suivant…](./4-exercise-ensure-secure-access.md)