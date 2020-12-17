---
title: Distribuire la soluzione di rilevamento del footfall basata sull'intelligenza artificiale in Azure e nell'hub di Azure Stack
description: Informazioni su come distribuire una soluzione di rilevamento del footfall basata sull'intelligenza artificiale per analizzare il traffico dei visitatori nei punti vendita al dettaglio usando Azure e l'hub di Azure Stack.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 2177b32474dea695967e197acbd4bc1e18422d7b
ms.sourcegitcommit: df7e3e6423c3d4e8a42dae3d1acfba1d55057258
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 12/09/2020
ms.locfileid: "96901491"
---
# <a name="deploy-an-ai-based-footfall-detection-solution-using-azure-and-azure-stack-hub"></a>Distribuire una soluzione di rilevamento del footfall basata sull'intelligenza artificiale usando Azure e l'hub di Azure Stack

Questo articolo descrive come distribuire una soluzione basata sull'intelligenza artificiale che genera informazioni dettagliate da azioni reali usando Azure, l'hub di Azure Stack e il Custom Vision AI Dev Kit.

In questa soluzione si apprenderà come:

> [!div class="checklist"]
> - Distribuire Cloud Native Application Bundle (CNAB) perimetrali. 
> - Distribuire un'app che si estende oltre i limiti del cloud.
> - Usare il Custom Vision AI Dev Kit per l'inferenza perimetrale.

