---
title: Distribuire la soluzione di rilevamento calpestio basata su intelligenza artificiale in Azure e nell'hub Azure Stack
description: Informazioni su come distribuire una soluzione di rilevamento calpestio basata su intelligenza artificiale per l'analisi del traffico dei visitatori nei negozi al dettaglio usando Azure e l'hub Azure Stack.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 6913cc522da447092dad0af24e148a3b2576495c
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910894"
---
# <a name="deploy-an-ai-based-footfall-detection-solution-using-azure-and-azure-stack-hub"></a>Distribuire una soluzione di rilevamento calpestio basata su intelligenza artificiale usando Azure e l'hub Azure Stack

Questo articolo descrive come distribuire una soluzione basata su intelligenza artificiale che genera informazioni dettagliate da azioni reali usando Azure, Azure Stack Hub e il Visione personalizzata AI Dev Kit.

In questa soluzione si apprenderà come:

> [!div class="checklist"]
> - Distribuire bundle di applicazioni native cloud (CNAB) al perimetro. 
> - Distribuire un'app che si estende sui limiti del cloud.
> - Usare Visione personalizzata AI Dev Kit per l'inferenza al perimetro.

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub è un'estensione di Azure. Azure Stack Hub offre l'agilità e l'innovazione di cloud computing all'ambiente locale, abilitando l'unico Cloud ibrido che consente di creare e distribuire app ibride ovunque.  
> 
> L'articolo [considerazioni sulla progettazione di app ibride](overview-app-design-considerations.md) esamina i pilastri della qualità del software (posizionamento, scalabilità, disponibilità, resilienza, gestibilità e sicurezza) per la progettazione, la distribuzione e la gestione di app ibride. Le considerazioni di progettazione consentono di ottimizzare la progettazione delle app ibride, riducendo al minimo le esigenze negli ambienti di produzione.

## <a name="prerequisites"></a>Prerequisiti

Prima di iniziare a usare questa guida alla distribuzione, assicurarsi di:

