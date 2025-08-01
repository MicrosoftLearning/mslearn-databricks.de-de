---
lab:
  title: Erweiterte Generierung von Datenabfragen mit Azure Databricks
---

# Erweiterte Generierung von Datenabfragen mit Azure Databricks

Retrieval Augmented Generation (RAG) ist ein innovativer Ansatz in der KI, der große Sprachmodelle durch die Integration externer Wissensquellen verbessert. Azure Databricks bietet eine robuste Plattform für die Entwicklung von RAG-Anwendungen, mit der unstrukturierte Daten in ein Format umgewandelt werden können, das für den Abruf und die Generierung von Antworten geeignet ist. Dieser Prozess umfasst eine Reihe von Schritten, darunter das Verstehen der Benutzeranfrage, das Abrufen relevanter Daten und die Generierung einer Antwort mithilfe eines Sprachmodells. Das von Azure Databricks bereitgestellte Framework unterstützt die schnelle Iteration und Bereitstellung von RAG-Anwendungen und gewährleistet hochwertige, domänenspezifische Antworten, die aktuelle Informationen und proprietäres Wissen enthalten können.

Dieses Lab dauert ungefähr **40** Minuten.

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

7. Warten Sie, bis das Skript abgeschlossen ist. Dies dauert in der Regel etwa 5 Minuten, in einigen Fällen kann es jedoch länger dauern.

## Erstellen eines Clusters

Azure Databricks ist eine verteilte Verarbeitungsplattform, die Apache Spark-*Cluster* verwendet, um Daten parallel auf mehreren Knoten zu verarbeiten. Jeder Cluster besteht aus einem Treiberknoten, um die Arbeit zu koordinieren, und Arbeitsknoten zum Ausführen von Verarbeitungsaufgaben. In dieser Übung erstellen Sie einen *Einzelknotencluster* , um die in der Lab-Umgebung verwendeten Computeressourcen zu minimieren (in denen Ressourcen möglicherweise eingeschränkt werden). In einer Produktionsumgebung erstellen Sie in der Regel einen Cluster mit mehreren Workerknoten.

> **Tipp**: Wenn Sie bereits über einen Cluster mit der Runtimeversion 16.4 LTS **<u>ML</u>** oder höher in Ihrem Azure Databricks-Arbeitsbereich verfügen, können Sie ihn verwenden, um diese Übung abzuschließen, und diesen Prozess überspringen.

1. Navigieren Sie im Azure-Portal zur Ressourcengruppe **msl-*xxxxxxx***, die vom Skript erstellt wurde (oder zur Ressourcengruppe, die Ihren vorhandenen Azure Databricks-Arbeitsbereich enthält).
1. Wählen Sie die Ressource Ihres Azure Databricks-Diensts aus (sie trägt den Namen **databricks-*xxxxxxx***, wenn Sie das Setupskript zum Erstellen verwendet haben).
1. Verwenden Sie auf der Seite **Übersicht** für Ihren Arbeitsbereich die Schaltfläche **Arbeitsbereich starten**, um Ihren Azure Databricks-Arbeitsbereich auf einer neuen Browserregisterkarte zu öffnen. Melden Sie sich an, wenn Sie dazu aufgefordert werden.

    > **Tipp**: Während Sie das Databricks-Arbeitsbereichsportal verwenden, werden möglicherweise verschiedene Tipps und Benachrichtigungen angezeigt. Schließen Sie diese, und folgen Sie den Anweisungen, um die Aufgaben in dieser Übung auszuführen.

1. Wählen Sie zunächst in der Randleiste auf der linken Seite die Aufgabe **(+) Neu** und dann **Cluster** aus.
1. Erstellen Sie auf der Seite **Neuer Cluster** einen neuen Cluster mit den folgenden Einstellungen:
    - **Clustername**: Cluster des *Benutzernamens* (der Standardclustername)
    - **Richtlinie:** Unrestricted
    - **Maschinelles Lernen**: Aktiviert
    - **Databricks Runtime**: 16.4 LTS
    - **Photon-Beschleunigung verwenden**: <u>Nicht</u> ausgewählt
    - **Workertyp**: Standard_D4ds_v5
    - **Einzelner Knoten**: Aktiviert

