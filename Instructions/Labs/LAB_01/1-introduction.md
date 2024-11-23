---
lab:
  title: Présentation
  module: 'LAB 01: Connect Microsoft 365 Copilot to your external data in real-time with message extension plugins built with .NET and Visual Studio'
---

# Présentation

Les extensions de message permettent d’utiliser des systèmes externes à partir de Microsoft Teams et de Microsoft Outlook. Les utilisateurs peuvent se servir des extensions de message pour rechercher, modifier et partager les données de ces systèmes dans des messages et des e-mails en tant que carte enrichie.

Supposons que vous disposez d’une API personnalisée que vous utilisez pour accéder aux informations de produits actuelles et pertinentes pour votre organisation. Vous souhaitez rechercher et partager ces informations dans Microsoft 365. Vous souhaitez également que Microsoft 365 Copilot utilise ces informations dans ses réponses.

Dans ce module, vous allez créer une extension de message. Votre extension de message utilise un bot pour communiquer avec Microsoft Teams, Microsoft Outlook et Microsoft 365 Copilot.

![Capture d’écran des résultats de la recherche retournés par une extension de message basée sur une recherche dans Microsoft Teams.](../media/1-search-results.png)

Cette extension utilise Microsoft Entra pour authentifier les utilisateurs, ce qui lui permet de renvoyer des données à partir de l’API en leur nom.

Une fois que l’utilisateur s’est authentifié, votre extension de message obtient des données de l’API et retourne les résultats de recherche, qui peuvent être incorporés dans des messages et des e-mails sous forme de carte enrichie, puis partagés.

![Capture d’écran des résultats de recherche qui utilisent des données d’une API externe dans Microsoft Teams.](../media/3-search-results-api.png)

![Capture d’écran du résultat de recherche incorporé dans un message dans Microsoft Teams.](../media/4-adaptive-card.png)

L’extension fonctionne avec Microsoft 365 Copilot en tant que plug-in, ce qui lui permet d’interroger les données de produits pour le compte de l’utilisateur et d’utiliser les données retournées dans ses réponses.

![Capture d’écran d’une réponse dans Microsoft 365 Copilot qui contient des informations retournées par le plug-in d’extension de message. Une carte adaptative s’affiche avec des informations sur le produit.](../media/5-copilot-answer.png)

À la fin de ce module, vous pouvez créer des extensions de message écrites en C# (exécution sur .NET). Elle peut être utilisée dans Microsoft Teams, Microsoft Outlook et Microsoft 365 Copilot. Elle peut interroger des données derrière des API protégées et retourner les résultats sous forme de cartes avec mise en forme complexe.

## Prérequis

- Connaissances de base de C#
- Connaissances de base de Bicep
- Connaissances de base de l’authentification
- Accès administrateur à un locataire Microsoft 365
- Accès à un abonnement Azure
- L’accès à Microsoft 365 Copilot est facultatif, et n’est nécessaire que pour réaliser l’**exercice 4 : tâche 5**.
- Visual Studio 2022 17.10+ avec [Teams Toolkit](/microsoftteams/platform/toolkit/toolkit-v4/teams-toolkit-fundamentals-vs) (composant des outils de développement Microsoft Teams) installé
- [.NET 8.0](https://dotnet.microsoft.com/download/dotnet/8.0)
- [Dev Proxy 0.19.1+](https://aka.ms/devproxy)

> [!NOTE]
> Le seul exercice de ce labo qui nécessite une licence Microsoft 365 Copilot est **l’exercice 4 : tâche 5**. Jusqu’ici, tout doit être réalisé, que votre locataire possède Copilot ou non.

## Durée du labo

  - **Durée estimée :** 150 minutes

## Objectifs d’apprentissage

À la fin de ce module, vous devez pouvoir :

- Comprendre ce que sont les extensions de message et comment les créer
- Créer une extension de message
- Découvrir comment authentifier les utilisateurs à l’aide d’une authentification unique et appeler une API personnalisée protégée par l’authentification Microsoft Entra
- Comprendre comment étendre et optimiser les extensions de message à utiliser avec Microsoft 365 Copilot

Lorsque vous êtes prêt à commencer, [passez au premier exercice…](./2-exercise-create-a-message-extension.md)
