---
lab:
  title: Implementieren von CI/CD-Workflows mit Azure Databricks
---

# Implementieren von CI/CD-Workflows mit Azure Databricks

Das Implementieren von Continuous Integration (CI)- und Continuous Deployment (CD)-Pipelines mit Azure Databricks und Azure DevOps oder Azure Databricks und GitHub umfasst das Einrichten einer Reihe automatisierter Schritte, um sicherzustellen, dass Codeänderungen integriert, getestet und effizient bereitgestellt werden. Der Prozess umfasst in der Regel das Herstellen einer Verbindung mit einem Git-Repository, das Ausführen von Aufträgen mit Azure Pipelines zum Erstellen von Code und zum Komponententest für Code sowie die Bereitstellung der Buildartefakte für die Verwendung in Databricks-Notizbüchern. Dieser Workflow ermöglicht einen stabilen Entwicklungszyklus, der Continuous Integration und Bereitstellung ermöglicht und modernen DevOps-Methoden entspricht.

Dieses Lab dauert ungefähr **40** Minuten.

>**Hinweis:** Sie benötigen ein Github-Konto und Azure DevOps-Zugriff, um diese Übung abzuschließen.

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
    rm -r /dbfs/FileStore
    mkdir /dbfs/FileStore
    wget -O /dbfs/FileStore/sample_sales.csv https://github.com/MicrosoftLearning/mslearn-databricks/raw/main/data/sample_sales.csv
     ```

3. Verwenden Sie Menüoption **&#9656; Zelle Ausführen** links neben der Zelle, um sie auszuführen. Warten Sie dann, bis der vom Code ausgeführte Spark-Auftrag, abgeschlossen ist.
   
## Einrichten des Github Repositorys und des Azure DevOps-Projekts

Nachdem Sie ein GitHub Repository mit einem Azure DevOps-Projekt verbunden haben, können Sie CI-Pipelines einrichten, die bei allen an Ihrem Repository vorgenommenen Änderungen ausgelöst werden.

1. Wechseln Sie zu Ihrem [GitHub-Konto](https://github.com/) , und erstellen Sie ein neues Repository für Ihr Projekt.

2. Klonen Sie das Repository mit `git clone` auf Ihren lokalen Computer.

3. Laden Sie die [CSV-Datei](https://github.com/MicrosoftLearning/mslearn-databricks/raw/main/data/sample_sales.csv) auf Ihr lokales  Repository herunter und committen Sie die Änderungen:

4. Laden Sie das [Databricks-Notizbuch](https://github.com/MicrosoftLearning/mslearn-databricks/raw/main/data/sample_sales_notebook.dbc) herunter, das zum Lesen der CSV-Datei und zum Durchführen der Datentransformation verwendet wird. Führen Sie für die Änderungen einen Commit aus.

5. Wechseln Sie zum [Azure DevOps-Portal](https://azure.microsoft.com/en-us/products/devops/) , und erstellen Sie ein neues Projekt.

6. Wechseln Sie in Ihrem Azure DevOps-Projekt zum Abschnitt **Repos**, und wählen Sie **Importieren** aus, um sie mit Ihrem GitHub Repository zu verbinden.

7. Navigieren Sie in der linken Leiste zu **Projekteinstellungen > Dienstverbindungen**.

8. Wählen Sie **Neue Dienstverbindung erstellen**, und dann **Azure Resource Manager** aus.

9. Wählen Sie im Bereich **Authentifizierungsmethode****Workload-Identitätsverbund (automatisch)** aus. Wählen Sie **Weiter** aus.

10. Wählen Sie in der **Bereichsebene**  **Abonnement** aus. Wählen Sie die Abonnement- und Ressourcengruppe aus, in der Sie Ihren Databricks-Arbeitsbereich erstellt haben.

11. Geben Sie einen Namen für Ihre Dienstverbindung ein, und prüfen Sie die Option **Allen Pipelines die Zugriffsberechtigung gewähren**. Wählen Sie **Speichern**.

Jetzt hat Ihr DevOps-Projekt Zugriff auf Ihren Databricks-Arbeitsbereich, und Sie können es mit Ihren Pipelines verbinden.

## Konfigurieren einer CI-Pipeline

1. Navigieren Sie in der linken Leiste zu **Pipelines**, und wählen Sie ** Pipeline erstellen** aus.

2. Wählen Sie **GitHub** als Quelle aus, und wählen Sie Ihr Repository aus.

3. Wählen Sie im Bereich **Pipeline konfigurieren** die **Startpipeline** aus, und verwenden Sie die folgende YAML-Konfiguration für die CI-Pipeline:

```yaml
trigger:
- main