> [!Tip]  
> ![Diagramma dei concetti sulle app ibride](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> L'hub di Microsoft Azure Stack è un'estensione di Azure, che offre all'ambiente locale l'agilità e l'innovazione del cloud computing, abilitando l'unico cloud ibrido che consente di creare e distribuire ovunque app ibride.  
> 
> L'articolo [Considerazioni per la progettazione di app ibride](overview-app-design-considerations.md) esamina i concetti fondamentali di qualità del software (posizionamento, scalabilità, disponibilità, resilienza, gestibilità e sicurezza) per la progettazione, la distribuzione e la gestione di app ibride. Le considerazioni di progettazione consentono di ottimizzare la progettazione delle app ibride, riducendo al minimo i rischi negli ambienti di produzione.

## <a name="prerequisites"></a>Prerequisiti

Prima di iniziare a usare questa guida alla distribuzione:

- Rivedere l'argomento [Modello di rilevamento del footfall](pattern-retail-footfall-detection.md).
- Ottenere un accesso utente ad Azure Stack Development Kit (ASDK) o un'istanza di sistema integrato dell'hub di Azure Stack, con i requisiti seguenti:
  - Installazione del [servizio app di Azure in un provider di risorse dell'hub di Azure Stack](/azure-stack/operator/azure-stack-app-service-overview.md). È necessario disporre dell'accesso operatore all'istanza dell'hub di Azure Stack o dell'accesso amministratore per poter eseguire l'installazione.
  - Sottoscrizione a un'offerta che offre il servizio app e la quota di archiviazione. Per creare un'offerta, è necessario disporre dell'accesso operatore.
- Ottenere l'accesso a una sottoscrizione di Azure.
  - Se non si dispone di una sottoscrizione di Azure, iscriversi per creare un [account di prova gratuito](https://azure.microsoft.com/free/) prima di iniziare.
- Creare due entità servizio nella directory:
  - Una configurata per usare le risorse di Azure, con accesso all'ambito della sottoscrizione di Azure.
  - Una configurata per usare le risorse dell'hub di Azure Stack, con accesso all'ambito della sottoscrizione dell'hub di Azure Stack.
  - Per altre informazioni sulla creazione di entità servizio e sull'autorizzazione dell'accesso, vedere [Usare un'identità dell'app per accedere alle risorse](/azure-stack/operator/azure-stack-create-service-principals.md). Per usare l'interfaccia della riga di comando di Azure, vedere [Creare un'entità servizio di Azure con l'interfaccia della riga di comando di Azure](/cli/azure/create-an-azure-service-principal-azure-cli?view=azure-cli-latest&preserve-view=true).
- Distribuire Servizi cognitivi di Azure in Azure o nell'hub di Azure Stack.
  - Leggere prima [altre informazioni su Servizi cognitivi](https://azure.microsoft.com/services/cognitive-services/).
  - Visitare quindi [Distribuire Servizi cognitivi di Azure nell'hub di Azure Stack](/azure-stack/user/azure-stack-solution-template-cognitive-services.md) per distribuire Servizi cognitivi nell'hub di Azure Stack. È prima necessario iscriversi per accedere all'anteprima.
- Clonare o scaricare una versione del Custom Vision AI Dev Kit non configurata di Azure. Per informazioni dettagliate, vedere [Vision AI DevKit](https://azure.github.io/Vision-AI-DevKit-Pages/).
- Creare un account Power BI.
- Chiave di sottoscrizione per l'API Viso di Servizi cognitivi di Azure e URL dell'endpoint. È possibile ottenere sia la chiave sia l'URL nella versione di [valutazione di Servizi cognitivi](https://azure.microsoft.com/try/cognitive-services/?api=face-api). In alternativa, seguire le istruzioni riportate in [Creare un account di Servizi cognitivi](/azure/cognitive-services/cognitive-services-apis-create-account).
- Installare le risorse di sviluppo seguenti:
  - [Interfaccia della riga di comando di Azure 2.0](/azure-stack/user/azure-stack-version-profiles-azurecli2.md)
  - [Docker CE](https://hub.docker.com/search/?type=edition&offering=community)
  - [Porter](https://porter.sh/). Porter consente di distribuire app cloud usando i manifesti di bundle CNAB disponibili.
  - [Visual Studio Code](https://code.visualstudio.com/)
  - [Azure IoT Tools per Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.azure-iot-tools)
  - [Estensione Python per Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=ms-python.python)
  - [Python](https://www.python.org/)

## <a name="deploy-the-hybrid-cloud-app"></a>Distribuire l'app cloud ibrida

Usare l'interfaccia della riga di comando di Porter per generare un set di credenziali, quindi distribuire l'app cloud.  

1. Clonare o scaricare il codice di esempio della soluzione da https://github.com/azure-samples/azure-intelligent-edge-patterns. 

1. Porter genererà un set di credenziali che consentono di automatizzare la distribuzione dell'app. Prima di eseguire il comando di generazione delle credenziali, assicurarsi che siano disponibili gli elementi seguenti:

    - Entità servizio per l'accesso alle risorse di Azure, inclusi l'ID dell'entità servizio, la chiave e il DNS tenant.
    - ID sottoscrizione della sottoscrizione di Azure.
    - Entità servizio per l'accesso alle risorse dell'hub di Azure Stack, inclusi l'ID dell'entità servizio, la chiave e il DNS tenant.
    - ID sottoscrizione della sottoscrizione dell'hub di Azure Stack.
    - Chiave per l'API Viso di Servizi cognitivi di Azure e URL dell'endpoint della risorsa.

1. Eseguire il processo di generazione delle credenziali di Porter e seguire le istruzioni:

   ```porter
   porter creds generate --tag intelligentedge/footfall-cloud-deployment:0.1.0
   ```

1. Per l'esecuzione, Porter richiede anche un set di parametri. Creare un file di testo per i parametri e immettere le coppie nome-valore seguenti. Rivolgersi all'amministratore dell'hub di Azure Stack per richiedere assistenza su qualsiasi valore necessario.

   > [!NOTE] 
   > Il valore `resource suffix` viene usato per assicurare che le risorse della distribuzione abbiano nomi univoci in Azure. Deve essere una stringa univoca di lettere e numeri e non deve superare gli 8 caratteri di lunghezza.

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

1. A questo punto è possibile distribuire l'app cloud ibrida usando Porter. Eseguire il comando di installazione e osservare come le risorse vengono distribuite in Azure e nell'hub di Azure Stack:

    ```porter
    porter install footfall-cloud –tag intelligentedge/footfall-cloud-deployment:0.1.0 –creds footfall-cloud-deployment –param-file "path-to-cloud-parameters-file.txt"
    ```

1. Al termine della distribuzione, prendere nota dei valori seguenti:
    - Stringa di connessione della fotocamera.
    - Stringa di connessione dell'account di archiviazione dell'immagine.
    - Nomi dei gruppi di risorse.

## <a name="prepare-the-custom-vision-ai-devkit"></a>Preparare il Custom Vision AI DevKit

A questo punto configurare il Custom Vision AI DevKit come illustrato nell'[avvio rapido di Vision AI DevKit](https://azure.github.io/Vision-AI-DevKit-Pages/docs/quick_start/). È anche possibile configurare e testare la fotocamera, usando la stringa di connessione ottenuta nel passaggio precedente.

## <a name="deploy-the-camera-app"></a>Distribuire l'app della fotocamera

Usare l'interfaccia della riga di comando di Porter per generare un set di credenziali, quindi distribuire l'app della fotocamera.

1. Porter genererà un set di credenziali che consentono di automatizzare la distribuzione dell'app. Prima di eseguire il comando di generazione delle credenziali, assicurarsi che siano disponibili gli elementi seguenti:

    - Entità servizio per l'accesso alle risorse di Azure, inclusi l'ID dell'entità servizio, la chiave e il DNS tenant.
    - ID sottoscrizione della sottoscrizione di Azure.
    - Stringa di connessione dell'account di archiviazione dell'immagine ottenuta durante la distribuzione dell'app cloud.

1. Eseguire il processo di generazione delle credenziali di Porter e seguire le istruzioni:

    ```porter
    porter creds generate --tag intelligentedge/footfall-camera-deployment:0.1.0
    ```

1. Per l'esecuzione, Porter richiede anche un set di parametri. Creare un file di testo per i parametri e immettere il testo seguente. Rivolgersi all'amministratore dell'hub di Azure Stack se non sono noti alcuni dei valori richiesti.

    > [!NOTE]
    > Il valore `deployment suffix` viene usato per assicurare che le risorse della distribuzione abbiano nomi univoci in Azure. Deve essere una stringa univoca di lettere e numeri e non deve superare gli 8 caratteri di lunghezza.

    ```porter
    iot_hub_name="Name of the IoT Hub deployed"
    deployment_suffix="Unique string here"
    ```

    Salvare il file di testo e prendere nota del relativo percorso.

4. A questo punto è possibile distribuire l'app della fotocamera usando Porter. Eseguire il comando di installazione e osservare come viene distribuito il servizio IoT Edge.

    ```porter
    porter install footfall-camera –tag intelligentedge/footfall-camera-deployment:0.1.0 –creds footfall-camera-deployment –param-file "path-to-camera-parameters-file.txt"
    ```

5. Verificare che la distribuzione della fotocamera sia completata visualizzando il feed della fotocamera in `https://<camera-ip>:3000/`, dove `<camara-ip>` corrisponde all'indirizzo IP della fotocamera. Questo passaggio può richiedere fino a 10 minuti.

## <a name="configure-azure-stream-analytics"></a>Configurare Analisi di flusso di Azure

Ora che i dati vengono trasmessi ad Analisi di flusso di Azure dalla fotocamera, è necessario creare manualmente un'autorizzazione perché possano comunicare con Power BI.

1. Nel portale di Azure aprire **Tutte le risorse**, quindi selezionare il processo *process-footfall\[suffissoutente\]* .

2. Nella sezione **Topologia processo** del riquadro del processo di Analisi di flusso selezionare l'opzione **Output**.

3. Selezionare il sink di output **traffic-output**.

4. Selezionare **Rinnova autorizzazione** e accedere all'account di Power BI.
  
    ![Richiesta di rinnovo dell'autorizzazione in Power BI](./media/solution-deployment-guide-retail-footfall-detection/image2.png)

5. Salvare le impostazioni di output.

6. Passare al riquadro **Panoramica** e selezionare **Avvia** per iniziare a inviare i dati a Power BI.

7. Selezionare **Ora** come orario di avvio per l'output del processo e selezionare **Avvia**. È possibile visualizzare lo stato del processo nella barra di notifica.

## <a name="create-a-power-bi-dashboard"></a>Creare un dashboard di Power BI

1. Al termine del processo, passare a [Power BI](https://powerbi.com/) e accedere con l'account aziendale o dell'istituto di istruzione. Se la query del processo di Analisi di flusso restituisce risultati, il set di dati *footfall-dataset* creato è presente nella scheda **Set di dati**.

2. Nell'area di lavoro di Power BI selezionare **+ Crea** per creare un nuovo dashboard denominato *Footfall Analysis.*

3. Nella parte superiore della finestra selezionare **Aggiungi riquadro**. Selezionare quindi **Dati in streaming personalizzati** e **Avanti**. Scegliere **footfall-dataset** in **Set di dati personali**. Selezionare **Scheda** nell'elenco a discesa **Tipo di visualizzazione** e aggiungere **age** a **Campi**. Selezionare **Avanti** per immettere un nome per il riquadro e quindi selezionare **Applica** per creare il riquadro.

4. È possibile aggiungere altri campi e schede in base alle esigenze.

## <a name="test-your-solution"></a>Testare la soluzione

Osservare come i dati nelle schede create in Power BI cambiano a seconda delle diverse persone che passano di fronte alla fotocamera. È possibile che le inferenze siano visualizzate fino a 20 secondi dopo essere state registrate.

## <a name="remove-your-solution"></a>Rimuovere la soluzione

Se si vuole rimuovere la soluzione, eseguire i comandi seguenti tramite Porter, usando gli stessi file di parametri creati per la distribuzione:

```porter
porter uninstall footfall-cloud –tag intelligentedge/footfall-cloud-deployment:0.1.0 –creds footfall-cloud-deployment –param-file "path-to-cloud-parameters-file.txt"

porter uninstall footfall-camera –tag intelligentedge/footfall-camera-deployment:0.1.0 –creds footfall-camera-deployment –param-file "path-to-camera-parameters-file.txt"
```

## <a name="next-steps"></a>Passaggi successivi

- Altre informazioni sulle [considerazioni per la progettazione di app ibride](overview-app-design-considerations.md)
- Esaminare e proporre miglioramenti del [codice di questo esempio in GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/footfall-analysis).
