---
lab:
  title: "Exercice\_4\_- Explorer le code source du plug-in"
  module: 'LAB 02: Build your own message extension plugin with TypeScript (TS) for Microsoft 365 Copilot'
---

# Exercice 4 - Explorer le code source du plug-in

Dans cet exercice, vous allez passer en revue le code d’application afin de comprendre le fonctionnement d’une **extension de message**.

## Tâche 1 - Examiner le manifeste

Le cœur de toute application Microsoft 365 est son manifeste de l’application. C’est là que vous fournissez les informations dont Microsoft 365 a besoin pour accéder à votre application.

Dans votre **répertoire de travail**, ouvrez le fichier **appPackackage/manifest.json**. Ce fichier JSON est placé dans une archive zip avec deux fichiers d’icônes pour créer le package d’application. La propriété **icons** inclut des chemins d’accès à ces icônes.

```json
"icons": {
    "color": "Northwind-Logo3-192-${{TEAMSFX_ENV}}.png",
    "outline": "Northwind-Logo3-32.png"
},
```

Notez le jeton `${{TEAMSFX_ENV}}` dans l’un des noms d’icônes. Teams Toolkit remplacera ce jeton par le nom de votre environnement, tel que **local** ou **dev** (pour un déploiement Azure en cours de développement). Ainsi, la couleur de l’icône change en fonction de l’environnement.

### Description de l’application

Examinez maintenant le **nom** et la **description**. Notez que la **description** est relativement longue ! Cela est important pour que les utilisateurs et Copilot puissent apprendre ce que fait votre application et quand l’utiliser.

```json
    "name": {
        "short": "Northwind Inventory",
        "full": "Northwind Inventory App"
    },
    "description": {
        "short": "App allows you to find and update product inventory information",
        "full": "Northwind Inventory is the ultimate tool for managing your product inventory. With its intuitive interface and powerful features, you'll be able to easily find your products by name, category, inventory status, and supplier city. You can also update inventory information with the app. \n\n **Why Choose Northwind Inventory:** \n\n Northwind Inventory is the perfect solution for businesses of all sizes that need to keep track of their inventory. Whether you're a small business owner or a large corporation, Northwind Inventory can help you stay on top of your inventory management needs. \n\n **Features and Benefits:** \n\n - Easy Product Search through Microsoft 365 Copilot. Simply start by saying, 'Find northwind dairy products that are low on stock' \r - Real-Time Inventory Updates: Keep track of inventory levels in real-time and update them as needed \r  - User-Friendly Interface: Northwind Inventory's intuitive interface makes it easy to navigate and use \n\n **Availability:** \n\n To use Northwind Inventory, you'll need an active Microsoft 365 account . Ensure that your administrator enables the app for your Microsoft 365 account."
    },
```

### Définition du bot

Faites défiler un peu vers le bas jusqu’à **composerExtensions**. L’extension compose est le nom historique de l’extension de message ; c’est là que les extensions de message de l’application sont définies. Les extensions de message communiquent à l’aide d’Azure Bot Framework ; cela fournit un canal de communication rapide et sécurisé entre Microsoft 365 et votre application. Lors de la première exécution de votre projet, Teams Toolkit a inscrit un bot et placera son **botID** ici.

```json
    "composeExtensions": [
        {
            "botId": "${{BOT_ID}}",
            "commands": [
                {
                    ...
```

### Définitions de commande

Cette extension de message comprend deux commandes, qui sont définies dans le tableau `commands`. Si vous avez terminé l’exercice précédent, il y aura également une troisième commande pour rechercher par nom de société. Nous allons ignorer la première commande pour le moment, car c’est la plus complexe. La commande suivante permet à Copilot (ou à un utilisateur) de rechercher des produits à prix réduits dans une catégorie Northwind. Cette commande accepte un seul paramètre, **categoryName**.

```json
{
    "id": "discountSearch",
    "context": [
        "compose",
        "commandBox"
    ],
    "description": "Search for discounted products by category",
    "title": "Discounts",
    "type": "query",
    "parameters": [
        {
            "name": "categoryName",
            "title": "Category name",
            "description": "Enter the category to find discounted products",
            "inputType": "text"
        }
    ]
},
```

Revenons maintenant à la première commande, **inventorySearch**, qui comprend 5 paramètres, ce qui permet d’obtenir des requêtes beaucoup plus sophistiquées.

