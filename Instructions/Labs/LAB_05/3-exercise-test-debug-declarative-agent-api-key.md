---
lab:
  title: "Exercice\_2\_: tester l’agent déclaratif dans Microsoft\_365 Copilot\_Chat"
  module: 'LAB 05: Authenticate your API plugin for declarative agents with secured APIs'
---

# Exercice 2 : tester l’agent déclaratif dans Microsoft 365 Copilot Chat

Dans cet exercice, vous allez tester et déployer votre agent déclaratif sur Microsoft 365 et le tester à l’aide de Microsoft 365 Copilot Chat.

### Durée de l’exercice

- **Durée estimée :** 10 minutes

## Tâche 1 : tester l’agent déclaratif avec le plug-in d’API dans Microsoft 365 Copilot Chat

La dernière étape consiste à tester l’agent déclaratif avec le plug-in API dans Microsoft 365 Copilot.

Dans Visual Studio Code :

1. Dans la barre d’activité, ouvrez l’extension **Teams Toolkit**.
1. Dans le panneau d’extension **Teams Toolkit**, dans la section **Comptes**, assurez-vous que vous êtes connecté à votre locataire Microsoft 365.

  ![Capture d’écran de Teams Toolkit montrant l’état de la connexion à Microsoft 365.](../media/LAB_05/3-teams-toolkit-account.png)

1. Dans la barre d’activité, basculez vers la vue Exécuter et Déboguer.
1. Dans la liste des configurations, choisissez **Déboguer dans Copilot (Edge)** et appuyez sur le bouton lecture pour démarrer le débogage.

  ![Capture d’écran de l’option de débogage dans Visual Studio Code.](../media/LAB_05/3-vs-code-debug.png)

  Visual Studio Code ouvre un nouveau navigateur web avec Microsoft 365 Copilot Chat. Si vous y êtes invité, connectez-vous à votre compte Microsoft 365.

Dans un navigateur web :

1. Dans le panneau latéral, sélectionnez l’agent **da-repairs-keylocal**.

  ![Capture d’écran de l’agent personnalisé affiché dans Microsoft 365 Copilot.](../media/LAB_05/3-copilot-agent-sidebar.png)

1. Tapez `What repairs are assigned to Karin?` dans la zone de texte d’invite, puis envoyez l’invite.
1. Vérifiez que vous souhaitez envoyer des données au plug-in d’API à l’aide du bouton **Toujours autoriser**.

  ![Capture d’écran de l’invite pour autoriser l’envoi de données à l’API.](../media/LAB_05/3-allow-data.png)

1. Attendez que l’agent réponde.

  ![Capture d’écran de la réponse de l’agent personnalisé à l’invite de l’utilisateur.](../media/LAB_05/3-copilot-response.png)

Arrêtez la session de débogage dans Visual Studio Code lorsque vous avez terminé le test.