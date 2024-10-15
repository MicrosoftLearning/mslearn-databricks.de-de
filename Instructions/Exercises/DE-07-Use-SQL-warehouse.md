---
lab:
  title: Verwenden eines SQL-Warehouse in Azure Databricks
---

# Verwenden eines SQL-Warehouse in Azure Databricks

SQL ist eine Branchenstandardsprache zum Abfragen und Bearbeiten von Daten. Viele Datenanalysten führen Datenanalysen mithilfe von SQL aus, um Tabellen in einer relationalen Datenbank abzufragen. Azure Databricks umfasst SQL-Funktionen, die auf Spark- und Delta Lake-Technologien basieren, um eine relationale Datenbankebene über Dateien in einem Data Lake bereitzustellen.

Diese Übung dauert ca. **30** Minuten.

## Bereitstellen eines Azure Databricks-Arbeitsbereichs

> **Tipp**: Wenn Sie bereits über einen *Premium* - oder *Testarbeitsbereich* für Azure Databricks verfügen, können Sie dieses Verfahren überspringen und Ihren vorhandenen Arbeitsbereich verwenden.

Diese Übung enthält ein Skript zum Bereitstellen eines neuen Azure Databricks-Arbeitsbereichs. Das Skript versucht, eine Azure Databricks-Arbeitsbereichsressource im *Premium*-Tarif in einer Region zu erstellen, in der Ihr Azure-Abonnement über ein ausreichendes Kontingent für die in dieser Übung erforderlichen Computekerne verfügt. Es wird davon ausgegangen, dass Ihr Benutzerkonto über ausreichende Berechtigungen im Abonnement verfügt, um eine Azure Databricks-Arbeitsbereichsressource zu erstellen. Wenn das Skript aufgrund unzureichender Kontingente oder Berechtigungen fehlschlägt, können Sie versuchen, [einen Azure Databricks-Arbeitsbereich interaktiv im Azure-Portal zu erstellen](https://learn.microsoft.com/azure/databricks/getting-started/#--create-an-azure-databricks-workspace).

1. Melden Sie sich in einem Webbrowser am [Azure-Portal](https://portal.azure.com) unter `https://portal.azure.com` an.
2. Verwenden Sie rechts neben der Suchleiste oben auf der Seite die Schaltfläche **[\>_]**, um eine neue Cloud Shell-Instanz im Azure-Portal zu erstellen. Wählen Sie eine ***PowerShell***-Umgebung aus, und erstellen Sie Speicher, falls Sie dazu aufgefordert werden. Die Cloud Shell bietet eine Befehlszeilenschnittstelle in einem Bereich am unteren Rand des Azure-Portals, wie hier gezeigt:

    ![Azure-Portal mit einem Cloud Shell-Bereich](./images/cloud-shell.png)

    > **Hinweis:** Wenn Sie zuvor eine Cloudshell erstellt haben, die eine *Bash*-Umgebung verwendet, verwenden Sie das Dropdownmenü links oben im Bereich „Cloudshell“, um sie in ***PowerShell*** zu ändern.

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
7. Warten Sie, bis das Skript abgeschlossen ist. Dies dauert in der Regel etwa 5 Minuten, in einigen Fällen kann es jedoch länger dauern. Während Sie warten, lesen Sie den Artikel [Was ist Data Warehousing in Azure Databricks?](https://learn.microsoft.com/azure/databricks/sql/) in der Dokumentation zu Azure Databricks.

## Anzeigen und Starten eines SQL-Warehouse

1. Wenn die Azure Databricks-Arbeitsbereichsressource bereitgestellt wurde, wechseln Sie im Azure-Portal zu dieser Ressource.
1. Verwenden Sie auf der Seite **Übersicht** für Ihren Azure Databricks-Arbeitsbereich die Schaltfläche **Arbeitsbereich starten**, um Ihren Azure Databricks-Arbeitsbereich auf einer neuen Browserregisterkarte zu öffnen. Melden Sie sich an, wenn Sie dazu aufgefordert werden.

    > **Tipp**: Während Sie das Databricks-Arbeitsbereichsportal verwenden, werden möglicherweise verschiedene Tipps und Benachrichtigungen angezeigt. Schließen Sie diese, und folgen Sie den Anweisungen, um die Aufgaben in dieser Übung auszuführen.

1. Zeigen Sie das Azure Databricks-Arbeitsbereichsportal an, und beachten Sie, dass die Randleiste auf der linken Seite die Namen der Aufgabenkategorien enthält.
1. Wählen Sie in der Randleiste unter **SQL** die Option **SQL-Warehouses** aus.
1. Beachten Sie, dass der Arbeitsbereich bereits ein SQL Warehouse mit dem Namen **Starter Warehouse** enthält.
1. Wählen Sie im Menü **Aktionen** (**&#8285;**) für das SQL-Warehouse die Option **Bearbeiten** aus. Legen Sie dann die Eigenschaft **Clustergröße** auf **2X-Klein** fest, und speichern Sie Ihre Änderungen.
1. Verwenden Sie die Schaltfläche **Start**, um das SQL-Warehouse zu starten (was ein oder zwei Minuten dauern kann).

> **Hinweis**: Wenn Ihr SQL-Warehouse nicht gestartet werden kann, verfügt Ihr Abonnement möglicherweise über ein unzureichendes Kontingent in der Region, in der Ihr Azure Databricks-Arbeitsbereich bereitgestellt wird. Siehe [Erforderliches Azure vCPU-Kontingent](https://docs.microsoft.com/azure/databricks/sql/admin/sql-endpoints#required-azure-vcpu-quota) für Details. In diesem Fall können Sie versuchen, eine Kontingenterhöhung anzufordern, wie in der Fehlermeldung beschrieben wird, die beim Fehlschlagen des Starts des Warehouse angezeigt wird. Alternativ können Sie versuchen, Ihren Arbeitsbereich zu löschen und in einer anderen Region einen neuen zu erstellen. Sie können einen Bereich als Parameter für das Setupskript wie folgt angeben: `./setup.ps1 eastus`

## Erstellen eines Datenbankschemas

1. Wenn Ihr SQL Warehouse *ausgeführt* wird, wählen Sie in der Seitenleiste **SQL-Editor** aus.
2. Beachten Sie im Bereich **Schemabrowser**, dass der Katalog *hive_metastore* eine Datenbank mit dem Namen **default** enthält.
3. Geben Sie im Bereich **Neue Abfrage** den folgenden SQL-Code ein:

    ```sql
   CREATE DATABASE retail_db;
    ```

4. Verwenden Sie die Schaltfläche **&#9658; Ausführen (1000)**, um den SQL-Code auszuführen.
5. Wenn der Code erfolgreich ausgeführt wurde, verwenden Sie im Bereich **Schemabrowser** die Schaltfläche „Aktualisieren“ am oberen Rand des Bereichs, um die Liste zu aktualisieren. Erweitern Sie dann **hive_metastore** und **retail_db**, und stellen Sie fest, dass die Datenbank zwar erstellt wurde, aber keine Tabellen enthält.

Sie können die Datenbank **default** für Ihre Tabellen verwenden, aber beim Erstellen eines analytischen Datenspeichers empfiehlt es sich, für spezifische Daten benutzerdefinierte Datenbanken zu erstellen.

## Erstellen einer Tabelle

1. Laden Sie die Datei [`products.csv`](https://raw.githubusercontent.com/MicrosoftLearning/mslearn-databricks/main/data/products.csv) auf Ihren lokalen Computer herunter und speichern Sie sie als **produkte.csv**.
1. Wählen Sie im Azure Databricks-Arbeitsbereichsportal in der Randleiste **(+) Neu** und dann ** Daten** aus.
1. Wählen Sie auf der Seite **Daten hinzufügen** die Option **Tabelle erstellen oder ändern** aus und laden Sie die Datei **products.csv** hoch, die Sie auf Ihren Computer heruntergeladen haben.
1. Wählen Sie auf der Seite **Tabelle aus Dateiupload erstellen oder ändern** das Schema **retail_db** aus und legen Sie den Tabellennamen auf **Produkte** fest. Wählen Sie dann unten rechts auf der Seite die Option **Tabelle erstellen** aus.
1. Wenn die Tabelle erstellt wurde, überprüfen Sie die Details.

Die Möglichkeit zum Erstellen einer Tabelle durch Importieren von Daten aus einer Datei erleichtert das Auffüllen einer Datenbank. Sie können Auch Spark SQL verwenden, um Tabellen mithilfe von Code zu erstellen. Die Tabellen selbst sind Metadatendefinitionen im Hive-Metastore, und die darin enthaltenen Daten werden im Delta-Format im Databricks File System(DBFS)-Speicher gespeichert.

## Erstellen eines Dashboards

1. Klicken Sie in der Seitenleiste auf **(+) Neu**, und wählen Sie dann **Dashboard** aus.
2. Wählen Sie den neuen Dashboard-Namen aus und ändern Sie ihn in `Retail Dashboard`.
3. Wählen Sie auf der Registerkarte **Daten** die Option **Aus SQL erstellen** aus, und verwenden Sie die folgende Abfrage:

    ```sql
   SELECT ProductID, ProductName, Category
   FROM retail_db.products; 
    ```

4. Wählen Sie **Ausführen** aus und benennen Sie dann das Dataset „Unbenannt“ in `Products and Categories` um.
5. Wählen Sie die Registerkarte **Canvas** und dann **Visualisierung hinzufügen** aus.
6. Legen Sie im Visualisierungs-Editor die folgenden Eigenschaften fest:
    
    - **Dataset**: Produkte und Kategorien
    - **Visualisierung**: Balken
    - **X-Achse**: COUNT(ProductID)
    - **Y-Achse**: Kategorie

7. Wählen Sie **Veröffentlichen** aus, um das Dashboard so anzuzeigen, wie es Benutzende sehen werden.

Dashboards sind eine hervorragende Möglichkeit, Datentabellen und Visualisierungen mit Geschäftsbenutzerinnen und -benutzern zu teilen. Sie regelmäßiges Aktualisieren und Versenden von Dashboards an Abonnenten per E-Mail planen.

## Bereinigung

Wählen Sie im Azure Databricks-Portal auf der Seite **SQL Warehouses** Ihr SQL Warehouse aus, und wählen Sie **&#9632; Beenden** aus, um es herunterzufahren.

Wenn Sie die Erkundung von Azure Databricks abgeschlossen haben, löschen Sie die erstellten Ressourcen, um unnötige Azure-Kosten zu vermeiden und Kapazität in Ihrem Abonnement freizugeben.
