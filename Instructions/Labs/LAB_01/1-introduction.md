---
lab:
  title: Présentation
  module: 'LAB 03: Connect Copilot for Microsoft 365 to your external data in real-time with message extension plugins built with .NET and Visual Studio'
---

# Présentation

Les extensions de message permettent d’utiliser des systèmes externes à partir de Microsoft Teams et de Microsoft Outlook. Les utilisateurs peuvent se servir des extensions de message pour rechercher et modifier des données et partager les informations de ces systèmes dans des cartes avec une mise en forme complexe dans des messages et des e-mails.

Supposons que vous disposez d’une liste SharePoint Online avec des informations de produit actuelles et pertinentes pour votre organisation. Vous souhaitez rechercher et partager ces informations dans Microsoft 365. Vous souhaitez également que Copilot pour Microsoft 365 utilise ces informations dans ses réponses.

:::image type="content" source="../media/1-sharepoint-online-product-support-site.png" alt-text="Capture d’écran de la page d’accueil du site d’équipe SharePoint Online du support produit. Une liste de produits récemment publiés s’affiche." lightbox="../media/1-sharepoint-online-product-support-site.png":::

Dans ce module, vous allez créer une extension de message. Votre extension de message utilise un bot pour communiquer avec Microsoft Teams, Microsoft Outlook et Copilot pour Microsoft 365.

:::image type="content" source="../media/2-search-results-nuget.png" alt-text="Capture d’écran des résultats de la recherche retournés par une extension de message basée sur une recherche dans Microsoft Teams." lightbox="../media/2-search-results-nuget.png":::

L’extension utilise Microsoft Entra pour authentifier les utilisateurs, ce qui lui permet de retourner des données à partir de SharePoint Online à l’aide de l’API Microsoft Graph.

:::image type="content" source="../media/3-sign-in.png" alt-text="Capture d’écran d’une demande d’authentification dans une extension de message basée sur une recherche. Un lien de connexion s’affiche." lightbox="../media/3-sign-in.png":::

Une fois que l’utilisateur s’authentifie, votre extension de message reçoit des informations sur le produit de SharePoint Online à l’aide de l’API Microsoft Graph. Elle retourne les résultats de la recherche qui peuvent être incorporés puis partagés par le biais de cartes avec mise en forme complexe dans des messages et des e-mails.

:::image type="content" source="../media/4-search-results-sharepoint-online.png" alt-text="Capture d’écran des résultats de la recherche retournés par une extension de message basée sur une recherche dans Microsoft Teams. Les résultats de la recherche sont retournés à partir de SharePoint Online. Chaque résultat de recherche affiche le nom du produit, la catégorie et l’image du produit." lightbox="../media/4-search-results-sharepoint-online.png":::

:::image type="content" source="../media/5-adaptive-card.png" alt-text="Capture d’écran des résultats de recherche incorporés dans un message dans Microsoft Teams. Les résultats de la recherche sont affichés sous la forme d’une carte adaptative avec le nom du produit, la catégorie, le volume d’appels et la date de version. Un bouton d’action Afficher permet aux utilisateurs et aux utilisatrices d’accéder à l’élément de liste de produits dans SharePoint Online." lightbox="../media/5-adaptive-card.png":::

L’extension fonctionne avec Copilot pour Microsoft 365 en tant que plug-in, ce qui lui permet d’interroger la liste SharePoint Online pour le compte de l’utilisateur ou de l’utilisatrice et d’accéder aux données retournées dans ses réponses.

:::image type="content" source="../media/6-copilot-answer.png" alt-text="Capture d’écran d’une réponse dans Copilot pour Microsoft 365 qui contient des informations retournées par le plug-in d’extension de message. Une carte adaptative s’affiche avec des informations sur le produit." lightbox="../media/6-copilot-answer.png":::

À la fin de ce module, vous pouvez créer des extensions de message écrites en C# (exécution sur .NET). Elle peut être utilisée dans Microsoft Teams, Microsoft Outlook et Copilot pour Microsoft 365. Elle peut interroger des données derrière des API protégées et retourner les résultats sous forme de cartes avec mise en forme complexe.

## Prérequis

- Connaissances de base de C#
- Connaissances de base de Bicep
- Connaissances de base de l’authentification
- Accès administrateur à un locataire Microsoft 365
- Accès à un abonnement Azure
- L’accès à Copilot pour Microsoft 365 est nécessaire pour effectuer un seul exercice, sinon il est facultatif.
- Visual Studio 2022 17.9 avec [Teams Toolkit](/microsoftteams/platform/toolkit/toolkit-v4/teams-toolkit-fundamentals-vs) (outils de développement Microsoft Teams) installé
- [.NET 8.0](https://dotnet.microsoft.com/download/dotnet/8.0)

Lorsque vous êtes prêt, [passez à l’exercice suivant](./2-exercise-create-a-message-extension.md).
