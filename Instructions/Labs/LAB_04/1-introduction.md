---
lab:
  title: Présentation
  module: 'LAB 04: Add custom knowledge to declarative agents using Microsoft Graph connectors and Visual Studio Code'
---

# Présentation

Les agents Microsoft 365 Copilot vous permettent de créer des assistants basés sur l’intelligence artificielle optimisés pour des scénarios spécifiques. À l’aide d’instructions, vous définissez le contexte du copilote et configurez le ton de sa voix ou la façon dont il doit répondre. En configurant les connaissances de l’agent, vous lui donnez accès à des données externes qui ne font pas partie du modèle LLM (grand modèle de langage), afin qu’il puisse répondre plus précisément. 

## Exemple de scénario

Supposons que vous travaillez dans un service informatique d’une grande organisation. Votre organisation normalise l’informatique par le biais de différentes stratégies qu’elle stocke dans un système spécialisé. Vous et vos collègues du service informatique recevez régulièrement des questions qui sont abordées dans les stratégies. La recherche de réponses dans le système de gestion des stratégies prend beaucoup de temps. Vous souhaitez fournir à votre organisation un assistant alimenté par l’IA capable de répondre aux questions de vos collègues à l’aide d’informations faisant autorité issues des stratégies.

## Objectifs d’apprentissage

À la fin de ce module, vous pourrez créer des agents déclaratifs pour Microsoft 365 Copilot. Vous allez comprendre comment configurer leurs instructions afin de les optimiser pour un scénario spécifique. Vous allez également comprendre comment les intégrer aux connecteurs Microsoft Graph pour leur donner accès à des données externes, qui ne font pas partie du LLM de Microsoft 365 Copilot.

## Prérequis

- Connaître Microsoft 365 Copilot et son fonctionnement, niveau débutant
- Savoir créer un agent déclaratif Microsoft 365 Copilot
- Savoir créer un connecteur Graph
- Client Microsoft 365 avec Microsoft 365 Copilot et privilèges d’administrateur client
- [Visual Studio Code](https://code.visualstudio.com/) avec l’extension [Teams Toolkit](https://marketplace.visualstudio.com/items?itemName=TeamsDevApp.ms-teams-vscode-extension) installée
- [Azure Functions Core Tools](https://learn.microsoft.com/azure/azure-functions/functions-run-local#install-the-azure-functions-core-tools)
- [Node.js v18](https://nodejs.org/)
