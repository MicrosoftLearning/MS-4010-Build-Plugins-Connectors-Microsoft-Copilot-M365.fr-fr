---
lab:
  title: "Exercice\_1\_: créer une connexion externe pour le connecteur Graph"
  module: 'LAB 04: Add custom knowledge to declarative agents using Microsoft Graph connectors and Visual Studio Code'
---

# Exercice 1 : créer une connexion externe pour le connecteur Graph

L’extension d’un agent déclaratif avec des connaissances lui donne accès à des informations supplémentaires qui ne font pas partie de son grand modèle de langage. À l’aide de connecteurs Graph, vous pouvez ingérer des données externes dans Microsoft 365, où elles sont disponibles pour différentes expériences utilisateur, notamment Microsoft 365 Copilot. Lors de la configuration des paramètres de connaissances d’un agent Copilot, vous pouvez l’intégrer à une connexion externe créée par un connecteur Graph.

### Durée de l’exercice

- **Durée estimée :** 10 minutes

## Tâche 1 : télécharger un exemple de projet et se connecter aux ressources

Lorsque vous intégrez un agent Copilot à un connecteur Graph, vous devez spécifier l’ID de la connexion externe que le connecteur a créée. En règle générale, vous déployez des connecteurs Graph séparément des agents Copilot. Pour effectuer cet exercice, déployez un connecteur Graph existant que vous référencez dans les étapes ultérieures.

Commencez par télécharger l’exemple de projet de connecteur Graph.

1. Dans un navigateur web, accédez à [https://aka.ms/learn-gc-ts-policies](https://aka.ms/learn-gc-ts-policies). Vous obtenez une invite pour télécharger un fichier ZIP avec l’exemple de projet.
1. Enregistrez le fichier zip sur votre ordinateur.
    1. Créez un dossier dans **Documents**.
    1. Extrayez le contenu du fichier zip téléchargé dans le dossier que vous venez de créer.
    1. Ouvrez le dossier  dans Visual Studio Code.

Dans Visual Studio Code :

1. Dans le menu Fichier, choisissez l’option **Ouvrir le dossier…**
1. Ouvrez le dossier de projet que vous venez d’extraire dans votre **dossier Documents**.
1. Dans la **barre d’activité** (barre latérale), ouvrez l’extension **Teams Toolkit**.
1. Dans le volet **Comptes**, vérifiez que vous êtes connecté à votre **locataire Microsoft 365**.
1. Dans le volet **Comptes**, vérifiez que vous êtes connecté à votre **abonnement Azure**.

    ![Capture d’écran de Teams Toolkit montrant les comptes connectés.](../media/LAB_04/3-teams-toolkit-accounts.png)

> [!NOTE]
> Si vous n’avez pas de licence Microsoft 365 Copilot complète, vous pouvez voir « Accès Copilot désactivé ». Les exercices peuvent toujours être effectués, bien que vous ne puissiez pas tester entièrement l’agent dans Microsoft 365 Copilot Chat.

## Tâche 2 : exécuter un projet et créer une connexion à Microsoft 365

1. Démarrez le projet en appuyant sur <kbd>F5</kbd>. Teams Toolkit crée une inscription d’application Microsoft Entra dans votre locataire qui permet au connecteur Graph de communiquer avec votre locataire Microsoft 365. Teams Toolkit démarre également la fonction Azure déclenchée par le minuteur qui héberge le connecteur Graph.

> [!IMPORTANT]
> Cette étape peut prendre jusqu’à 10 minutes ou plus, ne fermez pas tant que vous n’avez pas terminé l’exercice.

1. Avant que le connecteur Graph puisse s’exécuter, vous devez donner votre consentement aux autorisations dont l’application Entra a besoin. Pour accorder le consentement, utilisez les instructions du volet **Terminal** associé à la tâche de **démarrage de l’hôte func:**.

    ![Capture d’écran de Visual Studio Code montrant le message de consentement des autorisations.](../media/LAB_04/3-consent-message.png)

1. Ouvrez l’URL de consentement dans un navigateur web. Connectez-vous avec votre compte professionnel qui appartient à votre locataire Microsoft 365. Accordez les autorisations requises à l’application à l’aide du bouton **Accorder le consentement administrateur**.

    ![Capture d’écran du portail Microsoft Entra ID dans lequel un utilisateur peut accorder son consentement.](../media/LAB_04/3-consent-microsoft-entra-id.png)

1. Une fois que vous accordez le consentement aux autorisations requises, le connecteur Graph continue. Dans le volet **Terminal**, notez la sortie du connecteur Graph. Le connecteur Graph crée une connexion externe, configure le schéma et ingère l’exemple de contenu sur votre locataire Microsoft 365.
1. L’exécution du connecteur prend 5 à 10 minutes. Une fois terminé, arrêtez le débogage en appuyant sur le bouton **Arrêter** dans la barre d’outils de débogage.

    ![Capture d’écran du terminal Visual Studio Code avec sortie du connecteur Graph.](../media/LAB_04/3-connector-done.png)
