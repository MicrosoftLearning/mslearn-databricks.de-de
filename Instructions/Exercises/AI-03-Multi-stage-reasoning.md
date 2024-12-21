---
lab:
  title: Mehrstufiges Reasoning mit LangChain unter Verwendung von Azure Databricks und Azure OpenAI
---

# Mehrstufiges Reasoning mit LangChain unter Verwendung von Azure Databricks und Azure OpenAI

Mehrstufiges Reasoning ist ein innovativer Ansatz in der KI, bei dem komplexe Probleme in kleinere, leichter zu handhabende Phasen zerlegt werden. LangChain, ein Softwareframework, erleichtert die Erstellung von Anwendungen, die große Sprachmodelle (LLMs) nutzen. Durch die Integration mit Azure Databricks ermöglicht LangChain das nahtlose Laden von Daten, Modell-Wrapping und die Entwicklung von anspruchsvollen KI-Agenten. Diese Kombination ist besonders leistungsfähig bei der Verarbeitung komplizierter Aufgaben, die ein tiefes Verständnis des Kontextes und die Fähigkeit zum Denken über mehrere Schritte hinweg erfordern.

Dieses Lab dauert ungefähr **30** Minuten.

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
        - Australien (Osten)
        - Kanada, Osten
        - East US
        - USA (Ost) 2
        - Frankreich, Mitte
        - Japan, Osten
        - USA Nord Mitte
        - Schweden, Mitte
        - Schweiz, Norden
        - UK, Süden
    - **Name:** *Wählen Sie einen Namen Ihrer Wahl aus.*
    - **Tarif**: Standard S0.

> \* Azure OpenAI-Ressourcen werden durch regionale Kontingente eingeschränkt. Die aufgeführten Regionen enthalten das Standardkontingent für die in dieser Übung verwendeten Modelltypen. Durch die zufällige Auswahl einer Region wird das Risiko reduziert, dass eine einzelne Region ihr Kontingentlimit in Szenarien erreicht, in denen Sie ein Abonnement für andere Benutzer freigeben. Wenn später in der Übung ein Kontingentlimit erreicht wird, besteht eventuell die Möglichkeit, eine andere Ressource in einer anderen Region zu erstellen.

3. Warten Sie, bis die Bereitstellung abgeschlossen ist. Wechseln Sie dann zur bereitgestellten Azure OpenAI-Ressource im Azure-Portal.

4. Wählen Sie im linken Fensterbereich unter **Ressourcenverwaltung** die Option **Tasten und Endpunkt**.

5. Kopieren Sie den Endpunkt und einen der verfügbaren Schlüssel, da Sie ihn später in dieser Übung verwenden werden.

## Erforderliche Modelle bereitstellen

Azure bietet ein webbasiertes Portal namens **Azure AI Studio**, das Sie zur Bereitstellung, Verwaltung und Untersuchung von Modellen verwenden können. Sie beginnen Ihre Erkundung von Azure OpenAI, indem Sie Azure AI Studio verwenden, um ein Modell bereitzustellen.

> **Hinweis**: Während Sie Azure AI Studio verwenden, werden möglicherweise Meldungsfelder mit Vorschlägen für auszuführende Aufgaben angezeigt. Sie können diese schließen und die Schritte in dieser Übung ausführen.

1. Scrollen Sie im Azure-Portal auf der Seite **Übersicht** für Ihre Azure OpenAI-Ressource nach unten zum Abschnitt **Erste Schritte** und klicken Sie auf die Schaltfläche, um zu **Azure AI Studio** zu gelangen.
   