```json
{
    "id": "inventorySearch",
    "context": [
        "compose",
        "commandBox"
    ],
    "description": "Search products by name, category, inventory status, supplier location, stock level",
    "title": "Product inventory",
    "type": "query",
    "parameters": [
        {
            "name": "productName",
            "title": "Product name",
            "description": "Enter a product name here",
            "inputType": "text"
        },
        {
            "name": "categoryName",
            "title": "Category name",
            "description": "Enter the category of the product",
            "inputType": "text"
        },
        {
            "name": "inventoryStatus",
            "title": "Inventory status",
            "description": "Enter what status of the product inventory. Possible values are 'in stock', 'low stock', 'on order', or 'out of stock'",
            "inputType": "text"
        },
        {
            "name": "supplierCity",
            "title": "Supplier city",
            "description": "Enter the supplier city of product",
            "inputType": "text"
        },
        {
            "name": "stockQuery",
            "title": "Stock level",
            "description": "Enter a range of integers such as 0-42 or 100- (for >100 items). Only use if you need an exact numeric range.",
            "inputType": "text"
        }
    ]
},
```

Copilot est en mesure de les remplir en fonction des descriptions et l’extension de message retournera une liste de produits filtrés par tous les paramètres non vides.

## Tâche 2 - Examiner le code du bot

