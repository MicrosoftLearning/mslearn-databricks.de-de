---
lab:
  title: Automatisieren der Datenerfassung und -verarbeitung mithilfe von Azure Databricks
---

# Automatisieren der Datenerfassung und -verarbeitung mithilfe von Azure Databricks

Databricks Jobs ist ein leistungsstarker Dienst, der die Automatisierung von Datenerfassungs- und Verarbeitungsworkflows ermöglicht. Es ermöglicht die Orchestrierung komplexer Datenpipelines, die Aufgaben wie das Erfassen von Rohdaten aus verschiedenen Quellen, das Transformieren dieser Daten mithilfe von Delta Live Tables und das Speichern in Delta Lake zur weiteren Analyse umfassen kann. Mit Azure Databricks können Benutzerinnen und Benutzer ihre Datenverarbeitungsaufgaben automatisch planen und ausführen, um sicherzustellen, dass Daten immer auf dem neuesten Stand sind und für Entscheidungsprozesse verfügbar sind.

Dieses Lab dauert ungefähr **20** Minuten.

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

7. Warten Sie, bis das Skript abgeschlossen ist. Dies dauert in der Regel etwa 5 Minuten, in einigen Fällen kann es jedoch länger dauern. Während Sie warten, lesen Sie den Artikel [Planen und Orchestrieren von Workflows](https://learn.microsoft.com/azure/databricks/jobs/) in der Azure Databricks-Dokumentation.

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

## Erstellen eines Notebooks und Abrufen von Quelldaten

1. Verwenden Sie in der Randleiste den Link **(+) Neu** um ein **Notebook** zu erstellen und ändern Sie den Standard-Notebooknamen (**Untitled Notebook *[date]***) auf **Datenverarbeitung**. Wählen Sie dann in der Dropdown-Liste **Verbinden mit** Ihren Cluster aus, falls er noch nicht ausgewählt ist. Wenn der Cluster nicht ausgeführt wird, kann es eine Minute dauern, bis er gestartet wird.

2. Geben Sie in der ersten Zelle des Notebooks den folgenden Code ein, der mit *Shellbefehlen* die Datendateien von GitHub in das von Ihrem Cluster verwendete Dateisystem herunterlädt.

     ```python
    %sh
    rm -r /dbfs/FileStore
    mkdir /dbfs/FileStore
    wget -O /dbfs/FileStore/sample_sales_data.csv https://github.com/MicrosoftLearning/mslearn-databricks/raw/main/data/sample_sales_data.csv
     ```

3. Verwenden Sie Menüoption **&#9656; Zelle Ausführen** links neben der Zelle, um sie auszuführen. Warten Sie dann, bis der vom Code ausgeführte Spark-Auftrag, abgeschlossen ist.

## Automatisieren der Datenverarbeitung mit Azure Databricks-Aufträgen

1. Ersetzen Sie den Code in der ersten Zeile des Notebooks durch den folgenden Code. Führen Sie ihn dann aus, um die Daten in einen Datenframe zu laden:

     ```python
    # Load the sample dataset into a DataFrame
    df = spark.read.csv('/FileStore/*.csv', header=True, inferSchema=True)
    df.show()
     ```

1. Fahren Sie mit der Maus unter die vorhandene Codezelle und verwenden Sie das angezeigte Symbol **+ Code**, um eine neue Codezelle hinzuzufügen. Geben Sie dann in die neue Zelle den folgenden Code ein und führen Sie ihn aus, um die Verkaufsdaten nach Produktkategorie zu aggregieren:

     ```python
    from pyspark.sql.functions import col, sum

    # Aggregate sales data by product category
    sales_by_category = df.groupBy('product_category').agg(sum('transaction_amount').alias('total_sales'))
    sales_by_category.show()
     ```

1. Verwenden Sie in der Randleiste den Link **(+) Neu**, um einen **Auftrag** zu erstellen.

1. Ändern Sie den Standardauftragsnamen (**Neuer Auftrag *[Datum]***) in `Automated job`.

1. Konfigurieren Sie die unbenannte Aufgabe im Auftrag mit den folgenden Einstellungen:
    - **Aufgabenname**: `Run notebook`
    - **Typ**: Notebook
    - **Quelle**: Arbeitsbereich
    - **Pfad**: *Wählen Sie Ihr* *Notebook für* die Datenverarbeitung aus
    - **Cluster**: *Cluster auswählen*

1. Wählen Sie **Aufgabe erstellen**.

1. Wählen Sie **Jetzt ausführen** aus

    **Tipp**: Im rechten Bereich können Sie unter **Zeitplan** die Option **Auslöser hinzufügen** auswählen und einen Zeitplan für die Ausführung des Auftrags (z. B. täglich, wöchentlich) einrichten. Für diese Übung werden wir sie jedoch manuell ausführen.

1. Wählen Sie im Auftragsbereich die Registerkarte **Ausführen** aus, und überwachen Sie die Ausführung des Auftrags.

1. Nach dem erfolgreichen Ausführen des Auftrags können Sie ihn in der Liste **Ausgeführt** auswählen und die Ausgabe überprüfen.

    Sie haben die Erfassung und Verarbeitung von Daten mithilfe von Azure Databricks Jobs erfolgreich eingerichtet und automatisiert. Sie können diese Lösung jetzt skalieren, um komplexere Datenpipelines zu verarbeiten und in andere Azure-Dienste für eine robuste Datenverarbeitungsarchitektur zu integrieren.

## Bereinigen

Wählen Sie zunächst im Azure Databricks-Portal auf der Seite **Compute** Ihren Cluster und dann **&#9632; Beenden** aus, um ihn herunterzufahren.

Wenn Sie die Erkundung von Azure Databricks abgeschlossen haben, löschen Sie die erstellten Ressourcen, um unnötige Azure-Kosten zu vermeiden und Kapazität in Ihrem Abonnement freizugeben.
