# Übung 04 - Feinabstimmung großer Sprachmodelle mit Azure Databricks und Azure OpenAI

## Ziel
Diese Übung führt Sie durch den Prozess der Feinabstimmung eines großen Sprachmodells (LLM) mit Azure Databricks und Azure OpenAI. Sie erfahren, wie Sie die Umgebung einrichten, Daten vorverarbeitern und ein LLM für benutzerdefinierte Daten optimieren, um bestimmte NLP-Aufgaben zu erfüllen.

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
- Klicken Sie auf der Registerkarte „Bibliotheken“ Ihres Clusters auf „Neu installieren“.
- Installieren Sie die folgenden Python-Pakete:
    1. transformers
    2. datasets
    3. azure-ai-openai
- Optional können Sie auch alle anderen notwendigen Pakete wie Torch installieren.

### Neues Notizbuch erstellen
- Gehen Sie zum Abschnitt „Arbeitsbereich“ und klicken Sie auf „Erstellen“ > „Notizbuch“.
- Benennen Sie Ihr Notebook (z. B. Fine-Tuning-GPT4) und wählen Sie Python als Standardsprache.
- Hängen Sie das Notizbuch an Ihren Cluster.

## Schritt 4 - Datensatz vorbereiten

- Laden Sie den Datensatz
    1. Sie können jeden Textdatensatz verwenden, der für Ihre Feinabstimmung geeignet ist. Verwenden wir beispielsweise das IMDB-Dataset für die Stimmungsanalyse.
    2. Führen Sie in Ihrem Notebook den folgenden Code aus, um den Datensatz zu laden

    ```python
    from datasets import load_dataset

    dataset = load_dataset("imdb")
    ```

- Vorverarbeiten der Daten
    1. Tokenisieren Sie die Textdaten mit dem Tokenizer aus der Transformers-Bibliothek.
    2. Fügen Sie in Ihrem Notizbuch den folgenden Code ein:

    ```python
    from transformers import GPT2Tokenizer

    tokenizer = GPT2Tokenizer.from_pretrained("gpt2")

    def tokenize_function(examples):
        return tokenizer(examples["text"], padding="max_length", truncation=True)

    tokenized_datasets = dataset.map(tokenize_function, batched=True)
    ```

- Vorbereiten von Daten für die Feinabstimmung
    1. Teilen Sie die Daten in Trainings- und Validierungssätze auf.
    2. Fügen Sie in Ihrem Notizbuch Folgendes hinzu:

    ```python
    small_train_dataset = tokenized_datasets["train"].shuffle(seed=42).select(range(1000))
    small_eval_dataset = tokenized_datasets["test"].shuffle(seed=42).select(range(500))
    ```

## Schritt 5 – Feinabstimmung des GPT-4-Modells

- Einrichten der OpenAI-API
    1. Sie benötigen Ihren Azure OpenAI API-Schlüssel und Endpunkt.
    2. Richten Sie in Ihrem Notizbuch die API-Anmeldeinformationen ein:

    ```python
    import openai

    openai.api_type = "azure"
    openai.api_key = "YOUR_AZURE_OPENAI_API_KEY"
    openai.api_base = "YOUR_AZURE_OPENAI_ENDPOINT"
    openai.api_version = "2023-05-15"
    ```
- Feinabstimmen des Modells
    1. Die Feinabstimmung von GPT-4 erfolgt durch die Anpassung von Hyperparametern und die Fortsetzung des Trainingsprozesses auf Ihrem spezifischen Datensatz.
    2. Die Feinabstimmung kann komplexer sein und erfordert unter Umständen eine Stapelung von Daten, die Anpassung von Schulungsschleifen usw.
    3. Verwenden Sie die folgende Vorlage:

    ```python
    from transformers import GPT2LMHeadModel, Trainer, TrainingArguments

    model = GPT2LMHeadModel.from_pretrained("gpt2")

    training_args = TrainingArguments(
        output_dir="./results",
        evaluation_strategy="epoch",
        learning_rate=2e-5,
        per_device_train_batch_size=2,
        per_device_eval_batch_size=2,
        num_train_epochs=3,
        weight_decay=0.01,
    )

    trainer = Trainer(
        model=model,
        args=training_args,
        train_dataset=small_train_dataset,
        eval_dataset=small_eval_dataset,
    )

    trainer.train()
    ```
    4. Dieser Code stellt ein grundlegendes Framework für die Ausbildung bereit. Die Parameter und Datensätze müssten für bestimmte Fälle angepasst werden.

- Überwachen des Schulungsvorgangs
    1. Databricks ermöglicht die Überwachung des Schulungsvorgangs über die Notebook-Oberfläche und integrierte Tools wie MLflow zur Nachverfolgung.

## Schritt 6: Feinabstimmungsmodell bewerten

- Vorhersagen generieren
    1. Nach der Feinabstimmung erstellen Sie Vorhersagen für den Bewertungsdatensatz.
    2. Fügen Sie in Ihrem Notizbuch Folgendes hinzu:

    ```python
    predictions = trainer.predict(small_eval_dataset)
    print(predictions)
    ```

- Bewerten Sie die Modellleistung
    1. Verwenden Sie Metriken wie Genauigkeit, Präzision, Rückruf und F1-Score, um das Modell zu bewerten.
    2. Beispiel:

    ```python
    from sklearn.metrics import accuracy_score

    preds = predictions.predictions.argmax(-1)
    labels = predictions.label_ids
    accuracy = accuracy_score(labels, preds)
    print(f"Accuracy: {accuracy}")
    ```

- Speichern Sie das Feinabstimmungsmodell
    1. Speichern Sie das feinabgestimmte Modell in Ihrer Azure Databricks-Umgebung oder im Azure-Speicher zur späteren Verwendung.
    2. Beispiel:

    ```python
    model.save_pretrained("/dbfs/mnt/fine-tuned-gpt4/")
    ```

## Schritt 7: Feinabstimmungsmodell bereitstellen
- Packen eines Modells für die Bereitstellung (Vorschau)
    1. Konvertieren Sie das Modell in ein Format, das mit Azure OpenAI oder einem anderen Bereitstellungsdienst kompatibel ist.

- Bereitstellen des Modells
    1. Verwenden Sie Azure OpenAI für die Bereitstellung, indem Sie das Modell über Azure Machine Learning oder direkt beim OpenAI-Endpunkt registrieren.

- Testen Sie das bereitgestellte Modell
    1. Führen Sie Tests durch, um sicherzustellen, dass sich das bereitgestellte Modell erwartungsgemäß verhält und reibungslos in die Anwendungen integriert werden kann.

## Schritt 8: Ressourcen bereinigen
- Beenden des Clusters
    1. Gehen Sie zurück zur Seite Berechnen, wählen Sie Ihren Cluster aus und klicken Sie auf Beenden, um den Cluster zu stoppen.

- Optional: Löschen Sie den Databricks-Dienst:
    1. Um weitere Gebühren zu vermeiden, sollten Sie den Databricks-Arbeitsbereich löschen, wenn dieses Lab nicht Teil eines größeren Projekts oder Lernpfads ist.

Diese Übung stellte eine umfassende Anleitung zur Feinabstimmung großer Sprachmodelle wie GPT-4 mit Azure Databricks und Azure OpenAI bereit. Wenn Sie diese Schritte befolgen, können Sie die Modelle für bestimmte Aufgaben fein abstimmen, ihre Leistung bewerten und sie für reale Anwendungen einsetzen.