Ouvrez maintenant le fichier **src/searchApp.ts**, qui contient le code du bot qui utilise le [Kit de développement logiciel (SDK) Bot Builder](https://learn.microsoft.com/azure/bot-service/index-bf-sdk) pour communiquer avec Azure Bot Framework. Notez que le bot étend une classe de Kit de développement logiciel (SDK) **TeamsActivityHandler**.

```typescript
export class SearchApp extends TeamsActivityHandler {
  constructor() {
    super();
  }

  ...
```

### Requête d’extension de message

L’application est en mesure de gérer les messages (appelés **activités**) provenant de Microsoft 365 en remplaçant les méthodes de **TeamsActivityHandler**.

La première d’entre elles est une activité **Requête d’extension de message**. Cette fonction est appelée lorsqu’un utilisateur tape dans une extension de message ou lorsque Copilot l’appelle. Le gestionnaire distribue la requête en fonction du **commandID**. Il s’agit du même commandID que celui utilisé dans le manifeste de l’application.

```typescript
  // Handle search message extension
  public async handleTeamsMessagingExtensionQuery(
    context: TurnContext,
    query: MessagingExtensionQuery
  ): Promise<MessagingExtensionResponse> {

    switch (query.commandId) {
      case productSearchCommand.COMMAND_ID: {
        return productSearchCommand.handleTeamsMessagingExtensionQuery(context, query);
      }
      case discountedSearchCommand.COMMAND_ID: {
        return discountedSearchCommand.handleTeamsMessagingExtensionQuery(context, query);
      }
    }
  }
  ...
```

### Actions de carte adaptative

L’autre type d’activités que notre application doit gérer sont les actions de carte adaptative, par exemple lorsqu’un utilisateur sélectionne **Mettre à jour le stock** ou **Réorganiser** sur une carte adaptative. Étant donné qu’il n’existe pas de méthode spécifique pour une action de carte adaptative, le code remplace `onInvokeActivity()`, qui est une classe d’activité beaucoup plus large qui inclut des requêtes d’extension de message. Pour cette raison, le code vérifie manuellement le nom de l’activité et transmet au gestionnaire approprié. Si le nom d’activité n’est pas destiné à une action de carte adaptative, la clause `else` exécute l’implémentation de base de `onInvokeActivity()`, laquelle, entre autres choses, appellera notre méthode `handleTeamsMessagingExtensionQuery()` si l’activité **Appeler** est une requête.

```typescript
import {
  TeamsActivityHandler,
  TurnContext,
  MessagingExtensionQuery,
  MessagingExtensionResponse,
  InvokeResponse
} from "botbuilder";
import productSearchCommand from "./messageExtensions/productSearchCommand";
import discountedSearchCommand from "./messageExtensions/discountSearchCommand";
import revenueSearchCommand from "./messageExtensions/revenueSearchCommand";
import actionHandler from "./adaptiveCards/cardHandler";

export class SearchApp extends TeamsActivityHandler {
  constructor() {
    super();
  }

  // Handle search message extension
  public async handleTeamsMessagingExtensionQuery(
    context: TurnContext,
    query: MessagingExtensionQuery
  ): Promise<MessagingExtensionResponse> {

    switch (query.commandId) {
      case productSearchCommand.COMMAND_ID: {
        return productSearchCommand.handleTeamsMessagingExtensionQuery(context, query);
      }
      case discountedSearchCommand.COMMAND_ID: {
        return discountedSearchCommand.handleTeamsMessagingExtensionQuery(context, query);
      }
    }

  }

  // Handle adaptive card actions
  public async onInvokeActivity(context: TurnContext): Promise<InvokeResponse> {
    let runEvents = true;
    // console.log (`🎬 Invoke activity received: ${context.activity.name}`);
    try {
      if(context.activity.name==='adaptiveCard/action'){
        switch (context.activity.value.action.verb) {
          case 'ok': {
            return actionHandler.handleTeamsCardActionUpdateStock(context);
          }
          case 'restock': {
            return actionHandler.handleTeamsCardActionRestock(context);
          }
          case 'cancel': {
            return actionHandler.handleTeamsCardActionCancelRestock(context);
          }
          default:
            runEvents = false;
            return super.onInvokeActivity(context);
        }
      } else {
          runEvents = false;
          return super.onInvokeActivity(context);
      }
    } 
```

## Tâche 3 - Examiner le code de commande d’extension de message

Dans le but de rendre le code plus modulaire, lisible et réutilisable, chaque commande d’extension de message est placée dans son propre module TypeScript. Consultez **src/messageExtensions/discountSearchCommand.ts** pour obtenir un exemple.

Observez tout d’abord que le module exporte une constante `COMMAND_ID`, qui contient le même **commandID** que celui trouvé dans le manifeste de l’application, et permet à l’instruction switch dans **searchApp.ts** de fonctionner correctement.

Il fournit ensuite une fonction, `handleTeamsMessagingExtensionQuery()`, pour gérer les requêtes entrantes de **produits à prix réduits par catégorie**.

```typescript
async function handleTeamsMessagingExtensionQuery(
    context: TurnContext,
    query: MessagingExtensionQuery
): Promise<MessagingExtensionResponse> {

    // Seek the parameter by name, don't assume it's in element 0 of the array
    let categoryName = cleanupParam(query.parameters.find((element) => element.name === "categoryName")?.value);
    console.log(`💰 Discount query #${++queryCount}: Discounted products with categoryName=${categoryName}`);

    const products = await getDiscountedProductsByCategory(categoryName);

    console.log(`Found ${products.length} products in the Northwind database`)
    const attachments = [];
    products.forEach((product) => {
        const preview = CardFactory.heroCard(product.ProductName,
            `Avg discount ${product.AverageDiscount}%<br />Supplied by ${product.SupplierName} of ${product.SupplierCity}`,
            [product.ImageUrl]);

        const resultCard = cardHandler.getEditCard(product);
        const attachment = { ...resultCard, preview };
        attachments.push(attachment);
    });
    return {
        composeExtension: {
            type: "result",
            attachmentLayout: "list",
            attachments: attachments,
        },
    };
}
```

Notez que l’index du tableau `query.parameters` peut ne pas correspondre à la position du paramètre dans le manifeste. Bien qu’il s’agisse généralement d’un problème de commande à plusieurs paramètres uniquement, le code obtient toujours la valeur en fonction du nom du paramètre plutôt que du codage en dur d’un index.

Après avoir nettoyé le paramètre (en le réglant et en partant du fait que Copilot suppose parfois que « **\***  » est un caractère générique correspondant à tout), le code appelle la couche d’accès aux données Northwind sur `getDiscountedProductsByCategory()`.

Il itère ensuite dans les produits et crée deux cartes pour chacun d’entre eux :

- Une carte d’_aperçu_, qui est implémentée en tant que carte de **bannière**. Celle-ci s’affiche dans les résultats de la recherche dans l’interface utilisateur et dans certaines citations dans Copilot.

- Une carte de _résultat_, qui est implémentée en tant que carte **adaptative** qui inclut tous les détails.

Dans la tâche suivante, nous allons passer en revue le code de carte adaptative et consulter le concepteur de carte adaptative.

## Tâche 4 - Examiner les cartes adaptatives et le code associé

Les cartes adaptatives du projet se trouvent dans le dossier **src/adaptiveCards/**. Il existe 3 cartes, chacune implémentée en tant que fichier JSON.

- **editCard.json** : il s’agit de la carte initiale affichée par l’extension de message ou une référence Copilot.

- **successCard.json** : lorsqu’un utilisateur effectue une action, cette carte s’affiche pour indiquer la réussite. Elle est quasiment identique à la carte d’édition, sauf qu’elle inclut un message à l’utilisateur.

- **errorCard.json** : en cas d’échec d’une action, cette carte s’affiche.

Examinons la carte d’édition dans le **Concepteur de carte adaptative**. Ouvrez votre navigateur web à l’adresse [https://adaptivecards.io](https://adaptivecards.io) et sélectionnez l’option **Concepteur** en haut.

![Capture d’écran du concepteur de carte adaptative.](../media/5-01-adaptive-card-designer-01.png)

Notez les expressions de liaison de données telles que `"text": "📦 ${productName}",`. Cela lie la propriété `productName` dans les données au texte de la carte.

Sélectionnez maintenant **Microsoft Teams** comme application hôte 1️. Collez tout le contenu de **editCard.json** dans l’éditeur de charge utile de carte 2️ et le contenu de **sampleData.json** dans l’exemple d’éditeur de données 3️. L’exemple de données est identique à un produit tel qu’il est fourni dans le code. Vous devez voir la carte comme affichée, à l’exception d’une petite erreur qui survient en raison de l’incapacité du concepteur à afficher l’un des formats de carte adaptative.

![Capture d’écran de Copilot de la carte adaptative affichée en fonction du JSON.](../media/5-01-adaptive-card-designer-02.png)

En haut de la page, essayez de modifier le **thème** et l’**appareil émulé** pour voir à quoi la carte ressemblerait avec un thème sombre ou sur un appareil mobile. Il s’agit de l’outil utilisé pour créer des cartes adaptatives pour l’exemple d’application.

Revenez maintenant dans Visual Studio Code et ouvrez **cardHandler.ts**. La fonction `getEditCard()` est appelée à partir de chacune des commandes d’extension de message pour obtenir une carte de **résultat**. Le code lit le JSON de carte adaptative, qui est considéré comme un modèle, puis le lie aux données de produit. Le résultat est plus que du JSON : la même carte que le modèle, avec toutes les expressions de liaison de données renseignées. Enfin, le module `CardFactory` est utilisé pour convertir le JSON final en objet de carte adaptative pour le rendu.

```typescript
function getEditCard(product: ProductEx): any {

    var template = new ACData.Template(editCard);
    var card = template.expand({
        $root: {
            productName: product.ProductName,
            unitsInStock: product.UnitsInStock,
            productId: product.ProductID,
            categoryId: product.CategoryID,
            imageUrl: product.ImageUrl,
            supplierName: product.SupplierName,
            supplierCity: product.SupplierCity,
            categoryName: product.CategoryName,
            inventoryStatus: product.InventoryStatus,
            unitPrice: product.UnitPrice,
            quantityPerUnit: product.QuantityPerUnit,
            unitsOnOrder: product.UnitsOnOrder,
            reorderLevel: product.ReorderLevel,
            unitSales: product.UnitSales,
            inventoryValue: product.InventoryValue,
            revenue: product.Revenue,
            averageDiscount: product.AverageDiscount
        }
    });
    return CardFactory.adaptiveCard(card);
}
```

En faisant défiler vers le bas, vous verrez le gestionnaire de chacun des boutons d’action de la carte. La carte envoie des données lorsqu’un bouton d’action est actionné, et plus particulièrement `data.txtStock`, qui est la zone d’entrée **Quantity** sur la carte, et `data.productId`, qui est envoyé dans chaque action de carte pour indiquer au code quel produit mettre à jour.

```typescript
async function handleTeamsCardActionUpdateStock(context: TurnContext) {

    const request = context.activity.value;
    const data = request.action.data;
    console.log(`🎬 Handling update stock action, quantity=${data.txtStock}`);

    if (data.txtStock && data.productId) {

        const product = await getProductEx(data.productId);
        product.UnitsInStock = Number(data.txtStock);
        await updateProduct(product);

        var template = new ACData.Template(successCard);
        var card = template.expand({
            $root: {
                productName: product.ProductName,
                unitsInStock: product.UnitsInStock,
                productId: product.ProductID,
                categoryId: product.CategoryID,
                imageUrl: product.ImageUrl,
                ...
```

Comme vous pouvez le voir, le code obtient ces deux valeurs, met à jour la base de données, puis envoie une nouvelle carte qui contient un message et les données mises à jour.

## Félicitations

Vous avez terminé l’exercice 5 et le labo de plug-in Extensions de message Microsoft 365 Copilot. Merci beaucoup d’avoir effectué ces labos !

[Passez au résumé du labo...](./7-summary.md)
