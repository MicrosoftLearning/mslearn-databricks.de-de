# Übung 03 - Mehrstufiges Reasoning mit LangChain unter Verwendung von Azure Databricks und GPT-4

## Ziel
Diese Übung soll Sie dazu anleiten, ein mehrstufiges Reasoning-System mit LangChain auf Azure Databricks aufzubauen. Sie lernen, wie man einen Vektorindex erstellt, Einbettungen speichert, eine abruferbasierte Kette aufbaut, eine Bilderzeugungskette konstruiert und diese schließlich mit Hilfe des OpenAI-Modells von GPT-4 zu einem Multi-Chain-System kombiniert.

## Anforderungen
Ein aktives Azure-Abonnement. Wenn Sie keine Version besitzen, können Sie sich für eine [kostenlose Testversion](https://azure.microsoft.com/en-us/free/) registrieren.

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

## Schritt 3: Erforderliche Bibliotheken installieren

- Öffnen Sie ein neues Notizbuch in Ihrem Arbeitsbereich.
- Installieren Sie die erforderlichen Bibliotheken mit den folgenden Befehlen:

```python
%pip install langchain openai faiss-cpu
```

- OpenAI API konfigurieren

```python
import os
os.environ["OPENAI_API_KEY"] = "your-openai-api-key"
```

## Schritt 4: Vektorindex erstellen und Einbettungen speichern

- Laden Sie den Datensatz
    1. Laden Sie einen Beispieldatensatz, für den Sie Einbettungen erzeugen möchten. In dieser Übung werden wir einen kleinen Textdatensatz verwenden.

    ```python
    sample_texts = [
        "Azure Databricks is a fast, easy, and collaborative Apache Spark-based analytics platform.",
        "LangChain is a framework designed to simplify the creation of applications using large language models.",
        "GPT-4 is a powerful language model developed by OpenAI."
    ]
    ```
- Generieren von Einbettungen
    1. Verwenden Sie das OpenAI GPT-4-Modell, um Einbettungen für diese Texte zu erzeugen.

    ```python
    from langchain.embeddings.openai import OpenAIEmbeddings

    embeddings_model = OpenAIEmbeddings()
    embeddings = embeddings_model.embed_documents(sample_texts)
    ``` 

- Speichern der Einbettungen mit FAISS
    1. Verwenden Sie FAISS, um einen Vektorindex für einen effizienten Abruf zu erstellen.

    ```python
    import faiss
    import numpy as np

    dimension = len(embeddings[0])
    index = faiss.IndexFlatL2(dimension)
    index.add(np.array(embeddings))
    ```

## Schritt 5: Erstellen einer Retriever-basierten Kette
- Definieren Sie einen Retriever
    1. Erstellen Sie einen Retriever, der den Vektorindex nach den ähnlichsten Texten durchsuchen kann.

    ```python
    from langchain.chains import RetrievalQA
    from langchain.vectorstores.faiss import FAISS

    vector_store = FAISS(index, embeddings_model)
    retriever = vector_store.as_retriever()  
    ```

- Erstellen der RetrievalQA-Kette
    1. Erstellen Sie ein QA-System mit dem Retriever und dem GPT-4-Modell.
    
    ```python
    from langchain.llms import OpenAI
    from langchain.chains.question_answering import load_qa_chain

    llm = OpenAI(model_name="gpt-4")
    qa_chain = load_qa_chain(llm, retriever)
    ```

- Testen Sie das QA-System
    1. Stellen Sie eine Frage zu den von Ihnen eingebetteten Texten

    ```python
    result = qa_chain.run("What is Azure Databricks?")
    print(result)
    ```

## Schritt 6: Eine Bilderzeugungskette aufbauen

- Einrichten des Bilderzeugungsmodells
    1. Konfigurieren Sie die Bilderzeugungsfunktionen mit GPT-4.

    ```python
    from langchain.chains import SimpleChain

    def generate_image(prompt):
        # Assuming you have an endpoint or a tool to generate images from text.
        return f"Generated image for prompt: {prompt}"

    image_generation_chain = SimpleChain(input_variables=["prompt"], output_variables=["image"], transform=generate_image)
    ```

- Testen Sie die Bilderzeugungskette
    1. Generieren Sie ein Bild basierend auf einer Textaufforderung.

    ```python
    prompt = "A futuristic city with flying cars"
    image_result = image_generation_chain.run(prompt=prompt)
    print(image_result)
    ```

## Schritt 7: Ketten zu einem Multi-Chain-System kombinieren
- Kombinieren von Ketten
    1. Integrieren Sie die Retriever-basierte QA-Kette und die Bilderzeugungskette in ein Multi-Chain-System.

    ```python
    from langchain.chains import MultiChain

    multi_chain = MultiChain(
        chains=[
            {"name": "qa", "chain": qa_chain},
            {"name": "image_generation", "chain": image_generation_chain}
        ]
    )
    ```

- Führen Sie das Multi-Chain-System aus
    1. Übergeben Sie eine Aufgabe, die sowohl Textabruf als auch Bilderzeugung umfasst.

    ```python
    multi_task_input = {
        "qa": {"question": "Tell me about LangChain."},
        "image_generation": {"prompt": "A conceptual diagram of LangChain in use"}
    }

    multi_task_output = multi_chain.run(multi_task_input)
    print(multi_task_output)
    ```

## Schritt 8: Ressourcen bereinigen
- Beenden des Clusters
    1. Gehen Sie zurück zur Seite Berechnen, wählen Sie Ihren Cluster aus und klicken Sie auf Beenden, um den Cluster zu stoppen.

- Optional: Löschen Sie den Databricks-Dienst:
    1. Um weitere Gebühren zu vermeiden, sollten Sie den Databricks-Arbeitsbereich löschen, wenn dieses Lab nicht Teil eines größeren Projekts oder Lernpfads ist.

Dies schließt die Übung zur mehrstufigen Begründung mit LangChain mithilfe von Azure Databricks ab.