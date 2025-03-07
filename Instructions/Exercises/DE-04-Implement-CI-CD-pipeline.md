---
lab:
  title: Implementieren von CI/CD-Workflows mit Azure Databricks
---

# Implementieren von CI/CD-Workflows mit Azure Databricks

Die Implementierung von CI/CD-Workflows mit GitHub Actions und Azure Databricks kann Ihren Entwicklungsprozess optimieren und die Automatisierung verbessern. GitHub-Aktionen bieten eine leistungsstarke Plattform zum Automatisieren von Softwareworkflows, einschließlich Continuous Integration (CI) und Continuous Delivery (CD). Wenn diese Workflows in Azure Databricks integriert sind, können komplexe Datenaufgaben wie das Ausführen von Notebooks oder das Bereitstellen von Updates in Databricks-Umgebungen ausgeführt werden. Sie können beispielsweise GitHub-Aktionen verwenden, um die Bereitstellung von Databricks-Notebooks zu automatisieren, Databricks-Dateisystemuploads zu verwalten und die Databricks-CLI in Ihren Workflows einzurichten. Diese Integration sorgt dafür, dass der Entwicklungszyklus effizienter und fehlersicherer ist, insbesondere für datengesteuerte Anwendungen.

Dieses Lab dauert ungefähr **30** Minuten.

> **Hinweis**: Die Benutzeroberfläche von Azure Databricks wird kontinuierlich verbessert. Die Benutzeroberfläche kann sich seit der Erstellung der Anweisungen in dieser Übung geändert haben.

> **Hinweis:** Sie benötigen ein GitHub-Konto und einen Git-Client (z. B. das Git-Befehlszeilen-Tool), der auf Ihrem lokalen Computer installiert ist, um diese Übung abzuschließen.

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

