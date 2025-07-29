---
lab:
  title: Feinabstimmung großer Sprachmodelle mit Azure Databricks und Azure OpenAI
---

# Feinabstimmung großer Sprachmodelle mit Azure Databricks und Azure OpenAI

Mit Azure Databricks können Benutzende nun die Leistungsfähigkeit von LLMs für spezielle Aufgaben nutzen, indem sie diese mit ihren eigenen Daten abstimmen und so die domänenspezifische Leistung verbessern. Um ein Sprachmodell mit Azure Databricks zu optimieren, können Sie die Mosaik AI Model-Training-Schnittstelle verwenden, die den Prozess der vollständigen Modelloptimierung vereinfacht. Mit dieser Funktion können Sie ein Modell mit Ihren benutzerdefinierten Daten feinabstimmen, wobei die Prüfpunkte in MLflow gespeichert werden, sodass Sie die vollständige Kontrolle über das feinabgestimmte Modell behalten.

Dieses Lab dauert ungefähr **60** Minuten.

> **Hinweis**: Die Benutzeroberfläche von Azure Databricks wird kontinuierlich verbessert. Die Benutzeroberfläche kann sich seit der Erstellung der Anweisungen in dieser Übung geändert haben.

## Vor der Installation

Sie benötigen ein [Azure-Abonnement](https://azure.microsoft.com/free), in dem Sie Administratorzugriff besitzen.

## Bereitstellen einer Azure OpenAI-Ressource

Wenn Sie noch keine Azure OpenAI-Ressource haben, stellen Sie eine in Ihrem Azure-Abonnement bereit.

1. Melden Sie sich beim **Azure-Portal** unter `https://portal.azure.com` an.
2. Erstellen Sie eine **Azure OpenAI-Ressource** mit den folgenden Einstellungen:
    - **Abonnement:** *Wählen Sie ein Azure-Abonnement aus, das für den Zugriff auf den Azure OpenAI-Dienst freigegeben wurde.*
    - **Ressourcengruppe**: *Wählen Sie eine Ressourcengruppe aus, oder erstellen Sie eine*.
    - **Region:** *Treffen Sie eine **zufällige** Auswahl aus einer der folgenden Regionen*\*
        - USA (Ost) 2
        - USA Nord Mitte
        - Schweden, Mitte
        - Schweiz, Westen
    - **Name:** *Wählen Sie einen Namen Ihrer Wahl aus.*
    - **Tarif**: Standard S0.

> \* Azure OpenAI-Ressourcen werden durch regionale Kontingente eingeschränkt. Die aufgeführten Regionen enthalten das Standardkontingent für die in dieser Übung verwendeten Modelltypen. Durch die zufällige Auswahl einer Region wird das Risiko reduziert, dass eine einzelne Region ihr Kontingentlimit in Szenarien erreicht, in denen Sie ein Abonnement für andere Benutzer freigeben. Wenn später in der Übung ein Kontingentlimit erreicht wird, besteht eventuell die Möglichkeit, eine andere Ressource in einer anderen Region zu erstellen.

3. Warten Sie, bis die Bereitstellung abgeschlossen ist. Wechseln Sie dann zur bereitgestellten Azure OpenAI-Ressource im Azure-Portal.

4. Wählen Sie im linken Fensterbereich unter **Ressourcenverwaltung** die Option **Tasten und Endpunkt**.

5. Kopieren Sie den Endpunkt und einen der verfügbaren Schlüssel, da Sie ihn später in dieser Übung verwenden werden.

6. Starten Sie Cloud Shell und führen Sie `az account get-access-token` aus, um ein temporäres Autorisierungstoken für API-Tests zu erhalten. Notieren Sie dies zusammen mit dem zuvor kopierten Endpunkt und Schlüssel.

    >**Hinweis:** Sie müssen nur den Feldwert `accessToken` und **nicht** die gesamte JSON-Ausgabe kopieren.

## Bereitstellen des erforderlichen Modells

Azure bietet ein webbasiertes Portal mit dem Namen **Azure AI Foundry**, das Sie zur Bereitstellung, Verwaltung und Untersuchung von Modellen verwenden können. Sie beginnen Ihre Erkundung von Azure OpenAI, indem Sie Azure AI Foundry verwenden, um ein Modell bereitzustellen.

> **Hinweis:** Während Sie Azure AI Foundry verwenden, werden möglicherweise Meldungsfelder mit Vorschlägen für auszuführende Aufgaben angezeigt. Sie können diese schließen und die Schritte in dieser Übung ausführen.

1. Scrollen Sie im Azure-Portal auf der Seite **Übersicht** für Ihre Azure OpenAI-Ressource nach unten zum Abschnitt **Erste Schritte**, und wählen Sie die Schaltfläche aus, um zu **Azure AI Foundry** zu gelangen.
   
1. Wählen Sie in Azure AI Foundry im linken Bereich die Seite **Bereitstellungen** aus, und sehen Sie sich Ihre vorhandenen Modellbereitstellungen an. Falls noch nicht vorhanden, erstellen Sie eine neue Bereitstellung des **gpt-4o**-Modells mit den folgenden Einstellungen:
    - **Bereitstellungsname**: *gpt-4o*
    - **Bereitstellungstyp**: Standard
    - **Modellversion**: *Standardversion verwenden*
    - **Ratenbegrenzung für Token pro Minute**: 10 Tsd.\*
    - **Inhaltsfilter**: Standard
    - **Dynamische Quote aktivieren**: Deaktiviert
    
> \* Ein Ratenlimit von 10.000 Token pro Minute ist mehr als ausreichend, um diese Übung auszuführen und gleichzeitig Kapazität für andere Personen zu schaffen, die das gleiche Abonnement nutzen.

## Bereitstellen eines Azure Databricks-Arbeitsbereichs

> **Tipp**: Wenn Sie bereits über einen Azure Databricks-Arbeitsbereich verfügen, können Sie dieses Verfahren überspringen und Ihren vorhandenen Arbeitsbereich verwenden.

1. Melden Sie sich beim **Azure-Portal** unter `https://portal.azure.com` an.
2. Erstellen Sie eine **Azure Databricks**-Ressource mit den folgenden Einstellungen:
    - **Abonnement**: *Wählen Sie das gleiche Azure-Abonnement aus, das Sie zum Erstellen Ihrer Azure OpenAI-Ressource verwendet haben*
    - **Ressourcengruppe**: *Die gleiche Ressourcengruppe, in der Sie Ihre Azure OpenAI-Ressource erstellt haben*
    - **Region**: *Die gleiche Region, in der Sie Ihre Azure OpenAI-Ressource erstellt haben*
    - **Name:** *Wählen Sie einen Namen Ihrer Wahl aus.*
    - **Preisstufe**: *Premium* oder *Testversion*

3. Wählen Sie **Überprüfen + Erstellen** und warten Sie, bis die Bereitstellung abgeschlossen ist. Wechseln Sie dann zur Ressource, und starten Sie den Arbeitsbereich.

## Erstellen eines Clusters

Azure Databricks ist eine verteilte Verarbeitungsplattform, die Apache Spark-*Cluster* verwendet, um Daten parallel auf mehreren Knoten zu verarbeiten. Jeder Cluster besteht aus einem Treiberknoten, um die Arbeit zu koordinieren, und Arbeitsknoten zum Ausführen von Verarbeitungsaufgaben. In dieser Übung erstellen Sie einen *Einzelknotencluster* , um die in der Lab-Umgebung verwendeten Computeressourcen zu minimieren (in denen Ressourcen möglicherweise eingeschränkt werden). In einer Produktionsumgebung erstellen Sie in der Regel einen Cluster mit mehreren Workerknoten.

> **Tipp**: Wenn Sie bereits über einen Cluster mit der Runtimeversion 15.4 LTS **<u>ML</u>** oder einer höheren Runtimeversion in Ihrem Azure Databricks-Arbeitsbereich verfügen, können Sie ihn verwenden, um diese Übung abzuschließen, und dieses Verfahren überspringen.

1. Navigieren Sie im Azure-Portal zu der Ressourcengruppe, in der der Azure Databricks-Arbeitsbereich erstellt wurde.
2. Wählen Sie Ihre Azure Databricks Service-Ressource aus.
3. Verwenden Sie auf der Seite **Übersicht** für Ihren Arbeitsbereich die Schaltfläche **Arbeitsbereich starten**, um Ihren Azure Databricks-Arbeitsbereich auf einer neuen Browserregisterkarte zu öffnen. Melden Sie sich an, wenn Sie dazu aufgefordert werden.

> **Tipp**: Während Sie das Databricks-Arbeitsbereichsportal verwenden, werden möglicherweise verschiedene Tipps und Benachrichtigungen angezeigt. Schließen Sie diese, und folgen Sie den Anweisungen, um die Aufgaben in dieser Übung auszuführen.

4. Wählen Sie zunächst in der Randleiste auf der linken Seite die Aufgabe **(+) Neu** und dann **Cluster** aus.
5. Erstellen Sie auf der Seite **Neuer Cluster** einen neuen Cluster mit den folgenden Einstellungen:
    - **Clustername**: Cluster des *Benutzernamens* (der Standardclustername)
    - **Richtlinie:** Unrestricted
    - **Maschinelles Lernen**: Aktiviert
    - **Databricks Runtime**: 15.4 LTS
    - **Photon-Beschleunigung verwenden**: <u>Nicht</u> ausgewählt
    - **Workertyp**: Standard_D4ds_v5
    - **Einzelner Knoten**: Aktiviert

6. Warten Sie, bis der Cluster erstellt wurde. Es kann ein oder zwei Minuten dauern.

> **Hinweis**: Wenn Ihr Cluster nicht gestartet werden kann, verfügt Ihr Abonnement möglicherweise über ein unzureichendes Kontingent in der Region, in der Ihr Azure Databricks-Arbeitsbereich bereitgestellt wird. Details finden Sie unter [Der Grenzwert für CPU-Kerne verhindert die Clustererstellung](https://docs.microsoft.com/azure/databricks/kb/clusters/azure-core-limit). In diesem Fall können Sie versuchen, Ihren Arbeitsbereich zu löschen und in einer anderen Region einen neuen zu erstellen.

## Erstellen eines neuen Notebooks und Erfassen von Daten

1. Verwenden Sie in der Randleiste den Link ** (+) Neu**, um ein **Notebook** zu erstellen. Wählen Sie in der Dropdownliste **Verbinden** Ihren Cluster aus, wenn er noch nicht ausgewählt ist. Wenn der Cluster nicht ausgeführt wird, kann es eine Minute dauern, bis er gestartet wird.

1. Geben Sie in der ersten Zelle des Notebook die folgende SQL-Abfrage ein, um ein neues Volume zu erstellen, mit dem die Daten dieser Übung in Ihrem Standardkatalog gespeichert werden:

    ```python
   %sql 
   CREATE VOLUME <catalog_name>.default.fine_tuning;
    ```

1. Ersetzen Sie `<catalog_name>` durch den Namen des Standardkatalogs. Sie können den Namen überprüfen, indem Sie auf der Randleiste die Option **Katalog** auswählen.
1. Verwenden Sie Menüoption **&#9656; Zelle Ausführen** links neben der Zelle, um sie auszuführen. Warten Sie dann, bis der vom Code ausgeführte Spark-Auftrag, abgeschlossen ist.
1. Führen Sie in einer neuen Zelle den folgenden Code aus, der einen *Shellbefehl* verwendet, um Daten aus GitHub in Ihren Unity-Katalog herunterzuladen.

    ```python
   %sh
   wget -O /Volumes/<catalog_name>/default/fine_tuning/training_set.jsonl https://github.com/MicrosoftLearning/mslearn-databricks/raw/main/data/training_set.jsonl
   wget -O /Volumes/<catalog_name>/default/fine_tuning/validation_set.jsonl https://github.com/MicrosoftLearning/mslearn-databricks/raw/main/data/validation_set.jsonl
    ```

3. Führen Sie in einer neuen Zelle den folgenden Code mit den Zugriffsinformationen aus, die Sie am Anfang dieser Übung kopiert haben, um beständige Umgebungsvariablen für die Authentifizierung bei Verwendung von Azure OpenAI-Ressourcen zuzuweisen:

    ```python
   import os

   os.environ["AZURE_OPENAI_API_KEY"] = "your_openai_api_key"
   os.environ["AZURE_OPENAI_ENDPOINT"] = "your_openai_endpoint"
   os.environ["TEMP_AUTH_TOKEN"] = "your_access_token"
    ```
     
## Überprüfen der Tokenanzahl

Sowohl `training_set.jsonl` als auch `validation_set.jsonl` bestehen aus verschiedenen Konversationsbeispielen zwischen `user` und `assistant`, die als Datenpunkte für das Training und die Validierung des fein abgestimmten Modells dienen. Während die Datasets für diese Übung als klein betrachtet werden, ist es wichtig, beim Arbeiten mit größeren Datasets zu berücksichtigen, dass die LLMs eine maximale Kontextlänge in Bezug auf Token aufweisen. Daher können Sie die Tokenanzahl Ihrer Datasets überprüfen, bevor Sie Ihr Modell trainieren, und sie bei Bedarf überarbeiten. 

1. Führen Sie in einer neuen Zelle den folgenden Code aus, um die Tokenanzahl für jede Datei zu überprüfen:

    ```python
   import json
   import tiktoken
   import numpy as np
   from collections import defaultdict

   encoding = tiktoken.get_encoding("cl100k_base")

   def num_tokens_from_messages(messages, tokens_per_message=3, tokens_per_name=1):
       num_tokens = 0
       for message in messages:
           num_tokens += tokens_per_message
           for key, value in message.items():
               num_tokens += len(encoding.encode(value))
               if key == "name":
                   num_tokens += tokens_per_name
       num_tokens += 3
       return num_tokens

   def num_assistant_tokens_from_messages(messages):
       num_tokens = 0
       for message in messages:
           if message["role"] == "assistant":
               num_tokens += len(encoding.encode(message["content"]))
       return num_tokens

   def print_distribution(values, name):
       print(f"\n##### Distribution of {name}:")
       print(f"min / max: {min(values)}, {max(values)}")
       print(f"mean / median: {np.mean(values)}, {np.median(values)}")

   files = ['/Volumes/<catalog_name>/default/fine_tuning/training_set.jsonl', '/Volumes/<catalog_name>/default/fine_tuning/validation_set.jsonl']

   for file in files:
       print(f"File: {file}")
       with open(file, 'r', encoding='utf-8') as f:
           dataset = [json.loads(line) for line in f]

       total_tokens = []
       assistant_tokens = []

       for ex in dataset:
           messages = ex.get("messages", {})
           total_tokens.append(num_tokens_from_messages(messages))
           assistant_tokens.append(num_assistant_tokens_from_messages(messages))

       print_distribution(total_tokens, "total tokens")
       print_distribution(assistant_tokens, "assistant tokens")
       print('*' * 75)
    ```

Als Referenz weist das in dieser Übung verwendete Modell GPT-4o das Kontextlimit (kombinierte Gesamtanzahl der Token im Eingabeprompt und in der generierten Antwort) von 128.000 Token auf.

## Hochladen von Feinabstimmungsdateien in Azure OpenAI

Bevor Sie mit der Feinabstimmung des Modells beginnen, müssen Sie einen OpenAI-Client initialisieren und die Feinabstimmungsdateien zu seiner Umgebung hinzufügen und Datei-IDs generieren, die zum Initialisieren des Auftrags verwendet werden.

1. Führen Sie in einer neuen Zelle den folgenden Code aus:

     ```python
    import os
    from openai import AzureOpenAI

    client = AzureOpenAI(
      azure_endpoint = os.getenv("AZURE_OPENAI_ENDPOINT"),
      api_key = os.getenv("AZURE_OPENAI_API_KEY"),
      api_version = "2024-05-01-preview"  # This API version or later is required to access seed/events/checkpoint features
    )

    training_file_name = '/Volumes/<catalog_name>/default/fine_tuning/training_set.jsonl'
    validation_file_name = '/Volumes/<catalog_name>/default/fine_tuning/validation_set.jsonl'

    training_response = client.files.create(
        file = open(training_file_name, "rb"), purpose="fine-tune"
    )
    training_file_id = training_response.id

    validation_response = client.files.create(
        file = open(validation_file_name, "rb"), purpose="fine-tune"
    )
    validation_file_id = validation_response.id

    print("Training file ID:", training_file_id)
    print("Validation file ID:", validation_file_id)
     ```

## Übermitteln des Feinabstimmungsauftrags

Nachdem die Feinabstimmungsdateien erfolgreich hochgeladen wurden, können Sie Ihren Trainingsauftrag für die Optimierung übermitteln. Es ist nicht ungewöhnlich, dass das Training mehr als eine Stunde dauert. Nach Abschluss des Trainings können Sie die Ergebnisse in Azure AI Foundry anzeigen, indem Sie im linken Bereich die Option **Feinabstimmung** auswählen.

1. Führen Sie in einer neuen Zelle den folgenden Code aus, um den Trainingsauftrag zur Feinabstimmung auszuführen:

    ```python
   response = client.fine_tuning.jobs.create(
       training_file = training_file_id,
       validation_file = validation_file_id,
       model = "gpt-4o",
       seed = 105 # seed parameter controls reproducibility of the fine-tuning job. If no seed is specified one will be generated automatically.
   )

   job_id = response.id
    ```

Der Parameter `seed` steuert die Reproduzierbarkeit des Feinabstimmungsauftrags. Die Übergabe derselben Seed- und Auftragsparameter sollte zu denselben Ergebnissen führen, kann aber in seltenen Fällen abweichen. Wenn kein Startwert angegeben ist, wird automatisch einer generiert.

2. In einer neuen Zelle können Sie den folgenden Code ausführen, um den Status des Feinabstimmungsauftrags zu überwachen:

    ```python
   print("Job ID:", response.id)
   print("Status:", response.status)
    ```

>**Hinweis:** Sie können den Auftragsstatus auch in AI Foundry überwachen, indem Sie auf der linken Randleiste die Option **Feinabstimmung** auswählen.

3. Sobald sich der Auftragsstatus in `succeeded` ändert, führen Sie den folgenden Code aus, um die Endergebnisse zu erhalten:

    ```python
   response = client.fine_tuning.jobs.retrieve(job_id)

   print(response.model_dump_json(indent=2))
   fine_tuned_model = response.fine_tuned_model
    ```
   
## Bereitstellen des optimierten Modells

Da Sie nun über ein fein abgestimmtes Modell verfügen, können Sie es als benutzerdefiniertes Modell bereitstellen und wie jedes andere bereitgestellte Modell entweder im **Chat**-Playground von Azure AI Foundry oder über die Chatvervollständigungs-API verwenden.

1. Führen Sie in einer neuen Zelle den folgenden Code aus, um Ihr fein abgestimmtes Modell bereitzustellen:
   
    ```python
   import json
   import requests

   token = os.getenv("TEMP_AUTH_TOKEN")
   subscription = "<YOUR_SUBSCRIPTION_ID>"
   resource_group = "<YOUR_RESOURCE_GROUP_NAME>"
   resource_name = "<YOUR_AZURE_OPENAI_RESOURCE_NAME>"
   model_deployment_name = "gpt-4o-ft"

   deploy_params = {'api-version': "2023-05-01"}
   deploy_headers = {'Authorization': 'Bearer {}'.format(token), 'Content-Type': 'application/json'}

   deploy_data = {
       "sku": {"name": "standard", "capacity": 1},
       "properties": {
           "model": {
               "format": "OpenAI",
               "name": "<YOUR_FINE_TUNED_MODEL>",
               "version": "1"
           }
       }
   }
   deploy_data = json.dumps(deploy_data)

   request_url = f'https://management.azure.com/subscriptions/{subscription}/resourceGroups/{resource_group}/providers/Microsoft.CognitiveServices/accounts/{resource_name}/deployments/{model_deployment_name}'

   print('Creating a new deployment...')

   r = requests.put(request_url, params=deploy_params, headers=deploy_headers, data=deploy_data)

   print(r)
   print(r.reason)
   print(r.json())
    ```

2. Führen Sie in einer neuen Zelle den folgenden Code aus, um Ihr angepasstes Modell in einem Chatvervollständigungs-Aufruf zu verwenden:
   
    ```python
   import os
   from openai import AzureOpenAI

   client = AzureOpenAI(
     azure_endpoint = os.getenv("AZURE_OPENAI_ENDPOINT"),
     api_key = os.getenv("AZURE_OPENAI_API_KEY"),
     api_version = "2024-02-01"
   )

   response = client.chat.completions.create(
       model = "gpt-4o-ft", # model = "Custom deployment name you chose for your fine-tuning model"
       messages = [
           {"role": "system", "content": "You are a helpful assistant."},
           {"role": "user", "content": "Does Azure OpenAI support customer managed keys?"},
           {"role": "assistant", "content": "Yes, customer managed keys are supported by Azure OpenAI."},
           {"role": "user", "content": "Do other Azure AI services support this too?"}
       ]
   )

   print(response.choices[0].message.content)
    ```
 
## Bereinigen

Wenn Sie mit Ihrer Azure OpenAI-Ressource fertig sind, denken Sie daran, die Bereitstellung oder die gesamte Ressource im **Azure-Portal** auf `https://portal.azure.com` zu löschen.

Wählen Sie zunächst im Azure Databricks-Portal auf der Seite **Compute** Ihren Cluster und dann **&#9632; Beenden** aus, um ihn herunterzufahren.

Wenn Sie die Erkundung von Azure Databricks abgeschlossen haben, löschen Sie die erstellten Ressourcen, um unnötige Azure-Kosten zu vermeiden und Kapazität in Ihrem Abonnement freizugeben.
