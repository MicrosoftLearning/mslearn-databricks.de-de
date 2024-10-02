---
lab:
  title: Bereitstellung von Workloads mit Azure Databricks Workflows
---

# Bereitstellung von Workloads mit Azure Databricks Workflows

Azure Databricks Workflows bieten eine robuste Plattform für die effiziente Bereitstellung von Workloads. Mit Features wie Azure Databricks Jobs und Delta Live Tables können Benutzende komplexe Datenverarbeitung, maschinelles Lernen und Analysepipelines orchestrieren.

Dieses Lab dauert ungefähr **40** Minuten.

## Bereitstellen eines Azure Databricks-Arbeitsbereichs

> **Tipp**: Wenn Sie bereits über einen Azure Databricks-Arbeitsbereich verfügen, können Sie dieses Verfahren überspringen und Ihren vorhandenen Arbeitsbereich verwenden.

Diese Übung enthält ein Skript zum Bereitstellen eines neuen Azure Databricks-Arbeitsbereichs. Das Skript versucht, eine Azure Databricks-Arbeitsbereichsressource im *Premium*-Tarif in einer Region zu erstellen, in der Ihr Azure-Abonnement über ein ausreichendes Kontingent für die in dieser Übung erforderlichen Computekerne verfügt. Es wird davon ausgegangen, dass Ihr Benutzerkonto über ausreichende Berechtigungen im Abonnement verfügt, um eine Azure Databricks-Arbeitsbereichsressource zu erstellen. Wenn das Skript aufgrund unzureichender Kontingente oder Berechtigungen fehlschlägt, können Sie versuchen, [einen Azure Databricks-Arbeitsbereich interaktiv im Azure-Portal zu erstellen](https://learn.microsoft.com/azure/databricks/getting-started/#--create-an-azure-databricks-workspace).

1. Melden Sie sich in einem Webbrowser am [Azure-Portal](https://portal.azure.com) unter `https://portal.azure.com` an.

2. Verwenden Sie rechts neben der Suchleiste oben auf der Seite die Schaltfläche **[\>_]**, um eine neue Cloud Shell-Instanz im Azure-Portal zu erstellen. Wählen Sie eine ***PowerShell***-Umgebung aus, und erstellen Sie Speicher, falls Sie dazu aufgefordert werden. Die Cloud Shell bietet eine Befehlszeilenschnittstelle in einem Bereich am unteren Rand des Azure-Portals, wie hier gezeigt:

    ![Azure-Portal mit einem Cloud Shell-Bereich](./images/cloud-shell.png)

    > **Hinweis:** Wenn Sie zuvor eine Cloudshell erstellt haben, die eine *Bash*-Umgebung verwendet, verwenden Sie das Dropdownmenü links oben im Bereich „Cloudshell“, um sie in ***PowerShell*** zu ändern.

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
    - **Knotentyp**: Standard_D4ds_v5
    - **Beenden nach** *20* **Minuten Inaktivität**

1. Warten Sie, bis der Cluster erstellt wurde. Es kann ein oder zwei Minuten dauern.

    > **Hinweis**: Wenn Ihr Cluster nicht gestartet werden kann, verfügt Ihr Abonnement möglicherweise über ein unzureichendes Kontingent in der Region, in der Ihr Azure Databricks-Arbeitsbereich bereitgestellt wird. Details finden Sie unter [Der Grenzwert für CPU-Kerne verhindert die Clustererstellung](https://docs.microsoft.com/azure/databricks/kb/clusters/azure-core-limit). In diesem Fall können Sie versuchen, Ihren Arbeitsbereich zu löschen und in einer anderen Region einen neuen zu erstellen. Sie können einen Bereich als Parameter für das Setupskript wie folgt angeben: `./mslearn-databricks/setup.ps1 eastus`
        
## Erstellen eines Notebook und Erfassen von Daten

1. Verwenden Sie in der Randleiste den Link ** (+) Neu**, um ein **Notebook** zu erstellen.

2. Wählen Sie in der Dropdownliste **Verbinden** Ihren Cluster aus, wenn er noch nicht ausgewählt ist. Wenn der Cluster nicht ausgeführt wird, kann es eine Minute dauern, bis er gestartet wird.

3. Geben Sie in der ersten Zelle des Notebooks den folgenden Code ein, der mit *Shellbefehlen* die Datendateien von GitHub in das von Ihrem Cluster verwendete Dateisystem herunterlädt.

     ```python
    %sh
    rm -r /dbfs/workflow_lab
    mkdir /dbfs/workflow_lab
    wget -O /dbfs/workflow_lab/2019.csv https://github.com/MicrosoftLearning/mslearn-databricks/raw/main/data/2019_edited.csv
    wget -O /dbfs/workflow_lab/2020.csv https://github.com/MicrosoftLearning/mslearn-databricks/raw/main/data/2020_edited.csv
    wget -O /dbfs/workflow_lab/2021.csv https://github.com/MicrosoftLearning/mslearn-databricks/raw/main/data/2021_edited.csv
     ```

4. Verwenden Sie Menüoption **&#9656; Zelle Ausführen** links neben der Zelle, um sie auszuführen. Warten Sie dann, bis der vom Code ausgeführte Spark-Auftrag, abgeschlossen ist.

## Erstellen von Auftragsaufgaben

Sie implementieren Ihren Datenverarbeitungs- und Analyseworkflow mithilfe von Aufgaben. Ein Auftrag besteht aus einer oder mehreren Aufgaben. Sie können Auftragsaufgaben erstellen, die Notebooks, JARS, Delta Live Tables-Pipelines oder Python-, Scala- und Spark-Übertragungen sowie Java-Anwendungen ausführen. In dieser Übung werden Sie eine Aufgabe als Notebook erstellen, die Daten extrahiert, transformiert und in Visualisierungsdiagramme lädt. 

1. Verwenden Sie in der Randleiste den Link ** (+) Neu**, um ein **Notebook** zu erstellen.

2. Ändern Sie den Standardnamen des Notebooks (**Unbenanntes Notebook *[Datum]***) in `ETL task` und wählen Sie in der Dropdown-Liste **Verbinden** Ihren Cluster aus, falls er nicht bereits ausgewählt ist. Wenn der Cluster nicht ausgeführt wird, kann es eine Minute dauern, bis er gestartet wird.

3. Geben Sie in die erste Zelle des Notebooks den folgenden Code ein, der ein Schema für die Daten definiert und die Datensätze in einen Dataframe lädt:

    ```python
   from pyspark.sql.types import *
   from pyspark.sql.functions import *
   orderSchema = StructType([
        StructField("SalesOrderNumber", StringType()),
        StructField("SalesOrderLineNumber", IntegerType()),
        StructField("OrderDate", DateType()),
        StructField("CustomerName", StringType()),
        StructField("Email", StringType()),
        StructField("Item", StringType()),
        StructField("Quantity", IntegerType()),
        StructField("UnitPrice", FloatType()),
        StructField("Tax", FloatType())
   ])
   df = spark.read.load('/workflow_lab/*.csv', format='csv', schema=orderSchema)
   display(df.limit(100))
    ```

4. Verwenden Sie unter der vorhandenen Codezelle das Symbol **+**, um eine neue Codezelle hinzuzufügen. Geben Sie dann in die neue Zelle den folgenden Code ein und führen Sie ihn aus, um doppelte Zeilen zu entfernen und die `null`-Einträge durch die richtigen Werte zu ersetzen:

     ```python
    from pyspark.sql.functions import col
    df = df.dropDuplicates()
    df = df.withColumn('Tax', col('UnitPrice') * 0.08)
    df = df.withColumn('Tax', col('Tax').cast("float"))
     ```
    > **Hinweis**: Nach der Aktualisierung der Werte in der Spalte **Steuer** wird der Datentyp wieder auf `float` gesetzt. Dies ist darauf zurückzuführen, dass sich der Datentyp nach der Berechnung in `double` ändert. Da `double` einen höheren Speicherbedarf hat als `float`, ist es für die Leistung besser, die Spalte zurück nach `float` zu schreiben.

5. Führen Sie in einer neuen Codezelle den folgenden Code aus, um die Auftragsdaten zu aggregieren und zu gruppieren:

    ```python
   yearlySales = df.select(year("OrderDate").alias("Year")).groupBy("Year").count().orderBy("Year")
   display(yearlySales)
    ```

6. Wählen Sie oberhalb der Ergebnistabelle **+** und dann **Visualisierung** aus, um den Visualisierungs-Editor anzuzeigen, und wenden Sie dann die folgenden Optionen an:

   Registerkarte **Allgemeines**:
    - **Visualisierungstyp**: Balken
    - **X-Spalte**: Jahr
    - **Y-Spalte**: *Eine neue Spalte hinzufügen und***Count**auswählen. *Wenden Sie die Aggregation* **Summe** *aus*.
   
   Registerkarte **X-Achse**:
    - **Skalierung**: Kategorial

8. Wählen Sie **Speichern**.

## Erstellen des Workflows

Azure Databricks verwaltet die Aufgabenorchestrierung, Clusterverwaltung, Überwachung und Fehlerberichterstattung für alle Ihre Aufträge. Sie können Ihre Aufträge sofort ausführen, in regelmäßigen Abständen über ein benutzerfreundliches Planungssystem, wann immer neue Dateien an einem externen Speicherort eintreffen, oder kontinuierlich, um sicherzustellen, dass eine Instanz des Auftrags immer ausgeführt wird.

1. Wählen Sie in der linken Randleiste **Workflows**.

2. Wählen Sie im Bereich Workflows die Option **Auftrag erstellen**.

3. Ändern Sie den Standardauftragsnamen (**Neuer Auftrag *[Datum]***) in `ETL job`.

4. Geben Sie im Feld **Aufgabenname** einen Namen für die Aufgabe ein.

5. Wählen Sie im Dropdownmenü **Typ** **Notebook** aus.

6. Wählen Sie im Feld **Pfad** Ihr **ETL-Aufgaben-Notebook**.

7. Wählen Sie **Aufgabe erstellen**.

8. Wählen Sie **Jetzt ausführen** aus.

9. Sobald der Auftrag ausgeführt wird, können Sie seine Ausführung überwachen, indem Sie in der linken Randleiste **Auftragsausführungen** auswählen.

10. Nachdem der Auftrag erfolgreich ausgeführt wurde, können Sie ihn auswählen und dessen Ausgabe überprüfen.

Außerdem können Sie Aufträge auf einer getriggerten Basis ausführen, z. B. einen Workflow nach einem Zeitplan. Um eine regelmäßige Auftragsausführung zu planen, können Sie die Auftragsaufgabe öffnen und **Trigger hinzufügen** im rechten Randbereich wählen.

   ![Workflow-Aufgabenbereich](./images/workflow-schedule.png)
    
## Bereinigung

Wählen Sie im Azure Databricks Portal auf der Seite **Compute** Ihren Cluster aus und wählen Sie **&#9632; Stop**, um ihn herunterzufahren.

Wenn Sie die Erkundung von Azure Databricks abgeschlossen haben, löschen Sie die erstellten Ressourcen, um unnötige Azure-Kosten zu vermeiden und Kapazität in Ihrem Abonnement freizugeben.
