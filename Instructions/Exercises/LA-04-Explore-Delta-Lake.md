---
lab:
  title: Verwenden von Delta Lake in Azure Databricks
---

# Verwenden von Delta Lake in Azure Databricks

Delta Lake ist ein Open-Source-Projekt zur Erstellung einer transaktionalen Datenspeicherebene für Spark in einem Data Lake. Delta Lake bietet Unterstützung für relationale Semantik für Batch- und Streamingdatenvorgänge und ermöglicht die Erstellung einer *Lakehouse*-Architektur, in der Apache Spark zum Verarbeiten und Abfragen von Daten in Tabellen verwendet werden kann, die auf zugrunde liegenden Dateien im Data Lake basieren.

Dieses Lab dauert ungefähr **30** Minuten.

> **Hinweis**: Die Benutzeroberfläche von Azure Databricks wird kontinuierlich verbessert. Die Benutzeroberfläche kann sich seit der Erstellung der Anweisungen in dieser Übung geändert haben.

## Bereitstellen eines Azure Databricks-Arbeitsbereichs

> **Tipp**: Wenn Sie bereits über einen Azure Databricks-Arbeitsbereich verfügen, können Sie dieses Verfahren überspringen und Ihren vorhandenen Arbeitsbereich verwenden.

Diese Übung enthält ein Skript zum Bereitstellen eines neuen Azure Databricks-Arbeitsbereichs. Das Skript versucht, eine Azure Databricks-Arbeitsbereichsressource im *Premium*-Tarif in einer Region zu erstellen, in der Ihr Azure-Abonnement über ein ausreichendes Kontingent für die in dieser Übung erforderlichen Computekerne verfügt. Es wird davon ausgegangen, dass Ihr Benutzerkonto über ausreichende Berechtigungen im Abonnement verfügt, um eine Azure Databricks-Arbeitsbereichsressource zu erstellen. Wenn das Skript aufgrund unzureichender Kontingente oder Berechtigungen fehlschlägt, können Sie versuchen, [einen Azure Databricks-Arbeitsbereich interaktiv im Azure-Portal zu erstellen](https://learn.microsoft.com/azure/databricks/getting-started/#--create-an-azure-databricks-workspace).

1. Melden Sie sich in einem Webbrowser am [Azure-Portal](https://portal.azure.com) unter `https://portal.azure.com` an.
2. Verwenden Sie die Taste **[\>_]** rechts neben der Suchleiste oben auf der Seite, um eine neue Cloud Shell im Azure-Portal zu erstellen, und wählen Sie eine ***PowerShell***-Umgebung aus. Die Cloud Shell bietet eine Befehlszeilenschnittstelle in einem Bereich am unteren Rand des Azure-Portals, wie hier gezeigt:

    ![Azure-Portal mit einem Cloud Shell-Bereich](./images/cloud-shell.png)

    > **Hinweis**: Wenn Sie zuvor eine Cloud-Shell erstellt haben, die eine *Bash*-Umgebung verwendet, wechseln Sie zu ***PowerShell***.

3. Beachten Sie, dass Sie die Größe der Cloud-Shell ändern können, indem Sie die Trennlinie oben im Bereich ziehen oder die Symbole **&#8212;**, **&#10530;** und **X** oben rechts im Bereich verwenden, um den Bereich zu minimieren, zu maximieren und zu schließen. Weitere Informationen zur Verwendung von Azure Cloud Shell finden Sie in der [Azure Cloud Shell-Dokumentation](https://docs.microsoft.com/azure/cloud-shell/overview).

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

1. Wählen Sie in der linken Seitenleiste die Option **(+) Neue** Aufgabe und dann **Cluster** aus (ggf. im Untermenü **Mehr** suchen).

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

Jetzt erstellen wir ein Spark Notebook und importieren die Daten, mit denen wir in dieser Übung arbeiten werden.

1. Verwenden Sie in der Randleiste den Link ** (+) Neu**, um ein **Notebook** zu erstellen.

1. Ändern Sie den Standardnamen des Notebooks (**Unbenanntes Notebook *[Datum]***) in `Explore Delta Lake` und wählen Sie in der Dropdown-Liste **Verbinden** Ihren Cluster aus, falls er nicht bereits ausgewählt ist. Wenn der Cluster nicht ausgeführt wird, kann es eine Minute dauern, bis er gestartet wird.

1. Geben Sie in der ersten Zelle des Notebooks den folgenden Code ein, der mit *Shellbefehlen* die Datendateien von GitHub in das von Ihrem Cluster verwendete Dateisystem herunterlädt.

    ```python
    %sh
    rm -r /dbfs/delta_lab
    mkdir /dbfs/delta_lab
    wget -O /dbfs/delta_lab/products.csv https://raw.githubusercontent.com/MicrosoftLearning/mslearn-databricks/main/data/products.csv
    ```

1. Verwenden Sie Menüoption **&#9656; Zelle Ausführen** links neben der Zelle, um sie auszuführen. Warten Sie dann, bis der vom Code ausgeführte Spark-Auftrag, abgeschlossen ist.

1. Verwenden Sie unter der vorhandenen Codezelle das Symbol **+ Code**, um eine neue Codezelle hinzuzufügen. Geben Sie dann in die neue Zelle den folgenden Code ein, und führen Sie ihn aus, um die Daten aus der Datei zu laden und die ersten 10 Zeilen anzuzeigen.

    ```python
   df = spark.read.load('/delta_lab/products.csv', format='csv', header=True)
   display(df.limit(10))
    ```

## Laden der Dateidaten in eine Deltatabelle

Die Daten wurden in einen DataFrame geladen. Wir werden sie nun in einer Delta-Tabelle speichern.

1. Fügen Sie eine neue Codezelle hinzu, und verwenden Sie diese, um den folgenden Code auszuführen:

    ```python
   delta_table_path = "/delta/products-delta"
   df.write.format("delta").save(delta_table_path)
    ```

    Die Daten für eine Delta Lake-Tabelle werden im Parquet-Format gespeichert. Außerdem wird eine Protokolldatei erstellt, um die an den Daten vorgenommenen Änderungen zu verfolgen.

1. Fügen Sie eine neue Codezelle hinzu und führen Sie darin den folgenden Shell-Befehl aus, um den Inhalt des Ordners anzuzeigen, in dem die Deltadaten gespeichert wurden.

    ```
    %sh
    ls /dbfs/delta/products-delta
    ```

1. Die Dateidaten im Delta-Format können in ein **DeltaTable**-Objekt geladen werden, das Sie verwenden können, um die Daten in der Tabelle anzuzeigen und zu aktualisieren. Führen Sie den folgenden Code in einer neuen Zelle aus, um die Daten zu aktualisieren und den Preis des Produkts 771 um 10 % zu reduzieren.

    ```python
   from delta.tables import *
   from pyspark.sql.functions import *
   
   # Create a deltaTable object
   deltaTable = DeltaTable.forPath(spark, delta_table_path)
   # Update the table (reduce price of product 771 by 10%)
   deltaTable.update(
       condition = "ProductID == 771",
       set = { "ListPrice": "ListPrice * 0.9" })
   # View the updated data as a dataframe
   deltaTable.toDF().show(10)
    ```

    Die Aktualisierung wird in den Daten im Delta-Ordner gespeichert und in allen neuen DataFrames, die von dort geladen werden, widergespiegelt.

1. Führen Sie den folgenden Code aus, um einen neuen DataFrame aus den Delta-Tabellendaten zu erstellen:

    ```python
   new_df = spark.read.format("delta").load(delta_table_path)
   new_df.show(10)
    ```

## Erkunden von Protokollierung und *time-travel*

Datenänderungen werden protokolliert, sodass Sie die *time-travel*-Funktionen von Delta Lake verwenden können, um frühere Versionen der Daten anzuzeigen. 

1. Verwenden Sie in einer neuen Codezelle den folgenden Code, um die ursprüngliche Version der Produktdaten anzuzeigen:

    ```python
   new_df = spark.read.format("delta").option("versionAsOf", 0).load(delta_table_path)
   new_df.show(10)
    ```

1. Das Protokoll enthält den vollständigen Verlauf der Änderungen der Daten. Verwenden Sie den folgenden Code, um einen Datensatz der letzten 10 Änderungen anzuzeigen:

    ```python
   deltaTable.history(10).show(10, False, True)
    ```

## Erstellen von Katalogtabellen

Bisher haben Sie mit Delta-Tabellen gearbeitet, indem Sie Daten aus dem Ordner mit den Parquet-Dateien laden, auf denen die Tabelle basiert. Sie können *Katalogtabellen* definieren, die die Daten kapseln und eine benannte Tabellenentität bereitstellen, auf die Sie im SQL-Code verweisen können. Spark unterstützt zwei Arten von Katalogtabellen für Delta Lake:

- *Externe* Tabellen, die durch den Pfad zu den Dateien definiert werden, die die Tabellendaten enthalten
- *Verwaltete* Tabellen, die im Metastore definiert sind

### Erstellen einer externen Tabelle

1. Verwenden Sie den folgenden Code, um eine neue Datenbank mit dem Namen **AdventureWorks** zu erstellen, und erstellen Sie dann eine externe Tabelle mit dem Namen **ProductsExternal** in dieser Datenbank, basierend auf dem Pfad zu den zuvor definierten Delta-Dateien:

    ```python
   spark.sql("CREATE DATABASE AdventureWorks")
   spark.sql("CREATE TABLE AdventureWorks.ProductsExternal USING DELTA LOCATION '{0}'".format(delta_table_path))
   spark.sql("DESCRIBE EXTENDED AdventureWorks.ProductsExternal").show(truncate=False)
    ```

    Beachten Sie, dass die Eigenschaft **Location** der neuen Tabelle den Pfad enthält, den Sie angegeben haben.

1. Verwenden Sie den folgenden Code, um die Tabelle abzufragen:

    ```sql
   %sql
   USE AdventureWorks;
   SELECT * FROM ProductsExternal;
    ```

### Erstellen einer verwalteten Tabelle

1. Führen Sie den folgenden Code aus, um eine verwaltete Tabelle mit dem Namen **ProductsManaged** auf der Grundlage des Datenframes zu erstellen (und anschließend zu beschreiben), den Sie ursprünglich aus der Datei **products.csv** geladen haben (bevor Sie den Preis des Produkts 771 aktualisiert haben).

    ```python
   df.write.format("delta").saveAsTable("AdventureWorks.ProductsManaged")
   spark.sql("DESCRIBE EXTENDED AdventureWorks.ProductsManaged").show(truncate=False)
    ```

    Sie haben keinen Pfad für die von der Tabelle verwendeten Parquet-Dateien angegeben. Dieser wird für Sie im Hive-Metastore verwaltet und in der **Location**-Eigenschaft in der Tabellenbeschreibung angezeigt.

1. Verwenden Sie den folgenden Code, um die verwaltete Tabelle abzufragen, und beachten Sie, dass die Syntax dieselbe ist wie bei einer verwalteten Tabelle:

    ```sql
   %sql
   USE AdventureWorks;
   SELECT * FROM ProductsManaged;
    ```

### Vergleichen externer und verwalteter Tabellen

1. Verwenden Sie den folgenden Code, um die Tabellen in der Datenbank **AdventureWorks** auflisten:

    ```sql
   %sql
   USE AdventureWorks;
   SHOW TABLES;
    ```

1. Nun verwenden Sie den folgenden Code, um die Ordner anzuzeigen, auf denen diese Tabellen basieren:

    ```Bash
    %sh
    echo "External table:"
    ls /dbfs/delta/products-delta
    echo
    echo "Managed table:"
    ls /dbfs/user/hive/warehouse/adventureworks.db/productsmanaged
    ```

1. Verwenden Sie den folgenden Code, um beide Tabellen aus der Datenbank zu löschen:

    ```sql
   %sql
   USE AdventureWorks;
   DROP TABLE IF EXISTS ProductsExternal;
   DROP TABLE IF EXISTS ProductsManaged;
   SHOW TABLES;
    ```

1. Führen Sie nun die Zelle erneut aus, die den folgenden Code enthält, um den Inhalt der Delta-Ordner anzuzeigen:

    ```Bash
    %sh
    echo "External table:"
    ls /dbfs/delta/products-delta
    echo
    echo "Managed table:"
    ls /dbfs/user/hive/warehouse/adventureworks.db/productsmanaged
    ```

    Die Dateien für die verwaltete Tabelle werden automatisch gelöscht, wenn die Tabelle gelöscht wird. Die Dateien für die externe Tabelle werden jedoch beibehalten. Beim Löschen einer externen Tabelle werden nur die Tabellenmetadaten aus der Datenbank entfernt. Die Datendateien werden nicht gelöscht.

1. Verwenden Sie den folgenden Code, um eine neue Tabelle in der Datenbank zu erstellen, die auf den Delta-Dateien im Ordner **products-delta** basiert:

    ```sql
   %sql
   USE AdventureWorks;
   CREATE TABLE Products
   USING DELTA
   LOCATION '/delta/products-delta';
    ```

1. Verwenden Sie den folgenden Code, um die neue Tabelle abzufragen:

    ```sql
   %sql
   USE AdventureWorks;
   SELECT * FROM Products;
    ```

    Weil die Tabelle auf den vorhandenen Delta-Dateien basiert, die den protokollierten Änderungsverlauf enthalten, spiegelt sie die Änderungen wider, die Sie zuvor an den Produktdaten vorgenommen haben.

## Optimieren des Tabellenlayouts

Die physische Speicherung von Tabellendaten und zugeordneten Indexdaten kann neu organisiert werden, um Speicherplatz zu reduzieren und die E/A-Effizienz beim Zugriff auf die Tabelle zu verbessern. Dies ist besonders nützlich, nachdem wesentliche Einfüge-, Aktualisierungs- oder Löschvorgänge in einer Tabelle ausgeführt wurden.

1. Verwenden Sie in einer neuen Codezelle den folgenden Code, um das Layout zu optimieren und alte Versionen von Datendateien in der Delta-Tabelle zu bereinigen:

     ```python
    spark.sql("OPTIMIZE Products")
    spark.conf.set("spark.databricks.delta.retentionDurationCheck.enabled", "false")
    spark.sql("VACUUM Products RETAIN 24 HOURS")
     ```

Bei Delta Lake gibt es eine Sicherheitsüberprüfung, um zu verhindern, dass Sie einen gefährlichen VACUUM-Befehl ausführen. In Databricks Runtime gilt: Wenn Sie sicher sind, dass für diese Tabelle keine Vorgänge ausgeführt werden, die länger dauern als das Aufbewahrungsintervall, das Sie angeben möchten, können Sie diese Sicherheitsüberprüfung deaktivieren, indem Sie die Spark-Konfigurationseigenschaft `spark.databricks.delta.retentionDurationCheck.enabled` auf `false` festlegen.

> **Hinweis:** Wenn Sie VACUUM auf eine Delta-Tabelle anwenden, verlieren Sie die Möglichkeit der Zeitreise zurück zu einer Version, die älter als der angegebene Datenaufbewahrungszeitraum ist.

## Bereinigen

Wählen Sie zunächst im Azure Databricks-Portal auf der Seite **Compute** Ihren Cluster und dann **&#9632; Beenden** aus, um ihn herunterzufahren.

Wenn Sie die Erkundung von Azure Databricks abgeschlossen haben, löschen Sie die erstellten Ressourcen, um unnötige Azure-Kosten zu vermeiden und Kapazität in Ihrem Abonnement freizugeben.
