# Übung 02 – Abruf der erweiterten Generation mithilfe von Azure Databricks

## Ziel
Diese Übung führt Sie durch die Einrichtung eines RAG-Workflows (Retrieval Augmented Generation) auf Azure Databricks. Der Prozess umfasst die Aufnahme von Daten, die Erstellung von Vektoreinbettungen, die Speicherung dieser Einbettungen in einer Vektordatenbank und deren Verwendung zur Erweiterung der Eingabe für ein generatives Modell.

## Anforderungen
Ein aktives Azure-Abonnement. Wenn Sie keine Version besitzen, können Sie sich für eine [kostenlose Testversion](https://azure.microsoft.com/en-us/free/) registrieren.

## Geschätzte Dauer: 40 Minuten

## Schritt 1: Bereitstellen von Azure Databricks
- Melden Sie sich beim Azure-Portal an.
    1. Gehen Sie zum Azure-Portal und melden Sie sich mit Ihren Anmeldedaten an.
- Erstellen Sie einen Databricks-Dienst:
    1. Navigieren Sie zu „Ressource erstellen“ > „Analyse“ > „Azure Databricks“.
    2. Geben Sie die erforderlichen Details wie Arbeitsbereichsname, Abonnement, Ressourcengruppe (neu erstellen oder vorhandene auswählen) und Standort ein.
    3. Wählen Sie das Preisniveau aus (wählen Sie „Standard“ für diese Übung).
    4. Klicken Sie auf „Überprüfen + erstellen“ und dann auf „Erstellen“, sobald die Validierung erfolgreich war.

## Schritt 2: Starten Sie Workspace und erstellen Sie einen Cluster
- Starten des Databricks-Workspace:
    1. Sobald die Bereitstellung abgeschlossen ist, gehen Sie zur Ressource und klicken Sie auf "Workspace starten".
- Erstellen eines Spark-Clusters:
    1. Klicken Sie im Workspace von Databricks in der Seitenleiste auf "Berechnen" und dann auf "Berechnung erstellen".
    2. Geben Sie den Namen des Clusters an und wählen Sie eine Laufzeitversion von Spark aus.
    3. Wählen Sie den Arbeitertyp als „Standard“ und den Knotentyp basierend auf den verfügbaren Optionen (wählen Sie kleinere Knoten für Kosteneffizienz).
    4. Klicken Sie auf "Berechnung erstellen".

## Schritt 3: Datenvorbereitung
- Erfassung von Daten
    1. Laden Sie einen Beispieldatensatz von Wikipedia-Artikeln von [hier](https://dumps.wikimedia.org/enwiki/latest/) herunter.
    2. Laden Sie den Datensatz in Azure Data Lake Storage oder direkt in das Azure Databricks-Dateisystem hoch.

- Laden von Daten in Azure Databricks
```python
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("RAG-DataPrep").getOrCreate()
raw_data_path = "/mnt/data/wiki_sample.json"  # Adjust the path as necessary

raw_df = spark.read.json(raw_data_path)
raw_df.show(5)
```

- Datenbereinigung und -vorverarbeitung
    1. Bereinigen und Vorverarbeitung der Daten zum Extrahieren relevanter Textfelder.

    ```python
    from pyspark.sql.functions import col

    clean_df = raw_df.select(col("title"), col("text"))
    clean_df = clean_df.na.drop()
    clean_df.show(5)
    ```
## Schritt 4: Generieren von Einbettungen
- Installieren der erforderlichen Bibliotheken
    1. Vergewissern Sie sich, dass die Bibliotheken für Transformatoren und Satztransformatoren installiert sind.

    ```python
    %pip install transformers sentence-transformers
    ```
- Generieren von Einbettungen
    1. Verwenden Sie ein vorab trainiertes Modell, um Einbettungen für den Text zu generieren.

    ```python
    from sentence_transformers import SentenceTransformer

    model = SentenceTransformer('sentence-transformers/all-MiniLM-L6-v2')

    def embed_text(text):
        return model.encode(text).tolist()

    # Apply the embedding function to the dataset
    from pyspark.sql.functions import udf
    from pyspark.sql.types import ArrayType, FloatType

    embed_udf = udf(embed_text, ArrayType(FloatType()))
    embedded_df = clean_df.withColumn("embeddings", embed_udf(col("text")))
    embedded_df.show(5)
    ```

## Schritt 5: Speichern von Einbettungen
- Speichern von Einbettungen in Delta-Tabellen
    1. Speichern Sie die eingebetteten Daten in einer Delta-Tabelle, um sie effizient abrufen zu können.

    ```python
    embedded_df.write.format("delta").mode("overwrite").save("/mnt/delta/wiki_embeddings")
    ```

    2. Erstellen Sie eine Delta-Tabelle

    ```python
    CREATE TABLE IF NOT EXISTS wiki_embeddings
     LOCATION '/mnt/delta/wiki_embeddings'
    ```
## Schritt 6: Implementieren der Vektorsuche
- Konfigurieren der Vektorsuche
    1. Verwenden Sie die Vektorsuchfunktionen von Databricks oder integrieren Sie sie in eine Vektordatenbank wie Milvus oder Pinecone.

    ```python
    from databricks.feature_store import FeatureStoreClient

    fs = FeatureStoreClient()

    fs.create_table(
        name="wiki_embeddings_vector_store",
        primary_keys=["title"],
        df=embedded_df,
        description="Vector embeddings for Wikipedia articles."
    )
    ```
- Führen Sie eine Vektorsuche durch
    1. Implementieren Sie eine Funktion zur Suche nach relevanten Dokumenten auf der Grundlage eines Abfragevektors.

    ```python
    def search_vectors(query_text, top_k=5):
        query_embedding = model.encode([query_text]).tolist()
        query_df = spark.createDataFrame([(query_text, query_embedding)], ["query_text", "query_embedding"])
        
        results = fs.search_table(
            name="wiki_embeddings_vector_store",
            vector_column="embeddings",
            query=query_df,
            top_k=top_k
        )
        return results

    query = "Machine learning applications"
    search_results = search_vectors(query)
    search_results.show()
    ```

## Schritt 7: Generative Augmentation
- Erweitern von Eingabeaufforderungen mit abgerufenen Daten:
    1. Kombinieren Sie die abgerufenen Daten mit der Abfrage des Benutzenden, um eine umfangreiche Eingabeaufforderung für die LLM zu erstellen.

    ```python
    def augment_prompt(query_text):
        search_results = search_vectors(query_text)
        context = " ".join(search_results.select("text").rdd.flatMap(lambda x: x).collect())
        return f"Query: {query_text}\nContext: {context}"

    prompt = augment_prompt("Explain the significance of the Turing test")
    print(prompt)
    ```

- Antworten mit LLM generieren:
    2. Verwenden Sie einen LLM wie GPT-3 oder ähnliche Modelle von Hugging Face, um Antworten zu generieren.

    ```python
    from transformers import GPT2LMHeadModel, GPT2Tokenizer

    tokenizer = GPT2Tokenizer.from_pretrained("gpt2")
    model = GPT2LMHeadModel.from_pretrained("gpt2")

    inputs = tokenizer(prompt, return_tensors="pt")
    outputs = model.generate(inputs["input_ids"], max_length=500, num_return_sequences=1)
    response = tokenizer.decode(outputs[0], skip_special_tokens=True)

    print(response)
    ```

## Schritt 8: Auswertung und Optimierung
- Bewertung der Qualität der generierten Antworten:
    1. Beurteilen Sie die Relevanz, Kohärenz und Genauigkeit der generierten Antworten.
    2. Sammeln Sie Benutzerfeedback, und durchlaufen Sie den Prozess der Prompt-Augmentation.

- Optimieren Sie den RAG-Workflow:
    1. Experimentieren Sie mit unterschiedlichen Einbettungsmodellen, Chunk-Größen und Abrufparametern, um die Leistung zu optimieren.
    2. Überwachen Sie die Leistung des Systems, und nehmen Sie Anpassungen vor, um Die Genauigkeit und Effizienz zu verbessern.

## Schritt 9: Bereinigen der Ressourcen
- Beenden des Clusters
    1. Gehen Sie zurück zur Seite Berechnen, wählen Sie Ihren Cluster aus und klicken Sie auf Beenden, um den Cluster zu stoppen.

- Optional: Löschen Sie den Databricks-Dienst:
    1. Um weitere Gebühren zu vermeiden, sollten Sie den Databricks-Arbeitsbereich löschen, wenn dieses Lab nicht Teil eines größeren Projekts oder Lernpfads ist.

Wenn Sie diese Schritte befolgen, haben Sie ein RAG-System (Retrieval Augmented Generation) mit Azure Databricks implementiert. Dieses Labor zeigt, wie Daten vorverarbeitet, Einbettungen generiert, effizient gespeichert, Vektorsuchen durchgeführt und generative Modelle verwendet werden, um angereicherte Antworten zu erstellen. Der Ansatz kann an verschiedene Bereiche und Datensätze angepasst werden, um die Fähigkeiten von KI-gesteuerten Anwendungen zu verbessern.