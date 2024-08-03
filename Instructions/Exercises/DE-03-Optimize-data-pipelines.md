---
lab:
  title: Optimieren von Datenpipelines für eine bessere Leistung in Azure Databricks
---

# Optimieren von Datenpipelines für eine bessere Leistung in Azure Databricks

Die Optimierung von Datenpipelines in Azure Databricks kann die Leistung und Effizienz erheblich steigern. Die Verwendung von Auto Loader für die inkrementelle Datenerfassung, gekoppelt mit der Speicherebene von Delta Lake, gewährleistet Zuverlässigkeit und ACID-Transaktionen. Durch die Implementierung von Salting kann Datenschiefe verhindert werden, während Z-Sortierungsclustering Dateilesevorgänge optimiert, indem verwandte Informationen zusammengeführt werden. Die Autooptimierungsfunktionen von Azure Databricks und der kostenbasierte Optimierer können die Leistung weiter verbessern, indem Einstellungen basierend auf den Workloadanforderungen angepasst werden.

Dieses Lab dauert ungefähr **30** Minuten.

## Bereitstellen eines Azure Databricks-Arbeitsbereichs

> **Tipp**: Wenn Sie bereits über einen Azure Databricks-Arbeitsbereich verfügen, können Sie dieses Verfahren überspringen und Ihren vorhandenen Arbeitsbereich verwenden.

