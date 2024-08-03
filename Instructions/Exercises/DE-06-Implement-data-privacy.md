---
lab:
  title: Implementieren von Datenschutz und Governance mithilfe des Microsoft Purview und Unity Catalog mit Azure Databricks
---

# Implementieren von Datenschutz und Governance mithilfe des Microsoft Purview und Unity Catalog mit Azure Databricks

Microsoft Purview ermöglicht eine umfassende Datengovernance in Ihrem gesamten Datenbestand und lässt sich nahtlos in Azure Databricks integrieren, um Lakehouse-Daten zu verwalten und Metadaten in die Datenzuordnung einzubinden. Unity Catalog verbessert dies durch die Bereitstellung zentralisierter Datenverwaltung und -governance und vereinfacht die Sicherheit und Compliance in Databricks-Arbeitsbereichen.

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

## Einrichten des Unity Catalog

In den Metastore-Registern von Unity Catalog werden Metadaten über sicherheitsrelevante Objekte (z.B. Tabellen, Datenträger, externe Speicherorte und Freigaben) und die Berechtigungen für den Zugriff auf diese Objekte gespeichert. Jeder Metastore stellt einen 3-Level-Namespace (`catalog`.`schema``table`) zur Verfügung, durch den Daten organisiert werden können. Sie müssen einen Metastore für jede Region erstellen, in der Ihre Organisation tätig ist. Um mit Unity Catalog zu arbeiten, müssen sich die Benutzer in einem Arbeitsbereich befinden, der einem Metastore in ihrer Region zugeordnet ist.

1. Wählen Sie in der Randleiste **Katalog** aus.

2. Im Katalog-Explorer sollte ein Standard-Unity Catalog mit Ihrem Arbeitsbereichsnamen (**databricks-*xxxxx***, wenn Sie das Setupskript zum Erstellen verwendet haben) vorhanden sein. Wählen Sie den Katalog aus, und wählen Sie dann oben im rechten Bereich **Schema erstellen** aus.

3. Benennen Sie das neue Schema **E-Commerce**, wählen Sie den Speicherort aus, der mit Ihrem Arbeitsbereich erstellt wurde, und wählen Sie **Erstellen** aus.

4. Wählen Sie Ihren Katalog aus, und wählen Sie im rechten Bereich die Registerkarte ** Arbeitsbereiche** aus. Vergewissern Sie sich, dass Ihr Arbeitsbereich auf `Read & Write` zugreifen kann.

## Erfassen von Beispieldaten in Azure Databricks

