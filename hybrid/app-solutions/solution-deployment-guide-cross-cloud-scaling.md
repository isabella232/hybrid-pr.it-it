---
title: Distribuire un'app con scalabilità tra cloud in Azure e hub Azure Stack
description: Informazioni su come distribuire un'app con scalabilità tra cloud in Azure e hub Azure Stack.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 740a8c0ec904fe8eb3f9744626bc9dd6655bdb52
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911524"
---
# <a name="deploy-an-app-that-scales-cross-cloud-using-azure-and-azure-stack-hub"></a>Distribuire un'app scalabile tra cloud usando Azure e l'hub Azure Stack

Informazioni su come creare una soluzione tra cloud per fornire un processo attivato manualmente per passare da un'app Web ospitata nell'hub Azure Stack a un'app Web ospitata in Azure con scalabilità automatica tramite Gestione traffico. Questo processo garantisce un'utilità cloud flessibile e scalabile in fase di caricamento.

Con questo modello, il tenant potrebbe non essere pronto per l'esecuzione dell'app nel cloud pubblico. Tuttavia, potrebbe non essere economicamente fattibile per l'azienda mantenere la capacità richiesta nell'ambiente locale per gestire i picchi di domanda per l'app. Il tenant può sfruttare l'elasticità del cloud pubblico con la soluzione locale.

In questa soluzione verrà compilato un ambiente di esempio per:

> [!div class="checklist"]
> - Creare un'app Web a più nodi.
> - Configurare e gestire il processo di distribuzione continua (CD).
> - Pubblicare l'app Web nell'hub Azure Stack.
> - Creare una versione.
> - Informazioni su come monitorare e tenere traccia delle distribuzioni.

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub è un'estensione di Azure. Azure Stack Hub offre l'agilità e l'innovazione di cloud computing all'ambiente locale, abilitando l'unico Cloud ibrido che consente di creare e distribuire app ibride ovunque.  
> 
> L'articolo [considerazioni sulla progettazione di app ibride](overview-app-design-considerations.md) esamina i pilastri della qualità del software (posizionamento, scalabilità, disponibilità, resilienza, gestibilità e sicurezza) per la progettazione, la distribuzione e la gestione di app ibride. Le considerazioni di progettazione consentono di ottimizzare la progettazione delle app ibride, riducendo al minimo le esigenze negli ambienti di produzione.

## <a name="prerequisites"></a>Prerequisiti

