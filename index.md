---
title: Online gehostete Anweisungen
permalink: index.html
layout: home
---

# Azure Databricks-Übungen

Diese Übungen sind dafür konzipiert, die folgenden Schulungsinhalte auf Microsoft Learn zu unterstützen:

- [Implementieren einer Datenanalyselösung mit Azure Databricks](https://learn.microsoft.com/training/paths/data-engineer-azure-databricks/)
- [Implementieren einer Machine Learning-Lösung mit Azure Databricks](https://learn.microsoft.com/training/paths/build-operate-machine-learning-solutions-azure-databricks/)

Um diese Übungen abzuschließen, benötigen Sie ein Azure-Abonnement, in dem Sie über Administratorzugriff verfügen.

{% assign exercises = site.pages | where_exp:"page", "page.url contains '/Instructions'" %} {% for activity in exercises  %}
- [{{ activity.lab.title }}]({{ site.github.url }}{{ activity.url }}) | {% endfor %}