1. Warten Sie, bis der Cluster erstellt wurde. Es kann ein oder zwei Minuten dauern.

> **Hinweis**: Wenn Ihr Cluster nicht gestartet werden kann, verfügt Ihr Abonnement möglicherweise über ein unzureichendes Kontingent in der Region, in der Ihr Azure Databricks-Arbeitsbereich bereitgestellt wird. Details finden Sie unter [Der Grenzwert für CPU-Kerne verhindert die Clustererstellung](https://docs.microsoft.com/azure/databricks/kb/clusters/azure-core-limit). In diesem Fall können Sie versuchen, Ihren Arbeitsbereich zu löschen und in einer anderen Region einen neuen zu erstellen. Sie können einen Bereich als Parameter für das Setupskript wie folgt angeben: `./mslearn-databricks/setup.ps1 eastus`

## Installieren der erforderlichen Bibliotheken

1. Verwenden Sie in der Randleiste den Link ** (+) Neu**, um ein **Notebook** zu erstellen. Wählen Sie in der Dropdownliste **Verbinden** Ihren Cluster aus, wenn er noch nicht ausgewählt ist. Wenn der Cluster nicht ausgeführt wird, kann es eine Minute dauern, bis er gestartet wird.
1. In der ersten Codezelle geben Sie den folgenden Code ein und führen ihn aus, um die erforderlichen Bibliotheken zu installieren:
   
    ```python
   %pip install faiss-cpu
   dbutils.library.restartPython()
    ```
   
## Erfassen von Daten

1. Laden Sie auf einer neuen Browserregisterkarte die [Beispieldatei](https://github.com/MicrosoftLearning/mslearn-databricks/raw/main/data/enwiki-latest-pages-articles.xml) herunter, die in dieser Übung als Datenquelle verwendet wird: `https://github.com/MicrosoftLearning/mslearn-databricks/raw/main/data/enwiki-latest-pages-articles.xml`.
1. Wählen Sie auf der Registerkarte „Databricks-Arbeitsbereich“ bei geöffnetem Notebook den **Katalog-Explorer (STRG+ALT+C)** aus, und wählen Sie das ➕-Symbol zum **Hinzufügen von Daten** aus.
1. Wählen Sie auf der Seite **Daten hinzufügen** **Dateien in DBFS hochladen** aus.
1. Benennen Sie auf der Seite **DBFS** das Zielverzeichnis `RAG_lab`, und laden Sie die zuvor gespeicherte XML-Datei hoch.
1. Wählen Sie in der Randleiste **Arbeitsbereich** aus, und öffnen Sie Ihr Notebook erneut.
1. Führen Sie in einer neuen Codezelle den folgenden Code aus, um einen Dataframe aus den Rohdaten zu erstellen:

    ```python
   from pyspark.sql import SparkSession

   # Create a Spark session
   spark = SparkSession.builder \
       .appName("RAG-DataPrep") \
       .getOrCreate()

   # Read the XML file
   raw_df = spark.read.format("xml") \
       .option("rowTag", "page") \
       .load("/FileStore/tables/RAG_lab/enwiki_latest_pages_articles.xml")

   # Show the DataFrame
   raw_df.show(5)

   # Print the schema of the DataFrame
   raw_df.printSchema()
    ```

1. Verwenden Sie Menüoption **&#9656; Zelle Ausführen** links neben der Zelle, um sie auszuführen. Warten Sie dann, bis der vom Code ausgeführte Spark-Auftrag, abgeschlossen ist.
1. Führen Sie in einer neuen Zelle den folgenden Code aus, um die Daten zu bereinigen und vorzuverarbeiten und die relevanten Textfelder zu extrahieren:

    ```python
   from pyspark.sql.functions import col

   clean_df = raw_df.select(col("title"), col("revision.text._VALUE").alias("text"))
   clean_df = clean_df.na.drop()
   clean_df.show(5)
    ```

## Einbettungen generieren und Vektorsuche implementieren

FAISS (Facebook AI Similarity Search) ist eine Open-Source-Vektordatenbankbibliothek, die von Meta AI entwickelt wurde und für eine effiziente Ähnlichkeitssuche und Clustering von dichten Vektoren entwickelt wurde. FAISS ermöglicht schnelle und skalierbare Nearest-Neighbor-Suchvorgänge und kann in hybride Suchsysteme integriert werden, um vektorbasierte Ähnlichkeit mit herkömmlichen schlüsselwortbasierten Methoden zu kombinieren und so die Relevanz von Suchergebnissen zu verbessern.

1. Führen Sie in einer neuen Zelle den folgenden Code aus, um das vortrainierte `all-MiniLM-L6-v2`-Modell zu laden und Text in Einbettungen zu konvertieren:

    ```python
   from sentence_transformers import SentenceTransformer
   import numpy as np
    
   # Load pre-trained model
   model = SentenceTransformer('all-MiniLM-L6-v2')
    
   # Function to convert text to embeddings
   def text_to_embedding(text):
       embeddings = model.encode([text])
       return embeddings[0]
    
   # Convert the DataFrame to a Pandas DataFrame
   pandas_df = clean_df.toPandas()
    
   # Apply the function to get embeddings
   pandas_df['embedding'] = pandas_df['text'].apply(text_to_embedding)
   embeddings = np.vstack(pandas_df['embedding'].values)
    ```

1. Führen Sie in einer neuen Zelle den folgenden Code aus, um den FAISS-Index zu erstellen und abzufragen:

    ```python
   import faiss
    
   # Create a FAISS index
   d = embeddings.shape[1]  # dimension
   index = faiss.IndexFlatL2(d)  # L2 distance
   index.add(embeddings)  # add vectors to the index
    
   # Perform a search
   query_embedding = text_to_embedding("Anthropology fields")
   k = 1  # number of nearest neighbors
   distances, indices = index.search(np.array([query_embedding]), k)
    
   # Get the results
   results = pandas_df.iloc[indices[0]]
   display(results)
    ```

Überprüfen Sie, ob die Ausgabe die entsprechende Wiki-Seite für die Eingabeaufforderung findet.

## Erweitern von Prompts mit abgerufenen Daten

Jetzt können wir die Kapazitäten von großen Sprachmodellen verbessern, indem wir ihnen zusätzlichen Kontext aus externen Datenquellen bereitstellen. Auf diese Weise können die Modelle genauere und kontextbezogenere Antworten geben.

1. Führen Sie in einer neuen Zelle den folgenden Code aus, um die abgerufenen Daten mit der Abfrage des Benutzers zu kombinieren und eine umfangreiche Eingabeaufforderung für den LLM zu erstellen.

    ```python
   from transformers import pipeline
    
   # Load the summarization model
   summarizer = pipeline("summarization", model="facebook/bart-large-cnn", framework="pt")
    
   # Extract the string values from the DataFrame column
   text_data = results["text"].tolist()
    
   # Pass the extracted text data to the summarizer function
   summary = summarizer(text_data, max_length=512, min_length=100, do_sample=True)
    
   def augment_prompt(query_text):
       context = " ".join([item['summary_text'] for item in summary])
       return f"{context}\n\nQuestion: {query_text}\nAnswer:"
    
   prompt = augment_prompt("Explain the significance of Anthropology")
   print(prompt)
    ```

1. Führen Sie in einer neuen Zelle den folgenden Code aus, um einen LLM zur Generierung von Antworten zu verwenden.

    ```python
   from transformers import GPT2LMHeadModel, GPT2Tokenizer

   tokenizer = GPT2Tokenizer.from_pretrained("gpt2")
   model = GPT2LMHeadModel.from_pretrained("gpt2")

   inputs = tokenizer(prompt, return_tensors="pt")
   outputs = model.generate(
       inputs["input_ids"], 
       max_length=300, 
       num_return_sequences=1, 
       repetition_penalty=2.0, 
       top_k=50, 
       top_p=0.95, 
       temperature=0.7,
       do_sample=True
   )
   response = tokenizer.decode(outputs[0], skip_special_tokens=True)

   print(response)
    ```

## Bereinigen

Wählen Sie zunächst im Azure Databricks-Portal auf der Seite **Compute** Ihren Cluster und dann **&#9632; Beenden** aus, um ihn herunterzufahren.

Wenn Sie die Erkundung von Azure Databricks abgeschlossen haben, löschen Sie die erstellten Ressourcen, um unnötige Azure-Kosten zu vermeiden und Kapazität in Ihrem Abonnement freizugeben.
