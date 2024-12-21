---
lab:
  title: Erkunden Sie große Sprachmodelle mit Azure Databricks
---

# Erkunden Sie große Sprachmodelle mit Azure Databricks

Große Sprachmodelle (LLMs) können bei der Verarbeitung natürlicher Sprache (NLP) eine große Hilfe sein, wenn sie in Azure Databricks und Hugging Face Transformers integriert werden. Azure Databricks stellt eine nahtlose Plattform für den Zugriff auf, die Feinabstimmung und die Bereitstellung von LLMs bereit, einschließlich vorab trainierter Modelle aus der umfangreichen Bibliothek von Hugging Face. Für die Modellinferenz vereinfacht die Pipelines-Klasse von Hugging Face die Verwendung vorab trainierter Modelle und unterstützt eine Vielzahl von NLP-Aufgaben direkt in der Databricks-Umgebung.

Dieses Lab dauert ungefähr **30** Minuten.

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

## Installieren der erforderlichen Bibliotheken

1. Wählen Sie auf der Seite Ihres Clusters die Registerkarte „**Bibliotheken“** aus.

2. Wählen Sie „**Neu installieren**“ aus.

3. Wählen Sie **PyPI** als Bibliotheksquelle aus und geben Sie `transformers==4.44.0` in das Feld **Paket** ein.

4. Wählen Sie **Installieren** aus.

## Laden Sie vorab trainierte Modelle

1. Gehen Sie im Databricks-Arbeitsbereich zum Abschnitt **Arbeitsbereich**.

2. Wählen Sie „**Erstellen**“ und dann „**Notizbuch**“ aus.

3. Geben Sie Ihrem Notizbuch einen Namen und wählen Sie `Python` als Sprache aus.

4. In der ersten Codezelle den folgenden Code aufrufen und ausführen:

     ```python
    from transformers import pipeline

    # Load the summarization model
    summarizer = pipeline("summarization")

    # Load the sentiment analysis model
    sentiment_analyzer = pipeline("sentiment-analysis")

    # Load the translation model
    translator = pipeline("translation_en_to_fr")

    # Load a general purpose model for zero-shot classification and few-shot learning
    classifier = pipeline("zero-shot-classification")
     ```
Dadurch werden alle erforderlichen Modelle für die in dieser Übung vorgestellten NLP-Aufgaben geladen.

### Text zusammenfassen

Eine Zusammenfassungs-Pipeline generiert knappe Zusammenfassungen aus längeren Texten. Durch die Angabe eines Längenbereichs (`min_length`, `max_length`) und ob Stichproben verwendet werden sollen oder nicht (`do_sample`), können wir bestimmen, wie präzise oder kreativ die generierte Zusammenfassung sein wird. 

1. Geben Sie in der neuen Zelle den folgenden Code ein.

     ```python
    text = "Large language models (LLMs) are advanced AI systems capable of understanding and generating human-like text by learning from vast datasets. These models, which include OpenAI's GPT series and Google's BERT, have transformed the field of natural language processing (NLP). They are designed to perform a wide range of tasks, from translation and summarization to question-answering and creative writing. The development of LLMs has been a significant milestone in AI, enabling machines to handle complex language tasks with increasing sophistication. As they evolve, LLMs continue to push the boundaries of what's possible in machine learning and artificial intelligence, offering exciting prospects for the future of technology."
    summary = summarizer(text, max_length=75, min_length=25, do_sample=False)
    print(summary)
     ```

2. Führen Sie die Zelle aus, um den zusammengefassten Text anzuzeigen.

### Analysieren von Stimmungen

Die Pipeline zur Stimmungsanalyse bestimmt die Stimmung eines bestimmten Textes. Es klassifiziert den Text in Kategorien wie positiv, negativ oder neutral.

1. Geben Sie in der neuen Zelle den folgenden Code ein.

     ```python
    text = "I love using Azure Databricks for NLP tasks!"
    sentiment = sentiment_analyzer(text)
    print(sentiment)
     ```

2. Führen Sie die Zelle aus, um das Ergebnis der Stimmungsanalyse anzuzeigen.

### Übersetzen von Text

Die Übersetzungs-Pipeline konvertiert Text von einer Sprache in eine andere. In dieser Übung wurde die Aufgabe `translation_en_to_fr` verwendet, was bedeutet, dass jeder beliebige Text vom Englischen ins Französische übersetzt wird.

1. Geben Sie in der neuen Zelle den folgenden Code ein.

     ```python
    text = "Hello, how are you?"
    translation = translator(text)
    print(translation)
     ```

2. Führen Sie die Zelle aus, um den übersetzten Text auf Französisch zu sehen.

### Klassifizieren von Text

Die Zero-Shot-Klassifizierungspipeline ermöglicht es einem Modell, Text in Kategorien einzuteilen, die es während der Schulung nicht gesehen hat. Daher sind vordefinierte Kennzeichnungen als `candidate_labels` Parameter erforderlich.

1. Geben Sie in der neuen Zelle den folgenden Code ein.

     ```python
    text = "Azure Databricks is a powerful platform for big data analytics."
    labels = ["technology", "health", "finance"]
    classification = classifier(text, candidate_labels=labels)
    print(classification)
     ```

2. Führen Sie die Zelle aus, um die Ergebnisse der Zero-Shot-Klassifizierung zu sehen.

## Bereinigen

Wählen Sie zunächst im Azure Databricks-Portal auf der Seite **Compute** Ihren Cluster und dann **&#9632; Beenden** aus, um ihn herunterzufahren.

Wenn Sie die Erkundung von Azure Databricks abgeschlossen haben, löschen Sie die erstellten Ressourcen, um unnötige Azure-Kosten zu vermeiden und Kapazität in Ihrem Abonnement freizugeben.