- Sottoscrizione di Azure. Se necessario, creare un [account gratuito](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) prima di iniziare.
- Un sistema integrato di Azure Stack Hub o una distribuzione di Azure Stack Development Kit (Gabriele).
  - Per istruzioni sull'installazione di Azure Stack Hub, vedere [Install the Gabriele](/azure-stack/asdk/asdk-install.md).
  - Per uno script di automazione post-distribuzione di Gabriele, vedere:[https://github.com/mattmcspirit/azurestack](https://github.com/mattmcspirit/azurestack)
  - Per il completamento di questa installazione potrebbero essere necessarie alcune ore.
- Distribuire i servizi PaaS del [servizio app](/azure-stack/operator/azure-stack-app-service-deploy.md) nell'hub Azure stack.
- [Consente di creare piani/offerte](/azure-stack/operator/service-plan-offer-subscription-overview.md) all'interno dell'ambiente Azure stack Hub.
- [Creare una sottoscrizione tenant](/azure-stack/operator/azure-stack-subscribe-plan-provision-vm.md) all'interno dell'ambiente Azure stack Hub.
- Creare un'app Web all'interno della sottoscrizione tenant. Prendere nota del nuovo URL dell'app Web per usarlo in seguito.
- Distribuire Azure Pipelines macchina virtuale (VM) nella sottoscrizione tenant.
- È necessaria una macchina virtuale Windows Server 2016 con .NET 3,5. Questa macchina virtuale verrà compilata nella sottoscrizione tenant nell'hub Azure Stack come agente di compilazione privato.
- [Windows Server 2016 con immagine di macchina virtuale SQL 2017](/azure-stack/operator/azure-stack-add-vm-image.md) è disponibile nel Marketplace di hub Azure stack. Se questa immagine non è disponibile, usare un operatore Azure Stack hub per assicurarsi che venga aggiunta all'ambiente.

## <a name="issues-and-considerations"></a>Considerazioni e problemi

### <a name="scalability"></a>Scalabilità

Il componente principale del ridimensionamento tra cloud è la possibilità di fornire scalabilità immediata e su richiesta tra l'infrastruttura cloud pubblica e locale, fornendo un servizio coerente e affidabile.

### <a name="availability"></a>Disponibilità

Assicurarsi che le app distribuite localmente siano configurate per la disponibilità elevata tramite la configurazione hardware locale e la distribuzione software.

### <a name="manageability"></a>Gestione

La soluzione tra cloud garantisce una gestione semplice e un'interfaccia familiare tra gli ambienti. PowerShell è consigliato per la gestione multipiattaforma.

## <a name="cross-cloud-scaling"></a>Scalabilità tra cloud

### <a name="get-a-custom-domain-and-configure-dns"></a>Ottenere un dominio personalizzato e configurare DNS

Aggiornare il file di zona DNS per il dominio. Azure AD verificherà la proprietà del nome di dominio personalizzato. Usare [DNS di Azure](https://docs.microsoft.com/azure/dns/dns-getstarted-portal) per i record DNS di Azure/Office 365/esterni in Azure oppure aggiungere la voce DNS in [un registrar DNS diverso](https://support.office.com/article/Create-DNS-records-for-Office-365-when-you-manage-your-DNS-records-b0f3fdca-8a80-4e8e-9ef3-61e8a2a9ab23/).

1. Registrare un dominio personalizzato con un registrar pubblico.
2. Accedere al registrar per il dominio. Per eseguire gli aggiornamenti DNS, potrebbe essere necessario un amministratore approvato.
3. Aggiornare il file di zona DNS per il dominio aggiungendo la voce DNS fornita da Azure AD. La voce DNS non influirà sul routing della posta elettronica o sui comportamenti di hosting Web.

### <a name="create-a-default-multi-node-web-app-in-azure-stack-hub"></a>Creare un'app Web multinodo predefinita nell'hub Azure Stack

Configurare l'integrazione continua ibrida e la distribuzione continua (CI/CD) per distribuire app Web in Azure e Azure Stack Hub e per eseguire il push delle modifiche in entrambi i cloud.

> [!Note]  
> È necessario Azure Stack Hub con le immagini appropriate per l'esecuzione (Windows Server e SQL) e la distribuzione del servizio app. Per altre informazioni, vedere la documentazione del servizio app [prerequisiti per la distribuzione del servizio app nell'Hub Azure stack](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).

### <a name="add-code-to-azure-repos"></a>Aggiungi codice a Azure Repos

Azure Repos

1. Accedere a Azure Repos con un account che disponga dei diritti di creazione del progetto per Azure Repos.

    CI/CD ibrido può essere applicato sia al codice dell'app che al codice dell'infrastruttura. USA [modelli di Azure Resource Manager](https://azure.microsoft.com/resources/templates/) per lo sviluppo cloud privato e ospitato.

    ![Connettersi a un progetto in Azure Repos](media/solution-deployment-guide-cross-cloud-scaling/image1.JPG)

2. **Clonare il repository** creando e aprendo l'app Web predefinita.

    ![Clonare il repository nell'app Web di Azure](media/solution-deployment-guide-cross-cloud-scaling/image2.png)

### <a name="create-self-contained-web-app-deployment-for-app-services-in-both-clouds"></a>Creare una distribuzione di app Web indipendente per i servizi app in entrambi i cloud

1. Modificare il file **WebApplication. csproj** . Selezionare `Runtimeidentifier` e aggiungere `win10-x64` . Vedere la documentazione sulla [distribuzione indipendente](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) .

    ![Modificare il file di progetto dell'app Web](media/solution-deployment-guide-cross-cloud-scaling/image3.png)

2. Archiviare il codice per Azure Repos usando Team Explorer.

3. Verificare che il codice dell'app sia stato archiviato Azure Repos.

## <a name="create-the-build-definition"></a>Creare la definizione di compilazione

1. Accedere a Azure Pipelines per verificare la possibilità di creare definizioni di compilazione.

2. Add **-r WIN10-x64** code. Questa aggiunta è necessaria per attivare una distribuzione autonoma con .NET Core.

    ![Aggiungere codice all'app Web](media/solution-deployment-guide-cross-cloud-scaling/image4.png)

3. Eseguire la compilazione. Il processo di [compilazione della distribuzione autonoma](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) pubblicherà gli artefatti eseguiti in Azure e in hub Azure stack.

## <a name="use-an-azure-hosted-agent"></a>Usare un agente ospitato di Azure

L'uso di un agente di compilazione ospitato in Azure Pipelines è un'opzione utile per compilare e distribuire le app Web. La manutenzione e gli aggiornamenti vengono eseguiti automaticamente da Microsoft Azure, consentendo un ciclo di sviluppo continuo e senza interruzioni.

### <a name="manage-and-configure-the-cd-process"></a>Gestire e configurare il processo CD

Azure Pipelines e Azure DevOps Services forniscono una pipeline altamente configurabile e gestibile per i rilasci in più ambienti come gli ambienti di sviluppo, staging, controllo di qualità e produzione; inclusa la richiesta di approvazioni in fasi specifiche.

## <a name="create-release-definition"></a>Creare una definizione della versione

1. Selezionare il pulsante **più** per aggiungere una nuova versione nella scheda **versioni** della sezione **compilazione e versione** di Azure DevOps Services.

    ![Creare una definizione di versione](media/solution-deployment-guide-cross-cloud-scaling/image5.png)

2. Applicare il modello di distribuzione del servizio app Azure.

   ![Applicare il modello di distribuzione del servizio app Azure](meDia/solution-deployment-guide-cross-cloud-scaling/image6.png)

3. In **Aggiungi artefatto**aggiungere l'artefatto per l'app Azure cloud Build.

   ![Aggiungere un elemento alla compilazione cloud di Azure](media/solution-deployment-guide-cross-cloud-scaling/image7.png)

4. In scheda pipeline selezionare la **fase,** il collegamento all'attività dell'ambiente e impostare i valori dell'ambiente cloud di Azure.

   ![Impostare i valori dell'ambiente cloud di Azure](media/solution-deployment-guide-cross-cloud-scaling/image8.png)

5. Impostare il **nome dell'ambiente** e selezionare la **sottoscrizione di Azure** per l'endpoint cloud di Azure.

      ![Selezionare la sottoscrizione di Azure per l'endpoint cloud di Azure](media/solution-deployment-guide-cross-cloud-scaling/image9.png)

6. In **nome servizio app**impostare il nome del servizio app di Azure richiesto.

      ![Imposta il nome del servizio app di Azure](media/solution-deployment-guide-cross-cloud-scaling/image10.png)

7. Immettere "Hosted VS2017" nella **coda dell'agente per l'** ambiente ospitato nel cloud di Azure.

      ![Impostare la coda agente per l'ambiente ospitato nel cloud di Azure](media/solution-deployment-guide-cross-cloud-scaling/image11.png)

8. Nel menu Distribuisci servizio app Azure selezionare il **pacchetto o la cartella** validi per l'ambiente. Selezionare **OK** per **percorso cartella**.
  
      ![Selezionare il pacchetto o la cartella per l'ambiente del servizio app Azure](media/solution-deployment-guide-cross-cloud-scaling/image12.png)

      ![Selezionare il pacchetto o la cartella per l'ambiente del servizio app Azure](media/solution-deployment-guide-cross-cloud-scaling/image13.png)

9. Salvare tutte le modifiche e tornare alla **pipeline di rilascio**.

    ![Salva le modifiche nella pipeline di versione](media/solution-deployment-guide-cross-cloud-scaling/image14.png)

10. Aggiungere un nuovo elemento selezionando la compilazione per l'App Hub Azure Stack.

    ![Aggiungi nuovo artefatto per Azure Stack App Hub](media/solution-deployment-guide-cross-cloud-scaling/image15.png)

11. Aggiungere un altro ambiente applicando la distribuzione del servizio app Azure.

    ![Aggiungi ambiente alla distribuzione del servizio app Azure](media/solution-deployment-guide-cross-cloud-scaling/image16.png)

12. Assegnare al nuovo ambiente il nome "Azure Stack".

    ![Ambiente del nome nella distribuzione del servizio app Azure](media/solution-deployment-guide-cross-cloud-scaling/image17.png)

13. Trovare l'ambiente Azure Stack nella scheda **attività** .

    ![Ambiente Azure Stack](media/solution-deployment-guide-cross-cloud-scaling/image18.png)

14. Selezionare la sottoscrizione per l'endpoint Azure Stack.

    ![Selezionare la sottoscrizione per l'endpoint Azure Stack](media/solution-deployment-guide-cross-cloud-scaling/image19.png)

15. Impostare il nome dell'app Web Azure Stack come nome del servizio app.
    ![Imposta il nome dell'app Web Azure Stack](media/solution-deployment-guide-cross-cloud-scaling/image20.png)

16. Selezionare l'agente di Azure Stack.

    ![Selezionare l'agente di Azure Stack](media/solution-deployment-guide-cross-cloud-scaling/image21.png)

17. Nella sezione Distribuisci servizio app Azure selezionare il **pacchetto o la cartella** validi per l'ambiente. Selezionare **OK** per percorso cartella.

    ![Selezionare la cartella per la distribuzione del servizio app Azure](media/solution-deployment-guide-cross-cloud-scaling/image22.png)

    ![Selezionare la cartella per la distribuzione del servizio app Azure](media/solution-deployment-guide-cross-cloud-scaling/image23.png)

18. In scheda variabile aggiungere una variabile denominata `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS` , impostarne il valore su **true**e l'ambito su Azure stack.

    ![Aggiungere una variabile alla distribuzione app Azure](media/solution-deployment-guide-cross-cloud-scaling/image24.png)

19. Selezionare l'icona del trigger di distribuzione **continua** in entrambi gli artefatti e abilitare il trigger di distribuzione **continua** .

    ![Selezionare il trigger di distribuzione continua](media/solution-deployment-guide-cross-cloud-scaling/image25.png)

20. Selezionare l'icona condizioni **pre-distribuzione** nell'ambiente Azure stack e impostare il trigger su **dopo il rilascio.**

    ![Selezionare le condizioni di pre-distribuzione](media/solution-deployment-guide-cross-cloud-scaling/image26.png)

21. Salvare tutte le modifiche.

> [!Note]  
> Alcune impostazioni per le attività potrebbero essere state definite automaticamente come [variabili di ambiente](https://docs.microsoft.com/azure/devops/pipelines/release/variables?view=vsts&tabs=batch#custom-variables) quando si crea una definizione di versione da un modello. Queste impostazioni non possono essere modificate nelle impostazioni dell'attività. per modificare queste impostazioni è necessario invece selezionare l'elemento dell'ambiente padre.

## <a name="publish-to-azure-stack-hub-via-visual-studio"></a>Pubblicare nell'hub Azure Stack tramite Visual Studio

Creando gli endpoint, una Azure DevOps Services Build può distribuire le app del servizio di Azure nell'hub Azure Stack. Azure Pipelines si connette all'agente di compilazione, che si connette all'hub Azure Stack.

1. Accedere a Azure DevOps Services e passare alla pagina delle impostazioni dell'app.

2. Su **Impostazioni**, selezionare **Sicurezza**.

3. In **gruppi VSTS**selezionare **creatori endpoint**.

4. Nella scheda **Membri** selezionare **Aggiungi**.

5. In **Aggiungi utenti e gruppi**immettere un nome utente e selezionare l'utente dall'elenco di utenti.

6. Selezionare **Save changes** (Salva modifiche).

7. Nell'elenco **gruppi VSTS** selezionare **amministratori endpoint**.

8. Nella scheda **Membri** selezionare **Aggiungi**.

9. In **Aggiungi utenti e gruppi**immettere un nome utente e selezionare l'utente dall'elenco di utenti.

10. Selezionare **Save changes** (Salva modifiche).

Ora che le informazioni sull'endpoint esistono, il Azure Pipelines Azure Stack connessione hub è pronto per l'uso. L'agente di compilazione nell'hub Azure Stack ottiene le istruzioni da Azure Pipelines e quindi l'agente trasmette informazioni sugli endpoint per la comunicazione con l'hub Azure Stack.

## <a name="develop-the-app-build"></a>Sviluppare la compilazione dell'app

> [!Note]  
> È necessario Azure Stack Hub con le immagini appropriate per l'esecuzione (Windows Server e SQL) e la distribuzione del servizio app. Per altre informazioni, vedere [prerequisiti per la distribuzione del servizio app nell'Hub Azure stack](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).

Usare [modelli Azure Resource Manager](https://azure.microsoft.com/resources/templates/) come il codice dell'app web da Azure Repos per eseguire la distribuzione in entrambi i cloud.

### <a name="add-code-to-an-azure-repos-project"></a>Aggiungere codice a un progetto Azure Repos

1. Accedere a Azure Repos con un account che disponga dei diritti di creazione del progetto nell'hub Azure Stack.

2. **Clonare il repository** creando e aprendo l'app Web predefinita.

#### <a name="create-self-contained-web-app-deployment-for-app-services-in-both-clouds"></a>Creare una distribuzione di app Web indipendente per i servizi app in entrambi i cloud

1. Modificare il file **WebApplication. csproj** : selezionare `Runtimeidentifier` e quindi Aggiungi `win10-x64` . Per ulteriori informazioni, vedere la documentazione sulla [distribuzione indipendente](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) .

2. Usare Team Explorer per controllare il codice in Azure Repos.

3. Verificare che il codice dell'app sia stato archiviato Azure Repos.

### <a name="create-the-build-definition"></a>Creare la definizione di compilazione

1. Accedere a Azure Pipelines con un account che può creare una definizione di compilazione.

2. Passare alla pagina **Compila applicazione Web** per il progetto.

3. In **argomenti**aggiungere il codice **WIN10-x64** . Questa aggiunta è necessaria per attivare una distribuzione autonoma con .NET Core.

4. Eseguire la compilazione. Il processo di [compilazione della distribuzione autonoma](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) pubblicherà gli artefatti che possono essere eseguiti in Azure e in hub Azure stack.

#### <a name="use-an-azure-hosted-build-agent"></a>Usare un agente di compilazione ospitato in Azure

L'uso di un agente di compilazione ospitato in Azure Pipelines è un'opzione utile per compilare e distribuire le app Web. La manutenzione e gli aggiornamenti vengono eseguiti automaticamente da Microsoft Azure, consentendo un ciclo di sviluppo continuo e senza interruzioni.

### <a name="configure-the-continuous-deployment-cd-process"></a>Configurare il processo di distribuzione continua (CD)

Azure Pipelines e Azure DevOps Services forniscono una pipeline altamente configurabile e gestibile per le versioni in più ambienti quali sviluppo, staging, controllo di qualità (QA) e produzione. Questo processo può includere la richiesta di approvazioni in fasi specifiche del ciclo di vita dell'app.

#### <a name="create-release-definition"></a>Creare una definizione della versione

La creazione di una definizione di versione è il passaggio finale del processo di compilazione dell'app. Questa definizione di versione viene usata per creare una versione e distribuire una compilazione.

1. Accedere a Azure Pipelines e passare a **compilazione e versione** per il progetto.

2. Nella scheda **versioni** selezionare **[+]** , quindi scegliere **Crea definizione di versione**.

3. In **selezionare un modello**scegliere **app Azure distribuzione del servizio**e quindi selezionare **applica**.

4. In **Aggiungi artefatto**selezionare l'app Azure cloud Build dall' **origine (definizione di compilazione)**.

5. Nella scheda **pipeline** selezionare il collegamento **1 fase**, **1 attività** per visualizzare le **attività dell'ambiente**.

6. Nella scheda **attività** immettere Azure come **nome dell'ambiente** e selezionare il AzureCloud Traders-Web EP dall'elenco **sottoscrizione di Azure** .

7. Immettere il **nome del servizio app di Azure**, che si trova `northwindtraders` nella schermata successiva.

8. Per la fase Agent selezionare **Hosted VS2017** dall'elenco **coda agente** .

9. In **Distribuisci servizio app Azure**selezionare il **pacchetto o la cartella** validi per l'ambiente.

10. In **Seleziona file o cartella**selezionare **OK** in **percorso**.

11. Salvare tutte le modifiche e tornare alla **pipeline**.

12. Nella scheda **pipeline** selezionare **Aggiungi artefatto**e scegliere **NorthwindCloud Traders-vessel** dall'elenco di **origine (definizione di compilazione)** .

13. In **selezionare un modello**aggiungere un altro ambiente. Scegliere **app Azure distribuzione del servizio** e quindi selezionare **applica**.

14. Immettere `Azure Stack Hub` come **nome dell'ambiente**.

15. Nella scheda **attività** trovare e selezionare Azure stack Hub.

16. Dall'elenco **sottoscrizione di Azure** selezionare **AzureStack Traders-vessel EP** per l'endpoint dell'hub Azure stack.

17. Immettere il nome dell'app Web dell'hub Azure Stack come **nome del servizio app**.

18. In **Selezione agente**selezionare **AzureStack-b Douglas Fir** dall'elenco **coda agente** .

19. Per **Distribuisci servizio app Azure**selezionare il **pacchetto o la cartella** validi per l'ambiente. In **Seleziona file o cartella**selezionare **OK** per **percorso**cartella.

20. Nella scheda **variabile** individuare la variabile denominata `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS` . Impostare il valore della variabile su **true**e impostarne l'ambito su **Azure stack Hub**.

21. Nella scheda **pipeline** selezionare l'icona del **trigger di distribuzione continua** per l'artefatto NorthwindCloud Traders-Web e impostare il **trigger di distribuzione continua** su **abilitato**. Eseguire la stessa operazione per l'artefatto **NorthwindCloud Traders-vessel** .

22. Per l'ambiente Azure Stack Hub, selezionare l'icona **condizioni pre-distribuzione** impostare il trigger su **dopo la versione**.

23. Salvare tutte le modifiche.

> [!Note]  
> Alcune impostazioni per le attività di rilascio vengono definite automaticamente come [variabili di ambiente](https://docs.microsoft.com/azure/devops/pipelines/release/variables?view=vsts&tabs=batch#custom-variables) quando si crea una definizione di versione da un modello. Queste impostazioni non possono essere modificate nelle impostazioni dell'attività, ma possono essere modificate negli elementi dell'ambiente padre.

## <a name="create-a-release"></a>Creare una versione

1. Nella scheda **pipeline** aprire l'elenco **versione** e selezionare **Crea versione**.

2. Immettere una descrizione per la versione, verificare che siano selezionati gli elementi corretti, quindi selezionare **Crea**. Dopo alcuni istanti, viene visualizzato un banner che indica che la nuova versione è stata creata e il nome della versione viene visualizzato come collegamento. Selezionare il collegamento per visualizzare la pagina di riepilogo della versione.

3. Nella pagina di riepilogo della versione vengono visualizzati i dettagli relativi alla versione. Nella schermata seguente per "Release-2", la sezione **environments** Mostra lo **stato di distribuzione** per Azure come "in corso" e lo stato di Azure stack Hub è "succeeded". Quando lo stato della distribuzione per l'ambiente Azure diventa "SUCCEEDed", viene visualizzato un banner che indica che il rilascio è pronto per l'approvazione. Quando una distribuzione è in sospeso o non è riuscita, viene visualizzata un'icona di informazioni blu **(i)** . Passare il puntatore del mouse sull'icona per visualizzare un popup che contiene il motivo del ritardo o dell'errore.

4. Altre visualizzazioni, come l'elenco di versioni, visualizzano anche un'icona che indica che l'approvazione è in sospeso. Il popup per questa icona Mostra il nome dell'ambiente e altri dettagli relativi alla distribuzione. È facile per un amministratore vedere lo stato di avanzamento complessivo dei rilasci e vedere quali versioni sono in attesa di approvazione.

## <a name="monitor-and-track-deployments"></a>Monitorare e tenere traccia delle distribuzioni

1. Nella pagina Riepilogo **versione 2** selezionare **log**. Durante una distribuzione, in questa pagina viene visualizzato il log live dall'agente. Il riquadro sinistro mostra lo stato di ogni operazione nella distribuzione per ogni ambiente.

2. Selezionare l'icona person nella colonna **azione** per un'approvazione pre-distribuzione o post-distribuzione per verificare chi ha approvato (o rifiutato) la distribuzione e il messaggio fornito.

3. Al termine della distribuzione, nel riquadro destro viene visualizzato l'intero file di log. Selezionare un **passaggio** nel riquadro sinistro per visualizzare il file di log per un singolo passaggio, ad esempio **Initialize Job**. La possibilità di visualizzare i singoli log semplifica la traccia e il debug delle parti della distribuzione complessiva. **Salvare** il file di log per un passaggio o **scaricare tutti i log come zip**.

4. Aprire la scheda **Riepilogo** per visualizzare le informazioni generali sulla versione. Questa visualizzazione Mostra i dettagli relativi alla compilazione, gli ambienti in cui è stato distribuito, lo stato di distribuzione e altre informazioni sulla versione.

5. Selezionare un collegamento all'ambiente (**Azure** o **Hub Azure stack**) per visualizzare informazioni sulle distribuzioni esistenti e in sospeso in un ambiente specifico. Usare queste visualizzazioni come metodo rapido per verificare che la stessa compilazione sia stata distribuita in entrambi gli ambienti.

6. Aprire l' **app di produzione distribuita** in un browser. Per il sito web app Azure Services, ad esempio, aprire l'URL `https://[your-app-name\].azurewebsites.net` .

### <a name="integration-of-azure-and-azure-stack-hub-provides-a-scalable-cross-cloud-solution"></a>L'integrazione di Azure e Azure Stack Hub offre una soluzione scalabile tra cloud

Un servizio multicloud flessibile e affidabile offre sicurezza dei dati, backup e ridondanza, disponibilità coerente e rapida, archiviazione e distribuzione scalabili e routing conforme a geografica. Questo processo attivato manualmente garantisce un cambio di carico affidabile ed efficiente tra le app Web ospitate e la disponibilità immediata dei dati cruciali.

## <a name="next-steps"></a>Passaggi successivi

- Per altre informazioni sui modelli cloud di Azure, vedere [modelli di progettazione cloud](https://docs.microsoft.com/azure/architecture/patterns).