7. Warten Sie, bis das Skript abgeschlossen ist. Dies dauert in der Regel etwa 5 Minuten, in einigen Fällen kann es jedoch länger dauern. Während Sie warten, lesen Sie den Artikel [CI/CD-Workflow mit einem Databricks-Asset-Bundle und GitHub-Aktionen](https://learn.microsoft.com/azure/databricks/dev-tools/bundles/ci-cd-bundles) in der Azure-Databricks-Dokumentation.

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
   
## Einrichten eines GitHub-Repositorys

Nachdem Sie ein GitHub-Repository mit einem Azure Databricks-Arbeitsbereich verbunden haben, können Sie CI/CD-Pipelines einrichten, die bei jeder Änderung ausgelöst werden, die an Ihrem Repository vorgenommen wird.

1. Gehen Sie zu Ihrem [GitHub-Konto](https://github.com/) und erstellen Sie ein neues privates Repository mit einem geeigneten Namen (z. B. *databricks-cicd-repo*).

1. Klonen Sie das leere Repository mit dem Befehl [git clone](https://git-scm.com/docs/git-clone) auf Ihren lokalen Computer.

1. Laden Sie die erforderlichen Dateien für diese Übung in den lokalen Ordner Ihres Repositorys herunter:
   - [CSV-Datei](https://github.com/MicrosoftLearning/mslearn-databricks/raw/main/data/sample_sales.csv)
   - [Databricks-Notebook](https://github.com/MicrosoftLearning/mslearn-databricks/raw/main/data/sample_sales_notebook.py)
   - [Auftragskonfigurationsdatei](https://github.com/MicrosoftLearning/mslearn-databricks/raw/main/data/job-config.json):

1. Fügen Sie die Dateien in Ihrem lokalen Klon des Git-Repositorys [hinzu](https://git-scm.com/docs/git-add). Dann [committen](https://git-scm.com/docs/git-commit) Sie die Änderungen und [pushen](https://git-scm.com/docs/git-push) Sie sie in das Repository.

## Einrichten von geheimen Repositoryschlüsseln

Geheimnisse sind Variablen, die du in einer Organisation, einem Repository oder einer Repositoryumgebung erstellst. Die von Ihnen erstellten Geheimnisse können in GitHub Actions-Workflows verwendet werden. GitHub Actions kann ein Geheimnis nur lesen, wenn Sie das Geheimnis explizit in einen Workflow einschließen.

Da GitHub Actions-Workflows auf Ressourcen von Azure Databricks zugreifen müssen, werden Anmeldeinformationen für die Verifizierung als verschlüsselte Variablen gespeichert, die mit den CI/CD-Pipelines verwendet werden sollen.

Bevor Sie Repositoryschlüssel erstellen, müssen Sie in Azure Databricks ein persönliches Zugriffstoken generieren:

1. Wählen Sie in Ihrem Azure Databricks-Arbeitsbereich das Symbol *Benutzende* in der oberen Leiste aus und wählen Sie dann **Einstellungen** aus dem Dropdownmenü aus.

1. Auf der Seite **Fachkraft in der Entwicklung** neben **Zugriffstoken** **Verwalten** auswählen.

1. Wählen Sie **Neues Token generieren** und dann **Generieren** aus.

1. Kopieren Sie das angezeigte Token, und fügen Sie es an einer beliebigen Stelle ein, auf die Sie später verweisen können. Wählen Sie dann **Fertig** aus.

1. Wählen Sie nun auf Ihrer GitHub-Repository-Seite die Registerkarte **Einstellungen** aus.

   ![Registerkarte „GitHub-Einstellungen“](./images/github-settings.png)

1. Wählen Sie in der linken Randleiste **Geheime Schlüssel und Variablen** und dann ** Aktionen** aus.

1. Wählen Sie **Neuer Repositoryschlüssel** aus und fügen Sie jede dieser Variablen hinzu:
   - **Name:** DATABRICKS_HOST **Geheimer Schlüssel:** Fügen Sie die URL Ihres Databricks-Arbeitsbereichs hinzu.
   - **Name:** DATABRICKS_TOKEN **Geheimer Schlüssel:** Fügen Sie das zuvor generierte Zugriffstoken hinzu.

## Einrichten von CI-Pipelines

Nachdem Sie die erforderlichen Anmeldeinformationen für den Zugriff auf Ihren Azure Databricks-Arbeitsbereich von GitHub gespeichert haben, erstellen Sie einen Workflow, um das Erfassen von Daten zu automatisieren. Er wird bereitgestellt, wenn für den Hauptzweig des Repositorys ein Push-Commit oder eine Pull-Anfrage zusammengeführt wurde. Dieser Workflow stellt sicher, dass die im Azure Databricks-Arbeitsbereich verwendete Datenquelle immer auf dem neuesten Stand ist.

1. Wählen auf Ihrer Repository-Seite die Registerkarte **Aktionen** aus.

    ![Registerkarte „GitHub Actions“](./images/github-actions.png)

1. Wählen Sie **Workflow selbst einrichten** aus und geben Sie den folgenden Code ein:

     ```yaml
    name: CI Pipeline for Azure Databricks

    on:
      push:
        branches:
          - main
      pull_request:
        branches:
          - main

    jobs:
      deploy:
        runs-on: ubuntu-latest

        steps:
        - name: Checkout code
          uses: actions/checkout@v3

        - name: Set up Python
          uses: actions/setup-python@v4
          with:
            python-version: '3.x'

        - name: Install Databricks CLI
          run: |
            pip install databricks-cli

        - name: Configure Databricks CLI
          run: |
            databricks configure --token <<EOF
            ${{ secrets.DATABRICKS_HOST }}
            ${{ secrets.DATABRICKS_TOKEN }}
            EOF

        - name: Upload sample data to DBFS
          run: databricks fs cp sample_sales.csv dbfs:/FileStore/sample_sales.csv --overwrite
     ```

    Mit dem obigen Code wird Databricks CLI installiert und konfiguriert und die Beispieldaten aus Ihrem Repository in Ihren Arbeitsbereich kopiert.

1. Benennen Sie den Workflow **CI_pipeline.yml** und wählen Sie **Änderungen committen** aus. Die Pipeline wird automatisch ausgeführt, und Sie können den Status auf der Registerkarte **Aktionen** überprüfen.

1. Sobald der Arbeitsablauf abgeschlossen ist, gehen Sie zu Ihrer Arbeitsbereichsseite, wählen Sie **+ Neu** und erstellen Sie ein neues Notizbuch.
  
1. Führen Sie in der ersten Codezelle den folgenden Code aus:

     ```python
    %fs
    ls FileStore
     ``` 

    In der Ausgabe können Sie überprüfen, ob die Beispieldaten jetzt im Databricks-Dateisystem vorhanden sind und im Arbeitsbereich verwendet werden können.

## Einrichten von CD-Pipelines

Nachdem Sie den CI-Workflow eingerichtet haben, um das Erfassen von Daten zu automatisieren, erstellen Sie einen zweiten Workflow, um die Datenverarbeitung zu automatisieren. Der CD-Workflow führt ein Notebook als Job Runs aus, dessen Ausgabe auf der Seite **Job Runs** Ihres Azure Databricks-Arbeitsbereichs registriert wird. Das Notizbuch enthält alle Transformationsschritte, die die Daten durchlaufen müssen, bevor sie genutzt werden können.

1. Wechseln Sie zu Ihrer Arbeitsbereichsseite, wählen Sie **Compute** und dann Ihren Cluster aus.

1. Öffnen Sie auf der Seite des Clusters die Optionen links neben der Schaltfläche **Beenden** und wählen Sie dann **JSON anzeigen** aus. Kopieren Sie die ID des Clusters, da sie für die Einrichtung des Job-Laufs im Workflow benötigt wird.

1. Öffnen Sie die **job-config.json** in Ihrem Repository und ersetzen Sie *your_cluster_id* durch die soeben kopierte Cluster-ID. Ersetzen Sie außerdem */Workspace/Benutzende/Ihr_Benutzername/Ihr_Notebook* durch den Pfad in Ihrem Arbeitsbereich, in dem Sie das in der Pipeline verwendete Notebook speichern möchten. Führen Sie für die Änderungen einen Commit aus.

    > **Hinweis:** Wenn Sie zur Registerkarte **Aktionen** wechseln, sehen Sie, dass die CI-Pipeline erneut ausgeführt wurde. Da es immer dann ausgelöst werden soll, wenn ein Commit gepusht wird, wird durch Ändern von *job-config.json* die Pipeline wie erwartet bereitgestellt.

1. Erstellen Sie auf der Registerkarte **Aktionen** einen neuen Workflow mit dem Namen **CD_pipeline.yml** und geben Sie den folgenden Code ein:

     ```yaml
    name: CD Pipeline for Azure Databricks

    on:
      push:
        branches:
          - main

    jobs:
      deploy:
        runs-on: ubuntu-latest

        steps:
        - name: Checkout code
          uses: actions/checkout@v3

        - name: Set up Python
          uses: actions/setup-python@v4
          with:
            python-version: '3.x'

        - name: Install Databricks CLI
          run: pip install databricks-cli

        - name: Configure Databricks CLI
          run: |
            databricks configure --token <<EOF
            ${{ secrets.DATABRICKS_HOST }}
            ${{ secrets.DATABRICKS_TOKEN }}
            EOF
     
        - name: Import Notebook to Workspace
          run: databricks workspace import sample_sales_notebook.py /Workspace/Users/your_username/your_notebook -l python --overwrite

          env:
            DATABRICKS_TOKEN: ${{ secrets.DATABRICKS_TOKEN }}

        - name: Run Databricks Job
          run: |
            databricks jobs create --json-file job-config.json
            databricks jobs run-now --job-id $(databricks jobs list | grep -m 1 'CD pipeline' | awk '{print $1}')
          env:
            DATABRICKS_TOKEN: ${{ secrets.DATABRICKS_TOKEN }}
     ```

    Bevor Sie die Änderungen übernehmen, ersetzen Sie `/Workspace/Users/your_username/your_notebook` durch den Dateipfad, in den Sie das Notizbuch in Ihrem Azure Databricks-Arbeitsbereich importieren möchten.

1. Führen Sie für die Änderungen einen Commit aus.

    Mit diesem Code wird Databricks CLI erneut installiert und konfiguriert, das Notebook in Ihren Arbeitsbereich importiert und ein Joblauf erstellt, der es ausführt. Sie können den Fortschritt des Auftrags auf der Seite **Workflows** in Ihrem Arbeitsbereich überwachen. Überprüfen Sie die Ausgabe und stellen Sie sicher, dass die Datenprobe in einen Datenrahmen geladen und für die weitere Analyse modifiziert wurde.

## Bereinigen

Wählen Sie zunächst im Azure Databricks-Portal auf der Seite **Compute** Ihren Cluster und dann **&#9632; Beenden** aus, um ihn herunterzufahren.

Wenn Sie die Erkundung von Azure Databricks abgeschlossen haben, löschen Sie die erstellten Ressourcen, um unnötige Azure-Kosten zu vermeiden und Kapazität in Ihrem Abonnement freizugeben.
