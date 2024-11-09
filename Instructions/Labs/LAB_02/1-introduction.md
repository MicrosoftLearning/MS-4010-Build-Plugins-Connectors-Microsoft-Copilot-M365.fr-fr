---
lab:
  title: Présentation
  module: 'LAB 02: Build your own message extension plugin with TypeScript (TS) for Microsoft Copilot'
---

# Présentation

Dans ce projet, vous allez apprendre à utiliser les extensions de message Teams en tant que plug-ins dans Microsoft Copilot pour Microsoft 365. Le projet est basé sur l’exemple « Northwind Inventory » contenu dans ce même [référentiel GitHub](https://github.com/OfficeDev/Copilot-for-M365-Plugins-Samples/tree/main/samples/msgext-northwind-inventory-ts). En utilisant l’ancienne [base de données Northwind](https://learn.microsoft.com/dotnet/framework/data/adonet/sql/linq/downloading-sample-databases), vous disposez de nombreuses données d’entreprise simulées à utiliser.

Northwind exploite une activité d’e-commerce d’aliments spécialisés à Spokane, Washington. Dans ce labo, vous allez travailler avec l’application Northwind Inventory, qui donne accès à l’inventaire des produits et aux informations financières.

Cet exercice devrait prendre environ **60** minutes.

## Avant de commencer

- [**Commencez par**](./2-prepare-development-environment.md) configurer votre environnement de développement et en exécutant l’application.

- Dans [**l’exercice 1**](./3-exercise-1-run-message-extension.md), vous allez exécuter la même application qu’une [extension de message](https://learn.microsoft.com/microsoftteams/platform/messaging-extensions/what-are-messaging-extensions) dans Microsoft Teams et Outlook.

- Dans [**l’exercice 2**](./4-exercise-2-run-copilot-plugin.md), vous allez exécuter l’application en tant que plug-in de Copilot pour Microsoft 365. Vous allez expérimenter différentes invites et vous observerez comment le plug-in est appelé à l’aide de différents paramètres. Lorsque vous discutez avec Copilot, vous pouvez suivre les requêtes effectuées sur la console développeur.

- Dans [**l’exercice 3**](./5-exercise-3-add-new-command.md), vous allez apprendre à ajouter une nouvelle commande à l’application afin de pouvoir développer les fonctionnalités du plug-in et effectuer plus de tâches.

  ![Capture d’écran d’une carte adaptative affichant un produit.](../media/1-00-product-card-only.png)

- Enfin, dans **[l’exercice 4](./6-exercise-4-explore-plugin-source-code.md)** vous allez explorer le code pour voir comment il fonctionne plus en profondeur. Si vous n’avez pas encore Copilot, tout le reste fonctionne toujours comme une extension de message pour Microsoft 365.

Lorsque vous êtes prêt à commencer, [passez à l’exercice suivant…](./2-prepare-development-environment.md)