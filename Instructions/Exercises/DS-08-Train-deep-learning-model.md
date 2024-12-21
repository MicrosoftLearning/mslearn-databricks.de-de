---
lab:
  title: Trainieren eines Deep Learning-Modells
---

# Trainieren eines Deep Learning-Modells

In dieser Übung verwenden Sie die Bibliothek **PyTorch**, um ein Deep Learning-Modell in Azure Databricks zu trainieren. Anschließend verwenden Sie die Bibliothek **Horovod**, um das Deep Learning-Training über mehrere Workerknoten in einem Cluster zu verteilen.

Diese Übung dauert ca. **45** Minuten.

## Vorbereitung

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

    ```
    rm -r mslearn-databricks -f
    git clone https://github.com/MicrosoftLearning/mslearn-databricks
    ```

5. Nachdem das Repository geklont wurde, geben Sie den folgenden Befehl ein, um das Skript **setup.ps1** auszuführen, das einen Azure Databricks-Arbeitsbereich in einer verfügbaren Region bereitstellt:

    ```
    ./mslearn-databricks/setup.ps1
    ```

6. Wenn Sie dazu aufgefordert werden, wählen Sie aus, welches Abonnement Sie verwenden möchten (dies geschieht nur, wenn Sie Zugriff auf mehrere Azure-Abonnements haben).
7. Warten Sie, bis das Skript abgeschlossen ist. Dies dauert in der Regel etwa 5 Minuten, in einigen Fällen kann es jedoch länger dauern. Während Sie warten, lesen Sie den Artikel [Verteiltes Training](https://learn.microsoft.com/azure/databricks/machine-learning/train-model/distributed-training/) in der Azure Databricks-Dokumentation.

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

## Erstellen eines Notebooks

Sie führen Code aus, der die Spark MLLib-Bibliothek verwendet, um ein Machine Learning-Modell zu trainieren. Daher besteht der erste Schritt darin, ein neues Notebook in Ihrem Arbeitsbereich zu erstellen.

1. Verwenden Sie in der Randleiste den Link ** (+) Neu**, um ein **Notebook** zu erstellen.
1. Ändern Sie den Standardnamen des Notebooks (**Unbenanntes Notebook *[Datum]***) in **Deep Learning**, und wählen Sie in der Dropdownliste **Verbinden** Ihren Cluster aus, sofern er noch nicht ausgewählt ist. Wenn der Cluster nicht ausgeführt wird, kann es eine Minute dauern, bis er gestartet wird.

## Erfassen und Vorbereiten von Daten

Das Szenario für diese Übung basiert auf Beobachtungen von Pinguinen in der Antarktis. Das Ziel besteht darin, ein Machine Learning-Modell zu trainieren, um die Art eines beobachteten Pinguins anhand seines Standorts und seiner Körpermaße vorherzusagen.

> **Quellenangaben:** Das in dieser Übung verwendete Pinguin-Dataset ist eine Teilmenge der Daten, die von [Dr. Kristen Gorman](https://www.uaf.edu/cfos/people/faculty/detail/kristen-gorman.php) und der [Palmer-Station (Antarktis-Forschungsstation)](https://pal.lternet.edu/), ein Mitglied des [Long Term Ecological Research Network](https://lternet.edu/) (Netzwerk für ökologische und ökosystemare Langzeitforschung), gesammelt und zur Verfügung gestellt werden.

1. Geben Sie in der ersten Zelle des Notebooks den folgenden Code ein, der mit *Shell*-Befehlen die Pinguindaten von GitHub in das von Ihrem Cluster verwendete Dateisystem herunterlädt.

    ```bash
    %sh
    rm -r /dbfs/deepml_lab
    mkdir /dbfs/deepml_lab
    wget -O /dbfs/deepml_lab/penguins.csv https://raw.githubusercontent.com/MicrosoftLearning/mslearn-databricks/main/data/penguins.csv
    ```

1. Verwenden Sie Menüoption **&#9656; Zelle Ausführen** links neben der Zelle, um sie auszuführen. Warten Sie dann, bis der vom Code ausgeführte Spark-Auftrag, abgeschlossen ist.
1. Bereiten Sie nun die Daten für das maschinelle Lernen vor. Verwenden Sie unter der vorhandenen Codezelle das Symbol **+**, um eine neue Codezelle hinzuzufügen. Geben Sie dann den folgenden Code in die neue Zelle ein, und führen Sie ihn aus, um Folgendes zu tun:
    - Entfernen aller unvollständigen Zeilen
    - Codieren des Inselnamens (Zeichenfolge) als ganze Zahl
    - Verwenden geeigneter Datentypen
    - Normalisieren der numerischen Daten auf eine ähnliche Skala
    - Teilen Sie die Daten in zwei Datasets auf: eines zum Trainieren und ein weiteres zum Testen.

    ```python
   from pyspark.sql.types import *
   from pyspark.sql.functions import *
   from sklearn.model_selection import train_test_split
   
   # Load the data, removing any incomplete rows
   df = spark.read.format("csv").option("header", "true").load("/deepml_lab/penguins.csv").dropna()
   
   # Encode the Island with a simple integer index
   # Scale FlipperLength and BodyMass so they're on a similar scale to the bill measurements
   islands = df.select(collect_set("Island").alias('Islands')).first()['Islands']
   island_indexes = [(islands[i], i) for i in range(0, len(islands))]
   df_indexes = spark.createDataFrame(island_indexes).toDF('Island', 'IslandIdx')
   data = df.join(df_indexes, ['Island'], 'left').select(col("IslandIdx"),
                      col("CulmenLength").astype("float"),
                      col("CulmenDepth").astype("float"),
                      (col("FlipperLength").astype("float")/10).alias("FlipperScaled"),
                       (col("BodyMass").astype("float")/100).alias("MassScaled"),
                      col("Species").astype("int")
                       )
   
   # Oversample the dataframe to triple its size
   # (Deep learning techniques like LOTS of data)
   for i in range(1,3):
       data = data.union(data)
   
   # Split the data into training and testing datasets   
   features = ['IslandIdx','CulmenLength','CulmenDepth','FlipperScaled','MassScaled']
   label = 'Species'
      
   # Split data 70%-30% into training set and test set
   x_train, x_test, y_train, y_test = train_test_split(data.toPandas()[features].values,
                                                       data.toPandas()[label].values,
                                                       test_size=0.30,
                                                       random_state=0)
   
   print ('Training Set: %d rows, Test Set: %d rows \n' % (len(x_train), len(x_test)))
    ```

## Installieren und Importieren der PyTorch-Bibliotheken

PyTorch ist ein Framework zum Erstellen von Machine Learning-Modellen, einschließlich Deep Neural Networks (DNNs). Da wir die Verwendung von PyTorch zum Erstellen unseres Pinguinklassifizierers planen, müssen wir die PyTorch-Bibliotheken importieren, die wir verwenden möchten. PyTorch ist bereits auf Azure Databricks-Clustern mit einer ML Databricks-Laufzeit installiert (die spezifische Installation von PyTorch hängt davon ab, ob der Cluster über GPUs (Graphics Processing Units, Grafikprozessoren) verfügt, die über *cuda*zur Hochleistungsverarbeitung verwendet werden können).

1. Fügen Sie eine neue Codezelle hinzu, und führen Sie den folgenden Code aus, um die Verwendung von PyTorch vorzubereiten:

    ```python
   import torch
   import torch.nn as nn
   import torch.utils.data as td
   import torch.nn.functional as F
   
   # Set random seed for reproducability
   torch.manual_seed(0)
   
   print("Libraries imported - ready to use PyTorch", torch.__version__)
    ```

## Erstellen von Datenladern

PyTorch nutzt *Datenlader*, um Trainings- und Validierungsdaten in Batches zu laden. Wir haben die Daten bereits in numpy-Arrays geladen, müssen diese in PyTorch-Datasets (in denen die Daten in PyTorch *Tensor*-Objekte konvertiert werden) jedoch umschließen und Lader zum Lesen von Batches aus diesen Datasets erstellen.

1. Fügen Sie eine Zelle hinzu, und führen Sie den folgenden Code aus, um Datenlader vorzubereiten:

    ```python
   # Create a dataset and loader for the training data and labels
   train_x = torch.Tensor(x_train).float()
   train_y = torch.Tensor(y_train).long()
   train_ds = td.TensorDataset(train_x,train_y)
   train_loader = td.DataLoader(train_ds, batch_size=20,
       shuffle=False, num_workers=1)

   # Create a dataset and loader for the test data and labels
   test_x = torch.Tensor(x_test).float()
   test_y = torch.Tensor(y_test).long()
   test_ds = td.TensorDataset(test_x,test_y)
   test_loader = td.DataLoader(test_ds, batch_size=20,
                                shuffle=False, num_workers=1)
   print('Ready to load data')
    ```

## Definieren eines neuronalen Netzes

Jetzt sind wir bereit, unser neuronales Netzwerk zu definieren. In diesem Fall erstellen wir ein Netzwerk, das aus 3 vollständig verbundenen Ebenen besteht:

- Eine Eingabeebene, die einen Eingabewert für jedes Feature empfängt (in diesem Fall der Inselindex und vier Pinguinmessungen) und 10 Ausgaben generiert.
- Eine ausgeblendete Ebene, die zehn Eingaben von der Eingabeebene empfängt und zehn Ausgaben an die nächste Ebene sendet.
- Eine Ausgabeebene, die einen Vektor von Wahrscheinlichkeiten für jede der drei möglichen Pinguinarten generiert.

Während wir das Netzwerk trainieren, indem wir Daten durch das Netzwerk leiten, wendet die **forward**-Funktion *RELU*-Aktivierungsfunktionen auf die ersten beiden Ebenen an (um die Ergebnisse auf positive Zahlen zu beschränken) und gibt eine endgültige Ausgabeebene zurück, die eine *log_softmax*-Funktion verwendet, um einen Wert zurückzugeben, der eine Wahrscheinlichkeitsbewertung für jede der drei möglichen Klassen darstellt.

1. Führen Sie den folgenden Code aus, um das neuronale Netz zu definieren:

    ```python
   # Number of hidden layer nodes
   hl = 10
   
   # Define the neural network
   class PenguinNet(nn.Module):
       def __init__(self):
           super(PenguinNet, self).__init__()
           self.fc1 = nn.Linear(len(features), hl)
           self.fc2 = nn.Linear(hl, hl)
           self.fc3 = nn.Linear(hl, 3)
   
       def forward(self, x):
           fc1_output = torch.relu(self.fc1(x))
           fc2_output = torch.relu(self.fc2(fc1_output))
           y = F.log_softmax(self.fc3(fc2_output).float(), dim=1)
           return y
   
   # Create a model instance from the network
   model = PenguinNet()
   print(model)
    ```

## Erstellen von Funktionen zum Trainieren und Testen eines Modells eines neuronalen Netzes

Um das Modell zu trainieren, müssen wir die Trainingswerte wiederholt durch das Netzwerk weiterleiten, eine Verlustfunktion verwenden, um den Verlust zu berechnen, einen Optimierer verwenden, um die Anpassungen der Gewichtungs- und Verschiebungswerte zurückzuleiten, und das Modell anhand der zurückgehaltenen Testdaten validieren.

1. Verwenden Sie dazu den folgenden Code, um eine Funktion zum Trainieren und Optimieren des Modells und eine Funktion zum Testen des Modells zu erstellen.

    ```python
   def train(model, data_loader, optimizer):
       device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
       model.to(device)
       # Set the model to training mode
       model.train()
       train_loss = 0
       
       for batch, tensor in enumerate(data_loader):
           data, target = tensor
           #feedforward
           optimizer.zero_grad()
           out = model(data)
           loss = loss_criteria(out, target)
           train_loss += loss.item()
   
           # backpropagate adjustments to the weights
           loss.backward()
           optimizer.step()
   
       #Return average loss
       avg_loss = train_loss / (batch+1)
       print('Training set: Average loss: {:.6f}'.format(avg_loss))
       return avg_loss
              
               
   def test(model, data_loader):
       device = torch.device('cuda' if torch.cuda.is_available() else 'cpu')
       model.to(device)
       # Switch the model to evaluation mode (so we don't backpropagate)
       model.eval()
       test_loss = 0
       correct = 0
   
       with torch.no_grad():
           batch_count = 0
           for batch, tensor in enumerate(data_loader):
               batch_count += 1
               data, target = tensor
               # Get the predictions
               out = model(data)
   
               # calculate the loss
               test_loss += loss_criteria(out, target).item()
   
               # Calculate the accuracy
               _, predicted = torch.max(out.data, 1)
               correct += torch.sum(target==predicted).item()
               
       # Calculate the average loss and total accuracy for this epoch
       avg_loss = test_loss/batch_count
       print('Validation set: Average loss: {:.6f}, Accuracy: {}/{} ({:.0f}%)\n'.format(
           avg_loss, correct, len(data_loader.dataset),
           100. * correct / len(data_loader.dataset)))
       
       # return average loss for the epoch
       return avg_loss
    ```

## Trainieren eines Modells

Jetzt können Sie die Funktionen **train** und **test** verwenden, um das Modell eines neuronalen Netzes zu trainieren. Neuronale Netze werden iterativ über mehrere *Epochen* hinweg trainiert, wobei die Verlust- und Genauigkeitsstatistiken für jede Epoche protokolliert werden.

1. Verwenden Sie den folgenden Code, um das Modell zu trainieren:

    ```python
   # Specify the loss criteria (we'll use CrossEntropyLoss for multi-class classification)
   loss_criteria = nn.CrossEntropyLoss()
   
   # Use an optimizer to adjust weights and reduce loss
   learning_rate = 0.001
   optimizer = torch.optim.Adam(model.parameters(), lr=learning_rate)
   optimizer.zero_grad()
   
   # We'll track metrics for each epoch in these arrays
   epoch_nums = []
   training_loss = []
   validation_loss = []
   
   # Train over 100 epochs
   epochs = 100
   for epoch in range(1, epochs + 1):
   
       # print the epoch number
       print('Epoch: {}'.format(epoch))
       
       # Feed training data into the model
       train_loss = train(model, train_loader, optimizer)
       
       # Feed the test data into the model to check its performance
       test_loss = test(model, test_loader)
       
       # Log the metrics for this epoch
       epoch_nums.append(epoch)
       training_loss.append(train_loss)
       validation_loss.append(test_loss)
    ```

    Während der Trainingsprozess ausgeführt wird, wollen wir versuchen, zu verstehen, was passiert:

    - In jeder *Epoche* werden die gesamten Trainingsdaten durch das Netzwerk weitergeleitet. Es gibt fünf Features für jede Beobachtung und fünf entsprechende Knoten in der Eingabeebene. Deshalb werden die Features für jede Beobachtung als Vektor von fünf Werten an diese Ebene übergeben. Aus Effizienzgründen werden die Featurevektoren jedoch in Batches gruppiert. Daher wird jeweils eine Matrix aus mehreren Featurevektoren eingegeben.
    - Die Matrix von Featurewerten wird von einer Funktion verarbeitet, die eine gewichtete Summe mit initialisierten Gewichtungen und Verschiebungswerten berechnet. Das Ergebnis dieser Funktion wird dann von der Aktivierungsfunktion für die Eingabeebene verarbeitet, um die Werte einzuschränken, die an die Knoten in der nächsten Ebene übergeben werden.
    - Die gewichtete Summe und die Aktivierungsfunktionen werden auf jeder Ebene wiederholt. Beachten Sie, dass die Funktionen mit Vektoren und Matrizen arbeiten und nicht mit einzelnen Skalarwerten. Mit anderen Worten: Der forward-Durchlauf besteht im Wesentlichen aus einer Reihe von geschachtelten linearen Algebrafunktionen. Aus diesem Grund bevorzugen Datenwissenschaftler Computer mit GPUs (Graphical Processing Units, Grafikprozessoren), da diese für Matrix- und Vektorberechnungen optimiert sind.
    - In der letzten Ebene des Netzwerks enthalten die Ausgabevektoren einen berechneten Wert für jede mögliche Klasse (in diesem Fall die Klassen 0, 1 und 2). Dieser Vektor wird von einer *Verlustfunktion* verarbeitet, die bestimmt, wie weit die Werte von den erwarteten Werten basierend auf den tatsächlichen Klassen entfernt sind. Angenommen, die Ausgabe für die Beobachtung eines Eselspinguins (Klasse 1) ist \[0,3, 0,4, 0,3\]. Die richtige Vorhersage wäre \[0,0, 1,0, 0,0\], sodass die Varianz zwischen den vorhergesagten und tatsächlichen Werten (wie weit jeder vorhergesagte Wert von dem erwarteten Wert abweicht) \[0,3, 0,6, 0,3\] beträgt. Diese Varianz wird für jeden Batch aggregiert und als laufendes Aggregat verwaltet, um die Gesamtfehlerquote (*Verlust*) der Trainingsdaten für die Epoche zu berechnen.
    - Am Ende jeder Epoche werden die Validierungsdaten durch das Netzwerk geleitet, und der Verlust und die Genauigkeit (Anteil der richtigen Vorhersagen auf der Grundlage des höchsten Wahrscheinlichkeitswerts im Ausgabevektor) werden ebenfalls berechnet. Dies ist nützlich, da es uns ermöglicht, die Leistung des Modells nach jeder Epoche mit Daten zu vergleichen, für die es nicht trainiert wurde, um festzustellen, ob es für neue Daten gut verallgemeinert oder ob es für die Trainingsdaten *überangepasst* ist.
    - Nachdem alle Daten durch das Netzwerk weitergeleitet wurden, wird die Ausgabe der Verlustfunktion für die *Trainingsdaten* (aber <u>nicht</u> die *Validierungsdaten*) an den Optimierer übergeben. Die genauen Details, wie der Optimierer den Verlust verarbeitet, variieren je nach dem verwendeten Optimierungsalgorithmus. Grundsätzlich können wir uns das gesamte Netzwerk, von der Eingabeschicht bis zur Verlustfunktion, jedoch als eine große verschachtelte (*zusammengesetzte*) Funktion vorstellen. Der Optimierer wendet einige Differentialrechnungen an, um *partielle Ableitungen* für die Funktion in Bezug auf jede Gewichtung und jeden Verschiebungswert zu berechnen, die im Netzwerk verwendet wurden. Dies ist bei einer verschachtelten Funktion dank der so genannten *Kettenregel* möglich, mit der die Ableitung einer zusammengesetzten Funktion aus den Ableitungen ihrer inneren und äußeren Funktionen bestimmt werden kann. Sie müssen sich eigentlich nicht um die mathematischen Details (das Optimierungsprogramm erledigt das für Sie), doch das Endergebnis ist, dass die partiellen Ableitungen uns über die Steigung (oder den *Gradienten*) der Verlustfunktion in Bezug auf jeden Gewichtungs- und Verschiebungswert informieren. Mit anderen Worten, wir können bestimmen, ob wir die Gewichtungs- und Verschiebungswerte erhöhen oder verringern müssen, um den Verlust zu minimieren.
    - Nachdem bestimmt wurde, in welche Richtung die Gewichte und Verschiebungen angepasst werden sollen, verwendet der Optimierer die *Lernrate*, um zu ermitteln, um wie viel sie angepasst werden sollen, und arbeitet dann rückwärts durch das Netzwerk in einem Prozess namens *Backpropagation*, um den Gewichtungen und Verschiebungen in jeder Ebene neue Werte zuzuweisen.
    - In der nächsten Epoche wird der gesamte Trainings-, Validierungs- und Backpropagation-Prozess mit den überarbeiteten Gewichtungen und Verschiebungen aus der vorherigen Epoche wiederholt, was hoffentlich zu einer geringeren Verlustquote führt.
    - Der Prozess wird über 100 Epochen fortgesetzt.

## Überprüfen von Trainings- und Überprüfungsverlust

Nach Abschluss des Trainings können wir die Verlustmetriken untersuchen, die wir während des Trainings und der Überprüfung des Modells aufgezeichnet haben. Wir suchen eigentlich nach zwei Dingen:

- Der Verlust sollte mit jeder Epoche abnehmen, was zeigt, dass das Modell die richtigen Gewichtungen und Verschiebungen lernt, um die richtigen Bezeichnungen vorherzusagen.
- Der Trainingsverlust und der Validierungsverlust sollten einem ähnlichen Trend folgen, was zeigt, dass sich das Modell nicht zu stark an die Trainingsdaten anpasst.

1. Verwenden Sie den folgenden Code, um den Verlust zu plotten:

    ```python
   %matplotlib inline
   from matplotlib import pyplot as plt
   
   plt.plot(epoch_nums, training_loss)
   plt.plot(epoch_nums, validation_loss)
   plt.xlabel('epoch')
   plt.ylabel('loss')
   plt.legend(['training', 'validation'], loc='upper right')
   plt.show()
    ```

## Anzeigen der gelernten Gewichtungen und Verschiebungen

Das trainierte Modell besteht aus den endgültigen Gewichtungen und Verschiebungen, die während des Trainings vom Optimierer bestimmt wurden. Auf der Grundlage unseres Netzwerkmodells sollten wir die folgenden Werte für jede Ebene erwarten:

- Ebene 1 (*fc1*): Es gibt fünf Eingabewerte, die an zehn Ausgabeknoten weitergeleitet werden, sodass es 10 x 5 Gewichtungen und 10 Verschiebungswerte geben sollte.
- Ebene 2 (*fc2*): Es gibt zehn Eingabewerte, die an zehn Ausgabeknoten weitergeleitet werden, sodass es 10 x 10 Gewichtungen und 10 Verschiebungswerte geben sollte.
- Ebene 3 (*fc3*): Es gibt zehn Eingabewerte, die an drei Ausgabeknoten weitergeleitet werden, sodass es 3 x 10 Gewichtungen und 3 Verschiebungswerte geben sollte.

1. Verwenden Sie den folgenden Code, um die Ebenen im trainierten Modell anzuzeigen:

    ```python
   for param_tensor in model.state_dict():
       print(param_tensor, "\n", model.state_dict()[param_tensor].numpy())
    ```

## Speichern und Verwenden des trainierten Modells

Nachdem wir nun über ein trainiertes Modell verfügen, können wir seine trainierten Gewichtungen für die spätere Verwendung speichern.

1. Verwenden Sie den folgenden Code, um das Modell zu speichern:

    ```python
   # Save the model weights
   model_file = '/dbfs/penguin_classifier.pt'
   torch.save(model.state_dict(), model_file)
   del model
   print('model saved as', model_file)
    ```

1. Führen Sie den folgenden Code aus, um die Modellgewichtungen zu laden und für eine neue Pinguinbeobachtung die Art vorherzusagen:

    ```python
   # New penguin features
   x_new = [[1, 50.4,15.3,20,50]]
   print ('New sample: {}'.format(x_new))
   
   # Create a new model class and load weights
   model = PenguinNet()
   model.load_state_dict(torch.load(model_file))
   
   # Set model to evaluation mode
   model.eval()
   
   # Get a prediction for the new data sample
   x = torch.Tensor(x_new).float()
   _, predicted = torch.max(model(x).data, 1)
   
   print('Prediction:',predicted.item())
    ```

## Bereinigen

Wählen Sie zunächst im Azure Databricks-Portal auf der Seite **Compute** Ihren Cluster und dann **&#9632; Beenden** aus, um ihn herunterzufahren.

Wenn Sie die Erkundung von Azure Databricks abgeschlossen haben, löschen Sie die erstellten Ressourcen, um unnötige Azure-Kosten zu vermeiden und Kapazität in Ihrem Abonnement freizugeben.
