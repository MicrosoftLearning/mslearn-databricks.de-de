# Übung 07 - LLMOps mit Azure Databricks implementieren

## Ziel
Diese Übung leitet Sie durch den Prozess der Implementierung von Large Language Model Operations (LLMOps) mit Azure Databricks. Am Ende dieser Übung werden Sie wissen, wie Sie große Sprachmodelle (LLMs) in einer Produktionsumgebung mit Hilfe von Best Practices verwalten, bereitstellen und überwachen können.

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
- Databricks Workspace starten
    1. Sobald die Bereitstellung abgeschlossen ist, gehen Sie zur Ressource und klicken Sie auf "Workspace starten".
- Erstellen eines Spark-Clusters:
    1. Klicken Sie im Workspace von Databricks in der Seitenleiste auf "Berechnen" und dann auf "Berechnung erstellen".
    2. Geben Sie den Namen des Clusters an und wählen Sie eine Laufzeitversion von Spark aus.
    3. Wählen Sie den Arbeitertyp als „Standard“ und den Knotentyp basierend auf den verfügbaren Optionen (wählen Sie kleinere Knoten für Kosteneffizienz).
    4. Klicken Sie auf "Berechnung erstellen".

- Erforderliche Bibliotheken installieren
    1. Sobald Ihr Cluster läuft, navigieren Sie zur Registerkarte „Bibliotheken“.
    2. Installieren Sie die folgenden Bibliotheken:
        - azure-ai-openai (für die Verbindung zu Azure OpenAI)
        - mlflow (für die Modellverwaltung)
        - scikit-learn (für zusätzliche Modellevaluierung, falls erforderlich)

## Schritt 3: Modellverwaltung
- Hochladen oder Zugriff auf den LLM
    1. Wenn Sie über ein trainiertes Modell verfügen, laden Sie es in Ihr Databricks File System (DBFS) hoch oder verwenden Sie Azure OpenAI, um auf ein vorab trainiertes Modell zuzugreifen.
    2. Bei Verwendung von Azure OpenAI

    ```python
    from azure.ai.openai import OpenAIClient

    client = OpenAIClient(api_key="<Your_API_Key>")
    model = client.get_model("gpt-3.5-turbo")

    ```
- Versionierung des Modells mit MLflow
    1. MLflow-Verfolgung initialisieren

    ```python
    import mlflow

    mlflow.set_tracking_uri("databricks")
    mlflow.start_run()
    ```

- Das Modell protokollieren

```python
mlflow.pyfunc.log_model("model", python_model=model)
mlflow.end_run()

```

## Schritt 4: Modellbereitstellung
- Erstellen Sie eine REST-API für das Modell
    1. Erstellen Sie ein Databricks-Notizbuch für Ihre API.
    2. Definieren Sie die API-Endpunkte mit Flask oder FastAPI

    ```python
    from flask import Flask, request, jsonify
    import mlflow.pyfunc

    app = Flask(__name__)

    @app.route('/predict', methods=['POST'])
    def predict():
        data = request.json
        model = mlflow.pyfunc.load_model("model")
        prediction = model.predict(data["input"])
        return jsonify(prediction)

    if __name__ == '__main__':
        app.run(host='0.0.0.0', port=5000)
    ```
- Speichern Sie dieses Notizbuch und führen Sie es aus, um die API zu starten.

## Schritt 5: Modellüberwachung
- Einrichten der Protokollierung und Überwachung mit MLflow
    1. MLflow-Autologisierung in Ihrem Notebook aktivieren

    ```python
    mlflow.autolog()
    ```

    2. Verfolgen Sie Vorhersagen und Eingabedaten.

    ```python
    mlflow.log_param("input", data["input"])
    mlflow.log_metric("prediction", prediction)
    ```

- Benachrichtigungen für Modelldrift oder Leistungsprobleme implementieren
    1. Verwenden Sie Azure Databricks oder Azure Monitor, um Alarme für signifikante Änderungen der Modellleistung festzulegen.

## Schritt 6: Modellumschulung und Automatisierung
- Automatisierte Umschulungspipelines festlegen
    1. Erstellen Sie ein neues Databricks-Notizbuch für die Umschulung.
    2. Planen Sie den Umschulungsauftrag mit Databricks Jobs oder Azure Data Factory.
    3. Automatisieren Sie den Umschulungsprozess auf der Grundlage von Datendrift oder Zeitintervallen.

- Automatisches Bereitstellen des neu trainierten Modells
    1. Verwenden Sie MLflow's model_registry, um das eingesetzte Modell automatisch zu aktualisieren.
    2. Setzen Sie das neu trainierte Modell nach demselben Verfahren wie in Schritt 3 ein.

## Schritt 7: Verantwortungsvolle AI-Praktiken
- Erkennung und Abschwächung von Vorurteilen integrieren
    1. Verwenden Sie Fairlearn von Azure oder benutzerdefinierte Skripte, um den Vorurteil des Modells zu bewerten.
    2. Implementieren Sie Minderungsstrategien und protokollieren Sie die Ergebnisse mit MLflow.

- Umsetzung ethischer Richtlinien für den Einsatz von LLM
    1. Sorgen Sie für Transparenz bei Modellvorhersagen, indem Sie Eingabedaten und Vorhersagen protokollieren.
    2. Erstellen Sie Richtlinien für die Verwendung von Modellen und sorgen Sie für die Einhaltung der ethischen Standards.

Diese Übung stellte einen umfassenden Leitfaden für die Implementierung von LLMOps mit Azure Databricks bereit, der Modellmanagement, Bereitstellung, Überwachung, Umschulung und verantwortungsvolle KI-Praktiken behandelt. Die Befolgung dieser Schritte wird Ihnen helfen, LLMs in einer Produktionsumgebung effizient zu verwalten und zu betreiben.    