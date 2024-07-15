---
lab:
  title: Présentation
  module: 'LAB 02: Integrate external content with Copilot for Microsoft 365 using Microsoft Graph connectors built with .NET'
---

# Présentation

Supposons que vous disposez d’un système externe dans lequel vous stockez des articles de base de connaissances. Ces articles contiennent des informations sur les différents processus de votre organisation. Vous souhaitez être en mesure de retrouver et de découvrir facilement les informations pertinentes de Microsoft 365. Vous souhaitez également que Copilot pour Microsoft 365 inclue des informations de ces articles de base de connaissances dans ses réponses.

Pour exposer ces informations externes dans Microsoft 365, vous allez créer un connecteur Microsoft Graph personnalisé. Les connecteurs Microsoft Graph se connectent à votre système externe (1) pour récupérer du contenu, utiliser les informations de Microsoft Entra ID pour vous authentifier auprès de Microsoft 365 (2) et importer le contenu dans Microsoft 365 à l’aide de l’API Microsoft Graph (3).

:::image type="content" source="../media/1-graph-connector-concept.png" alt-text="Diagramme montrant le travail conceptuel d’un connecteur Microsoft Graph.":::

Dans ce module, vous allez découvrir les connecteurs Microsoft Graph et pourquoi vous devez envisager de les utiliser dans votre organisation. Vous créez un connecteur Microsoft Graph qui importe des fichiers Markdown locaux dans Microsoft 365. Vous découvrez également comment vous assurer que le contenu externe que vous importez est accessible uniquement aux personnes disposant d’autorisations appropriées. Enfin, vous optimisez votre connecteur Microsoft Graph à utiliser avec Copilot pour Microsoft 365.

## Prérequis

- Connaissances de base de C#
- Connaissances de base de l’authentification
- Accès à un [locataire de développeur Microsoft 365](https://developer.microsoft.com/microsoft-365/dev-program?ocid=MSlearn)
- [.NET 8.0](https://dotnet.microsoft.com/download/dotnet/8.0)

Lorsque vous êtes prêt à commencer, [passez à l’exercice suivant...](./2-exercise-configure-connection-schema.md)