- Vedere l'argomento relativo al [modello di rilevamento calpestio](pattern-retail-footfall-detection.md) .
- Ottenere l'accesso utente a un'istanza di sistema integrata Azure Stack Development Kit (Gabriele) o Azure Stack Hub con:
  - Il [servizio app Azure nel provider di risorse dell'Hub Azure stack](/azure-stack/operator/azure-stack-app-service-overview.md) installato. È necessario l'accesso dell'operatore all'istanza dell'hub Azure Stack o collaborare con l'amministratore per installare.
  - Sottoscrizione a un'offerta che fornisce il servizio app e la quota di archiviazione. Per creare un'offerta, è necessario disporre dell'accesso operator.
- Ottenere l'accesso a una sottoscrizione di Azure.
  - Se non si ha una sottoscrizione di Azure, iscriversi per ottenere un [account di valutazione gratuito](https://azure.microsoft.com/free/) prima di iniziare.
- Creare due entità servizio nella directory:
  - Una configurata per l'uso con le risorse di Azure, con accesso all'ambito della sottoscrizione di Azure.
  - Una configurata per l'uso con Azure Stack risorse dell'hub, con accesso all'ambito della sottoscrizione dell'hub Azure Stack.
  - Per altre informazioni sulla creazione di entità servizio e l'autorizzazione dell'accesso, vedere [usare un'identità dell'app per accedere alle risorse](/azure-stack/operator/azure-stack-create-service-principals.md). Se si preferisce usare l'interfaccia della riga di comando di Azure, vedere [creare un'entità servizio di Azure con la CLI](https://docs.microsoft.com/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest)di Azure.
- Distribuire servizi cognitivi di Azure in Azure o hub Azure Stack.
  - Per prima cosa, [Scopri di più su servizi cognitivi](https://azure.microsoft.com/services/cognitive-services/).
  - Visitare quindi [distribuire servizi cognitivi di Azure all'hub Azure stack](/azure-stack/user/azure-stack-solution-template-cognitive-services.md) per distribuire servizi cognitivi nell'hub Azure stack. Per prima cosa è necessario iscriversi per accedere all'anteprima.
- Clonare o scaricare un Azure Visione personalizzata AI Dev Kit non configurato. Per informazioni dettagliate, vedere i [DevKit di intelligenza artificiale](https://azure.github.io/Vision-AI-DevKit-Pages/).
- Iscriversi per un account Power BI.
- I servizi cognitivi di Azure API Viso la chiave di sottoscrizione e l'URL dell'endpoint. È possibile ottenere entrambi con la versione di valutazione gratuita di [prova Servizi cognitivi](https://azure.microsoft.com/try/cognitive-services/?api=face-api) . In alternativa, seguire le istruzioni riportate in [creare un account servizi cognitivi](/azure/cognitive-services/cognitive-services-apis-create-account).
- Installare le risorse di sviluppo seguenti:
  - [Interfaccia della riga di comando di Azure 2.0](/azure-stack/user/azure-stack-version-profiles-azurecli2.md)
  - [Docker CE](https://hub.docker.com/search/?type=edition&offering=community)
  - [Porter](https://porter.sh/). Si usa porter per distribuire app cloud usando i manifesti del bundle CNAB forniti per l'utente.
  - [Visual Studio Code](https://code.visualstudio.com/)
  - [Strumenti di Azure per la Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools)
  - [Estensione Python per Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=ms-python.python)
  - [Python](https://www.python.org/)

## <a name="deploy-the-hybrid-cloud-app"></a>Distribuire l'app cloud ibrida

Usare prima l'interfaccia della riga di comando di Porter per generare un set di credenziali e quindi distribuire l'app cloud.  

1. Clonare o scaricare il codice di esempio della soluzione da https://github.com/azure-samples/azure-intelligent-edge-patterns . 

1. Porter genererà un set di credenziali che consentono di automatizzare la distribuzione dell'app. Prima di eseguire il comando di generazione delle credenziali, assicurarsi che siano disponibili gli elementi seguenti:

    - Entità servizio per l'accesso alle risorse di Azure, inclusi l'ID dell'entità servizio, la chiave e il DNS tenant.
    - ID sottoscrizione per la sottoscrizione di Azure.
    - Entità servizio per l'accesso alle risorse di Azure Stack Hub, inclusi l'ID dell'entità servizio, la chiave e il DNS tenant.
    - ID sottoscrizione per la sottoscrizione dell'hub Azure Stack.
    - I servizi cognitivi di Azure API Viso la chiave e l'URL dell'endpoint di risorsa.

1. Eseguire il processo di generazione delle credenziali di Porter e seguire le istruzioni:

   ```porter
   porter creds generate --tag intelligentedge/footfall-cloud-deployment:0.1.0
   ```

1. Porter richiede anche un set di parametri per l'esecuzione. Creare un file di testo del parametro e immettere le coppie nome/valore seguenti. Chiedere all'amministratore di Azure Stack hub se è necessaria assistenza per uno dei valori richiesti.

   > [!NOTE] 
   > Il `resource suffix` valore viene usato per garantire che le risorse della distribuzione abbiano nomi univoci in Azure. Deve essere una stringa univoca di lettere e numeri, non più di 8 caratteri.

    ```porter
    azure_stack_tenant_arm="Your Azure Stack Hub tenant endpoint"
    azure_stack_storage_suffix="Your Azure Stack Hub storage suffix"
    azure_stack_keyvault_suffix="Your Azure Stack Hub keyVault suffix"
    resource_suffix="A unique string to identify your deployment"
    azure_location="A valid Azure region"
    azure_stack_location="Your Azure Stack Hub location identifier"
    powerbi_display_name="Your first and last name"
    powerbi_principal_name="Your Power BI account email address"
    ```
   Salvare il file di testo e prendere nota del relativo percorso.

1. A questo punto è possibile distribuire l'app cloud ibrido usando Porter. Eseguire il comando di installazione e osservare come le risorse vengono distribuite in Azure e Azure Stack Hub:

    ```porter
    porter install footfall-cloud –tag intelligentedge/footfall-cloud-deployment:0.1.0 –creds footfall-cloud-deployment –param-file "path-to-cloud-parameters-file.txt"
    ```

1. Al termine della distribuzione, prendere nota dei valori seguenti:
    - Stringa di connessione della fotocamera.
    - Stringa di connessione dell'account di archiviazione dell'immagine.
    - Nomi dei gruppi di risorse.

## <a name="prepare-the-custom-vision-ai-devkit"></a>Preparare il DevKit di intelligenza artificiale Visione personalizzata

Configurare quindi il Visione personalizzata AI Dev Kit come illustrato nella [Guida introduttiva a DevKit per intelligenza artificiale](https://azure.github.io/Vision-AI-DevKit-Pages/docs/quick_start/). È anche possibile configurare e testare la fotocamera, usando la stringa di connessione fornita nel passaggio precedente.

## <a name="deploy-the-camera-app"></a>Distribuire l'app della fotocamera

Usare l'interfaccia della riga di comando di Porter per generare un set di credenziali e quindi distribuire l'app della fotocamera.

1. Porter genererà un set di credenziali che consentono di automatizzare la distribuzione dell'app. Prima di eseguire il comando di generazione delle credenziali, assicurarsi che siano disponibili gli elementi seguenti:

    - Entità servizio per l'accesso alle risorse di Azure, inclusi l'ID dell'entità servizio, la chiave e il DNS tenant.
    - ID sottoscrizione per la sottoscrizione di Azure.
    - Stringa di connessione dell'account di archiviazione dell'immagine fornita quando è stata distribuita l'app cloud.

1. Eseguire il processo di generazione delle credenziali di Porter e seguire le istruzioni:

    ```porter
    porter creds generate --tag intelligentedge/footfall-camera-deployment:0.1.0
    ```

1. Porter richiede anche un set di parametri per l'esecuzione. Creare un file di testo del parametro e immettere il testo seguente. Se non si conoscono alcuni dei valori richiesti, rivolgersi all'amministratore dell'hub Azure Stack.

    > [!NOTE]
    > Il `deployment suffix` valore viene usato per garantire che le risorse della distribuzione abbiano nomi univoci in Azure. Deve essere una stringa univoca di lettere e numeri, non più di 8 caratteri.

    ```porter
    iot_hub_name="Name of the IoT Hub deployed"
    deployment_suffix="Unique string here"
    ```

    Salvare il file di testo e prendere nota del relativo percorso.

4. A questo punto è possibile distribuire l'app della fotocamera con Porter. Eseguire il comando di installazione e osservare come viene creata la IoT Edge distribuzione.

    ```porter
    porter install footfall-camera –tag intelligentedge/footfall-camera-deployment:0.1.0 –creds footfall-camera-deployment –param-file "path-to-camera-parameters-file.txt"
    ```

5. Verificare che la distribuzione della fotocamera sia completa visualizzando il feed della fotocamera in `https://<camera-ip>:3000/` , dove `<camara-ip>` è l'indirizzo IP della fotocamera. Questo passaggio può richiedere fino a 10 minuti.

## <a name="configure-azure-stream-analytics"></a>Configurare Analisi di flusso di Azure

Ora che i dati si propagano ad analisi di flusso di Azure dalla fotocamera, è necessario autorizzarli manualmente per comunicare con Power BI.

1. Dal portale di Azure aprire tutte le **risorse**e il *processo di \[ yoursuffix \] processo-calpestio* .

2. Nella sezione **Topologia processo** del riquadro del processo di Analisi di flusso selezionare l'opzione **Output**.

3. Selezionare il sink di output del **traffico** .

4. Selezionare **rinnova autorizzazione** e accedere all'account Power bi.
  
    ![Richiedi rinnovo autorizzazione in Power BI](./media/solution-deployment-guide-retail-footfall-detection/image2.png)

5. Salvare le impostazioni di output.

6. Passare al riquadro **Panoramica** e selezionare **Avvia** per iniziare a inviare i dati a Power bi.

7. Selezionare **Ora** come orario di avvio per l'output del processo e selezionare **Avvia**. È possibile visualizzare lo stato del processo nella barra di notifica.

## <a name="create-a-power-bi-dashboard"></a>Creare un dashboard Power BI

1. Una volta completato il processo, passare a [Power bi](https://powerbi.com/) e accedere con l'account aziendale o dell'Istituto di istruzione. Se la query del processo di analisi di flusso genera risultati, il set di dati *calpestio-DataSet* creato esiste nella scheda **set** di dati.

2. Dall'area di lavoro Power BI selezionare **+ Crea** per creare un nuovo dashboard denominato *calpestio Analysis.*

3. Nella parte superiore della finestra selezionare **Aggiungi riquadro**. Selezionare quindi **Dati in streaming personalizzati** e **Avanti**. Scegliere il **set di dati calpestio** nei **set**di dati. Selezionare **scheda** dall'elenco a discesa **tipo di visualizzazione** e aggiungere **Age** a **Fields**. Selezionare **Avanti** per immettere un nome per il riquadro e quindi selezionare **Applica** per creare il riquadro.

4. È possibile aggiungere ulteriori campi e schede come desiderato.

## <a name="test-your-solution"></a>testare la soluzione.

Osservare il modo in cui i dati nelle schede create in Power BI cambiano come persone diverse a fronte della fotocamera. Le inferenze possono richiedere fino a 20 secondi per essere visualizzate una volta registrate.

## <a name="remove-your-solution"></a>Rimuovere la soluzione

Se si vuole rimuovere la soluzione, eseguire i comandi seguenti usando Porter, usando gli stessi file di parametri creati per la distribuzione:

```porter
porter uninstall footfall-cloud –tag intelligentedge/footfall-cloud-deployment:0.1.0 –creds footfall-cloud-deployment –param-file "path-to-cloud-parameters-file.txt"

porter uninstall footfall-camera –tag intelligentedge/footfall-camera-deployment:0.1.0 –creds footfall-camera-deployment –param-file "path-to-camera-parameters-file.txt"
```

## <a name="next-steps"></a>Passaggi successivi

- Altre informazioni su [considerazioni sulla progettazione di app ibride]. (overview-app-design-considerations.md)
- Esaminare e proporre miglioramenti al [codice per questo esempio su GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/footfall-analysis).
