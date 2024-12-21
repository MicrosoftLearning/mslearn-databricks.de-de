---
lab:
  title: Erste Schritte mit Machine Learning in Azure Databricks
---

# Erste Schritte mit Machine Learning in Azure Databricks

In dieser Übung erkunden Sie Techniken zum Vorbereiten von Daten und Trainieren von Machine Learning-Modellen in Azure Databricks.

Diese Übung dauert ca. **45** Minuten.

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

## Einlesen von Daten

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

## Erkunden und Bereinigen der Daten
  
Nachdem Sie die Datendatei erfasst haben, können Sie sie in einen Datenframe laden und anzeigen.

1. Verwenden Sie unter der vorhandenen Codezelle das Symbol **+**, um eine neue Codezelle hinzuzufügen. Geben Sie dann in die neue Zelle den folgenden Code ein, und führen Sie ihn aus, um die Daten aus den Dateien zu laden und sie anzuzeigen.

    ```python
   df = spark.read.format("csv").option("header", "true").load("/ml_lab/penguins.csv")
   display(df)
    ```

    Der Code initiiert die erforderlichen *Spark-Aufträge* zum Laden der Daten, und die Ausgabe ist ein *pyspark.sql.dataframe.DataFrame*-Objekt namens *df*. Diese Informationen werden direkt unter dem Code angezeigt, und Sie können den Umschalter **&#9656** verwenden, um die Ausgabe **df: pyspark.sql.dataframe.DataFrame** zu erweitern und Details der darin enthaltenen Spalten und deren Datentypen anzuzeigen. Da diese Daten aus einer Textdatei geladen wurden und einige leere Werte enthielten, hat Spark allen Spalten den Datentyp **string** zugewiesen.
    
    Die Daten selbst bestehen aus Messungen der folgenden Details von Pinguinen, die in der Antarktis beobachtet wurden:
    
    - **Island**: Die Insel in der Antarktis, auf der der Pinguin beobachtet wurde.
    - **CulmenLength:** Die Länge des Pinguinschnabels in mm.
    - **CulmenDepth:** Die Tiefe des Pinguinschnabels in mm.
    - **FlipperLength:** Die Flossenlänge des Pinguins in mm.
    - **BodyMass:** Die Körpermasse des Pinguins in Gramm.
    - **Species:** Eine ganzzahliger Wert, der die Pinguinart darstellt:
      - **0**: *Adeliepinguin*
      - **1:** *Eselspinguin*
      - **2:** *Zügelpinguin*
      
    Unser Ziel in diesem Projekt ist es, die beobachteten Merkmale eines Pinguins (seine *Features*) zu verwenden, um seine Art vorherzusagen (in der Terminologie des maschinellen Lernens ist dies die *Bezeichnung*).
      
    Beachten Sie, dass einige Beobachtungen *NULL*-Werte enthalten oder dass Datenwerte für einige Features fehlen. Es ist nicht ungewöhnlich, dass die rohen Quelldaten, die Sie erfassen, solche Probleme aufweisen. Daher besteht der erste Schritt bei einem Projekt zum maschinellen Lernen in der Regel darin, die Daten gründlich zu untersuchen und zu bereinigen, damit sie sich besser zum Trainieren einer Machine Learning-Modells eignen.
    
