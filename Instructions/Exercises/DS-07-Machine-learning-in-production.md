---
lab:
  title: Verwalten eines Modells für maschinelles Lernen mit Azure Databricks
---

# Verwalten eines Modells für maschinelles Lernen mit Azure Databricks

Das Training eines Modells für maschinelles Lernen mit Azure Databricks beinhaltet die Nutzung einer einheitlichen Analyseplattform, die eine kollaborative Umgebung für Datenverarbeitung, Modelltraining und Bereitstellung bereitstellt. Azure Databricks ist mit MLflow integriert, um den Lebenszyklus des maschinellen Lernens zu verwalten, einschließlich der Verfolgung von Experimenten und der Bereitstellung von Modellen.

Diese Übung dauert ca. **20** Minuten.

## Vorbereitung

Sie benötigen ein [Azure-Abonnement](https://azure.microsoft.com/free), in dem Sie Administratorzugriff besitzen.

## Bereitstellen eines Azure Databricks-Arbeitsbereichs

> **Tipp**: Wenn Sie bereits über einen Azure Databricks-Arbeitsbereich verfügen, können Sie dieses Verfahren überspringen und Ihren vorhandenen Arbeitsbereich verwenden.

Diese Übung enthält ein Skript zum Bereitstellen eines neuen Azure Databricks-Arbeitsbereichs. Das Skript versucht, eine Azure Databricks-Arbeitsbereichsressource im *Premium*-Tarif in einer Region zu erstellen, in der Ihr Azure-Abonnement über ein ausreichendes Kontingent für die in dieser Übung erforderlichen Computekerne verfügt. Es wird davon ausgegangen, dass Ihr Benutzerkonto über ausreichende Berechtigungen im Abonnement verfügt, um eine Azure Databricks-Arbeitsbereichsressource zu erstellen. Wenn das Skript aufgrund unzureichender Kontingente oder Berechtigungen fehlschlägt, können Sie versuchen, [einen Azure Databricks-Arbeitsbereich interaktiv im Azure-Portal zu erstellen](https://learn.microsoft.com/azure/databricks/getting-started/#--create-an-azure-databricks-workspace).

1. Melden Sie sich in einem Webbrowser am [Azure-Portal](https://portal.azure.com) unter `https://portal.azure.com` an.
2. Verwenden Sie rechts neben der Suchleiste oben auf der Seite die Schaltfläche **[\>_]**, um eine neue Cloud Shell-Instanz im Azure-Portal zu erstellen. Wählen Sie eine ***PowerShell***-Umgebung aus, und erstellen Sie Speicher, falls Sie dazu aufgefordert werden. Die Cloud Shell bietet eine Befehlszeilenschnittstelle in einem Bereich am unteren Rand des Azure-Portals, wie hier gezeigt:

    ![Azure-Portal mit einem Cloud Shell-Bereich](./images/cloud-shell.png)

    > **Hinweis**: Wenn Sie zuvor eine Cloud Shell erstellt haben, die eine *Bash*-Umgebung verwendet, ändern Sie diese mithilfe des Dropdownmenüs oben links im Cloud Shell-Bereich zu ***PowerShell***.

3. Beachten Sie, dass Sie die Größe der Cloud Shell durch Ziehen der Trennzeichenleiste oben im Bereich ändern können oder den Bereich mithilfe der Symbole **&#8212;**, **&#9723;** und **X** oben rechts minimieren, maximieren und schließen können. Weitere Informationen zur Verwendung von Azure Cloud Shell finden Sie in der [Azure Cloud Shell-Dokumentation](https://docs.microsoft.com/azure/cloud-shell/overview).

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
7. Warten Sie, bis das Skript abgeschlossen ist. Dies dauert in der Regel etwa 5 Minuten, in einigen Fällen kann es jedoch länger dauern. Während Sie warten, lesen Sie den Artikel [Was ist maschinelles Lernen mit Databricks?](https://learn.microsoft.com/azure/databricks/machine-learning/) in der Azure Databricks-Dokumentation.

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
1. Ändern Sie den Standardnamen des Notebooks (**Unbenanntes Notebook *[Datum]***) in **Machine Learning**, und wählen Sie in der Dropdownliste **Verbinden** Ihren Cluster aus, sofern er noch nicht ausgewählt ist. Wenn der Cluster nicht ausgeführt wird, kann es eine Minute dauern, bis er gestartet wird.

## Daten erfassen und aufbereiten

Das Szenario für diese Übung basiert auf Beobachtungen von Pinguinen in der Antarktis. Das Ziel besteht darin, ein Machine Learning-Modell zu trainieren, um die Art eines beobachteten Pinguins anhand seines Standorts und seiner Körpermaße vorherzusagen.

> **Quellenangaben:** Das in dieser Übung verwendete Pinguin-Dataset ist eine Teilmenge der Daten, die von [Dr. Kristen Gorman](https://www.uaf.edu/cfos/people/faculty/detail/kristen-gorman.php) und der [Palmer-Station (Antarktis-Forschungsstation)](https://pal.lternet.edu/), ein Mitglied des [Long Term Ecological Research Network](https://lternet.edu/) (Netzwerk für ökologische und ökosystemare Langzeitforschung), gesammelt und zur Verfügung gestellt werden.

1. Geben Sie in der ersten Zelle des Notebooks den folgenden Code ein, der mit *Shell*-Befehlen die Pinguindaten von GitHub in das von Ihrem Cluster verwendete Dateisystem herunterlädt.

    ```bash
    %sh
    rm -r /dbfs/ml_lab
    mkdir /dbfs/ml_lab
    wget -O /dbfs/ml_lab/penguins.csv https://raw.githubusercontent.com/MicrosoftLearning/mslearn-databricks/main/data/penguins.csv
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
   
   data = spark.read.format("csv").option("header", "true").load("/hyperopt_lab/penguins.csv")
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
    
## Führen Sie eine Pipeline aus, um die Daten vorzuverarbeiten und ein ML-Modell zu trainieren

Bevor Sie Ihr Modell trainieren können, müssen Sie Schritte zur Merkmalstechnik durchführen und dann einen Algorithmus an die Daten anpassen. Um das Modell mit einigen Testdaten zur Erstellung von Vorhersagen zu verwenden, müssen Sie die gleichen Schritte der Merkmalstechnik auf die Testdaten anwenden. Eine effizientere Methode zum Erstellen und Verwenden von Modellen besteht darin, besteht darin, die Transformatoren, die zur Aufbereitung der Daten verwendet werden, und das Modell, mit dem sie trainiert werden, in einer *Pipeline* zu kapseln.

1. Verwenden Sie den folgenden Code, um eine Pipeline zu erstellen, die die Datenvorbereitungs- und Modelltrainingsschritte kapselt:

    ```python
   from pyspark.ml import Pipeline
   from pyspark.ml.feature import StringIndexer, VectorAssembler, MinMaxScaler
   from pyspark.ml.classification import LogisticRegression
   
   catFeature = "Island"
   numFeatures = ["CulmenLength", "CulmenDepth", "FlipperLength", "BodyMass"]
   
   # Define the feature engineering and model training algorithm steps
   catIndexer = StringIndexer(inputCol=catFeature, outputCol=catFeature + "Idx")
   numVector = VectorAssembler(inputCols=numFeatures, outputCol="numericFeatures")
   numScaler = MinMaxScaler(inputCol = numVector.getOutputCol(), outputCol="normalizedFeatures")
   featureVector = VectorAssembler(inputCols=["IslandIdx", "normalizedFeatures"], outputCol="Features")
   algo = LogisticRegression(labelCol="Species", featuresCol="Features", maxIter=10, regParam=0.3)
   
   # Chain the steps as stages in a pipeline
   pipeline = Pipeline(stages=[catIndexer, numVector, numScaler, featureVector, algo])
   
   # Use the pipeline to prepare data and fit the model algorithm
   model = pipeline.fit(train)
   print ("Model trained!")
    ```

    Da die Feature Engineering-Schritte jetzt in dem Modell, das von der Pipeline trainiert wird, gekapselt sind, können Sie das Modell mit den Testdaten verwenden, ohne jede Transformation anwenden zu müssen (sie werden automatisch vom Modell angewendet).

1. Verwenden Sie den folgenden Code, um die Pipeline auf die Testdaten anzuwenden und das Modell zu bewerten:

    ```python
   prediction = model.transform(test)
   predicted = prediction.select("Features", "probability", col("prediction").astype("Int"), col("Species").alias("trueLabel"))
   display(predicted)

   # Generate evaluation metrics
   from pyspark.ml.evaluation import MulticlassClassificationEvaluator
   
   evaluator = MulticlassClassificationEvaluator(labelCol="Species", predictionCol="prediction")
   
   # Simple accuracy
   accuracy = evaluator.evaluate(prediction, {evaluator.metricName:"accuracy"})
   print("Accuracy:", accuracy)
   
   # Class metrics
   labels = [0,1,2]
   print("\nIndividual class metrics:")
   for label in sorted(labels):
       print ("Class %s" % (label))
   
       # Precision
       precision = evaluator.evaluate(prediction, {evaluator.metricLabel:label,
                                                       evaluator.metricName:"precisionByLabel"})
       print("\tPrecision:", precision)
   
       # Recall
       recall = evaluator.evaluate(prediction, {evaluator.metricLabel:label,
                                                evaluator.metricName:"recallByLabel"})
       print("\tRecall:", recall)
   
       # F1 score
       f1 = evaluator.evaluate(prediction, {evaluator.metricLabel:label,
                                            evaluator.metricName:"fMeasureByLabel"})
       print("\tF1 Score:", f1)
   
   # Weighed (overall) metrics
   overallPrecision = evaluator.evaluate(prediction, {evaluator.metricName:"weightedPrecision"})
   print("Overall Precision:", overallPrecision)
   overallRecall = evaluator.evaluate(prediction, {evaluator.metricName:"weightedRecall"})
   print("Overall Recall:", overallRecall)
   overallF1 = evaluator.evaluate(prediction, {evaluator.metricName:"weightedFMeasure"})
   print("Overall F1 Score:", overallF1) 
    ```

## Registrieren und Bereitstellen des Modells

Sie haben das von jedem Experiment trainierte Modell bereits protokolliert, als Sie die Pipeline ausgeführt haben. Sie können Modelle auch *registrieren* und bereitstellen, damit sie für Clientanwendungen bereitgestellt werden können.

> **Hinweis:** Die Modellbereitstellung wird nur in Azure Databricks *Premium*-Arbeitsbereichen unterstützt und ist auf [bestimmte Regionen](https://learn.microsoft.com/azure/databricks/resources/supported-regions) beschränkt.

1. Wählen Sie **Experimente** im linken Fensterbereich.
1. Wählen Sie das Experiment aus, das mit dem Namen Ihres Notebooks erstellt wurde, und zeigen Sie die Detailseite für die letzte Experimentausführung an.
1. Verwenden Sie die Schaltfläche **Modell registrieren**, um das Modell zu registrieren, das in diesem Experiment protokolliert wurde, und erstellen Sie ein neues Modell namens **Pinguin Predictor**, wenn Sie dazu aufgefordert werden.
1. Wenn das Modell registriert wurde, zeigen Sie die Seite **Modelle** (in der Navigationsleiste auf der linken Seite) an, und wählen Sie das Modell **Pinguin Predictor** aus.
1. Verwenden Sie auf der Seite für das Modell **Penguin Predictor** die Schaltfläche **Modell für Rückschlüsse verwenden**, um einen neuen Echtzeitendpunkt mit den folgenden Einstellungen zu erstellen:
    - **Modell**: Penguin Predictor
    - **Modellversion**: 1
    - **Endpunkt**: predict-pinguin
    - **Computegröße**: Klein

    Der Bereitstellungsendpunkt wird in einem neuen Cluster gehostet, dessen Erstellung einige Minuten dauern kann.
  
1. Wenn der Endpunkt erstellt worden ist, verwenden Sie die Schaltfläche **Abfrageendpunkt** oben rechts, um eine Schnittstelle zu öffnen, über die Sie den Endpunkt testen können. Geben Sie dann in der Testschnittstelle auf der Registerkarte **Browser** die folgende JSON-Anforderung ein, und verwenden Sie die Schaltfläche **Anforderung senden**, um den Endpunkt aufzurufen und eine Vorhersage zu generieren.

    ```json
    {
      "dataframe_records": [
      {
         "Island": "Biscoe",
         "CulmenLength": 48.7,
         "CulmenDepth": 14.1,
         "FlipperLength": 210,
         "BodyMass": 4450
      }
      ]
    }
    ```

1. Experimentieren Sie mit einigen unterschiedlichen Werten für die Pinguinmerkmale, und beobachten Sie die zurückgegebenen Ergebnisse. Schließen Sie dann die Testschnittstelle.

## Löschen des Endpunkts

Wenn der Endpunkt nicht mehr benötigt wird, sollten Sie ihn löschen, um unnötige Kosten zu vermeiden.

Wählen Sie auf der Endpunktseite **predict-penguin** im Menü **&#8285;** die Option **Löschen** aus.

## Bereinigung

Wählen Sie zunächst im Azure Databricks-Portal auf der Seite **Compute** Ihren Cluster und dann **&#9632; Beenden** aus, um ihn herunterzufahren.

Wenn Sie die Erkundung von Azure Databricks abgeschlossen haben, löschen Sie die erstellten Ressourcen, um unnötige Azure-Kosten zu vermeiden und Kapazität in Ihrem Abonnement freizugeben.

> **Weitere Informationen:** Weitere Informationen finden Sie in der [Spark MLLib-Dokumentation](https://spark.apache.org/docs/latest/ml-guide.html).
