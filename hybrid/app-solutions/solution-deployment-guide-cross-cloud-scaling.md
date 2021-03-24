---
title: Distribuire un'app con scalabilità tra cloud in Azure e nell'hub di Azure Stack
description: Informazioni su come distribuire un'app con scalabilità tra cloud in Azure e nell'hub di Azure Stack.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: ed2ad5bed8f4bd80d4a40ab7600842d5544ff97d
ms.sourcegitcommit: 962334135b63ac99c715e7bc8fb9282648ba63c9
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/23/2021
ms.locfileid: "104895415"
---
# <a name="deploy-an-app-that-scales-cross-cloud-using-azure-and-azure-stack-hub"></a>Distribuire un'app con scalabilità tra cloud usando Azure e l'hub di Azure Stack

Informazioni su come creare una soluzione tra cloud per offrire un processo attivato manualmente per passare da un'app Web ospitata nell'hub di Azure Stack a un'app Web ospitata in Azure con scalabilità automatica tramite Gestione traffico. Questo processo garantisce un'utilità cloud flessibile e scalabile in fase di caricamento.

Con questo modello il tenant può non essere pronto a eseguire l'app nel cloud pubblico. Tuttavia, per l'azienda può non essere economicamente fattibile mantenere la capacità richiesta nell'ambiente locale per gestire i picchi di domanda per l'app. Il tenant può sfruttare l'elasticità del cloud pubblico con la soluzione locale in uso.

In questa soluzione si compilerà un ambiente di esempio per:

> [!div class="checklist"]
> - Creare un'app Web a più nodi.
> - Configurare e gestire il processo di distribuzione continua.
> - Pubblicare l'app Web nell'hub di Azure Stack.
> - Creare una versione.
> - Imparare a monitorare e tenere traccia delle distribuzioni.

