---
lab:
  title: Préparer votre environnement de développement
  module: 'LAB 03: Build your own message extension plugin with TypeScript (TS) for Microsoft Copilot'
---

# Préparer votre environnement de développement

Préparons tout d’abord votre environnement de développement, vos comptes et vos logiciels. Avant de commencer, vous devez effectuer les tâches suivantes.

## Tâche 1 - Installer les prérequis

> [!IMPORTANT]
> Pour mener ce projet à bien, vous aurez besoin d’un compte Microsoft 365 avec l’autorisation de charger des applications. Pour effectuer l’**Exercice 2**, le compte doit également être concédé sous licence pour Microsoft Copilot pour Microsoft 365.

Si vous utilisez un nouveau locataire, il est judicieux de vous connecter à la [page de Microsoft 365](https://office.com) à l’adresse [https://office.com](https://office.com) avant de commencer. Selon la configuration du locataire, vous pouvez être invité à configurer l’authentification multifacteur. Vérifiez que vous avez accès à Microsoft Teams et à Microsoft Outlook avant de continuer.

Les outils suivants ont déjà été installés dans le labo sur **MS-4010-DEVBOX**. Vérifiez qu’ils sont installés et fonctionnels :

1. [Visual Studio Code](https://code.visualstudio.com/) (dernière version)

1. [Explorateur Stockage Azure](https://azure.microsoft.com/products/storage/storage-explorer/) : téléchargez-le si vous souhaitez afficher et modifier la base de données Northwind utilisée dans cet exemple.

## Tâche 2 - Installer nvm-windows

Vous utiliserez cet outil pour installer Node.js et éventuellement changer les versions de Node en fonction des besoins de vos projets.

1. Dans un navigateur web, accédez à [https://github.com/coreybutler/nvm-windows/releases](https://github.com/coreybutler/nvm-windows/releases).
2. Recherchez la dernière version de mise en production et sélectionnez le fichier **nvm-setup.zip** à télécharger.  Le fichier va être téléchargé sur votre machine.
3. Ouvrez le dossier de fichiers et **extrayez** le contenu du dossier zip dans un dossier sur votre machine.
4. Dans le nouveau dossier, sélectionnez **nvm-setup.exe** pour ouvrir le fichier d’installation.
5. Suivez les invites du programme d’installation pour installer l’outil en utilisant les options par défaut.
6. Nvm pour Windows sera installé sur votre machine.

## Tâche 3 - Installer Node.js

Installez Node.js version 18.18.2 qui est compatible avec toutes les solutions de ce cours.

1. Ouvrez l’application **Invite de commandes**.
2. Entrez la commande `nvm install 18.18` pour installer Node.js.
3. La sortie nvm doit confirmer que la fin de l’installation.
4. Exécutez la commande `nvm use 18.18` pour utiliser cette version de Node.js.
5. Exécutez la commande `node -v` pour confirmer l’installation de la version 18.18.2.

Vous avez maintenant installé et configuré Node.js version 18.18.2.

## Tâche 4 - Télécharger l’exemple de code

[Clonez](https://github.com/OfficeDev/Copilot-for-M365-Plugins-Samples.git) ou [téléchargez](https://github.com/OfficeDev/Copilot-for-M365-Plugins-Samples.git) le référentiel d’exemples : [https://github.com/OfficeDev/Copilot-for-M365-Plugins-Samples/](https://github.com/OfficeDev/Copilot-for-M365-Plugins-Samples/).

Dans le référentiel cloné ou téléchargé, accédez au dossier **samples/msgext-northwind-inventory-ts**. Ces labos y font référence comme votre « **dossier de travail** », car c’est là que vous allez travailler.

## Tâche 3 - Copier des exemples de documents dans OneDrive

L’exemple d’application inclut certains documents auxquels Copilot doit se référer pendant les labos. Dans cette tâche, vous allez copier ces fichiers dans le OneDrive de votre utilisateur afin que Copilot puisse les trouver. Selon la configuration du locataire, vous pouvez être invité à configurer l’authentification multifacteur dans le cadre de ce processus.

1. Ouvrez votre navigateur web et accédez à Microsoft 365 ([https://www.office.com/](https://www.office.com/)). Connectez-vous à l’aide du compte Microsoft 365 que vous utiliserez tout au long du labo. Vous pouvez être invité à configurer l’authentification multifacteur.

1. À l’aide du menu des applications dans le coin supérieur gauche de la page 1️⃣, accédez à l’application OneDrive dans Microsoft 365 2️⃣.

    ![Capture d’écran de la navigation vers l’application OneDrive dans Microsoft 365.](../media/1-02-copy-sample-files-01.png)

1. Dans OneDrive, accédez à **Mes fichiers** 1️⃣. Si un dossier de documents existe, accédez-y également. Si ce n’est pas le cas, vous pouvez travailler directement dans l’emplacement **Mes fichiers**.

    ![Capture d’écran de la navigation vers vos documents dans OneDrive.](../media/1-02-copy-sample-files-02.png)

1. Sélectionnez maintenant **Ajouter nouveau** 1️ et **Dossier** 2️⃣ pour créer un dossier.

    ![Capture d’écran de l’ajout d’un nouveau dossier dans OneDrive.](../media/1-02-copy-sample-files-03.png)

1. Nommez le dossier **Northwind contracts** et sélectionnez **Créer**.

    ![Capture d’écran de l’affectation d’un nom au nouveau dossier « Northwind contracts ».](../media/1-02-copy-sample-files-03-b.png)

1. À présent, dans ce nouveau dossier, sélectionnez à nouveau **Ajouter nouveau 1️** mais sélectionnez **Chargement de fichiers** 2️ cette fois-ci.

    ![Capture d’écran de l’ajout de nouveaux fichiers au nouveau dossier.](../media/1-02-copy-sample-files-04.png)

1. Accédez maintenant au dossier **sampleDocs** dans votre **dossier de travail**. Mettez en surbrillance tous les fichiers 1️⃣ et sélectionnez **OK** 2️⃣ pour tous les charger.

    ![Capture d’écran du chargement des exemples de fichiers de ce référentiel dans le dossier.](../media/1-02-copy-sample-files-05.png)

En anticipant cette tâche, il y a de grandes chances que moteur de recherche Microsoft 365 les aura découverts lorsque vous serez prêts à les utiliser.

## Tâche 4 - Installer et configurer Teams Toolkit pour Visual Studio Code

Dans cette tâche, vous allez installer la version actuelle de [Teams Toolkit pour Visual Studio Code](https://learn.microsoft.com/microsoftteams/platform/toolkit/teams-toolkit-fundamentals?pivots=visual-studio-code-v5). La méthode la plus simple consiste à le faire à partir de Visual Studio Code.

> [!NOTE]
> N’installez pas la version préliminaire, car elle n’a pas été testée avec ce labo.

1. Ouvrez votre **dossier de travail** dans Visual Studio Code. Vous pouvez être invité à approuver les auteurs de ce dossier. Si c’est le cas, faites-le.

1. Dans la barre latérale gauche, sélectionnez **Extensions** 1️⃣. Entrez le mot **teams**dans la zone de recherche 2️ et recherchez **Teams Toolkit** dans les résultats de la recherche. Sélectionnez **Installer** 3️⃣.

    ![Capture d’écran de l’installation de Teams Toolkit dans Visual Studio Code.](../media/1-04-install-teams-toolkit-01.png)

1. Sélectionnez maintenant l’icône de **Teams Toolkit** à gauche 1️⃣. Si des options de création de projet sont proposées, vous êtes probablement dans le mauvais dossier. Dans le **menu Fichier de Visual Studio Code**, sélectionnez **Ouvrir le dossier** et ouvrez directement le dossier **msgext-northwind-inventory-ts**. Vous devez voir les sections des comptes, de l’environnement, etc. comme indiqué ci-dessous.

1. Sous **Comptes**, sélectionnez **Se connecter à Microsoft 365** 2️⃣ et connectez-vous avec votre compte Microsoft 365.

    ![Capture d’écran de la connexion à Microsoft 365 à partir de Teams Toolkit.](../media/1-04-setup-teams-toolkit-01.png)

1. Une fenêtre de navigateur s’ouvre et propose de se connecter à Microsoft 365. Lorsqu’il affiche **Vous êtes maintenant connecté, fermez cette page**, faites-le.

1. Enfin, vérifiez qu’une coche verte apparaît en regard de **Chargement indépendant activé**. Si ce n’est pas le cas, cela signifie que votre compte d’utilisateur n’est pas autorisé à charger des applications Teams. Cette autorisation est « désactivée » par défaut. Vous trouverez ici des [instructions pour permettre aux utilisateurs de charger des applications personnalisées](https://learn.microsoft.com/microsoftteams/teams-custom-app-policies-and-settings#allow-users-to-upload-custom-apps).

    ![Capture d’écran de la vérification de l’activation du chargement indépendant.](../media/1-04-setup-teams-toolkit-03.png)

> [!NOTE]
> Le Programme pour les développeurs Microsoft 365 n’inclut pas de licences Copilot pour Microsoft 365. Par conséquent, si vous décidez d’utiliser un locataire de développeur, vous ne pourrez tester l’exemple qu’en tant qu’extension de message.

## Vérifier votre travail

Après avoir effectué toutes les tâches ci-dessus, vous devez avoir installé et téléchargé les éléments suivants sur votre ordinateur :

- [Visual Studio Code](https://code.visualstudio.com/) (dernière version)

- [Node.js version 18.x](https://nodejs.org/download/release/v18.18.2/)

- [Explorateur Stockage Azure (facultatif)](https://azure.microsoft.com/products/storage/storage-explorer/)

- [Teams Toolkit pour Visual Studio Code](https://learn.microsoft.com/microsoftteams/platform/toolkit/teams-toolkit-fundamentals?pivots=visual-studio-code-v5)

- Exemple de référentiel : [https://github.com/OfficeDev/Copilot-for-M365-Plugins-Samples/](https://github.com/OfficeDev/Copilot-for-M365-Plugins-Samples/)

Si tout a été préparé correctement, vous êtes maintenant prêt à exécuter l’exemple d’application en tant qu’extension de message. 

[Passez à l’exercice suivant…](./3-exercise-1-run-message-extension.md)