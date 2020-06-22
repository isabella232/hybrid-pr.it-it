---
title: Indirizzare il traffico con un'app con distribuzione geografica usando Azure e l'hub Azure Stack
description: Informazioni su come indirizzare il traffico a endpoint specifici con una soluzione di app con distribuzione geografica con Azure e hub Azure Stack.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 8f2b7e48a62896acfce7293dcd4f18d5a43add01
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911325"
---
# <a name="direct-traffic-with-a-geo-distributed-app-using-azure-and-azure-stack-hub"></a>Indirizzare il traffico con un'app con distribuzione geografica usando Azure e l'hub Azure Stack

Informazioni su come indirizzare il traffico a endpoint specifici in base a diverse metriche usando il modello di app con distribuzione geografica. La creazione di un profilo di gestione traffico con routing basato su geografia e configurazione dell'endpoint garantisce che le informazioni vengano indirizzate agli endpoint in base ai requisiti internazionali, alla regolamentazione aziendale e internazionale e alle esigenze dei dati.

In questa soluzione verrà compilato un ambiente di esempio per:

> [!div class="checklist"]
> - Creare un'app con distribuzione geografica.
> - Usare gestione traffico per la destinazione dell'app.

## <a name="use-the-geo-distributed-apps-pattern"></a>Usare il modello di app con distribuzione geografica

Con il modello con distribuzione geografica, l'app si estende su più aree. Per impostazione predefinita, è possibile utilizzare il cloud pubblico, ma alcuni utenti potrebbero richiedere che i loro dati rimangano nella propria area geografica. Puoi indirizzare gli utenti al cloud più adatto in base ai requisiti.

### <a name="issues-and-considerations"></a>Considerazioni e problemi

#### <a name="scalability-considerations"></a>Considerazioni sulla scalabilità

La soluzione che verrà creata con questo articolo non è quella di adattare la scalabilità. Tuttavia, se usato in combinazione con altre soluzioni Azure e locali, è possibile soddisfare i requisiti di scalabilità. Per informazioni sulla creazione di una soluzione ibrida con la scalabilità automatica tramite Gestione traffico, vedere [creare soluzioni di scalabilità tra cloud con Azure](solution-deployment-guide-cross-cloud-scaling.md).

#### <a name="availability-considerations"></a>Considerazioni sulla disponibilità

Come nel caso delle considerazioni sulla scalabilità, questa soluzione non risolve direttamente la disponibilità. Tuttavia, Azure e le soluzioni locali possono essere implementate all'interno di questa soluzione per garantire la disponibilità elevata per tutti i componenti interessati.

### <a name="when-to-use-this-pattern"></a>Quando usare questo modello

- L'organizzazione dispone di rami internazionali che richiedono criteri personalizzati per la sicurezza e la distribuzione a livello di area.

- Ogni ufficio dell'organizzazione effettua il pull dei dati relativi a dipendenti, aziende e funzionalità, che richiedono l'attività di creazione di report in base alle normative locali e ai fusi orari.

- I requisiti per la scalabilità elevata sono soddisfatti con la scalabilità orizzontale delle app con più distribuzioni di app all'interno di una singola area e tra aree per gestire i requisiti di carico estremi.

### <a name="planning-the-topology"></a>Pianificazione della topologia

Prima di creare un footprint per le app distribuite, è utile sapere quanto segue:

- **Dominio personalizzato per l'app:** Qual è il nome di dominio personalizzato che i clienti useranno per accedere all'app? Per l'app di esempio, il nome di dominio personalizzato è *www \. scalableasedemo.com.*

- **Dominio di Traffic Manager:** Quando si crea un [profilo di gestione traffico di Azure](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-manage-profiles), viene scelto un nome di dominio. Questo nome viene combinato con il suffisso *trafficmanager.NET* per registrare una voce di dominio gestita da Gestione traffico. Per l'app di esempio il nome scelto è *scalable-ase-demo*. Di conseguenza, il nome di dominio completo gestito da Traffic Manager è *Scalable-ASE-demo.trafficmanager.NET*.

- **Strategia per la scalabilità del footprint dell'app:** Decidere se il footprint dell'app verrà distribuito tra più ambienti del servizio app in una singola area, più aree o una combinazione di entrambi gli approcci. La decisione deve essere basata sulle aspettative relative al modo in cui il traffico dei clienti produrrà e quanto il resto dell'infrastruttura di back-end di supporto di un'app possa essere ridimensionato. Con un'app senza stato del 100%, ad esempio, un'app può essere ridimensionata in modo massiccio usando una combinazione di più ambienti del servizio app per ogni area di Azure, moltiplicata per gli ambienti del servizio app distribuiti in più aree di Azure. Con più di 15 aree globali di Azure disponibili per la scelta, i clienti possono effettivamente creare un footprint di app su vasta scala mondiale. Per l'app di esempio usata in questo articolo, sono stati creati tre ambienti del servizio app in una singola area di Azure (Stati Uniti centro-meridionali).