> [!Tip]  
> ![diagramma delle colonne ibride](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> L'hub di Microsoft Azure Stack è un'estensione di Azure che offre all'ambiente locale l'agilità e l'innovazione del cloud computing, abilitando l'unico cloud ibrido che consente di creare e distribuire ovunque app ibride.  
> 
> L'articolo [Considerazioni per la progettazione di app ibride](overview-app-design-considerations.md) esamina i concetti fondamentali per la qualità del software (posizionamento, scalabilità, disponibilità, resilienza, gestibilità e sicurezza) per la progettazione, la distribuzione e la gestione di app ibride. Le considerazioni di progettazione consentono di ottimizzare la progettazione delle app ibride, riducendo al minimo i rischi negli ambienti di produzione.

## <a name="prerequisites"></a>Prerequisiti

- Sottoscrizione di Azure. Prima di iniziare, se necessario, creare un [account gratuito](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Un sistema integrato dell'hub di Azure Stack o una distribuzione di Azure Stack Development Kit (ASDK).
  - Per informazioni sull'installazione dell'hub di Azure Stack, vedere le [istruzioni di installazione di ASDK](/azure-stack/asdk/asdk-install).
  - Per uno script di automazione post-distribuzione di ASDK, vedere [https://github.com/mattmcspirit/azurestack](https://github.com/mattmcspirit/azurestack)
  - Per completare questa installazione possono essere necessarie alcune ore.
- Distribuire i servizi PaaS del [servizio app](/azure-stack/operator/azure-stack-app-service-deploy) nell'hub di Azure Stack.
- [Creare piani o offerte](/azure-stack/operator/service-plan-offer-subscription-overview) nell'ambiente dell'hub di Azure Stack.
- [Creare una sottoscrizione tenant](/azure-stack/operator/azure-stack-subscribe-plan-provision-vm) nell'ambiente dell'hub di Azure Stack.
- Creare un'app Web all'interno della sottoscrizione tenant. Prendere nota dell'URL della nuova app Web per usarlo in seguito.
- Distribuire la macchina virtuale (VM) di Azure Pipelines nella sottoscrizione tenant.
- È necessaria una macchina virtuale Windows Server 2016 con .NET 3.5. Questa macchina virtuale verrà compilata nella sottoscrizione tenant per l'hub di Azure Stack come agente di compilazione privato.
- [Windows Server 2016 con immagine VM di SQL 2017](/azure-stack/operator/azure-stack-add-vm-image) è disponibile nel Marketplace dell'hub di Azure Stack. Se questa immagine non è disponibile, usare un operatore dell'hub di Azure Stack per assicurarsi che venga aggiunta all'ambiente.

## <a name="issues-and-considerations"></a>Considerazioni e problemi

### <a name="scalability"></a>Scalabilità

Il componente chiave della scalabilità tra cloud è la possibilità di rendere immediatamente disponibile la scalabilità su richiesta tra l'infrastruttura cloud pubblica e quella locale, offrendo un servizio coerente e affidabile.

### <a name="availability"></a>Disponibilità

Assicurarsi che le app distribuite localmente siano configurate per la disponibilità elevata verificando la configurazione hardware locale e la distribuzione del software.

### <a name="manageability"></a>Gestione

La soluzione tra cloud garantisce una gestione senza problemi e un'interfaccia familiare tra gli ambienti. Si consiglia di usare PowerShell per la gestione multipiattaforma.

## <a name="cross-cloud-scaling"></a>Scalabilità tra cloud

### <a name="get-a-custom-domain-and-configure-dns"></a>Ottenere un dominio personalizzato e configurare il DNS

Aggiornare il file della zona DNS per il dominio. Azure AD verifica quindi la proprietà del nome di dominio personalizzato. Usare il [DNS di Azure](/azure/dns/dns-getstarted-portal) per i record DNS di Azure/Microsoft 365/esterni in Azure oppure aggiungere la voce DNS in un [registrar DNS diverso](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider).

1. Registrare un dominio personalizzato con un registrar pubblico.
2. Accedere al registrar per il dominio. Per eseguire gli aggiornamenti al DNS potrebbe essere necessario un amministratore approvato.
3. Aggiornare il file di zona DNS per il dominio aggiungendo la voce DNS specificata da Azure AD. La voce DNS non influirà sul routing della posta elettronica né sui comportamenti di hosting Web.

### <a name="create-a-default-multi-node-web-app-in-azure-stack-hub"></a>Creare un'app Web multinodo predefinita nell'hub di Azure Stack

Configurare l'integrazione continua e la distribuzione continua (CI/CD) ibride per distribuire le app Web in Azure e nell'hub di Azure Stack e per eseguire il push automatico delle modifiche in entrambi i cloud.

> [!Note]  
> Sono necessari l'hub di Azure Stack con immagini diffuse appropriate per l'esecuzione (Windows Server e SQL) e la distribuzione del servizio app. Per altre informazioni, vedere i [prerequisiti per la distribuzione del servizio app nell'hub di Azure Stack](/azure-stack/operator/azure-stack-app-service-before-you-get-started) nella documentazione del servizio app.

### <a name="add-code-to-azure-repos"></a>Aggiungere codice ad Azure Repos

Azure Repos

1. Accedere ad Azure Repos con un account provvisto di diritti di creazione progetti per Azure Repos.

    L'integrazione continua e la distribuzione continua (CI/CD) ibride possono essere applicate sia al codice dell'app sia al codice dell'infrastruttura. Usare [modelli di Azure Resource Manager](https://azure.microsoft.com/resources/templates/) sia per lo sviluppo cloud privato che per quello ospitato.

    ![Eseguire la connessione a un progetto in Azure Repos](media/solution-deployment-guide-cross-cloud-scaling/image1.JPG)

2. **Clonare il repository** creando e aprendo l'app Web predefinita.

    ![Clonare il repository nell'app Web di Azure](media/solution-deployment-guide-cross-cloud-scaling/image2.png)

### <a name="create-self-contained-web-app-deployment-for-app-services-in-both-clouds"></a>Creare una distribuzione di app Web autonoma per i servizi app in entrambi i cloud

1. Modificare il file **WebApplication.csproj**. Selezionare `Runtimeidentifier` e aggiungere `win10-x64`. Vedere la documentazione relativa alla [distribuzione autonoma](/dotnet/core/deploying/deploy-with-vs#simpleSelf).

    ![Modificare un file di progetto di app Web](media/solution-deployment-guide-cross-cloud-scaling/image3.png)

2. Archiviare il codice in Azure Repos usando Team Explorer.

3. Verificare che il codice dell'applicazione sia stato archiviato in Azure Repos.

## <a name="create-the-build-definition"></a>Creare la definizione di compilazione

1. Accedere ad Azure Pipelines per assicurarsi di poter creare definizioni di compilazione.

2. Aggiungere il codice **-r win10-x64**. Questa aggiunta è necessaria per attivare una distribuzione autonoma con .NET Core.

    ![Aggiungere codice all'app Web](media/solution-deployment-guide-cross-cloud-scaling/image4.png)

3. Eseguire la compilazione. Il processo di [compilazione della distribuzione autonoma](/dotnet/core/deploying/deploy-with-vs#simpleSelf) pubblicherà gli artefatti che vengono eseguiti in Azure e nell'hub di Azure Stack.

## <a name="use-an-azure-hosted-agent"></a>Usare un agente ospitato di Azure

L'uso di un agente di compilazione ospitato in Azure Pipelines è un'opzione utile per compilare e distribuire le app Web. La manutenzione e gli aggiornamenti vengono eseguiti automaticamente da Microsoft Azure, quindi il ciclo di sviluppo è continuo e ininterrotto.

### <a name="manage-and-configure-the-cd-process"></a>Gestire e configurare il processo di distribuzione continua

Azure Pipelines e Azure DevOps Services offrono una pipeline altamente configurabile e facilmente gestibile per i rilasci in più ambienti, ad esempio gli ambienti di sviluppo, gestione temporanea, controllo qualità e produzione, e include la richiesta di approvazioni in fasi specifiche.

## <a name="create-release-definition"></a>Creare una definizione della versione

1. Selezionare il pulsante con il **segno più** per aggiungere una nuova versione nella scheda **Releases** (Rilasci) della sezione **Build and Release** (Compilazione e versione) di Azure DevOps Services.

    ![Creare una definizione di versione](media/solution-deployment-guide-cross-cloud-scaling/image5.png)

2. Applicare il modello di distribuzione del servizio app di Azure.

   ![Applicare il modello di distribuzione del servizio app di Azure](meDia/solution-deployment-guide-cross-cloud-scaling/image6.png)

3. In **Aggiungi artefatto** aggiungere l'artefatto corrispondente all'app della build del cloud di Azure.

   ![Aggiungere l'artefatto alla build del cloud di Azure](media/solution-deployment-guide-cross-cloud-scaling/image7.png)

4. Nella scheda Pipeline selezionare il collegamento **Phase, Task** (Fase, Attività) dell'ambiente e impostare i valori di ambiente del cloud di Azure.

   ![Impostare i valori di ambiente del cloud di Azure](media/solution-deployment-guide-cross-cloud-scaling/image8.png)

5. Impostare il **nome dell'ambiente** e selezionare la **sottoscrizione di Azure** per l'endpoint cloud di Azure.

      ![Selezionare la sottoscrizione di Azure per l'endpoint cloud di Azure](media/solution-deployment-guide-cross-cloud-scaling/image9.png)

6. Come **nome del servizio app** specificare il nome del servizio app di Azure richiesto.

      ![Impostare il nome del servizio app di Azure](media/solution-deployment-guide-cross-cloud-scaling/image10.png)

7. Immettere "Hosted VS2017" in **Agent queue** (Coda agente) per l'ambiente ospitato del cloud di Azure.

      ![Impostare la coda agente per l'ambiente ospitato del cloud di Azure](media/solution-deployment-guide-cross-cloud-scaling/image11.png)

8. Nel menu Deploy Azure App Service (Distribuisci servizio app di Azure) selezionare l'opzione **Package or Folder** (Pacchetto o cartella) per l'ambiente. Selezionare **OK** per **folder location** (Percorso cartella).
  
      ![Selezionare il pacchetto o la cartella per l'ambiente del servizio app di Azure](media/solution-deployment-guide-cross-cloud-scaling/image12.png)

      ![Finestra di dialogo di selezione cartelle 1](media/solution-deployment-guide-cross-cloud-scaling/image13.png)

9. Salvare tutte le modifiche e tornare alla **pipeline di versione**.

    ![Salvare le modifiche nella pipeline di versione](media/solution-deployment-guide-cross-cloud-scaling/image14.png)

10. Aggiungere un nuovo artefatto selezionando la build per l'app hub di Azure Stack.

    ![Aggiungere un nuovo artefatto per l'app hub di Azure Stack](media/solution-deployment-guide-cross-cloud-scaling/image15.png)

11. Aggiungere un altro ambiente applicando la distribuzione del servizio app di Azure.

    ![Aggiungere l'ambiente alla distribuzione del servizio app di Azure](media/solution-deployment-guide-cross-cloud-scaling/image16.png)

12. Assegnare al nuovo ambiente il nome "Azure Stack".

    ![Assegnare un nome all'ambiente nella distribuzione del servizio app di Azure](media/solution-deployment-guide-cross-cloud-scaling/image17.png)

13. Individuare l'ambiente Azure Stack nella scheda **Tasks** (Attività).

    ![Ambiente Azure Stack](media/solution-deployment-guide-cross-cloud-scaling/image18.png)

14. Selezionare la sottoscrizione per l'endpoint di Azure Stack.

    ![Selezionare la sottoscrizione per l'endpoint di Azure Stack](media/solution-deployment-guide-cross-cloud-scaling/image19.png)

15. Impostare il nome dell'app Web Azure Stack come nome del servizio app.
    ![Impostare il nome dell'app Web Azure Stack](media/solution-deployment-guide-cross-cloud-scaling/image20.png)

16. Selezionare l'agente di Azure Stack.

    ![Selezionare l'agente di Azure Stack](media/solution-deployment-guide-cross-cloud-scaling/image21.png)

17. Nella sezione Deploy Azure App Service (Distribuisci servizio app di Azure) selezionare **pacchetto o la cartella** validi per l'ambiente. Selezionare **OK** per il percorso della cartella.

    ![Selezionare la cartella per la distribuzione del servizio app di Azure](media/solution-deployment-guide-cross-cloud-scaling/image22.png)

    ![Finestra di dialogo di selezione cartelle 2](media/solution-deployment-guide-cross-cloud-scaling/image23.png)

18. Nella scheda Variable (Variabile) aggiungere una variabile con nome `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`, impostare il valore come **true** e Azure Stack come ambito.

    ![Aggiungere una variabile alla distribuzione del servizio app di Azure](media/solution-deployment-guide-cross-cloud-scaling/image24.png)

19. Selezionare l'icona del trigger di distribuzione **Continuous** (Continua) in entrambi gli artefatti e abilitare il trigger di distribuzione **Continues** (Continua).

    ![Selezionare il trigger di distribuzione continua](media/solution-deployment-guide-cross-cloud-scaling/image25.png)

20. Selezionare l'icona delle condizioni **Pre-deployment** (Pre-distribuzione) nell'ambiente Azure Stack e impostare il trigger su **Dopo la versione**.

    ![Selezionare le condizioni di pre-distribuzione](media/solution-deployment-guide-cross-cloud-scaling/image26.png)

21. Salvare tutte le modifiche.

> [!Note]  
> Alcune impostazioni per le attività potrebbero essere state definite automaticamente come [variabili di ambiente](/azure/devops/pipelines/release/variables?tabs=batch#custom-variables) durante la creazione di una definizione di versione da un modello. Queste impostazioni non possono essere modificate nelle impostazioni dell'attività. Per modificarle è invece necessario selezionare l'elemento dell'ambiente padre.

## <a name="publish-to-azure-stack-hub-via-visual-studio"></a>Pubblicare nell'hub di Azure Stack con Visual Studio

Creando gli endpoint, una build di Azure DevOps Services può distribuire le app del servizio di Azure nell'hub di Azure Stack. Azure Pipelines si connette all'agente di compilazione, che si connette all'hub di Azure Stack.

1. Accedere ad Azure DevOps Services e passare alla pagina delle impostazioni dell'app.

2. Su **Impostazioni**, selezionare **Sicurezza**.

3. In **Gruppi VSTS** selezionare **Creatori di endpoint**.

4. Nella scheda **Membri** fare clic su **Aggiungi**.

5. In **Aggiungi utenti e gruppi** immettere un nome utente e selezionare l'utente dall'elenco di utenti.

6. Selezionare **Save changes** (Salva modifiche).

7. Nell'elenco **Gruppi VSTS** selezionare **Amministratori endpoint**.

8. Nella scheda **Membri** fare clic su **Aggiungi**.

9. In **Aggiungi utenti e gruppi** immettere un nome utente e selezionare l'utente dall'elenco di utenti.

10. Selezionare **Save changes** (Salva modifiche).

Poiché ora sono disponibili le informazioni sugli endpoint, la connessione da Azure Pipelines all'hub di Azure Stack è pronta per l'uso. L'agente di compilazione nell'hub di Azure Stack riceve le istruzioni da Azure Pipelines e quindi specifica le informazioni sugli endpoint per la comunicazione con l'hub di Azure Stack.

## <a name="develop-the-app-build"></a>Sviluppare la build dell'app

> [!Note]  
> Sono necessari l'hub di Azure Stack con immagini diffuse appropriate per l'esecuzione (Windows Server e SQL) e la distribuzione del servizio app. Per altre informazioni, vedere [Prerequisiti per la distribuzione del servizio app nell'hub di Azure Stack](/azure-stack/operator/azure-stack-app-service-before-you-get-started).

Usare i [modelli di Azure Resource Manager](https://azure.microsoft.com/resources/templates/) come codice dell'app Web da Azure Repos per la distribuzione in entrambi i cloud.

### <a name="add-code-to-an-azure-repos-project"></a>Aggiungere codice a un progetto di Azure Repos

1. Accedere ad Azure Repos con un account con diritti di creazione progetti per l'hub di Azure Stack.

2. **Clonare il repository** creando e aprendo l'app Web predefinita.

#### <a name="create-self-contained-web-app-deployment-for-app-services-in-both-clouds"></a>Creare una distribuzione di app Web autonoma per i servizi app in entrambi i cloud

1. Modificare il file **WebApplication.csproj**: Selezionare `Runtimeidentifier` e quindi aggiungere `win10-x64`. Per altre informazioni, vedere la documentazione relativa alla [distribuzione autonoma](/dotnet/core/deploying/deploy-with-vs#simpleSelf).

2. Usare Team Explorer per archiviare il codice in Azure Repos.

3. Verificare che il codice dell'app sia stato archiviato in Azure Repos.

### <a name="create-the-build-definition"></a>Creare la definizione di compilazione

1. Accedere ad Azure Pipelines con un account che può creare una definizione di compilazione.

2. Passare alla pagina di **compilazione dell'applicazione Web** per il progetto.

3. In **Argomenti** aggiungere il codice **-r win10-x64**. Questa aggiunta è necessaria per attivare una distribuzione autonoma con .NET Core.

4. Eseguire la compilazione. Il processo di [compilazione della distribuzione autonoma](/dotnet/core/deploying/deploy-with-vs#simpleSelf) pubblicherà gli artefatti che possono essere eseguiti in Azure e nell'hub di Azure Stack.

#### <a name="use-an-azure-hosted-build-agent"></a>Usare un agente di compilazione ospitato di Azure

L'uso di un agente di compilazione ospitato in Azure Pipelines è un'opzione utile per compilare e distribuire le app Web. La manutenzione e gli aggiornamenti vengono eseguiti automaticamente da Microsoft Azure, quindi il ciclo di sviluppo è continuo e ininterrotto.

### <a name="configure-the-continuous-deployment-cd-process"></a>Configurare il processo di distribuzione continua

Azure Pipelines e Azure DevOps Services offrono una pipeline altamente configurabile e facilmente gestibile per i rilasci in più ambienti, ad esempio gli ambienti di sviluppo, gestione temporanea, controllo qualità e produzione. Questo processo può includere la richiesta di approvazioni in fasi specifiche del ciclo di vita dell'app.

#### <a name="create-release-definition"></a>Creare una definizione della versione

La creazione di una definizione della versione è il passaggio finale del processo di compilazione dell'app. La definizione della versione viene usata per creare una versione e distribuire una build.

1. Accedere ad Azure Pipelines e passare a **Compilazione e versione** per il progetto.

2. Nella scheda **Versioni** selezionare **[+]** e quindi scegliere **Crea definizione di versione**.

3. In **Seleziona un modello** scegliere **Distribuzione Servizio app di Azure** e quindi selezionare **Applica**.

4. In **Aggiungi artefatto** selezionare l'app della build del cloud di Azure da **Origine (definizione di compilazione)** .

5. Nella scheda **Pipeline** selezionare il collegamento **fase 1**, **attività 1** a **Visualizza le attività dell'ambiente**.

6. Nella scheda **Attività** immettere Azure come **nome dell'ambiente** e selezionare AzureCloud Traders-Web EP dall'elenco delle **sottoscrizioni di Azure**.

7. Immettere il **nome del servizio app di Azure**, che è `northwindtraders` nella successiva acquisizione di schermata.

8. Per la fase Agente selezionare **Hosted VS2017** dall'elenco **Coda agente**.

9. In **Deploy Azure App Service** (Distribuisci servizio app di Azure) selezionare l'opzione **Package or folder** (Pacchetto o cartella) per l'ambiente.

10. In **Seleziona file o cartella** selezionare **OK** per **Percorso**.

11. Salvare tutte le modifiche e tornare a **Pipeline**.

12. Nella scheda **Pipeline** selezionare **Aggiungi artefatto** e scegliere **NorthwindCloud Traders-Vessel** dall'elenco **Origine (definizione di compilazione)** .

13. Aggiungere un altro ambiente in **Seleziona un modello**. Scegliere **Distribuzione Servizio app di Azure** e quindi selezionare **Applica**.

14. Immettere `Azure Stack Hub` come **nome dell'ambiente**.

15. Nella scheda **Attività** trovare e selezionare Azure Stack Hub.

16. Nell'elenco delle **sottoscrizioni di Azure** selezionare **AzureStack Traders-Vessel EP** per l'endpoint dell'hub di Azure Stack.

17. Immettere il nome dell'app Web Azure Stack Hub come **nome del servizio app**.

18. In **Selezione agente** scegliere **AzureStack -b Douglas Fir** dall'elenco **Coda agente**.

19. Per **Deploy Azure App Service** (Distribuisci servizio app di Azure) selezionare il **pacchetto o la cartella** validi per l'ambiente. In **Seleziona file o cartella** selezionare **OK** per la cartella **Percorso**.

20. Nella scheda **Variabile** individuare la variabile denominata `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`. Impostare il valore della variabile su **true** e impostare l'ambito su **Azure Stack Hub**.

21. Nella scheda **Pipeline** selezionare l'icona del **trigger di distribuzione continua** per l'artefatto NorthwindCloud Traders-Web e impostare il **trigger di distribuzione continua** su **Abilitato**. Eseguire la stessa operazione per l'artefatto **NorthwindCloud Traders-Vessel**.

22. Per l'ambiente Azure Stack Hub, selezionare l'icona delle **condizioni di pre-distribuzione** e impostare il trigger su **Dopo la versione**.

23. Salvare tutte le modifiche.

> [!Note]  
> Alcune impostazioni per le attività di rilascio vengono definite automaticamente come [variabili di ambiente](/azure/devops/pipelines/release/variables?tabs=batch#custom-variables) durante la creazione di una definizione di versione da un modello. Queste impostazioni non possono essere modificate nelle impostazioni delle attività, ma si possono modificare negli elementi dell'ambiente padre.

## <a name="create-a-release"></a>Creare una versione

1. Nella scheda **Pipeline** aprire l'elenco **Versione** e selezionare **Crea versione**.

2. Immettere una descrizione per la versione, verificare che siano selezionati gli artefatti corretti, quindi selezionare **Crea**. Dopo alcuni istanti viene visualizzato un banner che indica che la nuova versione è stata creata e il nome della versione viene visualizzato come collegamento. Selezionare il collegamento per visualizzare la pagina di riepilogo della versione,

3. che contiene i dettagli relativi alla versione. Nella seguente acquisizione di schermata per "Release-2", nella sezione **Ambienti** lo **stato della distribuzione** per Azure è "IN PROGRESS" (in corso) e lo stato per l'hub di Azure Stack è "SUCCEEDED" (riuscita). Quando lo stato della distribuzione per l'ambiente di Azure diventa "SUCCEEDED", viene visualizzato un banner che indica che la versione è pronta per l'approvazione. Quando una distribuzione è in sospeso o non riesce, viene visualizzata un'icona blu con la **(i)** di Informazioni. Passare il mouse sull'icona per visualizzare un elemento popup che contiene il motivo del ritardo o dell'errore.

4. Altre visualizzazioni, ad esempio l'elenco delle versioni, contengono anche un'icona che indica che l'approvazione è in sospeso. L'elemento popup per questa icona specifica il nome dell'ambiente e altri dettagli relativi alla distribuzione. È facile per un amministratore vedere lo stato di avanzamento complessivo delle versioni e quali versioni sono in attesa di approvazione.

## <a name="monitor-and-track-deployments"></a>Monitorare le distribuzioni

1. Nella pagina di riepilogo di **Release-2** selezionare **Log**. Durante una distribuzione, in questa pagina è visualizzato il log attivo dell'agente. Nel riquadro sinistro è riportato lo stato di ogni operazione della distribuzione per ogni ambiente.

2. Selezionare l'icona della persona nella colonna **Azione** per un'approvazione pre-distribuzione o post-distribuzione per verificare chi ha approvato (o rifiutato) la distribuzione e visualizzare il relativo messaggio.

3. Al termine della distribuzione, nel riquadro destro viene visualizzato l'intero file di log. Selezionare un **passaggio** nel riquadro sinistro per visualizzare il file di log per quel passaggio specifico, ad esempio l'**inizializzazione del processo**. La possibilità di visualizzare i singoli log semplifica il monitoraggio e il debug delle parti della distribuzione complessiva. **Salvare** il file di log per un passaggio o **scaricare tutti i log in un file zip**.

4. Aprire la scheda **Riepilogo** per visualizzare le informazioni generali sulla versione. Questa vista contiene i dettagli relativi alla build, gli ambienti in cui è stata distribuita, lo stato della distribuzione e altre informazioni sulla versione.

5. Selezionare un collegamento all'ambiente (**Azure** o **hub di Azure Stack**) per visualizzare informazioni sulle distribuzioni esistenti e in sospeso in un ambiente specifico. Usare queste viste come metodo rapido per verificare che la stessa build sia stata distribuita in entrambi gli ambienti.

6. Aprire l'**app di produzione distribuita** in un browser. Ad esempio, per il sito Web Servizi app di Azure aprire l'URL `https://[your-app-name\].azurewebsites.net`.

### <a name="integration-of-azure-and-azure-stack-hub-provides-a-scalable-cross-cloud-solution"></a>L'integrazione di Azure e dell'hub di Azure Stack offre una soluzione scalabile tra cloud

Un servizio multi-cloud flessibile e affidabile offre sicurezza dei dati, backup e ridondanza, disponibilità costante e rapida, archiviazione e distribuzione scalabili e routing conforme all'area geografica. Questo processo attivato manualmente garantisce un passaggio affidabile ed efficiente del carico tra le app Web ospitate e la disponibilità immediata dei dati cruciali.

## <a name="next-steps"></a>Passaggi successivi

- Per altre informazioni sui modelli cloud di Azure, vedere [Modelli di progettazione cloud](/azure/architecture/patterns).
