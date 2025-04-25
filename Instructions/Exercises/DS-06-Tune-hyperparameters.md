---
lab:
  title: Optimieren von Hyperparametern für maschinelles Lernen in Azure Databricks
---

# Optimieren von Hyperparametern für maschinelles Lernen in Azure Databricks

In dieser Übung verwenden Sie die Bibliothek **Optuna**, um Hyperparameter für das Training von Machine Learning-Modellen in Azure Databricks zu optimieren.

Diese Übung dauert ca. **30** Minuten.

> **Hinweis**: Die Benutzeroberfläche von Azure Databricks wird kontinuierlich verbessert. Die Benutzeroberfläche kann sich seit der Erstellung der Anweisungen in dieser Übung geändert haben.

## Vor der Installation

Sie benötigen ein [Azure-Abonnement](https://azure.microsoft.com/free), in dem Sie Administratorzugriff besitzen.

## Bereitstellen eines Azure Databricks-Arbeitsbereichs

> **Tipp**: Wenn Sie bereits über einen Azure Databricks-Arbeitsbereich verfügen, können Sie dieses Verfahren überspringen und Ihren vorhandenen Arbeitsbereich verwenden.

Diese Übung enthält ein Skript zum Bereitstellen eines neuen Azure Databricks-Arbeitsbereichs. Das Skript versucht, eine Azure Databricks-Arbeitsbereichsressource im *Premium*-Tarif in einer Region zu erstellen, in der Ihr Azure-Abonnement über ein ausreichendes Kontingent für die in dieser Übung erforderlichen Computekerne verfügt. Es wird davon ausgegangen, dass Ihr Benutzerkonto über ausreichende Berechtigungen im Abonnement verfügt, um eine Azure Databricks-Arbeitsbereichsressource zu erstellen. Wenn das Skript aufgrund unzureichender Kontingente oder Berechtigungen fehlschlägt, können Sie versuchen, [einen Azure Databricks-Arbeitsbereich interaktiv im Azure-Portal zu erstellen](https://learn.microsoft.com/azure/databricks/getting-started/#--create-an-azure-databricks-workspace).

1. Melden Sie sich in einem Webbrowser am [Azure-Portal](https://portal.azure.com) unter `https://portal.azure.com` an.
2. Verwenden Sie die Taste **[\>_]** rechts neben der Suchleiste oben auf der Seite, um eine neue Cloud Shell im Azure-Portal zu erstellen, und wählen Sie eine ***PowerShell***-Umgebung aus. Die Cloud Shell bietet eine Befehlszeilenschnittstelle in einem Bereich am unteren Rand des Azure-Portals, wie hier gezeigt:

    ![Azure-Portal mit einem Cloud Shell-Bereich](./images/cloud-shell.png)

    > **Hinweis**: Wenn Sie zuvor eine Cloud-Shell erstellt haben, die eine *Bash*-Umgebung verwendet, wechseln Sie zu ***PowerShell***.

3. Beachten Sie, dass Sie die Größe der Cloud-Shell ändern können, indem Sie die Trennlinie oben im Bereich ziehen oder die Symbole **&#8212;**, **&#10530;** und **X** oben rechts im Bereich verwenden, um den Bereich zu minimieren, zu maximieren und zu schließen. Weitere Informationen zur Verwendung von Azure Cloud Shell finden Sie in der [Azure Cloud Shell-Dokumentation](https://docs.microsoft.com/azure/cloud-shell/overview).

4. Geben Sie im PowerShell-Bereich die folgenden Befehle ein, um dieses Repository zu klonen:

    ```
    rm -r mslearn-databricks -f
    git clone https://github.com/MicrosoftLearning/mslearn-databricks
    ```

5. Nachdem das Repository geklont wurde, geben Sie den folgenden Befehl ein, um das Skript **setup.ps1** auszuführen, das einen Azure Databricks-Arbeitsbereich in einer verfügbaren Region bereitstellt:

    ```
    ./mslearn-databricks/setup.ps1
    ```

6. Wenn Sie dazu aufgefordert werden, wählen Sie aus, welches Abonnement Sie verwenden möchten (dies geschieht nur, wenn Sie Zugriff auf mehrere Azure-Abonnements haben).
7. Warten Sie, bis das Skript abgeschlossen ist. Dies dauert in der Regel etwa 5 Minuten, in einigen Fällen kann es jedoch länger dauern. Während Sie warten, lesen Sie den Artikel [Optimierung von Hyperparametern](https://learn.microsoft.com/azure/databricks/machine-learning/automl-hyperparam-tuning/) in der Azure Databricks-Dokumentation.

## Erstellen eines Clusters

Azure Databricks ist eine verteilte Verarbeitungsplattform, die Apache Spark-*Cluster* verwendet, um Daten parallel auf mehreren Knoten zu verarbeiten. Jeder Cluster besteht aus einem Treiberknoten, um die Arbeit zu koordinieren, und Arbeitsknoten zum Ausführen von Verarbeitungsaufgaben. In dieser Übung erstellen Sie einen *Einzelknotencluster* , um die in der Lab-Umgebung verwendeten Computeressourcen zu minimieren (in denen Ressourcen möglicherweise eingeschränkt werden). In einer Produktionsumgebung erstellen Sie in der Regel einen Cluster mit mehreren Workerknoten.

> **Tipp**: Wenn Sie bereits über einen Cluster mit einer Runtime 13.3 LTS **<u>ML</u>** oder einer höheren Runtimeversion in Ihrem Azure Databricks-Arbeitsbereich verfügen, können Sie ihn verwenden, um diese Übung abzuschließen, und dieses Verfahren überspringen.

1. Navigieren Sie im Azure-Portal zur Ressourcengruppe **msl-*xxxxxxx***, die vom Skript erstellt wurde (oder zur Ressourcengruppe, die Ihren vorhandenen Azure Databricks-Arbeitsbereich enthält).
1. Wählen Sie die Ressource Ihres Azure Databricks-Diensts aus (sie trägt den Namen **databricks-*xxxxxxx***, wenn Sie das Setupskript zum Erstellen verwendet haben).
1. Verwenden Sie auf der Seite **Übersicht** für Ihren Arbeitsbereich die Schaltfläche **Arbeitsbereich starten**, um Ihren Azure Databricks-Arbeitsbereich auf einer neuen Browserregisterkarte zu öffnen. Melden Sie sich an, wenn Sie dazu aufgefordert werden.

    > **Tipp**: Während Sie das Databricks-Arbeitsbereichsportal verwenden, werden möglicherweise verschiedene Tipps und Benachrichtigungen angezeigt. Schließen Sie diese, und folgen Sie den Anweisungen, um die Aufgaben in dieser Übung auszuführen.

1. Wählen Sie zunächst in der Randleiste auf der linken Seite die Aufgabe **(+) Neu** und dann **Cluster** aus.
1. Erstellen Sie auf der Seite **Neuer Cluster** einen neuen Cluster mit den folgenden Einstellungen:
    - **Clustername**: Cluster des *Benutzernamens* (der Standardclustername)
    - **Richtlinie:** Unrestricted
    - **Clustermodus**: Einzelknoten
    - **Zugriffsmodus**: Einzelner Benutzer (*Ihr Benutzerkonto ist ausgewählt*)
    - **Databricks-Runtimeversion**: *Wählen Sie die **<u>ML</u>**-Edition der neuesten Nicht-Betaversion der Runtime aus (**Nicht** eine Standard-Runtimeversion), die folgende Kriterien erfüllt:*
        - *Verwendet **keine** GPU*
        - *Umfasst Scala > **2.11***
        - *Umfasst Spark > **3.4***
    - **Photon-Beschleunigung verwenden**: <u>Nicht</u> ausgewählt
    - **Knotentyp**: Standard_D4ds_v5
    - **Beenden nach** *20* **Minuten Inaktivität**

1. Warten Sie, bis der Cluster erstellt wurde. Es kann ein oder zwei Minuten dauern.

> **Hinweis**: Wenn Ihr Cluster nicht gestartet werden kann, verfügt Ihr Abonnement möglicherweise über ein unzureichendes Kontingent in der Region, in der Ihr Azure Databricks-Arbeitsbereich bereitgestellt wird. Details finden Sie unter [Der Grenzwert für CPU-Kerne verhindert die Clustererstellung](https://docs.microsoft.com/azure/databricks/kb/clusters/azure-core-limit). In diesem Fall können Sie versuchen, Ihren Arbeitsbereich zu löschen und in einer anderen Region einen neuen zu erstellen. Sie können einen Bereich als Parameter für das Setupskript wie folgt angeben: `./mslearn-databricks/setup.ps1 eastus`

## Erstellen eines Notebooks

Sie führen Code aus, der die Spark MLLib-Bibliothek verwendet, um ein Machine Learning-Modell zu trainieren. Daher besteht der erste Schritt darin, ein neues Notebook in Ihrem Arbeitsbereich zu erstellen.

1. Verwenden Sie in der Randleiste den Link ** (+) Neu**, um ein **Notebook** zu erstellen.
1. Ändern Sie den Standardnamen des Notebooks (**Unbenanntes Notebook *[Datum]***) in **Hyperparameteroptimierung**, und wählen Sie in der Dropdownliste **Verbinden** Ihren Cluster aus, sofern er noch nicht ausgewählt ist. Wenn der Cluster nicht ausgeführt wird, kann es eine Minute dauern, bis er gestartet wird.

## Einlesen von Daten

Das Szenario für diese Übung basiert auf Beobachtungen von Pinguinen in der Antarktis. Das Ziel besteht darin, ein Machine Learning-Modell zu trainieren, um die Art eines beobachteten Pinguins anhand seines Standorts und seiner Körpermaße vorherzusagen.

> **Quellenangaben:** Das in dieser Übung verwendete Pinguin-Dataset ist eine Teilmenge der Daten, die von [Dr. Kristen Gorman](https://www.uaf.edu/cfos/people/faculty/detail/kristen-gorman.php) und der [Palmer-Station (Antarktis-Forschungsstation)](https://pal.lternet.edu/), ein Mitglied des [Long Term Ecological Research Network](https://lternet.edu/) (Netzwerk für ökologische und ökosystemare Langzeitforschung), gesammelt und zur Verfügung gestellt werden.

1. Geben Sie in der ersten Zelle des Notebooks den folgenden Code ein, der mit *Shell*-Befehlen die Pinguindaten von GitHub in das von Ihrem Cluster verwendete Dateisystem herunterlädt.

    ```bash
    %sh
    rm -r /dbfs/hyperparam_tune_lab
    mkdir /dbfs/hyperparam_tune_lab
    wget -O /dbfs/hyperparam_tune_lab/penguins.csv https://raw.githubusercontent.com/MicrosoftLearning/mslearn-databricks/main/data/penguins.csv
    ```

1. Verwenden Sie Menüoption **&#9656; Zelle Ausführen** links neben der Zelle, um sie auszuführen. Warten Sie dann, bis der vom Code ausgeführte Spark-Auftrag, abgeschlossen ist.
1. Bereiten Sie nun die Daten für das maschinelle Lernen vor. Verwenden Sie unter der vorhandenen Codezelle das Symbol **+**, um eine neue Codezelle hinzuzufügen. Geben Sie dann den folgenden Code in die neue Zelle ein, und führen Sie ihn aus, um Folgendes zu tun:
    - Entfernen aller unvollständigen Zeilen
    - Verwenden geeigneter Datentypen
    - Anzeigen einer Stichprobe der Daten
    - Teilen Sie die Daten in zwei Datasets auf: eines zum Trainieren und ein weiteres zum Testen.


    ```python
   from pyspark.sql.types import *
   from pyspark.sql.functions import *
   
   data = spark.read.format("csv").option("header", "true").load("/hyperparam_tune_lab/penguins.csv")
   data = data.dropna().select(col("Island").astype("string"),
                             col("CulmenLength").astype("float"),
                             col("CulmenDepth").astype("float"),
                             col("FlipperLength").astype("float"),
                             col("BodyMass").astype("float"),
                             col("Species").astype("int")
                             )
   display(data.sample(0.2))
   
   splits = data.randomSplit([0.7, 0.3])
   train = splits[0]
   test = splits[1]
   print ("Training Rows:", train.count(), " Testing Rows:", test.count())
    ```

## Optimieren von Hyperparameterwerten zum Trainieren eines Modells

Sie trainieren ein Machine Learning-Modell, indem Sie die Merkmale an einen Algorithmus anpassen, der die wahrscheinlichste Bezeichnung berechnet. Die Algorithmen nehmen die Trainingsdaten als Parameter und versuchen, eine mathematische Beziehung zwischen den Merkmalen und Bezeichnungen zu berechnen. Zusätzlich zu den Daten verwenden die meisten Algorithmen einen oder mehrere *Hyperparameter*, um die Art und Weise, wie die Beziehung berechnet wird, zu beeinflussen. Die Bestimmung der optimalen Hyperparameterwerte ist ein wichtiger Teil des iterativen Modelltrainingsprozesses.

Um die Ermittlung optimaler Hyperparameterwerte zu erleichtern, bietet Azure Databricks Unterstützung für [**Optuna**](https://optuna.readthedocs.io/en/stable/index.html) – eine Bibliothek, mit der Sie mehrere Hyperparameterwerte ausprobieren und die beste Kombination für Ihre Daten finden können.

Der erste Schritt bei der Verwendung von Optuna besteht darin, eine Funktion zu erstellen, die Folgendes leistet:

- Sie trainiert ein Modell mit mindestens einem Hyperparameterwert, der als Parameter an die Funktion übergeben wird.
- Sie berechnet eine Leistungsmetrik, die verwendet werden kann, um *Verluste* zu messen (wie weit das Modell von einer perfekten Vorhersageleistung entfernt ist)
- Sie gibt den Verlustwert zurück, sodass er iterativ optimiert (minimiert) werden kann, indem verschiedene Hyperparameterwerte ausprobiert werden.

1. Fügen Sie eine neue Zelle hinzu und verwenden Sie den folgenden Code, um eine Funktion zu erstellen, die den Wertebereich für die Hyperparameter definiert und die Pinguindaten verwendet, um ein Klassifizierungsmodell zu trainieren, das die Art eines Pinguins anhand seines Standorts und seiner Maße vorhersagt:

    ```python
   import optuna
   import mlflow # if you wish to log your experiments
   from pyspark.ml import Pipeline
   from pyspark.ml.feature import StringIndexer, VectorAssembler, MinMaxScaler
   from pyspark.ml.classification import DecisionTreeClassifier
   from pyspark.ml.evaluation import MulticlassClassificationEvaluator
   
   def objective(trial):
       # Suggest hyperparameter values (maxDepth and maxBins):
       max_depth = trial.suggest_int("MaxDepth", 0, 9)
       max_bins = trial.suggest_categorical("MaxBins", [10, 20, 30])

       # Define pipeline components
       cat_feature = "Island"
       num_features = ["CulmenLength", "CulmenDepth", "FlipperLength", "BodyMass"]
       catIndexer = StringIndexer(inputCol=cat_feature, outputCol=cat_feature + "Idx")
       numVector = VectorAssembler(inputCols=num_features, outputCol="numericFeatures")
       numScaler = MinMaxScaler(inputCol=numVector.getOutputCol(), outputCol="normalizedFeatures")
       featureVector = VectorAssembler(inputCols=[cat_feature + "Idx", "normalizedFeatures"], outputCol="Features")

       dt = DecisionTreeClassifier(
           labelCol="Species",
           featuresCol="Features",
           maxDepth=max_depth,
           maxBins=max_bins
       )

       pipeline = Pipeline(stages=[catIndexer, numVector, numScaler, featureVector, dt])
       model = pipeline.fit(train)

       # Evaluate the model using accuracy.
       predictions = model.transform(test)
       evaluator = MulticlassClassificationEvaluator(
           labelCol="Species",
           predictionCol="prediction",
           metricName="accuracy"
       )
       accuracy = evaluator.evaluate(predictions)

       # Since Optuna minimizes the objective, return negative accuracy.
       return -accuracy
    ```

1. Fügen Sie eine neue Zelle hinzu, und verwenden Sie den folgenden Code, um das Optimierungsexperiment durchzuführen:

    ```python
   # Optimization run with 5 trials:
   study = optuna.create_study()
   study.optimize(objective, n_trials=5)

   print("Best param values from the optimization run:")
   print(study.best_params)
    ```

1. Beobachten Sie, wie der Code die Trainingsfunktion 5 Mal iterativ ausführt und dabei versucht, den Verlust zu minimieren (basierend auf der Einstellung **n_trials**). Jeder Versuch wird von MLflow aufgezeichnet. Mit der Umschaltfläche **&#9656;** können Sie die Ausgabe der **MLflow-Ausführung** unter der Codezelle erweitern und auf den Hyperlink **Experiment** klicken, um sie anzuzeigen. Jeder Ausführung wird ein zufälliger Name zugewiesen.Sie können jede in der MLflow-Ausführungsanzeige betrachten, um Details zu Parametern und Metriken zu sehen, die aufgezeichnet wurden.
1. Wenn alle Ausführungen abgeschlossen sind, zeigt der Code die Details der besten gefundenen Hyperparameterwerte an (die Kombination, die zu dem geringsten Verlust geführt hat). In diesem Fall wird der Parameter **MaxBins** als Auswahl aus einer Liste mit drei möglichen Werten (10, 20 und 30) definiert. Der beste Wert gibt das nullbasierte Element in der Liste an (also 0=10, 1=20 und 2=30). Der Parameter **MaxDepth-** wird als zufällige ganze Zahl zwischen 0 und 10 definiert, und der ganzzahlige Wert, der das beste Ergebnis ergeben hat, wird angezeigt. 

## Bereinigen

Wählen Sie zunächst im Azure Databricks-Portal auf der Seite **Compute** Ihren Cluster und dann **&#9632; Beenden** aus, um ihn herunterzufahren.

Wenn Sie die Erkundung von Azure Databricks abgeschlossen haben, löschen Sie die erstellten Ressourcen, um unnötige Azure-Kosten zu vermeiden und Kapazität in Ihrem Abonnement freizugeben.

> **Weitere Informationen:** Weitere Informationen finden Sie unter [Optimieren von Hyperparametern](https://learn.microsoft.com/azure/databricks/machine-learning/automl-hyperparam-tuning/) in der Azure Databricks-Dokumentation.
