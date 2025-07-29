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

> **Tipp**: Wenn Sie bereits über einen Cluster mit der Runtimeversion 15.4 LTS **<u>ML</u>** oder einer höheren Runtimeversion in Ihrem Azure Databricks-Arbeitsbereich verfügen, können Sie ihn verwenden, um diese Übung abzuschließen, und dieses Verfahren überspringen.

1. Navigieren Sie im Azure-Portal zur Ressourcengruppe **msl-*xxxxxxx***, die vom Skript erstellt wurde (oder zur Ressourcengruppe, die Ihren vorhandenen Azure Databricks-Arbeitsbereich enthält).
1. Wählen Sie die Ressource Ihres Azure Databricks-Diensts aus (sie trägt den Namen **databricks-*xxxxxxx***, wenn Sie das Setupskript zum Erstellen verwendet haben).
1. Verwenden Sie auf der Seite **Übersicht** für Ihren Arbeitsbereich die Schaltfläche **Arbeitsbereich starten**, um Ihren Azure Databricks-Arbeitsbereich auf einer neuen Browserregisterkarte zu öffnen. Melden Sie sich an, wenn Sie dazu aufgefordert werden.

    > **Tipp**: Während Sie das Databricks-Arbeitsbereichsportal verwenden, werden möglicherweise verschiedene Tipps und Benachrichtigungen angezeigt. Schließen Sie diese, und folgen Sie den Anweisungen, um die Aufgaben in dieser Übung auszuführen.

1. Wählen Sie zunächst in der Randleiste auf der linken Seite die Aufgabe **(+) Neu** und dann **Cluster** aus.
1. Erstellen Sie auf der Seite **Neuer Cluster** einen neuen Cluster mit den folgenden Einstellungen:
    - **Clustername**: Cluster des *Benutzernamens* (der Standardclustername)
    - **Richtlinie:** Unrestricted
    - **Maschinelles Lernen**: Aktiviert
    - **Databricks Runtime**: 15.4 LTS
    - **Photon-Beschleunigung verwenden**: <u>Nicht</u> ausgewählt
    - **Workertyp**: Standard_D4ds_v5
    - **Einzelner Knoten**: Aktiviert

1. Warten Sie, bis der Cluster erstellt wurde. Es kann ein oder zwei Minuten dauern.