pool:
  vmImage: 'ubuntu-latest'

steps:
- task: UsePythonVersion@0
  inputs:
    versionSpec: '3.x'
    addToPath: true

- script: |
    pip install databricks-cli
  displayName: 'Install Databricks CLI'

- script: |
    databricks configure --token <<EOF
    <your-databricks-host>
    <your-databricks-token>
    EOF
  displayName: 'Configure Databricks CLI'

- script: |
    databricks fs cp dbfs:/FileStore/sample_sales.csv . --overwrite
  displayName: 'Download Sample Data from DBFS'
```

4. Ersetzen Sie `<your-databricks-host>` und `<your-databricks-token>` durch Ihre tatsächliche Databricks-Host-URL und -Token. Dadurch wird die Databricks-Befehlszeilenschnittstelle konfiguriert, bevor Sie versuchen, sie zu verwenden.

5. Klicken Sie auf **Speichern und ausführen**.

Diese YAML-Datei richtet eine CI-Pipeline ein, die durch Änderungen an der `main` Verzweigung Ihres Repositorys ausgelöst wird. Die Pipeline richtet eine Python-Umgebung ein, installiert die Databricks-CLI und lädt die Beispieldaten aus Ihrem Databricks-Arbeitsbereich herunter. Dies ist ein gängiges Setup für CI-Workflows.

## Konfigurieren einer CD-Pipeline

1. Navigieren Sie in der linken Leiste zu **Pipelines > Releases**, und wählen Sie **Release erstellen** aus.

2. Wählen Sie Ihre Buildpipeline als Artefaktquelle aus.

3. Fügen Sie eine Phase hinzu, und konfigurieren Sie die Aufgaben für die Bereitstellung in Azure Databricks:

```yaml
stages:
- stage: Deploy
  jobs:
  - job: DeployToDatabricks
    pool:
      vmImage: 'ubuntu-latest'
    steps:
    - task: UsePythonVersion@0
      inputs:
        versionSpec: '3.x'
        addToPath: true

    - script: |
        pip install databricks-cli
      displayName: 'Install Databricks CLI'

    - script: |
        databricks configure --token <<EOF
        <your-databricks-host>
        <your-databricks-token>
        EOF
      displayName: 'Configure Databricks CLI'

    - script: |
        databricks workspace import_dir /path/to/notebooks /Workspace/Notebooks
      displayName: 'Deploy Notebooks to Databricks'
```

Ersetzen Sie vor dem Ausführen dieser Pipeline `/path/to/notebooks` durch den Pfad zum Verzeichnis, in dem Sie Ihr Notizbuch in Ihrem Repository haben, und `/Workspace/Notebooks` durch den Dateipfad, in dem das Notizbuch in Ihrem Databricks-Arbeitsbereich gespeichert werden soll.

4. Klicken Sie auf **Speichern und ausführen**.

## Führen Sie die Pipelines aus

1. Fügen Sie in Ihrem lokalen Repository am Ende der `sample_sales.csv` Datei die folgende Zeile hinzu:

     ```sql
    2024-01-01,ProductG,1,500
     ```

2. Committen und pushen Sie Ihre Änderungen in das GitHub-Repository.

3. Die Änderungen im Repository lösen die CI-Pipeline aus. Stellen Sie sicher, dass die CI-Pipeline erfolgreich abgeschlossen wird.

4. Erstellen Sie ein neues Release in der Releasepipeline, und stellen Sie die Notizbücher für Databricks bereit. Stellen Sie sicher, dass die Notizbücher in Ihrem Databricks-Arbeitsbereich bereitgestellt und erfolgreich ausgeführt werden.

## Bereinigung

Wählen Sie zunächst im Azure Databricks-Portal auf der Seite **Compute** Ihren Cluster und dann **&#9632; Beenden** aus, um ihn herunterzufahren.

Wenn Sie die Erkundung von Azure Databricks abgeschlossen haben, löschen Sie die erstellten Ressourcen, um unnötige Azure-Kosten zu vermeiden und Kapazität in Ihrem Abonnement freizugeben.