1. Fügen Sie eine Zelle hinzu, und verwenden Sie diese, um die folgenden Zellen auszuführen, um die Zeilen mit unvollständigen Daten mithilfe der Methode **dropna** zu entfernen und um geeignete Datentypen auf die Daten anzuwenden, indem Sie die Methode **select** mit den Funktionen **col** und **astype** verwenden.

    ```python
   from pyspark.sql.types import *
   from pyspark.sql.functions import *
   
   data = df.dropna().select(col("Island").astype("string"),
                              col("CulmenLength").astype("float"),
                             col("CulmenDepth").astype("float"),
                             col("FlipperLength").astype("float"),
                             col("BodyMass").astype("float"),
                             col("Species").astype("int")
                             )
   display(data)
    ```
    
    Erneut können Sie die Details des zurückgegebenen Datenrahmens (dieses Mal mit dem Namen *data*) umschalten, um zu überprüfen, ob die Datentypen angewendet wurden, und Sie können die Daten überprüfen, um sicherzustellen, dass die Zeilen mit unvollständigen Daten entfernt wurden.
    
    In einem realen Projekt müssten Sie wahrscheinlich weitere Untersuchungen und Datenbereinigungen durchführen, um Fehler in den Daten zu beheben (oder zu entfernen), Ausreißer (untypisch große oder kleine Werte) zu identifizieren und zu entfernen oder die Daten so auszugleichen, dass für jede Bezeichnung, die Sie vorherzusagen versuchen, eine einigermaßen gleiche Anzahl von Zeilen vorhanden ist.

    > **Tipp**: Weitere Informationen zu Methoden und Funktionen, die Sie mit Datenframes verwenden können, finden Sie in der [Spark SQL-Referenz](https://spark.apache.org/docs/latest/sql-programming-guide.html).

## Teilen der Daten

Für die Zwecke dieser Übung gehen wir davon aus, dass die Daten nun bereinigt und bereit sind, um ein Machine Learning-Modell zu trainieren. Die Bezeichnung, die wir prognostizieren möchten, ist eine bestimmte Kategorie oder *Klasse* (die Pinguinart), daher ist das Machine Learning-Modell, das wir trainieren müssen, ein *Klassifizierungsmodell*. Die Klassifizierung (zusammen mit der *Regression*, die zur Vorhersage eines numerischen Wertes verwendet wird) ist eine Form des *überwachten* maschinellen Lernens, bei der wir Trainingsdaten verwenden, die bekannte Werte für die Bezeichnung enthalten, die wir vorhersagen möchten. Beim Training eines Modells geht es eigentlich nur darum, einen Algorithmus an die Daten anzupassen, um zu berechnen, wie die Featurewerte mit dem bekannten Bezeichnungswert korrelieren. Anschließend können wir das trainierte Modell auf eine neue Beobachtung, für die wir nur die Featurewerte kennen, anwenden und es den Bezeichnungswert vorhersehen lassen.

Um sicherzustellen, dass wir Vertrauen in unser trainiertes Modell haben können, besteht der typische Ansatz darin, das Modell nur mit *einigen* der Daten zu trainieren und einige Daten mit bekannten Bezeichnungswerten zurückzuhalten, die wir zum Testen des trainierten Modells verwenden können, um zu sehen, wie genau seine Vorhersagen sind. Um dieses Ziel zu erreichen, teilen wir den vollständigen Dataset in zwei zufällige Teilmengen auf. Wir verwenden 70 % der Daten für das Training und halten 30 % für Tests zurück.

1. Fügen Sie eine Codezelle mit dem folgenden Code hinzu, und führen Sie sie aus, um die Daten aufzuteilen.

    ```python
   splits = data.randomSplit([0.7, 0.3])
   train = splits[0]
   test = splits[1]
   print ("Training Rows:", train.count(), " Testing Rows:", test.count())
    ```

## Durchführen des Feature Engineerings

Nach der Bereinigung der Rohdaten führen Datenwissenschaftler in der Regel einige zusätzliche Arbeiten durch, um sie für das Modelltraining vorzubereiten. Dieser Prozess wird häufig als *Feature Engineering* bezeichnet und beinhaltet die iterative Optimierung der Merkmale im Trainingsdatensatz, um das bestmögliche Modell zu erstellen. Welche spezifischen Featuremodifikationen erforderlich sind, hängt von den Daten und dem gewünschten Modell ab. Es gibt jedoch einige allgemeine Feature-Engineering-Aufgaben, mit denen Sie sich vertraut machen sollten.

### Codieren kategorischer Features

Algorithmen des maschinellen Lernens basieren in der Regel auf der Suche nach mathematischen Beziehungen zwischen Features und Bezeichnungen. Das bedeutet, dass es in der Regel am besten ist, die Features in Ihren Trainingsdaten als *numerische* Werte zu definieren. In einigen Fällen haben Sie möglicherweise einige Features, die eher *kategorisch* als numerisch sind und als Zeichenfolgen ausgedrückt werden, z. B. der Name der Insel, auf der die Pinguinbeobachtung in unserem Dataset stattfand. Die meisten Algorithmen erwarten jedoch numerische Features. Daher müssen diese zeichenfolgenbasierten kategorischen Werte als Zahlen *codiert* werden. In diesem Fall verwenden wir **StringIndexer** aus der **Spark MLLib**-Bibliothek, um den Inselnamen als numerischen Wert zu codieren, indem jedem einzelnen Inselnamen ein eindeutiger ganzzahliger Index zugewiesen wird.

1. Führen Sie den folgenden Code aus, um die kategorischen Werte der Spalte **Island** als numerische Indizes zu codieren.

    ```python
   from pyspark.ml.feature import StringIndexer

   indexer = StringIndexer(inputCol="Island", outputCol="IslandIdx")
   indexedData = indexer.fit(train).transform(train).drop("Island")
   display(indexedData)
    ```

    In den Ergebnissen sollten Sie sehen, dass anstelle eines Inselnamens jede Zeile jetzt die Spalte **IslandIdx** mit einem ganzzahligen Wert enthält, der die Insel darstellt, auf der die Beobachtung aufgezeichnet wurde.

### Normalisieren (Skalieren) numerischer Features

Lenken wir unsere Aufmerksamkeit nun auf die numerischen Werte in unseren Daten. Diese Werte (**CulmenLength**, **CulmenDepth**, **FlipperLength** und **BodyMass**) stellen alle Maße einer Art oder einer anderen dar, aber sie sind in unterschiedlichen Maßstäben angegeben. Beim Trainieren von Modellen sind die Maßeinheiten nicht so wichtig wie die relativen Unterschiede zwischen verschiedenen Beobachtungen, und Merkmale, die durch größere Zahlen dargestellt werden, können oft den Algorithmus zum Trainieren des Modells dominieren und die Bedeutung des Merkmals bei der Berechnung einer Vorhersage verzerren. Um dies zu vermeiden, ist es üblich, die numerischen Featurewerte so zu *normalisieren*, dass sie alle auf der gleichen relativen Skala liegen (zum Beispiel ein Dezimalwert zwischen 0,0 und 1,0).

Der Code, den wir dafür verwenden werden, ist etwas komplizierter als die kategorisierte Codierung, die wir zuvor durchgeführt haben. Da wir mehrere Spaltenwerte gleichzeitig skalieren müssen, erstellen wir eine einzelne Spalte, die einen *Vektor* (im Wesentlichen ein Array) mit allen numerischen Merkmalen enthält, und wenden dann einen Skalierer an, um eine neue Vektorspalte mit den entsprechenden normalisierten Werten zu erzeugen.

1. Verwenden Sie den folgenden Code, um die numerischen Features zu normalisieren und einen Vergleich der Vektorspalten vor und nach der Normalisierung anzuzeigen.

    ```python
   from pyspark.ml.feature import VectorAssembler, MinMaxScaler

   # Create a vector column containing all numeric features
   numericFeatures = ["CulmenLength", "CulmenDepth", "FlipperLength", "BodyMass"]
   numericColVector = VectorAssembler(inputCols=numericFeatures, outputCol="numericFeatures")
   vectorizedData = numericColVector.transform(indexedData)
   
   # Use a MinMax scaler to normalize the numeric values in the vector
   minMax = MinMaxScaler(inputCol = numericColVector.getOutputCol(), outputCol="normalizedFeatures")
   scaledData = minMax.fit(vectorizedData).transform(vectorizedData)
   
   # Display the data with numeric feature vectors (before and after scaling)
   compareNumerics = scaledData.select("numericFeatures", "normalizedFeatures")
   display(compareNumerics)
    ```

    Die Spalte **numericFeatures** in den Ergebnissen enthält einen Vektor für jede Zeile. Der Vektor enthält vier nicht skalierte numerische Werte (die ursprünglichen Maße des Pinguins). Sie können den Umschalter **&#9656;** verwenden, um die diskreten Werte deutlicher zu sehen.
    
    Die Spalte **normalizedFeatures** enthält ebenfalls einen Vektor für jede Pinguinbeobachtung, doch sind dieses Mal die Werte im Vektor auf eine relative Skala normiert, die auf den Mindest- und Höchstwerten für jede Messung basiert.

### Vorbereiten von Features und Bezeichnungen für das Training

Nun fassen wir alles zusammen und erstellen eine einzige Spalte, die alle Merkmale enthält (den kodierten kategorischen Inselnamen und die normalisierten Pinguinmaße), und eine weitere Spalte mit der Klassenbezeichnung, für die wir ein Modell zur Vorhersage trainieren wollen (die Pinguinart).

1. Führen Sie den folgenden Code aus:

    ```python
   featVect = VectorAssembler(inputCols=["IslandIdx", "normalizedFeatures"], outputCol="featuresVector")
   preppedData = featVect.transform(scaledData)[col("featuresVector").alias("features"), col("Species").alias("label")]
   display(preppedData)
    ```

    Die Vektor **Features** enthalten fünf Werte (die codierte Insel und die normalisierte Schnabellänge, Schnabeltiefe, Flossenlänge und Körpermasse). Die Bezeichnung enthält einen einfachen ganzzahligen Code, der die Klasse der Pinguinart angibt.

## Trainieren eines Machine Learning-Modells

Nachdem die Trainingsdaten vorbereitet wurden, können Sie sie Trainieren eines Modells verwenden. Modelle werden mithilfe eines *Algorithmus* trainiert, der versucht, eine Beziehung zwischen den Features und Bezeichnungen herzustellen. Da Sie in diesem Fall ein Modell trainieren möchten, das eine Kategorie der *Klasse* vorhersagt, müssen Sie einen *Klassifizierungsalgorithmus* verwenden. Es gibt viele Algorithmen für die Klassifizierung. Beginnen wir mit einem bewährten Algorithmus: der logistischen Regression, die iterativ versucht, die optimalen Koeffizienten zu finden, die in einer logistischen Berechnung zur Vorhersage der Wahrscheinlichkeit eines jeden Klassenbezeichnungswerts auf die Feauredaten angewendet werden können. Um das Modell zu trainieren, passen Sie den logistischen Regressionsalgorithmus an die Trainingsdaten an.

1. Führen Sie folgenden Code aus, um ein Modell zu trainieren.

    ```python
   from pyspark.ml.classification import LogisticRegression

   lr = LogisticRegression(labelCol="label", featuresCol="features", maxIter=10, regParam=0.3)
   model = lr.fit(preppedData)
   print ("Model trained!")
    ```

    Die meisten Algorithmen unterstützen Parameter, mit denen Sie in einem gewissen Umfang die Art und Weise steuern können, wie das Modell trainiert wird. In diesem Fall müssen Sie mit dem logistischen Regressionsalgorithmus die Spalte mit dem Featurevektor und die Spalte mit der bekannten Bezeichnung identifizieren. Außerdem können Sie die maximale Anzahl der Iterationen angeben, die durchgeführt werden, um optimale Koeffizienten für die logistische Berechnung zu finden, sowie einen Regularisierungsparameter, der verwendet wird, um eine *Überanpassung* des Modells zu verhindern (d. h. eine logistische Berechnung, die mit den Trainingsdaten gut funktioniert, sich aber nicht gut verallgemeinern lässt, wenn sie auf neue Daten angewendet wird).

## Das Modell testen

Nachdem Sie nun ein trainiertes Modell haben, können Sie es mit den Daten, die Sie zurückgehalten haben, testen. Bevor Sie dies tun können, müssen Sie an den Testdaten dieselben Feature Engineering-Transformationen vornehmen wie an den Trainingsdaten (in diesem Fall den Inselnamen kodieren und die Messungen normalisieren). Anschließend können Sie das Modell verwenden, um Bezeichnungen für die Features in den Testdaten vorherzusagen und die vorhergesagten Bezeichnungen mit den tatsächlich bekannten Bezeichnungen zu vergleichen.

1. Verwenden Sie den folgenden Code, um die Testdaten vorzubereiten und dann Vorhersagen zu generieren:

    ```python
   # Prepare the test data
   indexedTestData = indexer.fit(test).transform(test).drop("Island")
   vectorizedTestData = numericColVector.transform(indexedTestData)
   scaledTestData = minMax.fit(vectorizedTestData).transform(vectorizedTestData)
   preppedTestData = featVect.transform(scaledTestData)[col("featuresVector").alias("features"), col("Species").alias("label")]
   
   # Get predictions
   prediction = model.transform(preppedTestData)
   predicted = prediction.select("features", "probability", col("prediction").astype("Int"), col("label").alias("trueLabel"))
   display(predicted)
    ```

    Die Ergebnisse umfassen die folgenden Spalten:
    
    - **features**: Die vorbereiteten Featuredaten aus dem Testdataset.
    - **probability**: Die vom Modell für jede Klasse berechnete Wahrscheinlichkeit. Diese besteht aus einem Vektor mit drei Wahrscheinlichkeitswerten (weil es drei Klassen gibt), die sich zu einem Gesamtwert von 1,0 addieren (es wird angenommen, dass der Pinguin mit 100%iger Wahrscheinlichkeit zu *einer* der drei Artklassen gehört).
    - **Vorhersage**: Die vorhergesagte Klassenbezeichnung (diejenige mit der höchsten Wahrscheinlichkeit).
    - **trueLabel**: Der tatsächliche bekannte Bezeichnungswert aus den Testdaten.
    
    Um die Effektivität des Modells zu bewerten, könnten Sie einfach die vorhergesagten und echten Bezeichnungen in diesen Ergebnissen vergleichen. Aussagekräftigere Metriken erhalten Sie jedoch, wenn Sie eine Modellauswertung verwenden – in diesem Fall eine Klassifizierungsauswertung für mehrere Klassen (da es mehrere mögliche Klassenbezeichnungen gibt).

1. Verwenden Sie den folgenden Code, um Auswertungsmetriken für ein Klassifizierungsmodell basierend auf den Ergebnissen aus den Testdaten abzurufen:

    ```python
   from pyspark.ml.evaluation import MulticlassClassificationEvaluator
   
   evaluator = MulticlassClassificationEvaluator(labelCol="label", predictionCol="prediction")
   
   # Simple accuracy
   accuracy = evaluator.evaluate(prediction, {evaluator.metricName:"accuracy"})
   print("Accuracy:", accuracy)
   
   # Individual class metrics
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
   
   # Weighted (overall) metrics
   overallPrecision = evaluator.evaluate(prediction, {evaluator.metricName:"weightedPrecision"})
   print("Overall Precision:", overallPrecision)
   overallRecall = evaluator.evaluate(prediction, {evaluator.metricName:"weightedRecall"})
   print("Overall Recall:", overallRecall)
   overallF1 = evaluator.evaluate(prediction, {evaluator.metricName:"weightedFMeasure"})
   print("Overall F1 Score:", overallF1)
    ```

    Zu den Auswertungsmetriken, die für die Klassifizierung mit mehreren Klassen berechnet werden, zählen:
    
    - **Genauigkeit:** Der Anteil aller Vorhersagen, die richtig waren.
    - Metriken pro Klasse:
      - **Präzision:** Der Anteil an Vorhersagen dieser Klasse, die korrekt waren.
      - **Trefferquote:** Der Anteil der tatsächlichen Instanzen dieser Klasse, die richtig vorhergesagt wurden.
      - **F1-Score:** Eine kombinierte Metrik für Genauigkeit und Abruf
    - Kombinierte (gewichtete) Metriken für Präzision, Abruf, und F1 für alle Klassen.
    
    > **Hinweis:** Auf den ersten Blick mag es so aussehen, als ob die Gesamtgenauigkeit die beste Methode zur Bewertung der Vorhersageleistung eines Modells darstellt. Sehen Sie sich jedoch dieses Beispiel an. Angenommen, Eselspinguine machen 95 % der Pinguinpopulation an Ihrem Studienstandort aus. Ein Modell, das immer die Bezeichnung **1** (die Klasse für Eselspinguine) vorhersagt, hat eine Genauigkeit von 0,95. Das bedeutet nicht, dass es ein gutes Modell ist, um anhand der Merkmale eine Pinguinart zu bestimmen! Aus diesem Grund untersuchen Datenwissenschaftler tendenziell zusätzliche Metriken, um besser zu verstehen, wie gut ein Klassifizierungsmodell Vorhersagen für jede mögliche Klassenbezeichnung trifft.

## Verwenden einer Pipeline

Sie haben Ihr Modell trainiert, indem Sie die erforderlichen Feature Engineering-Schritte durchgeführt und dann einen Algorithmus an die Daten angepasst haben. Um das Modell mit einigen Testdaten zur Erstellung von Vorhersagen zu verwenden (als *Rückschließen* bezeichnet), mussten Sie die gleichen Feature Engineering-Schritte auf die Testdaten anwenden. Eine effizientere Methode zum Erstellen und Verwenden von Modellen besteht darin, besteht darin, die Transformatoren, die zur Aufbereitung der Daten verwendet werden, und das Modell, mit dem sie trainiert werden, in einer *Pipeline* zu kapseln.

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

1. Verwenden Sie den folgenden Code, um die Pipeline auf die Testdaten anzuwenden:

    ```python
   prediction = model.transform(test)
   predicted = prediction.select("Features", "probability", col("prediction").astype("Int"), col("Species").alias("trueLabel"))
   display(predicted)
    ```

## Ausprobieren eines anderen Algorithmus

Bisher haben Sie ein Klassifizierungsmodell mit dem logistischen Regressionsalgorithmus trainiert. Nun wollen wir diese Phase in der Pipeline ändern, um einen anderen Algorithmus auszuprobieren.

1. Führen Sie den folgenden Code aus, um eine Pipeline zu erstellen, die einen Entscheidungsstruktur-Algorithmus verwendet:

    ```python
   from pyspark.ml import Pipeline
   from pyspark.ml.feature import StringIndexer, VectorAssembler, MinMaxScaler
   from pyspark.ml.classification import DecisionTreeClassifier
   
   catFeature = "Island"
   numFeatures = ["CulmenLength", "CulmenDepth", "FlipperLength", "BodyMass"]
   
   # Define the feature engineering and model steps
   catIndexer = StringIndexer(inputCol=catFeature, outputCol=catFeature + "Idx")
   numVector = VectorAssembler(inputCols=numFeatures, outputCol="numericFeatures")
   numScaler = MinMaxScaler(inputCol = numVector.getOutputCol(), outputCol="normalizedFeatures")
   featureVector = VectorAssembler(inputCols=["IslandIdx", "normalizedFeatures"], outputCol="Features")
   algo = DecisionTreeClassifier(labelCol="Species", featuresCol="Features", maxDepth=10)
   
   # Chain the steps as stages in a pipeline
   pipeline = Pipeline(stages=[catIndexer, numVector, numScaler, featureVector, algo])
   
   # Use the pipeline to prepare data and fit the model algorithm
   model = pipeline.fit(train)
   print ("Model trained!")
    ```

    Dieses Mal umfasst die Pipeline dieselben Phasen der Featurevorbereitung wie zuvor, verwendet aber einen *Entscheidungsstruktur*-Algorithmus, um das Modell zu trainieren.
    
   1. Führen Sie den folgenden Code aus, um die neue Pipeline mit den Testdaten zu verwenden:

    ```python
   # Get predictions
   prediction = model.transform(test)
   predicted = prediction.select("Features", "probability", col("prediction").astype("Int"), col("Species").alias("trueLabel"))
   
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

## Speichern des Modells

In der Realität würden Sie das Modell iterativ mit verschiedenen Algorithmen (und Parametern) trainieren, um das beste Modell für Ihre Daten zu finden. Für den Moment bleiben wir bei dem Entscheidungsstrukturmodell, das wir trainiert haben. Wir speichern es, damit wir es später mit einigen neuen Pinguinbeobachtungen verwenden können.

1. Verwenden Sie den folgenden Code, um das Modell zu speichern:

    ```python
   model.save("/models/penguin.model")
    ```

    Wenn Sie nun unterwegs waren und einen neuen Pinguin entdeckt haben, können Sie das gespeicherte Modell laden und es verwenden, um anhand der von Ihnen gemessenen Merkmale die Pinguinart bestimmen. Die Verwendung eines Modells zur Erstellung von Vorhersagen aus neuen Daten wird als *Rückschließen* bezeichnet.

1. Führen Sie den folgenden Code aus, um das Modell zu laden und es zu verwenden, um für eine neue Pinguinbeobachtung die Art vorherzusagen:

    ```python
   from pyspark.ml.pipeline import PipelineModel

   persistedModel = PipelineModel.load("/models/penguin.model")
   
   newData = spark.createDataFrame ([{"Island": "Biscoe",
                                     "CulmenLength": 47.6,
                                     "CulmenDepth": 14.5,
                                     "FlipperLength": 215,
                                     "BodyMass": 5400}])
   
   
   predictions = persistedModel.transform(newData)
   display(predictions.select("Island", "CulmenDepth", "CulmenLength", "FlipperLength", "BodyMass", col("prediction").alias("PredictedSpecies")))
    ```

## Bereinigung

Wählen Sie zunächst im Azure Databricks-Portal auf der Seite **Compute** Ihren Cluster und dann **&#9632; Beenden** aus, um ihn herunterzufahren.

Wenn Sie die Erkundung von Azure Databricks abgeschlossen haben, löschen Sie die erstellten Ressourcen, um unnötige Azure-Kosten zu vermeiden und Kapazität in Ihrem Abonnement freizugeben.

> **Weitere Informationen:** Weitere Informationen finden Sie in der [Spark MLLib-Dokumentation](https://spark.apache.org/docs/latest/ml-guide.html).