Diese Übung enthält ein Skript zum Bereitstellen eines neuen Azure Databricks-Arbeitsbereichs. Das Skript versucht, eine Azure Databricks-Arbeitsbereichsressource im *Premium*-Tarif in einer Region zu erstellen, in der Ihr Azure-Abonnement über ein ausreichendes Kontingent für die in dieser Übung erforderlichen Computekerne verfügt. Es wird davon ausgegangen, dass Ihr Benutzerkonto über ausreichende Berechtigungen im Abonnement verfügt, um eine Azure Databricks-Arbeitsbereichsressource zu erstellen. Wenn das Skript aufgrund unzureichender Kontingente oder Berechtigungen fehlschlägt, können Sie versuchen, [einen Azure Databricks-Arbeitsbereich interaktiv im Azure-Portal zu erstellen](https://learn.microsoft.com/azure/databricks/getting-started/#--create-an-azure-databricks-workspace).

1. Melden Sie sich in einem Webbrowser am [Azure-Portal](https://portal.azure.com) unter `https://portal.azure.com` an.

2. Verwenden Sie rechts neben der Suchleiste oben auf der Seite die Schaltfläche **[\>_]**, um eine neue Cloud Shell-Instanz im Azure-Portal zu erstellen. Wählen Sie eine ***PowerShell***-Umgebung aus, und erstellen Sie Speicher, falls Sie dazu aufgefordert werden. Die Cloud Shell bietet eine Befehlszeilenschnittstelle in einem Bereich am unteren Rand des Azure-Portals, wie hier gezeigt:

    ![Azure-Portal mit einem Cloud Shell-Bereich](./images/cloud-shell.png)

    > **Hinweis**: Wenn Sie zuvor eine Cloud Shell erstellt haben, die eine *Bash*-Umgebung verwendet, ändern Sie diese mithilfe des Dropdownmenüs oben links im Cloud Shell-Bereich zu ***PowerShell***.

3. Beachten Sie, dass Sie die Größe der Cloud Shell durch Ziehen der Trennzeichenleiste oben im Bereich ändern können oder den Bereich mithilfe der Symbole **&#8212;**, **&#9723;** und **X** oben rechts minimieren, maximieren und schließen können. Weitere Informationen zur Verwendung von Azure Cloud Shell finden Sie in der [Azure Cloud Shell-Dokumentation](https://docs.microsoft.com/azure/cloud-shell/overview).

4. Geben Sie im PowerShell-Bereich die folgenden Befehle ein, um dieses Repository zu klonen:

     ```powershell
    rm -r mslearn-databricks -f
    git clone https://github.com/MicrosoftLearning/mslearn-databricks
     ```

5. Nachdem das Repository geklont wurde, geben Sie den folgenden Befehl ein, um das Skript **setup.ps1** auszuführen, das einen Azure Databricks-Arbeitsbereich in einer verfügbaren Region bereitstellt:

     ```powershell
    ./mslearn-databricks/setup.ps1
     ```

6. Wenn Sie dazu aufgefordert werden, wählen Sie aus, welches Abonnement Sie verwenden möchten (dies geschieht nur, wenn Sie Zugriff auf mehrere Azure-Abonnements haben).

7. Warten Sie, bis das Skript abgeschlossen ist. Dies dauert in der Regel etwa 5 Minuten, in einigen Fällen kann es jedoch länger dauern. Während Sie warten, lesen Sie den Artikel [Einführung in Delta Lake](https://docs.microsoft.com/azure/databricks/delta/delta-intro) in der Azure Databricks-Dokumentation.

## Erstellen eines Clusters

Azure Databricks ist eine verteilte Verarbeitungsplattform, die Apache Spark-*Cluster* verwendet, um Daten parallel auf mehreren Knoten zu verarbeiten. Jeder Cluster besteht aus einem Treiberknoten, um die Arbeit zu koordinieren, und Arbeitsknoten zum Ausführen von Verarbeitungsaufgaben. In dieser Übung erstellen Sie einen *Einzelknotencluster* , um die in der Lab-Umgebung verwendeten Computeressourcen zu minimieren (in denen Ressourcen möglicherweise eingeschränkt werden). In einer Produktionsumgebung erstellen Sie in der Regel einen Cluster mit mehreren Workerknoten.

> **Tipp**: Wenn Sie bereits über einen Cluster mit einer Runtime 13.3 LTS oder einer höheren Runtimeversion in Ihrem Azure Databricks-Arbeitsbereich verfügen, können Sie ihn verwenden, um diese Übung abzuschließen und dieses Verfahren zu überspringen.

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
    - **Databricks-Runtimeversion**: 13.3 LTS (Spark 3.4.1, Scala 2.12) oder höher
    - **Photonbeschleunigung verwenden**: Ausgewählt
    - **Knotentyp**: Standard_DS3_v2
    - **Beenden nach** *20* **Minuten Inaktivität**

1. Warten Sie, bis der Cluster erstellt wurde. Es kann ein oder zwei Minuten dauern.

    > **Hinweis**: Wenn Ihr Cluster nicht gestartet werden kann, verfügt Ihr Abonnement möglicherweise über ein unzureichendes Kontingent in der Region, in der Ihr Azure Databricks-Arbeitsbereich bereitgestellt wird. Details finden Sie unter [Der Grenzwert für CPU-Kerne verhindert die Clustererstellung](https://docs.microsoft.com/azure/databricks/kb/clusters/azure-core-limit). In diesem Fall können Sie versuchen, Ihren Arbeitsbereich zu löschen und in einer anderen Region einen neuen zu erstellen. Sie können einen Bereich als Parameter für das Setupskript wie folgt angeben: `./mslearn-databricks/setup.ps1 eastus`

## Erstellen eines Notebook und Erfassen von Daten

1. Verwenden Sie in der Randleiste den Link ** (+) Neu**, um ein **Notebook** zu erstellen. Wählen Sie in der Dropdownliste **Verbinden** Ihren Cluster aus, wenn er noch nicht ausgewählt ist. Wenn der Cluster nicht ausgeführt wird, kann es eine Minute dauern, bis er gestartet wird.

2. Geben Sie in der ersten Zelle des Notebooks den folgenden Code ein, der mit *Shellbefehlen* die Datendateien von GitHub in das von Ihrem Cluster verwendete Dateisystem herunterlädt.

     ```python
    %sh
    rm -r /dbfs/nyc_taxi_trips
    mkdir /dbfs/nyc_taxi_trips
    wget -O /dbfs/nyc_taxi_trips/yellow_tripdata_2021-01.parquet https://github.com/MicrosoftLearning/mslearn-databricks/raw/main/data/yellow_tripdata_2021-01.parquet
     ```

3. Geben Sie in einer neuen Zelle den folgenden Code ein, um das Dataset in einen Datenrahmen zu laden.
   
     ```python
    # Load the dataset into a DataFrame
    df = spark.read.parquet("/nyc_taxi_trips/yellow_tripdata_2021-01.parquet")
    display(df)
     ```

4. Verwenden Sie Menüoption **&#9656; Zelle Ausführen** links neben der Zelle, um sie auszuführen. Warten Sie dann, bis der vom Code ausgeführte Spark-Auftrag, abgeschlossen ist.

## Optimieren der Datenerfassung mit dem Autoloader:

Die Optimierung der Datenerfassung ist entscheidend für die effiziente Verarbeitung großer Datasets. Der Autoloader ist so konzipiert, dass neue Datendateien verarbeitet werden, sobald sie im Cloudspeicher ankommen und er unterstützt verschiedene Dateiformate und Cloudspeicherdienste. 

Der Autoloader stellt eine strukturierte Streamingquelle namens `cloudFiles` bereit. Mithilfe eines Eingabeverzeichnispfads im Clouddateispeicher verarbeitet die `cloudFiles`-Quelle automatisch neue Dateien, sobald diese eingehen. Dabei können auch bereits vorhandene Dateien in diesem Verzeichnis verarbeitet werden. 

1. Führen Sie den folgenden Code in einer neuen Zelle aus, um einen Datenstrom basierend auf dem Ordner, der die Beispieldaten enthält, zu erstellen:

     ```python
     df = (spark.readStream
             .format("cloudFiles")
             .option("cloudFiles.format", "parquet")
             .option("cloudFiles.schemaLocation", "/stream_data/nyc_taxi_trips/schema")
             .load("/nyc_taxi_trips/"))
     df.writeStream.format("delta") \
         .option("checkpointLocation", "/stream_data/nyc_taxi_trips/checkpoints") \
         .option("mergeSchema", "true") \
         .start("/delta/nyc_taxi_trips")
     display(df)
     ```

2. Führen Sie den folgenden Code in einer neuen Zelle aus, um dem Datenstrom eine neue Parquet-Datei hinzuzufügen:

     ```python
    %sh
    rm -r /dbfs/nyc_taxi_trips
    mkdir /dbfs/nyc_taxi_trips
    wget -O /dbfs/nyc_taxi_trips/yellow_tripdata_2021-02_edited.parquet https://github.com/MicrosoftLearning/mslearn-databricks/raw/main/data/yellow_tripdata_2021-02_edited.parquet
     ```
   
Die neue Datei verfügt über eine neue Spalte, sodass der Datenstrom mit einem `UnknownFieldException` Fehler beendet wird. Bevor Ihr Stream diesen Fehler auslöst, führt Autoloader einen Schemarückschluss für den letzten Mikrobatch von Daten durch und aktualisiert den Schemaspeicherort mit dem neuesten Schema, indem neue Spalten am Ende des Schemas zusammengeführt werden. Die Datentypen vorhandener Spalten bleiben unverändert.

3. Führen Sie die Streamingcodezelle erneut aus, und stellen Sie sicher, dass der Tabelle zwei neue Spalten hinzugefügt wurden:

   ![Delta-Tabelle mit neuen Spalten](./images/autoloader-new-columns.png)
   
> Hinweis: Die `_rescued_data` Spalte enthält alle Daten, die aufgrund von Typkonflikten, Groß-/Kleinschreibungskonflikten oder fehlenden Spalten aus dem Schema nicht analysiert werden.

4. Wählen Sie **Unterbrechen** aus, um das Datenstreaming zu beenden.
   
Die Streamingdaten werden in Delta-Tabellen geschrieben. Delta Lake bietet eine Reihe von Verbesserungen gegenüber herkömmlichen Parquet-Dateien, einschließlich ACID-Transaktionen, Schemaentwicklung, Zeitreise und vereint Streaming- und Batchdatenverarbeitung, was es zu einer leistungsstarken Lösung für die Verwaltung von Big Data-Workloads macht.

## Optimierte Datentransformation

Datenschiefe ist eine erhebliche Herausforderung bei der verteilten Datenverarbeitung, insbesondere bei der Big Data-Verarbeitung mit Frameworks wie Apache Spark. Salting ist eine effektive Technik zum Optimieren der Datenschiefe, bei der den Schlüsseln vor der Partitionierung eine Zufallskomponente, das so genannte „Salt“, hinzugefügt wird. Dieser Prozess trägt dazu bei, Daten gleichmäßiger über Partitionen zu verteilen, was zu einer ausgewogeneren Workload und einer verbesserten Leistung führt.

1. Führen Sie in einer neuen Zelle den folgenden Code aus, um eine große schiefe Partition in kleinere Partitionen zu unterteilen, indem Sie eine *Salz*-Spalte mit zufälligen ganzzahligen Zahlen anfügen:

     ```python
    from pyspark.sql.functions import lit, rand

    # Convert streaming DataFrame back to batch DataFrame
    df = spark.read.parquet("/nyc_taxi_trips/*.parquet")
     
    # Add a salt column
    df_salted = df.withColumn("salt", (rand() * 100).cast("int"))

    # Repartition based on the salted column
    df_salted.repartition("salt").write.format("delta").mode("overwrite").save("/delta/nyc_taxi_trips_salted")

    display(df_salted)
     ```   

## Speicher optimieren

Delta Lake bietet eine Reihe von Optimierungsbefehlen, die die Leistung und Verwaltung von Datenspeichern erheblich verbessern können. Der `optimize` Befehl wurde entwickelt, um die Abfragegeschwindigkeit zu verbessern, indem Daten effizienter durch Techniken wie Komprimierung und Z-Sortierung organisiert werden.

Die Komprimierung konsolidiert kleinere Dateien in größere Dateien, was besonders für Leseabfragen von Vorteil sein kann. Bei der Z-Sortierung werden Datenpunkte so angeordnet, dass zusammengehörige Informationen nahe beieinander gespeichert werden, wodurch sich die Zeit, die für den Zugriff auf diese Daten bei Abfragen benötigt wird, verringert.

1. Führen Sie in einer neuen Zelle den folgenden Code aus, um die Komprimierung in der Delta-Tabelle auszuführen:

     ```python
    from delta.tables import DeltaTable

    delta_table = DeltaTable.forPath(spark, "/delta/nyc_taxi_trips")
    delta_table.optimize().executeCompaction()
     ```

2. Führen Sie in einer neuen Zelle den folgenden Code aus, um Z-Sortierungsclustering auszuführen:

     ```python
    delta_table.optimize().executeZOrderBy("tpep_pickup_datetime")
     ```

Mit dieser Technik werden verwandte Informationen in derselben Gruppe von Dateien zusammengeführt, um die Abfrageleistung zu verbessern.

## Bereinigung

Wählen Sie zunächst im Azure Databricks-Portal auf der Seite **Compute** Ihren Cluster und dann **&#9632; Beenden** aus, um ihn herunterzufahren.

Wenn Sie die Erkundung von Azure Databricks abgeschlossen haben, löschen Sie die erstellten Ressourcen, um unnötige Azure-Kosten zu vermeiden und Kapazität in Ihrem Abonnement freizugeben.