- **Convenzione di denominazione per gli ambienti del servizio app:** Ogni ambiente del servizio app richiede un nome univoco. Oltre uno o due ambienti del servizio app, è utile avere una convenzione di denominazione che consenta di identificare ogni ambiente del servizio app. Per l'app di esempio usata in questo argomento, è stata usata una convenzione di denominazione semplice. I nomi dei tre ambienti del servizio app sono *fe1ase*, *fe2ase*e *fe3ase*.

- **Convenzione di denominazione per le app:** Poiché verranno distribuite più istanze dell'app, è necessario un nome per ogni istanza dell'app distribuita. Con ambiente del servizio app per Power Apps, lo stesso nome di app può essere usato in più ambienti. Poiché ogni ambiente del servizio app ha un suffisso di dominio univoco, gli sviluppatori possono scegliere di riutilizzare esattamente lo stesso nome di app in ogni ambiente. Ad esempio, uno sviluppatore potrebbe avere app denominate come segue: *MyApp.foo1.p.azurewebsites.NET*, *MyApp.foo2.p.azurewebsites.NET*, *MyApp.foo3.p.azurewebsites.NET*e così via. Per l'app usata qui, ogni istanza dell'app ha un nome univoco. I nomi delle istanze dell'app usati sono *webfrontend1*, *webfrontend2* e *webfrontend3*.

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub è un'estensione di Azure. Azure Stack Hub offre l'agilità e l'innovazione di cloud computing all'ambiente locale, abilitando l'unico Cloud ibrido che consente di creare e distribuire app ibride ovunque.  
> 
> L'articolo [considerazioni sulla progettazione di app ibride](overview-app-design-considerations.md) esamina i pilastri della qualità del software (posizionamento, scalabilità, disponibilità, resilienza, gestibilità e sicurezza) per la progettazione, la distribuzione e la gestione di app ibride. Le considerazioni di progettazione consentono di ottimizzare la progettazione delle app ibride, riducendo al minimo le esigenze negli ambienti di produzione.

## <a name="part-1-create-a-geo-distributed-app"></a>Parte 1: creare un'app con distribuzione geografica

In questa parte verrà creata un'app Web.

> [!div class="checklist"]
> - Creazione di app Web e pubblicazione.
> - Aggiungere il codice Azure Repos.
> - Puntare la compilazione dell'app a più destinazioni cloud.
> - Gestire e configurare il processo CD.

### <a name="prerequisites"></a>Prerequisiti

Sono necessarie una sottoscrizione di Azure e l'installazione dell'hub Azure Stack.

### <a name="geo-distributed-app-steps"></a>Passaggi dell'app con distribuzione geografica

### <a name="obtain-a-custom-domain-and-configure-dns"></a>Ottenere un dominio personalizzato e configurare DNS

Aggiornare il file di zona DNS per il dominio. Azure AD possibile verificare la proprietà del nome di dominio personalizzato. Usare [DNS di Azure](https://docs.microsoft.com/azure/dns/dns-getstarted-portal) per i record DNS di Azure/Office 365/esterni in Azure oppure aggiungere la voce DNS in [un registrar DNS diverso](https://support.office.com/article/Create-DNS-records-for-Office-365-when-you-manage-your-DNS-records-b0f3fdca-8a80-4e8e-9ef3-61e8a2a9ab23/).

1. Registrare un dominio personalizzato con un registrar pubblico.

2. Accedere al registrar per il dominio. Potrebbe essere necessario un amministratore approvato per eseguire gli aggiornamenti DNS.

3. Aggiornare il file di zona DNS per il dominio aggiungendo la voce DNS fornita da Azure AD. La voce DNS non modifica i comportamenti, ad esempio il routing della posta elettronica o l'hosting Web.

### <a name="create-web-apps-and-publish"></a>Creare app Web e pubblicare

Configurare l'integrazione continua e la distribuzione continua (CI/CD) ibride per distribuire l'app Web in Azure e l'hub Azure Stack e per eseguire il push automatico delle modifiche in entrambi i cloud.

> [!Note]  
> È necessario Azure Stack Hub con le immagini appropriate per l'esecuzione (Windows Server e SQL) e la distribuzione del servizio app. Per altre informazioni, vedere [prerequisiti per la distribuzione del servizio app nell'Hub Azure stack](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).

#### <a name="add-code-to-azure-repos"></a>Aggiungi codice a Azure Repos

