---
title: Online gehostete Anweisungen
permalink: index.html
layout: home
---

# Azure Databricks-Übungen

Diese Übungen sind dafür konzipiert, die folgenden Schulungsinhalte auf Microsoft Learn zu unterstützen:

- [Implementieren einer Data Lakehouse-Analyselösung mit Azure Databricks](https://learn.microsoft.com/training/paths/data-engineer-azure-databricks/)
- [Implementieren einer Lösung für maschinelles Lernen mit Azure Databricks](https://learn.microsoft.com/training/paths/build-operate-machine-learning-solutions-azure-databricks/)
- [Implementieren einer Datentechniklösung mit Azure Databricks](https://learn.microsoft.com/training/paths/azure-databricks-data-engineer/)
- [Implementieren von Engineering mit generativer KI mit Azure Databricks](https://learn.microsoft.com/training/paths/implement-generative-ai-engineering-azure-databricks/)

Um diese Übungen abzuschließen, benötigen Sie ein Azure-Abonnement, in dem Sie über Administratorzugriff verfügen.

{% assign exercises = site.pages | where_exp:"page", "page.url contains '/Instructions'" %} {% for activity in exercises  %}
- [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) | {% endfor %}