> **Hinweis**: Wenn Ihr Cluster nicht gestartet werden kann, verfügt Ihr Abonnement möglicherweise über ein unzureichendes Kontingent in der Region, in der Ihr Azure Databricks-Arbeitsbereich bereitgestellt wird. Details finden Sie unter [Der Grenzwert für CPU-Kerne verhindert die Clustererstellung](https://docs.microsoft.com/azure/databricks/kb/clusters/azure-core-limit). In diesem Fall können Sie versuchen, Ihren Arbeitsbereich zu löschen und in einer anderen Region einen neuen zu erstellen. Sie können einen Bereich als Parameter für das Setupskript wie folgt angeben: `./mslearn-databricks/setup.ps1 eastus`

## Installieren der erforderlichen Bibliotheken

1. Wählen Sie auf der Seite Ihres Clusters die Registerkarte „**Bibliotheken“** aus.

2. Wählen Sie „**Neu installieren**“ aus.

3. Wählen Sie **PyPI** als Bibliotheksquelle aus und geben Sie `transformers==4.53.0` in das Feld **Paket** ein.

4. Wählen Sie **Installieren** aus.

5. Wiederholen Sie die obigen Schritte, um `databricks-vectorsearch==0.56` ebenfalls zu installieren.
   
## Erstellen eines Notebook und Erfassen von Daten

1. Verwenden Sie in der Randleiste den Link ** (+) Neu**, um ein **Notebook** zu erstellen. Wählen Sie in der Dropdownliste **Verbinden** Ihren Cluster aus, wenn er noch nicht ausgewählt ist. Wenn der Cluster nicht ausgeführt wird, kann es eine Minute dauern, bis er gestartet wird.

1. Geben Sie in der ersten Zelle des Notebook die folgende SQL-Abfrage ein, um ein neues Volume zu erstellen, mit dem die Daten dieser Übung in Ihrem Standardkatalog gespeichert werden:

    ```python
   %sql 
   CREATE VOLUME <catalog_name>.default.RAG_lab;
    ```

1. Ersetzen Sie `<catalog_name>` durch den Namen Ihres Arbeitsbereichs, da Azure Databricks automatisch einen Standardkatalog mit diesem Namen erstellt.
1. Verwenden Sie Menüoption **&#9656; Zelle Ausführen** links neben der Zelle, um sie auszuführen. Warten Sie dann, bis der vom Code ausgeführte Spark-Auftrag, abgeschlossen ist.
1. Führen Sie in einer neuen Zelle den folgenden Code aus, der einen *Shellbefehl* verwendet, um Daten aus GitHub in Ihren Unity-Katalog herunterzuladen.

    ```python
   %sh
   wget -O /Volumes/<catalog_name>/default/RAG_lab/enwiki-latest-pages-articles.xml https://github.com/MicrosoftLearning/mslearn-databricks/raw/main/data/enwiki-latest-pages-articles.xml
    ```

1. Führen Sie in einer neuen Zelle den folgenden Code aus, um einen Dataframe aus den Rohdaten zu erstellen:

    ```python
   from pyspark.sql import SparkSession

   # Create a Spark session
   spark = SparkSession.builder \
       .appName("RAG-DataPrep") \
       .getOrCreate()

   # Read the XML file
   raw_df = spark.read.format("xml") \
       .option("rowTag", "page") \
       .load("/Volumes/<catalog_name>/default/RAG_lab/enwiki-latest-pages-articles.xml")

   # Show the DataFrame
   raw_df.show(5)

   # Print the schema of the DataFrame
   raw_df.printSchema()
    ```

1. Führen Sie in einer neuen Zelle den folgenden Code aus, und ersetzen Sie `<catalog_name>` durch den Namen Ihres Unity-Katalogs, um die Daten zu bereinigen und vorzuverarbeiten und die relevanten Textfelder zu extrahieren:

    ```python
   from pyspark.sql.functions import col

   clean_df = raw_df.select(col("title"), col("revision.text._VALUE").alias("text"))
   clean_df = clean_df.na.drop()
   clean_df.write.format("delta").mode("overwrite").saveAsTable("<catalog_name>.default.wiki_pages")
   clean_df.show(5)
    ```

    Wenn Sie den **Katalog (CTRL + Alt + C)**-Explorer öffnen und dessen Fenster aktualisieren, sehen Sie die Delta-Tabelle, die in Ihrem Standard-Unity-Katalog erstellt wurde.

## Einbettungen generieren und Vektorsuche implementieren

Databricks' Mosaic AI Vector Search ist eine in die Azure Databricks Platform integrierte Vektordatenbanklösung. Es optimiert die Speicherung und den Abruf von Einbettungen unter Verwendung des Hierarchical Navigable Small World (HNSW)-Algorithmus. Sie ermöglicht eine effiziente Suche nach den nächsten Nachbarn, und ihre hybride Suchfunktion für Schlüsselwortähnlichkeit stellt durch die Kombination von vektorbasierten und schlagwortbasierten Suchtechniken relevantere Ergebnisse bereit.

1. Führen Sie in einer neuen Zelle die folgende SQL-Abfrage aus, um die Funktion „Änderungsdatenfeed“ in der Quelltabelle zu aktivieren, bevor Sie einen Delta-Sync-Index erstellen.

    ```python
   %sql
   ALTER TABLE <catalog_name>.default.wiki_pages SET TBLPROPERTIES (delta.enableChangeDataFeed = true)
    ```

2. Führen Sie den folgenden Code in einer neuen Zelle aus, um den Index für die Vektorsuche zu erstellen.

    ```python
   from databricks.vector_search.client import VectorSearchClient

   client = VectorSearchClient()

   client.create_endpoint(
       name="vector_search_endpoint",
       endpoint_type="STANDARD"
   )

   index = client.create_delta_sync_index(
     endpoint_name="vector_search_endpoint",
     source_table_name="<catalog_name>.default.wiki_pages",
     index_name="<catalog_name>.default.wiki_index",
     pipeline_type="TRIGGERED",
     primary_key="title",
     embedding_source_column="text",
     embedding_model_endpoint_name="databricks-gte-large-en"
    )
    ```
     
Wenn Sie den **Katalog (CTRL + Alt + C)**-Explorer öffnen und den Bereich aktualisieren, sehen Sie den Index, der in Ihrem Standard-Unity-Katalog erstellt wurde.

> **Hinweis:** Stellen Sie vor dem Ausführen der nächsten Codezelle sicher, dass der Index erfolgreich erstellt wurde. Klicken Sie dazu mit der rechten Maustaste auf den Index im Katalogbereich und wählen Sie **Im Katalog-Explorer öffnen**. Warten Sie, bis der Indexstatus **Online** lautet.

3. Führen Sie in einer neuen Zelle den folgenden Code aus, um auf der Grundlage eines Abfragevektors nach relevanten Dokumenten zu suchen.

    ```python
   results_dict=index.similarity_search(
       query_text="Anthropology fields",
       columns=["title", "text"],
       num_results=1
   )

   display(results_dict)
    ```

Überprüfen Sie, ob die Ausgabe die entsprechende Wiki-Seite für die Eingabeaufforderung findet.

## Erweitern von Prompts mit abgerufenen Daten

Jetzt können wir die Kapazitäten von großen Sprachmodellen verbessern, indem wir ihnen zusätzlichen Kontext aus externen Datenquellen bereitstellen. Auf diese Weise können die Modelle genauere und kontextbezogenere Antworten geben.

1. Führen Sie in einer neuen Zelle den folgenden Code aus, um die abgerufenen Daten mit der Abfrage des Benutzers zu kombinieren und eine umfangreiche Eingabeaufforderung für den LLM zu erstellen.

    ```python
   # Convert the dictionary to a DataFrame
   results = spark.createDataFrame([results_dict['result']['data_array'][0]])

   from transformers import pipeline

   # Load the summarization model
   summarizer = pipeline("summarization", model="facebook/bart-large-cnn", framework="pt")

   # Extract the string values from the DataFrame column
   text_data = results.select("_2").rdd.flatMap(lambda x: x).collect()

   # Pass the extracted text data to the summarizer function
   summary = summarizer(text_data, max_length=512, min_length=100, do_sample=True)

   def augment_prompt(query_text):
       context = " ".join([item['summary_text'] for item in summary])
       return f"Query: {query_text}\nContext: {context}"

   prompt = augment_prompt("Explain the significance of Anthropology")
   print(prompt)
    ```

3. Führen Sie in einer neuen Zelle den folgenden Code aus, um einen LLM zur Generierung von Antworten zu verwenden.

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
