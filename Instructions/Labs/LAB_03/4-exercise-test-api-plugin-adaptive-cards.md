---
lab:
  title: "Exercice\_3\_: tester l’agent déclaratif avec le plug-in d’API dans Microsoft\_365 Copilot"
  module: 'LAB 03: Use Adaptive Cards to show data in API plugins for declarative agents'
---

# Exercice 3 : tester l’agent déclaratif avec le plug-in d’API dans Microsoft 365 Copilot

La dernière étape consiste à tester l’agent déclaratif avec le plug-in API dans Microsoft 365 Copilot.

### Durée de l’exercice

- **Durée estimée :** 10 minutes

## Tâche 1 : configurer et démarrer le débogage

Dans Visual Studio Code :

1. Dans la **barre d’activité**, sélectionnez **Teams Toolkit**.
1. Dans la section **Comptes**, vérifiez que vous êtes connecté à votre locataire Microsoft 365 avec Microsoft 365 Copilot.

    ![Capture d’écran de la section des comptes Teams Toolkit dans Visual Studio Code.](../media/LAB_03/LAB_03/3-teams-toolkit-accounts.png)

1. Dans la **barre d’activité**, sélectionnez **Exécuter et déboguer**.
1. Sélectionnez la configuration **Débogage dans Copilot** et démarrez le débogage à l’aide du bouton **Démarrer le débogage**.  

    ![Capture d’écran de la configuration Débogage dans Copilot dans Visual Studio Code.](../media/LAB_03/LAB_03/3-visual-studio-code-start-debugging.png)

1. Visual Studio Code génère et déploie votre projet sur votre locataire Microsoft 365 et ouvre une nouvelle fenêtre de navigateur web.

## Tâche 2 : tester et examiner les résultats

Dans un navigateur web :

1. Lorsque vous y êtes invité, connectez-vous avec le compte qui appartient à votre locataire Microsoft 365 avec Microsoft 365 Copilot.
1. Dans la barre latérale, sélectionnez **Il Ristorante**.

    ![Capture d’écran de l’interface Microsoft 365 Copilot avec l’agent Il Ristorante sélectionné.](../media/LAB_03/LAB_03/3-copilot-select-agent.png)

1. Choisissez l’amorce de conversation **Qu’est-ce qu’on mange aujourd’hui ?** et envoyez l’invite.

    ![Capture d’écran de l’interface Microsoft 365 Copilot avec l’invite relative au repas.](../media/LAB_03/LAB_03/3-copilot-lunch-prompt.png)

1. Lorsque vous y êtes invité, examinez les données envoyées par l’agent à l’API et confirmez à l’aide du bouton **Autoriser une fois**.

    ![Capture d’écran de l’interface Microsoft 365 Copilot avec la confirmation relative au repas.](../media/LAB_03/LAB_03/3-copilot-lunch-confirm.png)

1. Attendez que l’agent réponde. Remarquez que la fenêtre contextuelle d’une citation comprend maintenant votre carte adaptative personnalisée avec des informations supplémentaires provenant de l’API.

    ![Capture d’écran de l’interface Microsoft 365 Copilot avec la réponse relative au repas.](../media/LAB_03/LAB_03/3-copilot-lunch-response.png)

1. Passez une commande en entrant dans la zone de texte d’invite **1x spaghetti, 1x thé glacé** et envoyez l’invite.
1. Examinez les données envoyées par l’agent à l’API et continuez à l’aide du bouton **Confirmer**.

    ![Capture d’écran de l’interface Microsoft 365 Copilot avec la confirmation de la commande.](../media/LAB_03/LAB_03/3-copilot-order-confirm.png)

1. Attendez que l’agent passe la commande et retourne le résumé de la commande. Notez que, étant donné que l’API retourne un seul élément, l’agent l’affiche à l’aide d’une carte adaptative et inclut la carte directement dans sa réponse.

    ![Capture d’écran de l’interface Microsoft 365 Copilot avec la réponse de la commande.](../media/LAB_03/LAB_03/3-copilot-order-response.png)

1. Revenez à Visual Studio Code et arrêtez le débogage.
1. Basculez vers l’onglet **Terminal** et fermez tous les terminaux actifs.

    ![Capture d’écran de l’onglet Terminal de Visual Studio Code avec l’option permettant de fermer tous les terminaux.](../media/LAB_03/LAB_03/3-visual-studio-code-close-terminal.png)