1. Wählen Sie in Azure AI Studio im linken Bereich die Seite "**Deployments**" aus und sehen Sie sich Ihre vorhandenen Modellbereitstellungen an. Falls noch nicht vorhanden, erstellen Sie eine neue Bereitstellung des **gpt-35-turbo-16k**-Modells mit den folgenden Einstellungen:
    - **Bereitstellungsname**: *gpt-35-turbo-16k*
    - **Modell**: gpt-35-turbo-16k *(wenn das 16k-Modell nicht verfügbar ist, wählen Sie gpt-35-turbo und benennen Sie Ihren Einsatz entsprechend)*
    - **Modellversion**: *Standardversion verwenden*
    - **Bereitstellungstyp**: Standard
    - **Ratenlimit für Token pro Minute**: 5K\*
    - **Inhaltsfilter**: Standard
    - **Dynamische Quote aktivieren**: Deaktiviert
    
1. Gehen Sie zurück zur Seite **Bereitstellungen** und erstellen Sie eine neue Bereitstellung des Modells **text-embedding-ada-002** mit den folgenden Einstellungen:
    - **Bereitstellungsname**: *text-embedding-ada-002*
    - **Modell**: text-embedding-ada-002
    - **Modellversion**: *Standardversion verwenden*
    - **Bereitstellungstyp**: Standard
    - **Ratenlimit für Token pro Minute**: 5K\*
    - **Inhaltsfilter**: Standard
    - **Dynamische Quote aktivieren**: Deaktiviert

> \* Ein Ratenlimit von 5.000 Token pro Minute ist mehr als ausreichend, um diese Aufgabe zu erfüllen und gleichzeitig Kapazität für andere Personen zu schaffen, die das gleiche Abonnement nutzen.

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

> **Tipp**: Wenn Sie bereits über einen Cluster mit einer Runtime 13.3 LTS **<u>ML</u>** oder einer höheren Runtimeversion in Ihrem Azure Databricks-Arbeitsbereich verfügen, können Sie ihn verwenden, um diese Übung abzuschließen, und dieses Verfahren überspringen.

1. Navigieren Sie im Azure-Portal zu der Ressourcengruppe, in der der Azure Databricks-Arbeitsbereich erstellt wurde.
2. Wählen Sie Ihre Azure Databricks Service-Ressource aus.
3. Verwenden Sie auf der Seite **Übersicht** für Ihren Arbeitsbereich die Schaltfläche **Arbeitsbereich starten**, um Ihren Azure Databricks-Arbeitsbereich auf einer neuen Browserregisterkarte zu öffnen. Melden Sie sich an, wenn Sie dazu aufgefordert werden.

> **Tipp**: Während Sie das Databricks-Arbeitsbereichsportal verwenden, werden möglicherweise verschiedene Tipps und Benachrichtigungen angezeigt. Schließen Sie diese, und folgen Sie den Anweisungen, um die Aufgaben in dieser Übung auszuführen.

4. Wählen Sie zunächst in der Randleiste auf der linken Seite die Aufgabe **(+) Neu** und dann **Cluster** aus.
5. Erstellen Sie auf der Seite **Neuer Cluster** einen neuen Cluster mit den folgenden Einstellungen:
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

6. Warten Sie, bis der Cluster erstellt wurde. Es kann ein oder zwei Minuten dauern.

