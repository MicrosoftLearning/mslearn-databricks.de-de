# Übung 05 – Auswerten großer Sprachmodelle mit Azure Databricks und Azure OpenAI

## Ziel
In dieser Übung erfahren Sie, wie Sie große Sprachmodelle (LLMs) mit Azure Databricks und dem GPT-4 OpenAI-Modell auswerten. Dazu gehört das Einrichten der Umgebung, das Definieren von Auswertungsmetriken und die Analyse der Leistung des Modells für bestimmte Aufgaben.

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

## Schritt 3: Installieren der erforderlichen Bibliotheken

- Melden Sie sich bei Ihrem Azure Databricks-Arbeitsbereich an.
- Erstellen Sie ein neues Notebook und wählen Sie den Standard-Cluster aus.
- Führen Sie die folgenden Befehle aus, um die erforderlichen Python-Bibliotheken zu montieren:

```python
%pip install openai
%pip install transformers
%pip install datasets
```

- Konfigurieren Sie den OpenAI API-Schlüssel:
    1. Fügen Sie Ihren Azure OpenAI API-Schlüssel zum Notizbuch hinzu:

    ```python
    import openai
    openai.api_key = "your-openai-api-key"
    ```

## Schritt 4: Bewertungsmetriken definieren
- Definieren gängiger Auswertungsmetriken:
    1. In diesem Schritt definieren Sie Auswertungsmetriken wie Perplexität, BLEU-Score, ROUGE-Score und Genauigkeit je nach Aufgabe.

    ```python
    from datasets import load_metric

    # Example: Load BLEU metric
    bleu_metric = load_metric("bleu")
    rouge_metric = load_metric("rouge")

    def compute_bleu(predictions, references):
        return bleu_metric.compute(predictions=predictions, references=references)

    def compute_rouge(predictions, references):
        return rouge_metric.compute(predictions=predictions, references=references)
    ```

- Aufgabenspezifische Metriken definieren:
    1. Definieren Sie je nach Anwendungsfall weitere relevante Metriken. Definieren Sie beispielsweise für die Stimmungsanalyse die Genauigkeit:

    ```python
    from sklearn.metrics import accuracy_score

    def compute_accuracy(predictions, references):
        return accuracy_score(references, predictions)
    ```

## Schritt 5: Datensatz vorbereiten
- Laden Sie einen Datensatz
    1. Verwenden Sie die Datensatzbibliothek, um einen vordefinierten Datensatz zu laden. Für diese Übung können Sie einen einfachen Datensatz wie den IMDB-Filmkritiken-Datensatz für die Stimmungsanalyse verwenden:

    ```python
    from datasets import load_dataset

    dataset = load_dataset("imdb")
    test_data = dataset["test"]
    ```

- Daten vorverarbeiten
    1. Tokenisierung und Vorverarbeitung des Datensatzes, damit er mit dem GPT-4-Modell kompatibel ist:

    ```python
    from transformers import GPT2Tokenizer

    tokenizer = GPT2Tokenizer.from_pretrained("gpt2")

    def preprocess_function(examples):
        return tokenizer(examples["text"], truncation=True, padding=True)

    tokenized_data = test_data.map(preprocess_function, batched=True)
    ```

## Schritt 6: GPT-4-Modell bewerten
- Vorhersagen generieren:
    1. Verwenden Sie das GPT-4-Modell, um Vorhersagen für den Testdatensatz zu erstellen.

    ```python
    def generate_predictions(input_texts):
    predictions = []
    for text in input_texts:
        response = openai.Completion.create(
            model="gpt-4",
            prompt=text,
            max_tokens=50
        )
        predictions.append(response.choices[0].text.strip())
    return predictions

    input_texts = tokenized_data["text"]
    predictions = generate_predictions(input_texts)
    ```

- Bewertungsmetriken berechnen
    1. Berechnen Sie die Bewertungsmetriken auf der Grundlage der vom GPT-4-Modell generierten Vorhersagen

    ```python
    # Example: Compute BLEU and ROUGE scores
    bleu_score = compute_bleu(predictions, tokenized_data["text"])
    rouge_score = compute_rouge(predictions, tokenized_data["text"])

    print("BLEU Score:", bleu_score)
    print("ROUGE Score:", rouge_score)
    ```

    2. Wenn Sie für eine bestimmte Aufgabe wie die Stimmungsanalyse evaluieren, berechnen Sie die Genauigkeit

    ```python
    # Assuming binary sentiment labels (positive/negative)
    actual_labels = test_data["label"]
    predicted_labels = [1 if "positive" in pred else 0 for pred in predictions]

    accuracy = compute_accuracy(predicted_labels, actual_labels)
    print("Accuracy:", accuracy)
    ```

## Schritt 7: Ergebnisse analysieren und interpretieren

- Interpretieren der Ergebnisse
    1. Analysieren Sie die BLEU-, ROUGE- oder Genauigkeitswerte, um zu bestimmen, wie gut das GPT-4-Modell bei Ihrer Aufgabe abschneidet.
    2. Diskutieren Sie mögliche Gründe für Abweichungen und überlegen Sie, wie Sie die Leistung des Modells verbessern können (z. B. Feinabstimmung, weitere Datenvorverarbeitung).

- Visualisieren Sie die Ergebnisse
    1. Optional können Sie die Ergebnisse mit Matplotlib oder einem anderen Visualisierungstool visualisieren.

    ```python
    import matplotlib.pyplot as plt

    # Example: Plot accuracy scores
    plt.bar(["Accuracy"], [accuracy])
    plt.ylabel("Score")
    plt.title("Model Evaluation Metrics")
    plt.show()
    ```

## Schritt 8: Mit unterschiedlichen Szenarien experimentieren

- Experimentieren Sie mit unterschiedlichen Eingabeaufforderungen
    1. Ändern Sie die Mechanik der Eingabeaufforderung, um zu sehen, wie sie sich auf die Leistung des Modells auswirkt.

- Auswertung mit unterschiedlichen Datensätzen
    1. Versuchen Sie, einen unterschiedlichen Datensatz zu verwenden, um die Vielseitigkeit des GPT-4-Modells für verschiedene Aufgaben zu bewerten.

- Bewertungsmetriken optimieren
    1. Experimentieren Sie mit Hyperparametern wie Temperatur, maximale Tokenanzahl usw., um die Bewertungsmetriken zu optimieren.

## Schritt 9: Bereinigen der Ressourcen
- Beenden des Clusters
    1. Gehen Sie zurück zur Seite Berechnen, wählen Sie Ihren Cluster aus und klicken Sie auf Beenden, um den Cluster zu stoppen.

- Optional: Löschen Sie den Databricks-Dienst:
    1. Um weitere Gebühren zu vermeiden, sollten Sie den Databricks-Arbeitsbereich löschen, wenn dieses Lab nicht Teil eines größeren Projekts oder Lernpfads ist.

Diese Übung leitet Sie durch den Prozess der Evaluierung eines großen Sprachmodells mit Azure Databricks und dem GPT-4 OpenAI-Modell. Durch diese Übung erhalten Sie Einblicke in die Leistung des Modells und verstehen, wie Sie das Modell für bestimmte Aufgaben verbessern und feinabstimmen können.