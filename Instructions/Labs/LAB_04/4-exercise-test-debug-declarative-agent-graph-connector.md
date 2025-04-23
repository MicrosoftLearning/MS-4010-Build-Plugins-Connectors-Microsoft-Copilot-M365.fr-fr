---
lab:
  title: "Exercice\_3\_: tester et déboguer"
  module: 'LAB 04: Add custom knowledge to declarative agents using Microsoft Graph connectors and Visual Studio Code'
---

# Exercice 3 : tester et déboguer

Dans cet exercice, vous allez tester et déployer votre agent déclaratif sur Microsoft 365 et le tester à l’aide de Microsoft 365 Copilot Chat.

### Durée de l’exercice

- **Durée estimée :** 5 minutes

## Tâche 1 : tester l’agent déclaratif dans Microsoft 365 Copilot

Pour tester votre agent déclaratif, déployez-le en tant qu’application sur votre locataire Microsoft 365. Après l’avoir ouvert dans Microsoft 365 Copilot, vérifiez qu’il fonctionne comme prévu.

Dans Visual Studio Code :

1. Dans la **barre d’activité** (barre latérale), ouvrez l’extension **Teams Toolkit**.
1. Dans le volet **Cycle de vie**, choisissez **Configurer**. Le kit de ressources Teams compile le projet d’agent déclaratif en tant qu’application et le charge dans Microsoft 365.
1. Ouvrez un navigateur web.

Dans un navigateur Web :

1. Accédez à [https://www.microsoft365.com/chat](https://www.microsoft365.com/chat).
1. Connectez-vous avec votre compte professionnel qui appartient à votre locataire Microsoft 365.
1. Dans Microsoft 365 Copilot, dans le panneau latéral, sélectionnez l’agent **Stratégies informatiques Contoso** pour l’activer.
1. Dans la zone de texte de conversation, demandez `What's the acceptable use policy at Contoso?`.
1. Attendez que l’agent réponde. Notez comment la réponse inclut des références à du contenu externe que le connecteur Graph a ingéré. L’URL de chaque référence pointe vers l’emplacement dans le système externe où le contenu est stocké.

    ![Capture d’écran de Microsoft 365 Copilot répondant à l’invite d’un utilisateur.](../media/LAB_04/3-copilot-response.png)