1. Herunterhladen der Beispieldaten:
   * [customers.csv](https://github.com/MicrosoftLearning/mslearn-databricks/raw/main/data/DE-05/customers.csv)
   * [products.csv](https://github.com/MicrosoftLearning/mslearn-databricks/raw/main/data/DE-05/products.csv)
   * [sales.csv](https://github.com/MicrosoftLearning/mslearn-databricks/raw/main/data/DE-05/sales.csv)

2. Wählen Sie im Azure Databricks-Arbeitsbereich oben im Katalog-Explorer die Option **+** und dann **Daten hinzufügen** aus.

3. Wählen Sie im neuen Fenster **Dateien auf Volume hochladen** aus.

4. Navigieren Sie im neuen Fenster zu Ihrem `ecommerce` Schema, erweitern Sie es, und wählen Sie **Volume erstellen** aus.

5. Benennen Sie das neue Volume **sample_data,** und wählen Sie **Erstellen** aus.

6. Wählen Sie das neue Volume aus, und laden Sie die Dateien `customers.csv`, `products.csv` und `sales.csv` hoch. Wählen Sie die Option **Hochladen**.

7. Verwenden Sie in der Randleiste den Link ** (+) Neu**, um ein **Notebook** zu erstellen. Wählen Sie in der Dropdownliste **Verbinden** Ihren Cluster aus, wenn er noch nicht ausgewählt ist. Wenn der Cluster nicht ausgeführt wird, kann es eine Minute dauern, bis er gestartet wird.

8. Geben Sie in der ersten Zelle des Notebooks den folgenden Code ein, um Tabellen aus den CSV-Dateien zu erstellen:

     ```python
    # Load Customer Data
    customers_df = spark.read.format("csv").option("header", "true").load("/Volumes/databricksxxxxxxx/ecommerce/sample_data/customers.csv")
    customers_df.write.saveAsTable("ecommerce.customers")

    # Load Sales Data
    sales_df = spark.read.format("csv").option("header", "true").load("/Volumes/databricksxxxxxxx/ecommerce/sample_data/sales.csv")
    sales_df.write.saveAsTable("ecommerce.sales")

    # Load Product Data
    products_df = spark.read.format("csv").option("header", "true").load("/Volumes/databricksxxxxxxx/ecommerce/sample_data/products.csv")
    products_df.write.saveAsTable("ecommerce.products")
     ```

>**Hinweis:** Ersetzen Sie im `.load` Dateipfad `databricksxxxxxxx` durch Ihren Katalognamen.

9. Navigieren Sie im Katalog-Explorer zum `sample_data` Volume, und überprüfen Sie, ob sich die neuen Tabellen darin befinden.
    
## Einrichten von Microsoft Purview

Microsoft Purview ist ein einheitlicher Datengovernancedienst, der Organisationen dabei hilft, ihre Daten in verschiedenen Umgebungen zu verwalten und zu schützen. Mit Features wie Verhinderung von Datenverlust, Information Protection und Complianceverwaltung bietet Microsoft Purview Tools zum Verständnis, Verwalten und Schützen von Daten während ihres gesamten Lebenszyklus.

1. Navigieren Sie zum [Azure-Portal](https://portal.azure.com/).

2. Wählen Sie **Ressource erstellen** aus, und suchen Sie nach **Microsoft Purview**.

3. Erstellen Sie eine **Microsoft Purview**- Ressource mit den folgenden Einstellungen:
    - **Abonnement:** *Wählen Sie Ihr Azure-Abonnement aus.*
    - **Ressourcengruppe**: *Wählen Sie dieselbe Ressourcengruppe wie Ihr Azure Databricks-Arbeitsbereich aus*
    - **Microsoft Purview-Kontoname**: *Ein eindeutiger Name Ihrer Wahl*
    - **Standort**: *Wählen Sie dieselbe Region wie Ihr Azure Databricks-Arbeitsbereich aus*

4. Klicken Sie auf **Überprüfen + erstellen**. Warten Sie auf die Überprüfung, und wählen Sie dann **Erstellen** aus.

5. Warten Sie, bis die Bereitstellung abgeschlossen ist. Wechseln Sie dann zur bereitgestellten Azure Purview-Ressource im Azure-Portal.

6. Navigieren Sie im Microsoft Purview-Governanceportal in der Randleiste zum Abschnitt **Datenzuordnung**.

7. Wählen Sie im Bereich **Datenquellen** die Option **Registrieren** aus.

8. Suchen Sie im Fenster **Datenquelle registrieren** nach **Azure Databricks**, und wählen Sie sie aus. Wählen Sie **Weiter**.

9. Geben Sie Ihrer Datenquelle einen eindeutigen Namen, und wählen Sie dann Ihren Azure Databricks-Arbeitsbereich aus. Wählen Sie **Registrieren** aus.

## Implementieren von Datenschutz- und Governancerichtlinien

1. Wählen Sie im Abschnitt **Datenzuordnung** der Randleiste **Klassifizierungen** aus.

2. Wählen Sie im Bereich **Klassifizierungen****+Neu** aus, und erstellen Sie eine neue Klassifizierung namens **PII** (personenbezogene Informationen). Wählen Sie **OK** aus.

3. Wählen Sie den **Datenkatalog** in der Randleiste aus, und navigieren Sie zur Tabelle **Kundinnen und Kunden**.

4. Wenden Sie die PII-Klassifizierung auf die Spalten „E-Mail“ und „Telefon“ an.

5. Wechseln Sie zu Azure Databricks, und öffnen Sie das zuvor erstellte Notebook.
 
6. Führen Sie in einer neuen Zelle den folgenden Code aus, um eine Datenzugriffsrichtlinie zu erstellen, um den Zugriff auf PII-Daten einzuschränken.

     ```sql
    CREATE OR REPLACE TABLE ecommerce.customers (
      customer_id STRING,
      name STRING,
      email STRING,
      phone STRING,
      address STRING,
      city STRING,
      state STRING,
      zip_code STRING,
      country STRING
    ) TBLPROPERTIES ('data_classification'='PII');

    GRANT SELECT ON TABLE ecommerce.customers TO ROLE data_scientist;
    REVOKE SELECT (email, phone) ON TABLE ecommerce.customers FROM ROLE data_scientist;
     ```

7. Versuchen Sie, die Kundentabelle als Benutzerin oder Benutzer mit der Rolle data_scientist abzufragen. Überprüfen Sie, ob der Zugriff auf PII-Spalten (E-Mail und Telefon) eingeschränkt ist.

## Bereinigung

Wählen Sie zunächst im Azure Databricks-Portal auf der Seite **Compute** Ihren Cluster und dann **&#9632; Beenden** aus, um ihn herunterzufahren.

Wenn Sie die Erkundung von Azure Databricks abgeschlossen haben, löschen Sie die erstellten Ressourcen, um unnötige Azure-Kosten zu vermeiden und Kapazität in Ihrem Abonnement freizugeben.
