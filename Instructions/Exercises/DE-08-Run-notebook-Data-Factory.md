---
lab:
  title: Automatisieren eines Azure Databricks-Notebooks mit Azure Data Factory
---

# Automatisieren eines Azure Databricks-Notebooks mit Azure Data Factory

Sie können Notebooks in Azure Databricks verwenden, um Datentechnikaufgaben auszuführen, wie etwa das Verarbeiten von Datendateien und das Laden von Daten in Tabellen. Wenn Sie diese Aufgaben als Teil einer Datentechnikpipeline orchestrieren müssen, können Sie Azure Data Factory verwenden.

Diese Übung dauert ca. **40** Minuten.

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

    ```
    rm -r mslearn-databricks -f
    git clone https://github.com/MicrosoftLearning/mslearn-databricks
    ```

5. Nachdem das Repository geklont wurde, geben Sie den folgenden Befehl ein, um das Skript **setup.ps1** auszuführen, das einen Azure Databricks-Arbeitsbereich in einer verfügbaren Region bereitstellt:

    ```
    ./mslearn-databricks/setup.ps1
    ```

6. Wenn Sie dazu aufgefordert werden, wählen Sie aus, welches Abonnement Sie verwenden möchten (dies geschieht nur, wenn Sie Zugriff auf mehrere Azure-Abonnements haben).
7. Warten Sie, bis das Skript abgeschlossen ist. Dies dauert in der Regel etwa 5 Minuten, in einigen Fällen kann es jedoch länger dauern. Während Sie warten, lesen Sie den Artikel [Was ist Azure Data Factory?](https://docs.microsoft.com/azure/data-factory/introduction)

## Erstellen einer Azure Data Factory-Ressource

Zusätzlich zu Ihrem Azure Databricks-Arbeitsbereich müssen Sie eine Azure Data Factory-Ressource in Ihrem Abonnement bereitstellen.

1. Schließen Sie im Azure-Portal den Cloud Shell-Bereich, und navigieren Sie zur Ressourcengruppe ***msl-*xxxxxxx****, die vom Setupskript erstellt wurde (oder zur Ressourcengruppe, die Ihren vorhandenen Azure Databricks-Arbeitsbereich enthält).
1. Wählen Sie auf der Symbolleiste **+ Erstellen** aus, und suchen Sie nach `Data Factory`. Erstellen Sie dann eine neue **Data Factory**-Ressource mit den folgenden Einstellungen:
    - **Abonnement:** *Ihr Abonnement*
    - **Ressourcengruppe**: msl-*xxxxxxx* (oder die Ressourcengruppe, die Ihren vorhandenen Azure Databricks-Arbeitsbereich enthält)
    - **Name:** *Ein eindeutiger Name, z. B. **adf-xxxxxxx***
    - **Region:** *Dieselbe Region wie Ihr Azure Databricks-Arbeitsbereich (oder eine andere verfügbare Region, wenn diese nicht aufgeführt ist)*
    - **Version:** V2
1. Stellen Sie beim Erstellen der neuen Ressource sicher, dass die Ressourcengruppe sowohl den Azure Databricks-Arbeitsbereich als auch die Azure Data Factory-Ressourcen enthält.

## Erstellen eines Notebooks

Sie können Notizbücher in Ihrem Azure Databricks-Arbeitsbereich erstellen, um Code auszuführen, der in einer Reihe von Programmiersprachen geschrieben wurde. In dieser Übung erstellen Sie ein einfaches Notebook, das Daten aus einer Datei erfasst und in einem Ordner im Databricks-Dateisystem (Databricks File System, DBFS) speichert.

1. Navigieren Sie im Azure-Portal zur Ressourcengruppe **msl-*xxxxxxx***, die vom Skript erstellt wurde (oder zur Ressourcengruppe, die Ihren vorhandenen Azure Databricks-Arbeitsbereich enthält)
1. Wählen Sie die Ressource Ihres Azure Databricks-Diensts aus (sie trägt den Namen **databricks-*xxxxxxx***, wenn Sie das Setupskript zum Erstellen verwendet haben).
1. Verwenden Sie auf der Seite **Übersicht** für Ihren Arbeitsbereich die Schaltfläche **Arbeitsbereich starten**, um Ihren Azure Databricks-Arbeitsbereich auf einer neuen Browserregisterkarte zu öffnen. Melden Sie sich an, wenn Sie dazu aufgefordert werden.

    > **Tipp**: Während Sie das Databricks-Arbeitsbereichsportal verwenden, werden möglicherweise verschiedene Tipps und Benachrichtigungen angezeigt. Schließen Sie diese, und folgen Sie den Anweisungen, um die Aufgaben in dieser Übung auszuführen.

1. Zeigen Sie das Azure Databricks-Arbeitsbereichsportal an, und beachten Sie, dass die Randleiste auf der linken Seite Symbole für die verschiedenen Aufgaben enthält, die Sie ausführen können.
1. Verwenden Sie in der Randleiste den Link ** (+) Neu**, um ein **Notebook** zu erstellen.
1. Ändern Sie den Standardnamen des Notebooks (**Unbenanntes Notebook *[Datum]***) in `Process Data`.
1. Geben Sie in der ersten Zelle des Notebooks den folgenden Code ein (aber führen Sie ihn nicht aus), um eine Variable für den Ordner einzurichten, in dem dieses Notebook Daten speichert.

    ```python
   # Use dbutils.widget define a "folder" variable with a default value
   dbutils.widgets.text("folder", "data")
   
   # Now get the parameter value (if no value was passed, the default set above will be used)
   folder = dbutils.widgets.get("folder")
    ```

1. Verwenden Sie unter der vorhandenen Codezelle das Symbol **+**, um eine neue Codezelle hinzuzufügen. Geben Sie dann in der neuen Zelle den folgenden Code ein (aber führen Sie ihn nicht aus), um Daten herunterzuladen und im Ordner zu speichern:

    ```python
   import urllib3
   
   # Download product data from GitHub
   response = urllib3.PoolManager().request('GET', 'https://raw.githubusercontent.com/MicrosoftLearning/mslearn-databricks/main/data/products.csv')
   data = response.data.decode("utf-8")
   
   # Save the product data to the specified folder
   path = "dbfs:/{0}/products.csv".format(folder)
   dbutils.fs.put(path, data, True)
    ```

1. Wählen Sie in der Randleiste auf der linken Seite **Arbeitsbereich** aus, und stellen Sie sicher, dass Ihre **Process Data**-Notebooks aufgeführt sind. Sie verwenden Azure Data Factory, um das Notebook als Teil einer Pipeline auszuführen.

    > **Hinweis:** Das Notebook kann praktisch jede benötigte Datenverarbeitungslogik enthalten. Dies ist ein einfaches Beispiel, das das Prinzip veranschaulicht.

## Aktivieren der Azure Databricks-Integration in Azure Data Factory

Um Azure Databricks aus einer Azure Data Factory-Pipeline zu verwenden, müssen Sie einen verknüpften Dienst in Azure Data Factory erstellen, der den Zugriff auf Ihren Azure Databricks-Arbeitsbereich ermöglicht.

### Erstellen eines Zugriffstokens

1. Wählen Sie im Azure Databricks-Portal in der oberen rechten Menüleiste den Benutzernamen aus, und wählen Sie dann in der Dropdownliste **Benutzereinstellungen** aus.
1. Wählen Sie auf der Seite **Benutzereinstellungen** die Option **Entwickler** aus. Wählen Sie dann neben **Zugriffstoken** die Option **Verwalten** aus.
1. Wählen Sie **Neues Token generieren** aus, und generieren Sie ein neues Token mit dem Kommentar *Data Factory* und einer leeren Lebensdauer (damit das Token nicht abläuft). Achten Sie darauf, das Token zu **kopieren, wenn es angezeigt wird, <u>bevor</u> Sie *Fertig*** auswählen.
1. Fügen Sie das kopierte Token in eine Textdatei ein, damit Sie es später in dieser Übung zur Hand haben.

## Verwenden einer Pipeline zum Ausführen des Databricks-Notebooks

Nachdem Sie nun einen verknüpften Dienst erstellt haben, können Sie ihn in einer Pipeline verwenden, um das zuvor angezeigte Notebook auszuführen.

### Erstellen einer Pipeline

1. Wählen Sie in Azure Data Factory Studio im Navigationsbereich die Option **Autor** aus.
2. Verwenden Sie auf der Seite **Autor** im Bereich **Factory-Ressourcen** das Symbol **+**, um eine **Pipeline** hinzuzufügen.
3. Ändern Sie im Bereich **Eigenschaften** für die neue Pipeline den Namen in `Process Data with Databricks`. Verwenden Sie dann die Schaltfläche **Eigenschaften** (die **&#128463;<sub>*</sub>** ähnelt) am rechten Ende der Symbolleiste verwenden, um den Bereich **Eigenschaften** auszublenden.
4. Erweitern Sie im Bereich **Aktivitäten** die Option **Databricks**, und ziehen Sie eine **Notebook**-Aktivität auf die Oberfläche des Pipeline-Designers, wie hier gezeigt:
5. Wenn die neue **Notebook1**-Aktivität ausgewählt ist, legen Sie die folgenden Eigenschaften im unteren Bereich fest:
    - **Allgemein**:
        - **Name**: `Process Data`
    - **Azure Databricks**:
        - **Mit Databricks verknüpfter Dienst**: *Wählen Sie den verknüpften Dienst **AzureDatabricks** aus, den Sie zuvor erstellt haben.*
    - **Einstellungen**:
        - **Notebook-Pfad**: *Zum Ordner **Users/your_user_name** navigieren und das **Process Data**-Notebook* auswählen
        - **Basisparameter**: *Hinzufügen eines neuen Parameters namens `folder` mit dem Wert `product_data`*
6. Verwenden Sie die Schaltfläche **Validieren** oberhalb der Pipeline-Designeroberfläche, um die Pipeline zu validieren. Verwenden Sie dann die Schaltfläche **Alle veröffentlichen**, um sie zu veröffentlichen (speichern).

### Erstellen eines verknüpften Diensts in Azure Data Factory

1. Kehren Sie zum Azure-Portal zurück und wählen Sie in der Ressourcengruppe **msl-*xxxxxxx*** die Azure Data Factory-Ressource **adf*xxxxxxx*** aus.
2. Wählen Sie auf der Seite **Übersicht** die Option **Studio starten** aus, um das Azure Data Factory Studio zu öffnen. Melden Sie sich an, wenn Sie dazu aufgefordert werden.
3. Verwenden Sie in Azure Data Factory Studio das Symbol **>>**, um den Navigationsbereich auf der linken Seite zu erweitern. Wählen Sie dann die Seite **Verwalten** aus.
4. Wählen Sie auf der Seite **Verwalten** auf der Registerkarte **Verknüpfte Dienste** die Option **+ Neu** aus, um einen neuen verknüpften Dienst hinzuzufügen.
5. Wählen Sie im Bereich **Neuer verknüpfter Dienst** die Registerkarte **Compute** oben aus. Wählen Sie dann **Azure Databricks** aus.
6. Fahren Sie fort, und erstellen Sie den verknüpften Dienst mit den folgenden Einstellungen:
    - **Name**: `AzureDatabricks`
    - **Beschreibung:** `Azure Databricks workspace`
    - **Verbindung über Integration Runtime herstellen**: AutoResolveIntegrationRuntime
    - **Kontoauswahlmethode**: Aus Azure-Abonnement
    - **Azure-Abonnement**: *Wählen Sie Ihr Abonnement aus.*
    - **Databricks-Arbeitsbereich**: *Wählen Sie Ihren Arbeitsbereich **databricksxxxxxxx** aus.*
    - **Cluster auswählen**: Neuer Auftragscluster
    - **Databrick-Arbeitsbereichs-URL**: *Automatisch auf Ihre Databricks-Arbeitsbereichs-URL festgelegt*
    - **Authentifizierungstyp**: Zugriffstoken
    - **Zugriffstoken**: *Fügen Sie Ihr Zugriffstoken ein*
    - **Clusterversion**: 13.3 LTS (Spark 3.4.1, Scala 2.12)
    - **Clusterknotentyp**: Standard_D4ds_v5
    - **Python-Version**: 3
    - **Workeroptionen**: Behoben
    - **Worker**: 1

### Führen Sie die Pipeline aus.

1. Wählen Sie oberhalb der Oberfläche des Pipeline-Designers die Option **Trigger hinzufügen** und dann **Jetzt auslösen** aus.
2. Wählen Sie im Bereich **Pipelineausführung** die Option **OK** aus, um die Pipeline auszuführen.
3. Wählen Sie im Navigationsbereich auf der linken Seite **Überwachen** aus, und beobachten Sie die Pipeline **Daten mit Databricks verarbeiten** auf der Registerkarte **Pipelineausführungen**. Die Ausführung kann eine Weile dauern, da die Pipeline dynamisch einen Spark-Cluster erstellt und das Notebook ausführt. Sie können die Schaltfläche **&#8635; Aktualisieren** auf der Seite **Pipelineausführungen** verwenden, um den Status zu aktualisieren.

    > **Hinweis**: Wenn Ihre Pipeline nicht ausgeführt werden kann, verfügt Ihr Abonnement möglicherweise über ein unzureichendes Kontingent in der Region, in der Ihr Azure Databricks-Arbeitsbereich bereitgestellt wird, um einen Auftragscluster zu erstellen. Details finden Sie unter [Der Grenzwert für CPU-Kerne verhindert die Clustererstellung](https://docs.microsoft.com/azure/databricks/kb/clusters/azure-core-limit). In diesem Fall können Sie versuchen, Ihren Arbeitsbereich zu löschen und in einer anderen Region einen neuen zu erstellen. Sie können einen Bereich als Parameter für das Setupskript wie folgt angeben: `./setup.ps1 eastus`

4. Wenn die Ausführung erfolgreich ist, wählen Sie ihren Namen aus, um die Ausführungsdetails anzuzeigen. Wählen Sie dann auf der Seite **Daten mit Databricks verarbeiten** im Abschnitt **Aktivitätsausführungen** die Aktivität **Daten verarbeiten** aus, und verwenden Sie das zugehörige Symbol ***Ausgabe***, um den JSON-Ausgabe-Code aus der Aktivität anzuzeigen. Dieser sollte wie folgt aussehen:

    ```json
    {
        "runPageUrl": "https://adb-..../run/...",
        "effectiveIntegrationRuntime": "AutoResolveIntegrationRuntime (East US)",
        "executionDuration": 61,
        "durationInQueue": {
            "integrationRuntimeQueue": 0
        },
        "billingReference": {
            "activityType": "ExternalActivity",
            "billableDuration": [
                {
                    "meterType": "AzureIR",
                    "duration": 0.03333333333333333,
                    "unit": "Hours"
                }
            ]
        }
    }
    ```

## Bereinigen

Wenn Sie die Erkundung von Azure Databricks abgeschlossen haben, können Sie die erstellten Ressourcen löschen, um unnötige Azure-Kosten zu vermeiden und Kapazität in Ihrem Abonnement freizugeben.