1. Accedere a Visual Studio con un **account che disponga dei diritti di creazione del progetto** in Azure Repos.

    CI/CD può essere applicato sia al codice dell'app che al codice dell'infrastruttura. USA [modelli di Azure Resource Manager](https://azure.microsoft.com/resources/templates/) per lo sviluppo cloud privato e ospitato.

    ![Connettersi a un progetto in Visual Studio](media/solution-deployment-guide-geo-distributed/image1.JPG)

2. **Clonare il repository** creando e aprendo l'app Web predefinita.

    ![Clonare il repository in Visual Studio](media/solution-deployment-guide-geo-distributed/image2.png)

### <a name="create-web-app-deployment-in-both-clouds"></a>Creare una distribuzione di app Web in entrambi i cloud

1. Modificare il file **WebApplication. csproj** : selezionare `Runtimeidentifier` e aggiungere `win10-x64` . Vedere la documentazione sulla [distribuzione indipendente](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) .

    ![Modificare il file di progetto dell'app Web in Visual Studio](media/solution-deployment-guide-geo-distributed/image3.png)

2. **Archiviare il codice per Azure Repos** usando Team Explorer.

3. Verificare che il **codice dell'applicazione** sia stato archiviato Azure Repos.

### <a name="create-the-build-definition"></a>Creare la definizione di compilazione

1. **Accedere a Azure Pipelines** per verificare la possibilità di creare definizioni di compilazione.

2. Aggiungere il `-r win10-x64` codice. Questa aggiunta è necessaria per attivare una distribuzione autonoma con .NET Core.

    ![Aggiungere codice alla definizione di compilazione in Azure Pipelines](media/solution-deployment-guide-geo-distributed/image4.png)

3. **Eseguire la compilazione**. Il processo di [compilazione della distribuzione autonoma](https://docs.microsoft.com/dotnet/core/deploying/deploy-with-vs#simpleSelf) pubblicherà gli artefatti che possono essere eseguiti in Azure e in hub Azure stack.

#### <a name="using-an-azure-hosted-agent"></a>Uso di un agente ospitato di Azure

L'uso di un agente ospitato in Azure Pipelines è un'opzione utile per compilare e distribuire le app Web. La manutenzione e gli aggiornamenti vengono eseguiti automaticamente da Microsoft Azure, che consente lo sviluppo, il test e la distribuzione senza interruzioni.

### <a name="manage-and-configure-the-cd-process"></a>Gestire e configurare il processo CD

Azure DevOps Services forniscono una pipeline altamente configurabile e gestibile per le versioni in più ambienti, ad esempio sviluppo, gestione temporanea, controllo di qualità e ambienti di produzione; inclusa la richiesta di approvazioni in fasi specifiche.

## <a name="create-release-definition"></a>Creare una definizione della versione

1. Selezionare il pulsante **più** per aggiungere una nuova versione nella scheda **versioni** della sezione **compilazione e versione** di Azure DevOps Services.

    ![Creare una definizione di versione in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image5.png)

2. Applicare il modello di distribuzione del servizio app Azure.

   ![Applicare il modello di distribuzione del servizio app Azure in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image6.png)

3. In **Aggiungi artefatto**aggiungere l'artefatto per l'app Azure cloud Build.

   ![Aggiungere un elemento alla build del cloud di Azure in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image7.png)

4. In scheda pipeline selezionare la **fase,** il collegamento all'attività dell'ambiente e impostare i valori dell'ambiente cloud di Azure.

   ![Impostare i valori dell'ambiente cloud di Azure in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image8.png)

5. Impostare il **nome dell'ambiente** e selezionare la **sottoscrizione di Azure** per l'endpoint cloud di Azure.

      ![Selezionare la sottoscrizione di Azure per l'endpoint cloud di Azure in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image9.png)

6. In **nome servizio app**impostare il nome del servizio app di Azure richiesto.

      ![Impostare il nome del servizio app di Azure in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image10.png)

7. Immettere "Hosted VS2017" nella **coda dell'agente per l'** ambiente ospitato nel cloud di Azure.

      ![Impostare la coda agente per l'ambiente ospitato nel cloud di Azure in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image11.png)

8. Nel menu Distribuisci servizio app Azure selezionare il **pacchetto o la cartella** validi per l'ambiente. Selezionare **OK** per **percorso cartella**.
  
      ![Selezionare il pacchetto o la cartella per app Azure ambiente del servizio in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image12.png)

      ![Selezionare il pacchetto o la cartella per app Azure ambiente del servizio in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image13.png)

9. Salvare tutte le modifiche e tornare alla **pipeline di rilascio**.

    ![Salva le modifiche nella pipeline di rilascio in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image14.png)

10. Aggiungere un nuovo elemento selezionando la compilazione per l'App Hub Azure Stack.

    ![Aggiungere un nuovo artefatto per Azure Stack App Hub in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image15.png)


11. Aggiungere un altro ambiente applicando la distribuzione del servizio app Azure.

    ![Aggiungere ambiente alla distribuzione del servizio app Azure in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image16.png)

12. Assegnare un nome al nuovo ambiente Azure Stack Hub.

    ![Ambiente del nome nella distribuzione del servizio app Azure in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image17.png)

13. Trovare l'ambiente dell'hub Azure Stack nella scheda **attività** .

    ![Ambiente Azure Stack Hub in Azure DevOps Services Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image18.png)

14. Selezionare la sottoscrizione per l'endpoint dell'hub Azure Stack.

    ![Selezionare la sottoscrizione per l'endpoint dell'hub Azure Stack in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image19.png)

15. Impostare il nome dell'app Web dell'hub Azure Stack come nome del servizio app.

    ![Impostare il nome dell'app Web dell'hub Azure Stack in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image20.png)

16. Selezionare l'agente dell'hub Azure Stack.

    ![Selezionare l'agente Hub Azure Stack in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image21.png)

17. Nella sezione Distribuisci servizio app Azure selezionare il **pacchetto o la cartella** validi per l'ambiente. Selezionare **OK** per percorso cartella.

    ![Selezionare la cartella per la distribuzione del servizio app Azure in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image22.png)

    ![Selezionare la cartella per la distribuzione del servizio app Azure in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image23.png)

18. Nella scheda variabile aggiungere una variabile denominata `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS` , impostarne il valore su **true**e l'ambito su Azure stack Hub.

    ![Aggiungere una variabile alla distribuzione app Azure in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image24.png)

19. Selezionare l'icona del trigger di distribuzione **continua** in entrambi gli artefatti e abilitare il trigger di distribuzione **continua** .

    ![Selezionare il trigger di distribuzione continua in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image25.png)

20. Selezionare l'icona condizioni **pre-distribuzione** nell'ambiente dell'hub Azure stack e impostare il trigger su **dopo la versione.**

    ![Selezionare le condizioni di pre-distribuzione in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image26.png)

21. Salvare tutte le modifiche.

> [!Note]  
> Alcune impostazioni per le attività potrebbero essere state definite automaticamente come [variabili di ambiente](https://docs.microsoft.com/azure/devops/pipelines/release/variables?view=vsts&tabs=batch#custom-variables) quando si crea una definizione di versione da un modello. Queste impostazioni non possono essere modificate nelle impostazioni dell'attività. per modificare queste impostazioni è necessario invece selezionare l'elemento dell'ambiente padre.

## <a name="part-2-update-web-app-options"></a>Parte 2: aggiornare le opzioni dell'app Web

[Servizio app di Azure](https://docs.microsoft.com/azure/app-service/overview) offre un servizio di hosting Web con scalabilità elevata e funzioni di auto-correzione.

![Servizio app di Azure](media/solution-deployment-guide-geo-distributed/image27.png)

> [!div class="checklist"]
> - Eseguire il mapping di un nome DNS personalizzato esistente ad app Web di Azure.
> - Usare un **record CNAME** e un **record** a per eseguire il mapping di un nome DNS personalizzato al servizio app.

### <a name="map-an-existing-custom-dns-name-to-azure-web-apps"></a>Esecuzione del mapping di un nome DNS personalizzato esistente con un app Web di Azure

> [!Note]  
> Usare un record CNAME per tutti i nomi DNS personalizzati ad eccezione di un dominio radice (ad esempio, northwind.com).

Per eseguire la migrazione di un sito in tempo reale e del relativo nome di dominio DNS al servizio app, vedere [Migrare un nome DNS attivo nel servizio app di Azure](https://docs.microsoft.com/azure/app-service/manage-custom-dns-migrate-domain).

### <a name="prerequisites"></a>Prerequisiti

Per completare questa soluzione:

- [Creare un'app del servizio app](https://docs.microsoft.com/azure/app-service/)o usare un'app creata per un'altra soluzione.

- Acquistare un nome di dominio e verificare l'accesso al registro DNS per il provider di dominio.

Aggiornare il file di zona DNS per il dominio. Azure AD verificherà la proprietà del nome di dominio personalizzato. Usare [DNS di Azure](https://docs.microsoft.com/azure/dns/dns-getstarted-portal) per i record DNS di Azure/Office 365/esterni in Azure oppure aggiungere la voce DNS in [un registrar DNS diverso](https://support.office.com/article/Create-DNS-records-for-Office-365-when-you-manage-your-DNS-records-b0f3fdca-8a80-4e8e-9ef3-61e8a2a9ab23/).

- Registrare un dominio personalizzato con un registrar pubblico.

- Accedere al registrar per il dominio. Per eseguire gli aggiornamenti DNS, potrebbe essere necessario un amministratore approvato.

- Aggiornare il file di zona DNS per il dominio aggiungendo la voce DNS fornita da Azure AD.

Ad esempio, per aggiungere le voci DNS per northwindcloud.com e www \. northwindcloud.com, configurare le impostazioni DNS per il dominio radice northwindcloud.com.

> [!Note]  
> È possibile acquistare un nome di dominio usando il [portale di Azure](https://docs.microsoft.com/azure/app-service/manage-custom-dns-buy-domain). Per eseguire il mapping di un nome DNS personalizzato a un'app Web, è necessario che il [piano di servizio app](https://azure.microsoft.com/pricing/details/app-service/) dell'app Web sia un livello a pagamento (**Condiviso**, **Base**, **Standard** o **Premium**).

### <a name="create-and-map-cname-and-a-records"></a>Creare ed eseguire il mapping di record CNAME e A

#### <a name="access-dns-records-with-domain-provider"></a>Accedere ai record DNS con il provider di dominio

> [!Note]  
>  Usare DNS di Azure per configurare un nome DNS personalizzato per le app Web di Azure. Per altre informazioni, vedere [Usare il servizio DNS di Azure per specificare impostazioni di dominio personalizzate per un servizio di Azure](https://docs.microsoft.com/azure/dns/dns-custom-domain).

1. Accedere al sito Web del provider principale.

2. Individuare la pagina relativa alla gestione dei record DNS. Ogni provider di dominio ha una propria interfaccia di record DNS. Cercare le aree del sito denominate **Domain Name** (Nome di dominio), **DNS** o **Name Server Management** (Gestione server dei nomi).

È possibile visualizzare la pagina record DNS nei **domini personali**. Trovare il collegamento nome **file di zona**, **record DNS**o **configurazione avanzata**.

La schermata seguente è un esempio di pagina di record DNS:

![Pagina record DNS di esempio](media/solution-deployment-guide-geo-distributed/image28.png)

1. In registrar, selezionare **Aggiungi o crea** per creare un record. Per alcuni provider esistono collegamenti diversi per aggiungere tipi di record diversi. Consultare la documentazione del provider.

2. Aggiungere un record CNAME per eseguire il mapping di un sottodominio al nome host predefinito dell'app.

   Per l' \. esempio di dominio www northwindcloud.com aggiungere un record CNAME che esegue il mapping del nome a `<app_name>.azurewebsites.net` .

Dopo aver aggiunto il record CNAME, la pagina dei record DNS è simile all'esempio seguente:

![Passaggio all'app di Azure nel portale](media/solution-deployment-guide-geo-distributed/image29.png)

### <a name="enable-the-cname-record-mapping-in-azure"></a>Abilitare il mapping dei record CNAME in Azure

1. In una nuova scheda accedere al portale di Azure.

2. Passare a Servizi app.

3. Selezionare app Web.

4. Nel riquadro di spostamento a sinistra della pagina dell'app nel portale di Azure selezionare **Domini personalizzati**.

5. Selezionare l' **+** icona accanto a **Aggiungi nome host**.

6. Digitare il nome di dominio completo, ad esempio `www.northwindcloud.com` .

7. Selezionare **Convalida**.

8. Se indicato, aggiungere altri record di altri tipi ( `A` o `TXT` ) ai record DNS del registrar. Azure fornirà i valori e i tipi dei record seguenti:

   a.  Un record **A** di cui eseguire il mapping all'indirizzo IP dell'app.

   b.  Un record **TXT** di cui eseguire il mapping al nome host predefinito dell'app `<app_name>.azurewebsites.net`. Il servizio app usa questo record solo in fase di configurazione per verificare la proprietà del dominio personalizzato. Dopo la verifica, eliminare il record TXT.

9. Completare questa attività nella scheda registrar e rivalidare fino a quando non viene attivato il pulsante **Aggiungi nome host** .

10. Verificare che il **tipo di record del nome host** sia impostato su **CNAME** (www.example.com o qualsiasi sottodominio).

11. Selezionare **Aggiungi il nome host**.

12. Digitare il nome di dominio completo, ad esempio `northwindcloud.com` .

13. Selezionare **Convalida**. L' **aggiunta** è attivata.

14. Verificare che il **tipo di record hostname** sia impostato su **un record** (example.com).

15. **Aggiungere il nome host**.

    La reflection dei nuovi nomi host nella pagina **domini personalizzati** dell'app potrebbe richiedere del tempo. Provare ad aggiornare il browser per visualizzare i dati più recenti.
  
    ![Domini personalizzati](media/solution-deployment-guide-geo-distributed/image31.png) 
  
    Se si verifica un errore, nella parte inferiore della pagina verrà visualizzata una notifica di errore di verifica. ![Errore di verifica del dominio](media/solution-deployment-guide-geo-distributed/image32.png)

> [!Note]  
>  I passaggi precedenti possono essere ripetuti per eseguire il mapping di un dominio con caratteri jolly ( \* northwindcloud.com). In questo modo è possibile aggiungere altri sottodomini a questo servizio app senza dover creare un record CNAME separato per ciascuno di essi. Per configurare questa impostazione, seguire le istruzioni del registrar.

#### <a name="test-in-a-browser"></a>Esegui test in un browser

Individuare i nomi DNS configurati in precedenza (ad esempio, `northwindcloud.com` o `www.northwindcloud.com` ).

## <a name="part-3-bind-a-custom-ssl-cert"></a>Parte 3: associare un certificato SSL personalizzato

In questa parte verrà:

> [!div class="checklist"]
> - Associare il certificato SSL personalizzato al servizio app.
> - Applicare HTTPS per l'app.
> - Automatizzare l'associazione di certificati SSL con gli script.

> [!Note]  
> Se necessario, ottenere un certificato SSL del cliente nel portale di Azure e associarlo all'app Web. Per altre informazioni, vedere l' [esercitazione sui certificati del servizio app](https://docs.microsoft.com/azure/app-service/web-sites-purchase-ssl-web-site).

### <a name="prerequisites"></a>Prerequisiti

Per completare questa soluzione:

- [Creare un'app del servizio app.](https://docs.microsoft.com/azure/app-service/)
- [Eseguire il mapping di un nome DNS personalizzato all'app Web.](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-domain)
- Acquisire un certificato SSL da un'autorità di certificazione attendibile e usare la chiave per firmare la richiesta.

### <a name="requirements-for-your-ssl-certificate"></a>Requisiti per il certificato SSL

Per poter essere usato nel servizio app, il certificato deve soddisfare tutti i requisiti seguenti:

- Firmato da un'autorità di certificazione attendibile.

- Esportato come file PFX protetto da password.

- Contiene una chiave privata con una lunghezza minima di 2048 bit.

- Contiene tutti i certificati intermedi nella catena di certificati.

> [!Note]  
> I **certificati di crittografia a curva ellittica (ecc)** funzionano con il servizio app, ma non sono inclusi in questa guida. Per assistenza nella creazione di certificati ECC, consultare un'autorità di certificazione.

#### <a name="prepare-the-web-app"></a>Preparare l'app Web

Per associare un certificato SSL personalizzato all'app Web, il [piano di servizio app](https://azure.microsoft.com/pricing/details/app-service/) deve essere nel livello **Basic**, **standard**o **Premium** .

#### <a name="sign-in-to-azure"></a>Accedere ad Azure

1. Aprire il [portale di Azure](https://portal.azure.com/) e passare all'app Web.

2. Nel menu a sinistra selezionare **Servizi app**e quindi selezionare il nome dell'app Web.

![Selezionare l'app Web in portale di Azure](media/solution-deployment-guide-geo-distributed/image33.png)

#### <a name="check-the-pricing-tier"></a>Scegliere il piano tariffario

1. Nella finestra di spostamento a sinistra della pagina dell'app Web scorrere fino alla sezione **Impostazioni** e selezionare **scalabilità verticale (piano di servizio app)**.

    ![Menu di scalabilità verticale nell'app Web](media/solution-deployment-guide-geo-distributed/image34.png)

1. Assicurarsi che l'app Web non sia **inclusa** nel livello gratuito o **condiviso** . Il livello corrente dell'app Web è evidenziato in una casella blu scuro.

    ![Controllare il piano tariffario nell'app Web](media/solution-deployment-guide-geo-distributed/image35.png)

SSL personalizzato non è supportato nel livello **gratuito** o **condiviso** . Per eseguire la scalabilità, seguire i passaggi nella sezione successiva o nella pagina scegliere il piano **tariffario** e passare a [caricare e associare il certificato SSL](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-ssl).

#### <a name="scale-up-your-app-service-plan"></a>Passare a un piano di servizio app superiore

1. Selezionare uno tra i livelli **Basic**, **Standard** o **Premium**.

2. Scegliere **Seleziona**.

![Scegliere il piano tariffario per l'app Web](media/solution-deployment-guide-geo-distributed/image36.png)

L'operazione di ridimensionamento viene completata quando viene visualizzata la notifica.

![Notifica di passaggio al livello superiore](media/solution-deployment-guide-geo-distributed/image37.png)

#### <a name="bind-your-ssl-certificate-and-merge-intermediate-certificates"></a>Associare il certificato SSL e unire i certificati intermedi

Unire più certificati nella catena.

1. **Aprire ogni certificato** ricevuto in un editor di testo.

2. Creare un file per il certificato Unito denominato *mergedcertificate. CRT*. In un editor di testo copiare il contenuto di ogni certificato nel file. L'ordine dei certificati deve corrispondere all'ordine nella catena di certificati, che inizia con il certificato dell'utente e termina con il certificato radice. Sarà simile a quanto indicato nell'esempio seguente:

    ```Text

    -----BEGIN CERTIFICATE-----

    <your entire Base64 encoded SSL certificate>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded intermediate certificate 1>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded intermediate certificate 2>

    -----END CERTIFICATE-----

    -----BEGIN CERTIFICATE-----

    <The entire Base64 encoded root certificate>

    -----END CERTIFICATE-----
    ```

#### <a name="export-certificate-to-pfx"></a>Esportare il certificato in un file PFX

Esportare il certificato SSL Unito con la chiave privata generata dal certificato.

Un file di chiave privata viene creato tramite OpenSSL. Per esportare il certificato in PFX, eseguire il comando seguente e sostituire i segnaposto `<private-key-file>` e `<merged-certificate-file>` con il percorso della chiave privata e il file di certificato Unito:

```powershell
openssl pkcs12 -export -out myserver.pfx -inkey <private-key-file> -in <merged-certificate-file>
```

Quando richiesto, definire una password di esportazione per il caricamento del certificato SSL nel servizio app in un secondo momento.

Quando si usano IIS o **Certreq.exe** per generare la richiesta di certificato, installare il certificato in un computer locale e quindi [esportare il certificato in pfx](https://technet.microsoft.com/library/cc754329(v=ws.11).aspx).

#### <a name="upload-the-ssl-certificate"></a>Caricare il certificato SSL

1. Selezionare **Impostazioni SSL** nel percorso di spostamento sinistro dell'app Web.

2. Selezionare **Carica certificato**.

3. In **file di certificato PFX**selezionare file PFX.

4. In **password certificato**Digitare la password creata durante l'esportazione del file PFX.

5. Selezionare **Carica**.

    ![Caricare il certificato SSL](media/solution-deployment-guide-geo-distributed/image38.png)

Al termine del caricamento del certificato, il servizio app viene visualizzato nella pagina **Impostazioni SSL** .

![Impostazioni SSL](media/solution-deployment-guide-geo-distributed/image39.png)

#### <a name="bind-your-ssl-certificate"></a>Associare il certificato SSL

1. Nella sezione **binding SSL** selezionare **Aggiungi binding**.

    > [!Note]  
    >  Se il certificato è stato caricato, ma non viene visualizzato nei nomi di dominio nell'elenco a discesa nome **host** , provare ad aggiornare la pagina del browser.

2. Nella pagina **Aggiungi binding SSL** usare gli elenchi a discesa per selezionare il nome di dominio da proteggere e il certificato da usare.

3. In **Tipo SSL** selezionare se usare l'SSL basato su [**indicazione nome server (SNI)**](https://en.wikipedia.org/wiki/Server_Name_Indication) o basato su IP.

    - **SSL basato su SNI**: è possibile aggiungere più associazioni SSL basate su SNI. Questa opzione consente di usare più certificati SSL per proteggere più domini nello stesso indirizzo IP. La maggior parte dei browser moderni (tra cui Internet Explorer, Chrome, Firefox e Opera) supporta SNI. Per altre informazioni sul supporto dei browser, vedere [Indicazione nome server](https://wikipedia.org/wiki/Server_Name_Indication).

    - **SSL basato su IP**: è possibile aggiungere una sola associazione SSL basata su IP. Questa opzione consente di usare solo un certificato SSL per proteggere un indirizzo IP pubblico dedicato. Per proteggere più domini, è necessario proteggerli tutti utilizzando lo stesso certificato SSL. Il protocollo SSL basato su IP è l'opzione tradizionale per l'associazione SSL.

4. Selezionare **Aggiungi binding**.

    ![Aggiungere il binding SSL](media/solution-deployment-guide-geo-distributed/image40.png)

Al termine del caricamento del certificato, il servizio app viene visualizzato nelle sezioni **associazioni SSL** .

![Il caricamento delle associazioni SSL è stato completato](media/solution-deployment-guide-geo-distributed/image41.png)

#### <a name="remap-the-a-record-for-ip-ssl"></a>Modificare il mapping del record A per IP SSL

Se l'SSL basato su IP non viene usato nell'app Web, passare a [testare HTTPS per il dominio personalizzato](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-ssl).

Per impostazione predefinita, l'app Web usa un indirizzo IP pubblico condiviso. Quando il certificato è associato a SSL basato su IP, il servizio app crea un nuovo indirizzo IP dedicato per l'app Web.

Quando viene eseguito il mapping di un record a all'app Web, è necessario aggiornare il registro di sistema con l'indirizzo IP dedicato.

La pagina **dominio personalizzato** viene aggiornata con il nuovo indirizzo IP dedicato. Copiare questo [indirizzo IP](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-domain), quindi modificare il mapping del [record](https://docs.microsoft.com/azure/app-service/app-service-web-tutorial-custom-domain) a in questo nuovo indirizzo IP.

#### <a name="test-https"></a>Testare HTTPS

In diversi browser passare a `https://<your.custom.domain>` per assicurarsi che l'app Web venga servita.

![passare ad app Web](media/solution-deployment-guide-geo-distributed/image42.png)

> [!Note]  
> Se si verificano errori di convalida del certificato, è possibile che venga generato un certificato autofirmato oppure che i certificati intermedi siano stati lasciati durante l'esportazione nel file PFX.

#### <a name="enforce-https"></a>Imporre HTTPS

Per impostazione predefinita, chiunque può accedere all'app Web tramite HTTP. È possibile reindirizzare tutte le richieste HTTP alla porta HTTPS.

Nella pagina app Web selezionare **Impostazioni SL**. Quindi in **Solo HTTPS**, selezionare **Attiva**.

![Applicare HTTPS](media/solution-deployment-guide-geo-distributed/image43.png)

Al termine dell'operazione, passare a uno degli URL HTTP che puntano all'app. Ad esempio:

- https://<app_name>. azurewebsites.net
- `https://northwindcloud.com`
- `https://www.northwindcloud.com`

#### <a name="enforce-tls-1112"></a>Applicare TLS 1.1/1.2

Per impostazione predefinita, l'app consente [TLS](https://wikipedia.org/wiki/Transport_Layer_Security) 1,0, che non è più considerato sicuro dagli standard di settore, ad esempio [PCI DSS](https://wikipedia.org/wiki/Payment_Card_Industry_Data_Security_Standard). Per applicare versioni di TLS più recenti, seguire questa procedura:

1. Nel spostamento a sinistra della pagina dell'app Web selezionare **Impostazioni SSL**.

2. In **versione TLS**selezionare la versione minima di TLS.

    ![Applicare TLS 1.1 o 1.2](media/solution-deployment-guide-geo-distributed/image44.png)

### <a name="create-a-traffic-manager-profile"></a>Creare un profilo di Gestione traffico

1. Selezionare **Crea una risorsa**  >  **rete**  >  **profilo di gestione traffico**  >  **Crea**.

2. In **Crea profilo di Gestione traffico** procedere come segue:

    1. In **nome**specificare un nome per il profilo. Questo nome deve essere univoco all'interno della zona manager.net del traffico e restituisce il nome DNS, trafficmanager.net, usato per accedere al profilo di gestione traffico.

    2. In **metodo di routing**selezionare il **metodo di routing geografico**.

    3. In **sottoscrizione**selezionare la sottoscrizione in cui creare il profilo.

    4. In **Gruppo di risorse** creare un nuovo gruppo di risorse in cui aggiungere il profilo.

    5. In **Località del gruppo di risorse** selezionare la località del gruppo di risorse. Questa impostazione si riferisce al percorso del gruppo di risorse e non ha alcun effetto sul profilo di gestione traffico distribuito a livello globale.

    6. Selezionare **Crea**.

    7. Una volta completata la distribuzione globale del profilo di gestione traffico, questa viene elencata nel rispettivo gruppo di risorse come una delle risorse.

        ![Gruppi di risorse in Crea profilo di gestione traffico](media/solution-deployment-guide-geo-distributed/image45.png)

### <a name="add-traffic-manager-endpoints"></a>Aggiungere endpoint di Gestione traffico

1. Nella barra di ricerca del portale cercare il nome del **profilo di gestione traffico** creato nella sezione precedente e selezionare il profilo di gestione traffico nei risultati visualizzati.

2. Nella sezione **Impostazioni** del **profilo di gestione traffico**selezionare **endpoint**.

3. Selezionare **Aggiungi**.

4. Aggiunta dell'endpoint dell'hub Azure Stack.

5. Per **tipo**selezionare **endpoint esterno**.

6. Specificare un **nome** per l'endpoint, idealmente il nome dell'hub Azure stack.

7. Per nome di dominio completo (**FQDN**) usare l'URL esterno per l'app Web Hub Azure stack.

8. In mapping geografico selezionare un'area/continente in cui si trova la risorsa. Ad esempio, **Europa.**

9. Nell'elenco a discesa paese/area geografica visualizzato selezionare il paese che si applica a questo endpoint. Ad esempio, **Germania**.

10. Mantenere deselezionata l'opzione **Aggiungi come disabilitato**.

11. Selezionare **OK**.

12. Aggiunta dell'endpoint di Azure:

    1. Per **tipo**selezionare **endpoint di Azure**.

    2. Consente di specificare un **nome** per l'endpoint.

    3. Per **tipo di risorsa di destinazione**selezionare **servizio app**.

    4. Per **risorsa di destinazione**selezionare **Scegli un servizio app** per visualizzare l'elenco delle app Web nella stessa sottoscrizione. In **risorsa**selezionare il servizio app usato come primo endpoint.

13. In mapping geografico selezionare un'area/continente in cui si trova la risorsa. Ad esempio, **America del Nord/America Centrale/Caraibi.**

14. Nell'elenco a discesa paese/area geografica che viene visualizzato lasciare vuoto questo punto per selezionare tutto il raggruppamento a livello di area precedente.

15. Mantenere deselezionata l'opzione **Aggiungi come disabilitato**.

16. Selezionare **OK**.

    > [!Note]  
    >  Creare almeno un endpoint con un ambito geografico di All (World) per fungere da endpoint predefinito per la risorsa.

17. Quando l'aggiunta di entrambi gli endpoint è completa, vengono visualizzati nel **profilo di gestione traffico** insieme al relativo stato di monitoraggio **online**.

    ![Stato dell'endpoint del profilo di gestione traffico](media/solution-deployment-guide-geo-distributed/image46.png)

#### <a name="global-enterprise-relies-on-azure-geo-distribution-capabilities"></a>Global Enterprise si basa sulle funzionalità di distribuzione geografica di Azure

L'indirizzamento del traffico dati tramite gli endpoint di gestione traffico di Azure e di geografia consente alle aziende globali di rispettare le normative regionali e di mantenere i dati conformi e sicuri, il che è fondamentale per il successo di posizioni aziendali locali e remote.

## <a name="next-steps"></a>Passaggi successivi

- Per altre informazioni sui modelli cloud di Azure, vedere [modelli di progettazione cloud](https://docs.microsoft.com/azure/architecture/patterns).