> **Hinweis**: Wenn Ihr Cluster nicht gestartet werden kann, verfügt Ihr Abonnement möglicherweise über ein unzureichendes Kontingent in der Region, in der Ihr Azure Databricks-Arbeitsbereich bereitgestellt wird. Details finden Sie unter [Der Grenzwert für CPU-Kerne verhindert die Clustererstellung](https://docs.microsoft.com/azure/databricks/kb/clusters/azure-core-limit). In diesem Fall können Sie versuchen, Ihren Arbeitsbereich zu löschen und in einer anderen Region einen neuen zu erstellen.

## Installieren der erforderlichen Bibliotheken

1. Gehen Sie im Databricks-Arbeitsbereich zum Abschnitt **Arbeitsbereich**.

2. Wählen Sie „**Erstellen**“ und dann „**Notizbuch**“ aus.

3. Geben Sie Ihrem Notizbuch einen Namen und wählen Sie `Python` als Sprache aus.

4. In der ersten Codezelle geben Sie den folgenden Code ein und führen ihn aus, um die erforderlichen Bibliotheken zu installieren:
   
     ```python
    %pip install langchain openai langchain_openai faiss-cpu
     ```

5. Nachdem die Installation abgeschlossen ist, starten Sie den Kernel in einer neuen Zelle neu:

     ```python
    %restart_python
     ```

6. Definieren Sie in einer neuen Zelle die Authentifizierungsparameter, die zur Initialisierung der OpenAI-Modelle verwendet werden, und ersetzen Sie `your_openai_endpoint` und `your_openai_api_key` durch den Endpunkt und den Schlüssel, die Sie zuvor aus Ihrer OpenAI-Ressource kopiert haben:

     ```python
    endpoint = "your_openai_endpoint"
    key = "your_openai_api_key"
     ```
     
## Vektorindex erstellen und Einbettungen speichern

Ein Vektorindex ist eine spezielle Datenstruktur, die eine effiziente Speicherung und Abfrage von hochdimensionalen Vektordaten ermöglicht, was für die Durchführung von schnellen Ähnlichkeitssuchen und Abfragen nach dem nächsten Nachbarn entscheidend ist. Einbettungen hingegen sind numerische Darstellungen von Objekten, die ihre Bedeutung in einer Vektorform erfassen und es Maschinen ermöglichen, verschiedene Arten von Daten, einschließlich Text und Bilder, zu verarbeiten und zu verstehen.

1. Führen Sie den folgenden Code in einer neuen Zelle aus, um einen Beispieldatensatz zu laden:

     ```python
    from langchain_core.documents import Document

    documents = [
         Document(page_content="Azure Databricks is a fast, easy, and collaborative Apache Spark-based analytics platform.", metadata={"date_created": "2024-08-22"}),
         Document(page_content="LangChain is a framework designed to simplify the creation of applications using large language models.", metadata={"date_created": "2024-08-22"}),
         Document(page_content="GPT-4 is a powerful language model developed by OpenAI.", metadata={"date_created": "2024-08-22"})
    ]
    ids = ["1", "2", "3"]
     ```
     
1. Führen Sie in einer neuen Zelle den folgenden Code aus, um Einbettungen nach dem `text-embedding-ada-002`-Modell zu erzeugen:

     ```python
    from langchain_openai import AzureOpenAIEmbeddings
     
    embedding_function = AzureOpenAIEmbeddings(
        deployment="text-embedding-ada-002",
        model="text-embedding-ada-002",
        azure_endpoint=endpoint,
        openai_api_key=key,
        chunk_size=1
    )
     ```
     
1. Führen Sie in einer neuen Zelle den folgenden Code aus, um einen Vektorindex zu erstellen, der das erste Textbeispiel als Referenz für die Vektordimension verwendet:

     ```python
    import faiss
      
    index = faiss.IndexFlatL2(len(embedding_function.embed_query("Azure Databricks is a fast, easy, and collaborative Apache Spark-based analytics platform.")))
     ```

## Eine Kette auf Retriever-Basis erstellen

Eine Retriever-Komponente ruft relevante Dokumente oder Daten auf der Grundlage einer Abfrage ab. Dies ist besonders nützlich bei Anwendungen, die die Integration großer Datenmengen für die Analyse erfordern, wie z. B. bei Systemen zur abrufgestützten Generierung.

1. Führen Sie in einer neuen Zelle den folgenden Code aus, um einen Retriever zu erstellen, der den Vektorindex nach den ähnlichsten Texten durchsuchen kann.

     ```python
    from langchain.vectorstores import FAISS
    from langchain_core.vectorstores import VectorStoreRetriever
    from langchain_community.docstore.in_memory import InMemoryDocstore

    vector_store = FAISS(
        embedding_function=embedding_function,
        index=index,
        docstore=InMemoryDocstore(),
        index_to_docstore_id={}
    )
    vector_store.add_documents(documents=documents, ids=ids)
    retriever = VectorStoreRetriever(vectorstore=vector_store)
     ```

1. Führen Sie in einer neuen Zelle den folgenden Code aus, um ein QA-System mit dem Retriever und dem Modell `gpt-35-turbo-16k` zu erstellen:
    
     ```python
    from langchain_openai import AzureChatOpenAI
    from langchain_core.prompts import ChatPromptTemplate
    from langchain.chains.combine_documents import create_stuff_documents_chain
    from langchain.chains import create_retrieval_chain
     
    llm = AzureChatOpenAI(
        deployment_name="gpt-35-turbo-16k",
        model_name="gpt-35-turbo-16k",
        azure_endpoint=endpoint,
        api_version="2023-03-15-preview",
        openai_api_key=key,
    )

    system_prompt = (
        "Use the given context to answer the question. "
        "If you don't know the answer, say you don't know. "
        "Use three sentences maximum and keep the answer concise. "
        "Context: {context}"
    )

    prompt1 = ChatPromptTemplate.from_messages([
        ("system", system_prompt),
        ("human", "{input}")
    ])

    chain = create_stuff_documents_chain(llm, prompt)

    qa_chain1 = create_retrieval_chain(retriever, chain)
     ```

1. Führen Sie in einer neuen Zelle den folgenden Code aus, um das QA-System zu testen:

     ```python
    result = qa_chain1.invoke({"input": "What is Azure Databricks?"})
    print(result)
     ```

Die Ergebnisausgabe sollte Ihnen eine Antwort zeigen, die auf dem relevanten Dokument im Beispieldatensatz und dem vom LLM erzeugten generativen Text basiert.

## Kombinieren von Ketten in einem Mehrkettensystem

Langchain ist ein vielseitiges Tool, das die Kombination mehrerer Ketten in ein Mehrkettensystem ermöglicht und die Funktionen von Sprachmodellen verbessert. Bei diesem Prozess werden verschiedene Komponenten in Zeichenfolge aneinandergereiht, die Eingaben parallel oder nacheinander verarbeiten können, um schließlich eine endgültige Antwort zu synthetisieren.

1. Führen Sie in einer neuen Zelle den folgenden Code aus, um eine zweite Kette zu erstellen

     ```python
    from langchain_core.prompts import ChatPromptTemplate
    from langchain_core.output_parsers import StrOutputParser

    prompt2 = ChatPromptTemplate.from_template("Create a social media post based on this summary: {summary}")

    qa_chain2 = ({"summary": qa_chain1} | prompt2 | llm | StrOutputParser())
     ```

1. Führen Sie in einer neuen Zelle den folgenden Code aus, um eine mehrstufige Kette mit einer bestimmten Eingabe aufzurufen:

     ```python
    result = qa_chain2.invoke({"input": "How can we use LangChain?"})
    print(result)
     ```

Die erste Kette stellt auf der Grundlage des bereitgestellten Beispiel-DataSets eine Antwort auf die Eingabe bereit, während die zweite Kette auf der Grundlage der Ausgabe der ersten Kette einen Beitrag in den sozialen Medien erstellt. Mit diesem Ansatz können Sie komplexere Textverarbeitungsaufgaben verarbeiten, indem Sie mehrere Schritte miteinander verketten.

## Bereinigen

Wenn Sie mit Ihrer Azure OpenAI-Ressource fertig sind, denken Sie daran, die Bereitstellung oder die gesamte Ressource im **Azure-Portal** auf `https://portal.azure.com` zu löschen.

Wählen Sie zunächst im Azure Databricks-Portal auf der Seite **Compute** Ihren Cluster und dann **&#9632; Beenden** aus, um ihn herunterzufahren.

Wenn Sie die Erkundung von Azure Databricks abgeschlossen haben, löschen Sie die erstellten Ressourcen, um unnötige Azure-Kosten zu vermeiden und Kapazität in Ihrem Abonnement freizugeben.
