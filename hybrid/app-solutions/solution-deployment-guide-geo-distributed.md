---
title: Indirizzare il traffico con un'app con distribuzione geografica usando Azure e l'hub di Azure Stack
description: Informazioni su come indirizzare il traffico a endpoint specifici tramite una soluzione app con distribuzione geografica usando Azure e l'hub di Azure Stack.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 27d07070becfa902a715b451baae7c81c7e4b46f
ms.sourcegitcommit: 56980e3c118ca0a672974ee3835b18f6e81b6f43
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 08/26/2020
ms.locfileid: "88886833"
---
# <a name="direct-traffic-with-a-geo-distributed-app-using-azure-and-azure-stack-hub"></a>Indirizzare il traffico con un'app con distribuzione geografica usando Azure e l'hub di Azure Stack

Informazioni su come indirizzare il traffico a endpoint specifici in base a diverse metriche usando il criterio delle app con distribuzione geografica. La creazione di un profilo di Gestione traffico con routing geografico e configurazione degli endpoint garantisce che le informazioni vengano indirizzate agli endpoint in base ai requisiti internazionali, alle normative aziendali e internazionali e alle esigenze a livello di dati.

In questa soluzione si compilerà un ambiente di esempio per:

> [!div class="checklist"]
> - Creare un'app con distribuzione geografica.
> - Usare Gestione traffico per definire la destinazione dell'app.

## <a name="use-the-geo-distributed-apps-pattern"></a>Usare il criterio delle app con distribuzione geografica

Con il criterio di distribuzione geografica, l'app può raggiungere più aree geografiche. È possibile usare come impostazione predefinita il cloud pubblico, ma alcuni utenti potrebbero richiedere che i loro dati risiedano nella loro area geografica. È possibile indirizzare gli utenti al cloud più adatto ai loro requisiti.

### <a name="issues-and-considerations"></a>Considerazioni e problemi

#### <a name="scalability-considerations"></a>Considerazioni sulla scalabilità

La soluzione che verrà creata in questo articolo non prevede l'adattamento in base alla scalabilità. Se tuttavia viene usata in combinazione con altre soluzioni Azure e locali, può soddisfare requisiti di scalabilità. Per informazioni sulla creazione di una soluzione ibrida con scalabilità automatica tramite Gestione traffico, vedere [Distribuire un'app scalabile tra cloud usando Azure e l'hub di Azure Stack](solution-deployment-guide-cross-cloud-scaling.md).

#### <a name="availability-considerations"></a>Considerazioni sulla disponibilità

Come nel caso delle considerazioni sulla scalabilità, questa soluzione non risolve direttamente la disponibilità. Tuttavia è possibile implementare al suo interno soluzioni Azure e locali, per garantire la disponibilità elevata per tutti i componenti interessati.

### <a name="when-to-use-this-pattern"></a>Quando usare questo modello

- L'organizzazione dispone di filiali internazionali che richiedono criteri personalizzati per la sicurezza e la distribuzione nelle aree specifiche.

- Ogni ufficio dell'organizzazione esegue il pull dei dati relativi a dipendenti, attività e strutture, pertanto è necessaria un'attività di creazione di report basata sulle normative e sui fusi orari locali.

- I requisiti di scalabilità elevata vengono soddisfatti con la scalabilità orizzontale delle applicazioni e con più distribuzioni di app in una singola area geografica e in più aree, al fine di gestire i requisiti di carico particolarmente elevati.

### <a name="planning-the-topology"></a>Pianificazione della topologia

Prima di creare un footprint per le app distribuite, è utile sapere quanto segue:

- **Dominio personalizzato per l'app:** nome di dominio personalizzato che i clienti useranno per accedere all'app. Per l'app di esempio il nome di dominio personalizzato è *www\.scalableasedemo.com*.

- **Dominio di Gestione traffico:** quando si crea un [profilo di Gestione traffico di Azure](/azure/traffic-manager/traffic-manager-manage-profiles) è necessario scegliere un nome di dominio. Questo nome viene combinato con il suffisso *trafficmanager.net* per registrare una voce di dominio gestita da Gestione traffico. Per l'app di esempio il nome scelto è *scalable-ase-demo*. Il nome di dominio completo gestito da Gestione traffico sarà quindi *scalable-ase-demo.trafficmanager.net*.

- **Strategia per la scalabilità del footprint dell'app:** decidere se il footprint dell'app verrà distribuito tra più ambienti del servizio app in una singola area geografica, in più aree o con una combinazione di entrambi gli approcci. La decisione dovrà basarsi sulla previsione dell'origine del traffico dei clienti, oltre che sull'efficacia della scalabilità nel resto dell'infrastruttura di back-end che supporta un'app. Ad esempio, un'app senza stato al 100% può essere ridimensionata notevolmente usando una combinazione di più ambienti del servizio app per ogni area di Azure, moltiplicati per gli ambienti del servizio app distribuiti tra più aree di Azure. Con oltre 15 aree di Azure globali tra cui scegliere, i clienti possono creare un footprint dell'app iperscalabile in tutto il mondo. Per l'app di esempio usata qui sono stati creati tre ambienti del servizio app in una singola area di Azure (Stati Uniti centro-meridionali).

- **Convenzione di denominazione per gli ambienti del servizio app:** ogni ambiente del servizio app richiede un nome univoco. Se si hanno più di due ambienti del servizio app, è utile implementare una convenzione di denominazione per facilitare l'identificazione di ogni ambiente del servizio app. Per l'app di esempio usata qui è stata usata una convenzione di denominazione semplice. I nomi dei tre ambienti del Servizio app di Azure sono *fe1ase*, *fe2ase* e *fe3ase*.

- **Convenzione di denominazione per le app:** poiché verranno distribuite più istanze dell'app, è necessario definire un nome per ogni istanza dell'app distribuita. Con l'ambiente del servizio app per Power Apps, lo stesso nome di app può essere usato in più ambienti. Poiché ogni ambiente del servizio app ha un suffisso univoco, gli sviluppatori possono scegliere di riusare esattamente lo stesso nome dell'app in ogni ambiente. Ad esempio, uno sviluppatore può avere app denominate come segue: *myapp.foo1.p.azurewebsites.net*, *myapp.foo2.p.azurewebsites.net*, *myapp.foo3.p.azurewebsites.net* e così via. Per l'app usata nel presente contesto, ogni istanza dell'app ha un nome univoco. I nomi delle istanze dell'app usati sono *webfrontend1*, *webfrontend2* e *webfrontend3*.

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> L'hub di Azure Stack è un'estensione di Azure che offre all'ambiente locale l'agilità e l'innovazione del cloud computing, abilitando l'unico cloud ibrido che consente di creare e distribuire ovunque app ibride.  
> 
> L'articolo [Considerazioni per la progettazione di app ibride](overview-app-design-considerations.md) esamina i concetti fondamentali di qualità del software (posizionamento, scalabilità, disponibilità, resilienza, gestibilità e sicurezza) per la progettazione, la distribuzione e la gestione di app ibride. Le considerazioni di progettazione consentono di ottimizzare la progettazione delle app ibride, riducendo al minimo i rischi negli ambienti di produzione.

## <a name="part-1-create-a-geo-distributed-app"></a>Parte 1: Creare un'app con distribuzione geografica

In questa parte si creerà un'app Web.

> [!div class="checklist"]
> - Creare e pubblicare app Web.
> - Aggiungere codice ad Azure Repos.
> - Indirizzare la compilazione dell'app su più destinazioni cloud.
> - Gestire e configurare il processo di recapito continuo.

### <a name="prerequisites"></a>Prerequisiti

Sono necessarie una sottoscrizione di Azure e l'installazione dell'hub di Azure Stack.

### <a name="geo-distributed-app-steps"></a>Procedura per app con distribuzione geografica

### <a name="obtain-a-custom-domain-and-configure-dns"></a>Ottenere un dominio personalizzato e configurare il DNS

Aggiornare il file della zona DNS per il dominio. Azure AD può quindi verificare la proprietà del nome di dominio personalizzato. Usare il [DNS di Azure](/azure/dns/dns-getstarted-portal) per i record DNS di Azure/Microsoft 365/esterni in Azure oppure aggiungere la voce DNS in un [registrar DNS diverso](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider).

1. Registrare un dominio personalizzato con un registrar pubblico.

2. Accedere al registrar per il dominio. Per eseguire gli aggiornamenti al DNS potrebbe essere necessario un amministratore approvato.

3. Aggiornare il file di zona DNS per il dominio aggiungendo la voce DNS specificata da Azure AD. La voce DNS non modifica comportamenti come il routing della posta elettronica o l'hosting Web.

### <a name="create-web-apps-and-publish"></a>Creare e pubblicare app Web

Configurare l'integrazione continua e il recapito continuo (CI/CD) ibridi per distribuire l'app Web in Azure e nell'hub di Azure Stack e per eseguire il push automatico delle modifiche in entrambi i cloud.

> [!Note]  
> Sono necessari l'hub di Azure Stack con immagini diffuse appropriate per l'esecuzione (Windows Server e SQL) e la distribuzione del servizio app. Per altre informazioni, vedere [Prerequisiti per la distribuzione del servizio app nell'hub di Azure Stack](/azure-stack/operator/azure-stack-app-service-before-you-get-started.md).

#### <a name="add-code-to-azure-repos"></a>Aggiungere codice ad Azure Repos

1. Accedere a Visual Studio con un **account provvisto di diritti di creazione progetti** in Azure Repos.

    L'integrazione continua e il recapito continuo (CI/CD) possono essere applicati sia al codice dell'app sia al codice dell'infrastruttura. Usare [modelli di Azure Resource Manager](https://azure.microsoft.com/resources/templates/) sia per lo sviluppo cloud privato che per quello ospitato.

    ![Connettersi a un progetto in Visual Studio](media/solution-deployment-guide-geo-distributed/image1.JPG)

2. **Clonare il repository** creando e aprendo l'app Web predefinita.

    ![Clonare il repository in Visual Studio](media/solution-deployment-guide-geo-distributed/image2.png)

### <a name="create-web-app-deployment-in-both-clouds"></a>Creare una distribuzione app Web in entrambi i cloud

1. Modificare il file **WebApplication.csproj**: Selezionare `Runtimeidentifier` e aggiungere `win10-x64`. Vedere la documentazione [Distribuzione autonoma](/dotnet/core/deploying/deploy-with-vs#simpleSelf).

    ![Modificare il file di progetto dell'app Web in Visual Studio](media/solution-deployment-guide-geo-distributed/image3.png)

2. **Archiviare il codice in Azure Repos** usando Team Explorer.

3. Verificare che il **codice dell'applicazione** sia stato archiviato in Azure Repos.

### <a name="create-the-build-definition"></a>Creare la definizione di compilazione

1. **Accedere ad Azure Pipelines** per verificare di poter creare definizioni di compilazione.

2. Aggiungere il codice `-r win10-x64`. Questa aggiunta è necessaria per attivare una distribuzione autonoma con .NET Core.

    ![Aggiungere codice alla definizione di compilazione in Azure Pipelines](media/solution-deployment-guide-geo-distributed/image4.png)

3. **Eseguire la compilazione**. Il processo di [compilazione della distribuzione autonoma](/dotnet/core/deploying/deploy-with-vs#simpleSelf) pubblicherà gli artefatti che possono essere eseguiti in Azure e nell'hub di Azure Stack.

#### <a name="using-an-azure-hosted-agent"></a>Uso di un agente ospitato di Azure

L'uso di un agente ospitato in Azure Pipelines è un'opzione utile per compilare e distribuire le app Web. La manutenzione e gli aggiornamenti vengono eseguiti automaticamente da Microsoft Azure, e questo consente di eseguire sviluppo, test e distribuzione senza interruzioni.

### <a name="manage-and-configure-the-cd-process"></a>Gestire e configurare il processo di distribuzione continua

Azure DevOps Services offre una pipeline altamente configurabile e facilmente gestibile per i rilasci in più ambienti (ad esempio gli ambienti di sviluppo, gestione temporanea, controllo qualità e produzione) e include la richiesta di approvazioni in fasi specifiche.

## <a name="create-release-definition"></a>Creare una definizione della versione

1. Selezionare il **pulsante con il segno più** per aggiungere una nuova versione nella scheda **Releases** (Rilasci) della sezione **Build and Release** (Compilazione e versione) di Azure DevOps Services.

    ![Creare una definizione di versione in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image5.png)

2. Applicare il modello di distribuzione del Servizio app di Azure.

   ![Applicare il modello di distribuzione del Servizio app di Azure in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image6.png)

3. In **Aggiungi artefatto** aggiungere l'artefatto corrispondente all'app della build del cloud di Azure.

   ![Aggiungere un elemento alla build del cloud di Azure in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image7.png)

4. Nella scheda Pipeline selezionare il collegamento **Phase, Task** (Fase, Attività) dell'ambiente e impostare i valori di ambiente del cloud di Azure.

   ![Impostare i valori dell'ambiente cloud di Azure in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image8.png)

5. Impostare il **nome dell'ambiente** e selezionare la **sottoscrizione di Azure** per l'endpoint cloud di Azure.

      ![Selezionare la sottoscrizione di Azure per l'endpoint cloud di Azure in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image9.png)

6. In **Nome del servizio app** impostare il nome del servizio app di Azure richiesto.

      ![Impostare il nome del servizio app di Azure in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image10.png)

7. Immettere "Hosted VS2017" in **Agent queue** (Coda agente) per l'ambiente ospitato del cloud di Azure.

      ![Impostare la coda agente per l'ambiente ospitato del cloud di Azure in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image11.png)

8. Nel menu Deploy Azure App Service (Distribuisci servizio app di Azure) selezionare l'opzione **Package or Folder** (Pacchetto o cartella) per l'ambiente. Selezionare **OK** per **folder location** (Percorso cartella).
  
      ![Selezionare il pacchetto o la cartella dell'ambiente Servizio app di Azure in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image12.png)

      ![Selezionare il pacchetto o la cartella dell'ambiente servizio app di Azure in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image13.png)

9. Salvare tutte le modifiche e tornare alla **pipeline di versione**.

    ![Salvare le modifiche nella pipeline di versione in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image14.png)

10. Aggiungere un nuovo artefatto selezionando la build dell'app hub di Azure Stack.

    ![Aggiungere un nuovo artefatto per l'hub di Azure Stack in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image15.png)


11. Aggiungere un altro ambiente applicando la Distribuzione Servizio app di Azure.

    ![Aggiungere un ambiente a Distribuzione servizio app di Azure in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image16.png)

12. Assegnare al nuovo ambiente il nome Azure Stack Hub.

    ![Assegnare un nome all'ambiente in Distribuzione servizio app di Azure in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image17.png)

13. Trovare l'ambiente Azure Stack Hub nella scheda **Tasks** (Attività).

    ![Ambiente Azure Stack Hub in Azure DevOps Services in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image18.png)

14. Selezionare la sottoscrizione per l'endpoint dell'hub di Azure Stack.

    ![Selezionare la sottoscrizione per l'endpoint dell'hub di Azure Stack in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image19.png)

15. Impostare il nome dell'app Web Azure Stack Hub come nome del servizio app.

    ![Impostare il nome dell'app Web Azure Stack Hub in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image20.png)

16. Selezionare l'agente dell'hub di Azure Stack.

    ![Selezionare l'agente dell'hub di Azure Stack in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image21.png)

17. Nella sezione Deploy Azure App Service (Distribuisci servizio app di Azure) selezionare l'opzione **Package or Folder** (Pacchetto o cartella) valida per l'ambiente. Selezionare **OK** per il percorso della cartella.

    ![Selezionare la cartella per Distribuzione Servizio app di Azure in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image22.png)

    ![Selezionare la cartella per Distribuzione Servizio app di Azure in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image23.png)

18. Nella scheda Variable (Variabile) aggiungere una variabile con nome `VSTS\_ARM\_REST\_IGNORE\_SSL\_ERRORS`, impostare il valore come **true** e l'hub di Azure Stack come ambito.

    ![Aggiungere una variabile a Distribuzione Servizio app di Azure in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image24.png)

19. Selezionare l'icona del trigger di distribuzione **Continuous** (Continua) in entrambi gli artefatti e abilitare il trigger di distribuzione **Continues** (Continua).

    ![Selezionare il trigger di distribuzione Continuous (Continua) in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image25.png)

20. Selezionare l'icona delle condizioni **Pre-deployment** (Pre-distribuzione) nell'ambiente dell'hub di Azure Stack e impostare il trigger su **Dopo la versione**.

    ![Selezionare condizioni pre-distribuzione in Azure DevOps Services](media/solution-deployment-guide-geo-distributed/image26.png)

21. Salvare tutte le modifiche.

> [!Note]  
> Alcune impostazioni per le attività potrebbero essere state definite automaticamente come [variabili di ambiente](/azure/devops/pipelines/release/variables?tabs=batch&view=vsts#custom-variables) durante la creazione di una definizione di versione da un modello. Queste impostazioni non possono essere modificate nelle impostazioni dell'attività. Per modificarle è invece necessario selezionare l'elemento dell'ambiente padre.

## <a name="part-2-update-web-app-options"></a>Parte 2: Aggiornare le opzioni dell'app Web

[Servizio app di Azure](/azure/app-service/overview) offre un servizio di hosting Web con scalabilità elevata e funzioni di auto-correzione.

![Servizio app di Azure](media/solution-deployment-guide-geo-distributed/image27.png)

> [!div class="checklist"]
> - Eseguire il mapping di un nome DNS personalizzato esistente ad app Web di Azure.
> - Usare un **record CNAME** e un **record A** per eseguire il mapping di un nome DNS personalizzato al servizio app.

### <a name="map-an-existing-custom-dns-name-to-azure-web-apps"></a>Esecuzione del mapping di un nome DNS personalizzato esistente con un app Web di Azure

> [!Note]  
> Usare un record CNAME per tutti i nomi DNS personalizzati, tranne che per il dominio radice (ad esempio northwind.com).

Per eseguire la migrazione di un sito in tempo reale e del relativo nome di dominio DNS al servizio app, vedere [Migrare un nome DNS attivo nel servizio app di Azure](/azure/app-service/manage-custom-dns-migrate-domain).

### <a name="prerequisites"></a>Prerequisiti

Per completare questa soluzione:

- [Creare un'app del servizio app](/azure/app-service/) oppure usare un'app creata per un'altra soluzione.

- Acquistare un nome di dominio e verificare l'accesso al registro DNS per il provider di dominio.

Aggiornare il file della zona DNS per il dominio. Azure AD verifica quindi la proprietà del nome di dominio personalizzato. Usare il [DNS di Azure](/azure/dns/dns-getstarted-portal) per i record DNS di Azure/Microsoft 365/esterni in Azure oppure aggiungere la voce DNS in un [registrar DNS diverso](/microsoft-365/admin/get-help-with-domains/create-dns-records-at-any-dns-hosting-provider).

- Registrare un dominio personalizzato con un registrar pubblico.

- Accedere al registrar per il dominio. Per eseguire gli aggiornamenti al DNS potrebbe essere necessario un amministratore approvato.

- Aggiornare il file di zona DNS per il dominio aggiungendo la voce DNS specificata da Azure AD.

Ad esempio, per aggiungere le voci DNS per northwindcloud.com e www\.northwindcloud.com, configurare le impostazioni DNS per il dominio radice northwindcloud.com.

> [!Note]  
> È possibile acquistare un nome di dominio usando il [portale di Azure](/azure/app-service/manage-custom-dns-buy-domain). Per eseguire il mapping di un nome DNS personalizzato a un'app Web, è necessario che il [piano di servizio app](https://azure.microsoft.com/pricing/details/app-service/) dell'app Web sia un livello a pagamento (**Condiviso**, **Base**, **Standard** o **Premium**).

### <a name="create-and-map-cname-and-a-records"></a>Creare ed eseguire il mapping di record CNAME e A

#### <a name="access-dns-records-with-domain-provider"></a>Accedere ai record DNS con il provider di dominio

> [!Note]  
>  È possibile usare DNS di Azure per configurare un nome DNS personalizzato per le app Web di Azure. Per altre informazioni, vedere [Usare il servizio DNS di Azure per specificare impostazioni di dominio personalizzate per un servizio di Azure](/azure/dns/dns-custom-domain).

1. Accedere al sito Web del provider principale.

2. Individuare la pagina relativa alla gestione dei record DNS. Ogni provider di dominio ha una propria interfaccia di record DNS. Cercare le aree del sito denominate **Domain Name** (Nome di dominio), **DNS** o **Name Server Management** (Gestione server dei nomi).

È possibile visualizzare la pagina dei record DNS in **My domains** (Domini personali). Trovare il collegamento con nome **File di zona**, **Record DNS** o **Configurazione avanzata**.

La schermata seguente è un esempio di pagina di record DNS:

![Pagina record DNS di esempio](media/solution-deployment-guide-geo-distributed/image28.png)

1. In Domain Name Registrar (Nome registrar del dominio) selezionare **Add or Create** (Aggiungi o crea) per creare un record. Per alcuni provider esistono collegamenti diversi per aggiungere tipi di record diversi. Vedere la documentazione del provider.

2. Aggiungere un record CNAME per eseguire il mapping di un sottodominio al nome host predefinito dell'app.

   Per l'esempio di dominio www\.northwindcloud.com, aggiungere un record CNAME che esegue il mapping del nome a `<app_name>.azurewebsites.net`.

Dopo l'aggiunta del record CNAME la pagina dei record DNS è simile all'esempio seguente:

![Passaggio all'app di Azure nel portale](media/solution-deployment-guide-geo-distributed/image29.png)

### <a name="enable-the-cname-record-mapping-in-azure"></a>Abilitare il mapping dei record CNAME in Azure

1. In una nuova finestra accedere al portale di Azure.

2. Passare a Servizi app.

3. Selezionare l'app Web.

4. Nel riquadro di spostamento a sinistra della pagina dell'app nel portale di Azure selezionare **Domini personalizzati**.

5. Selezionare l'icona **+** accanto ad **Aggiungi il nome host**.

6. Digitare il nome di dominio completo, ad esempio `www.northwindcloud.com`.

7. Selezionare **Convalida**.

8. Se indicato, aggiungere altri record di altri tipi (`A` o `TXT`) ai record DNS del registrar del dominio. Azure specificherà i valori e i tipi di record seguenti:

   a.  Un record **A** di cui eseguire il mapping all'indirizzo IP dell'app.

   b.  Un record **TXT** di cui eseguire il mapping al nome host predefinito dell'app `<app_name>.azurewebsites.net`. Il servizio app usa questo record solo in fase di configurazione per verificare la proprietà del dominio personalizzato. Dopo la verifica, eliminare il record TXT.

9. Completare questa attività nella scheda del registrar del dominio e riconvalidare fino a quando non viene attivato il pulsante **Aggiungi il nome host**.

10. Assicurarsi che **Tipo di record del nome host** sia impostato su **CNAME** (www.example.com o qualsiasi sottodominio).

11. Selezionare **Aggiungi il nome host**.

12. Digitare il nome di dominio completo, ad esempio `northwindcloud.com`.

13. Selezionare **Convalida**. Viene attivato **Aggiungi**.

14. Assicurarsi che **Tipo di record del nome host** sia impostato su **Record A** (example.com).

15. **Aggiungere il nome host**.

    La visualizzazione dei nuovi nomi host nella pagina **Domini personalizzati** dell'app potrebbe richiedere qualche minuto. Provare ad aggiornare il browser per visualizzare i dati più recenti.
  
    ![Domini personalizzati](media/solution-deployment-guide-geo-distributed/image31.png) 
  
    Se si verifica un errore, nella parte inferiore della pagina viene segnalato un errore di verifica. ![Errore di verifica del dominio](media/solution-deployment-guide-geo-distributed/image32.png)

> [!Note]  
>  È possibile ripetere i passaggi precedenti per eseguire il mapping di un dominio con caratteri jolly (\*.northwindcloud.com). In questo modo è possibile aggiungere altri sottodomini al servizio app senza dover creare un record CNAME separato per ciascuno di essi. Per configurare questa impostazione, seguire le istruzioni del registrar.

#### <a name="test-in-a-browser"></a>Eseguire test in un browser

Passare al nome o ai nomi DNS configurati in precedenza, ad esempio `northwindcloud.com` o `www.northwindcloud.com`.

## <a name="part-3-bind-a-custom-ssl-cert"></a>Parte 3: Associare un certificato SSL personalizzato

In questa parte si eseguiranno le operazioni seguenti:

> [!div class="checklist"]
> - Associare il certificato SSL personalizzato al Servizio app di Azure.
> - Applicare HTTPS per l'app.
> - Automatizzare l'associazione dei certificati SSL con gli script.

> [!Note]  
> Se necessario, ottenere un certificato SSL del cliente nel portale di Azure e associarlo all'app Web. Per altre informazioni, vedere l'[esercitazione sui certificati del Servizio app](/azure/app-service/web-sites-purchase-ssl-web-site).

### <a name="prerequisites"></a>Prerequisiti

Per completare questa soluzione:

- [Creare un'app del Servizio app di Azure](/azure/app-service/).
- [Eseguire il mapping di un nome DNS personalizzato all'app Web.](/azure/app-service/app-service-web-tutorial-custom-domain)
- Acquisire un certificato SSL da un'autorità di certificazione attendibile e usare la chiave per firmare la richiesta.

### <a name="requirements-for-your-ssl-certificate"></a>Requisiti per il certificato SSL

Per poter essere usato nel servizio app, il certificato deve soddisfare tutti i requisiti seguenti:

- Deve essere firmato da un'autorità di certificazione attendibile.

- Deve essere esportato come file PFX protetto da password.

- Deve contenere una chiave privata costituita da almeno 2048 bit.

- Deve contenere tutti i certificati intermedi nella catena di certificati.

> [!Note]  
> I **certificati di crittografia a curva ellittica (ECC)** funzionano con il Servizio app di Azure, ma non sono inclusi in questa guida. Per assistenza nella creazione di certificati ECC contattare un'autorità di certificazione.

#### <a name="prepare-the-web-app"></a>Preparare l'app Web

Per l'associazione di un certificato SSL personalizzato all'app Web, il [piano di servizio app](https://azure.microsoft.com/pricing/details/app-service/) in uso deve essere di livello **Basic**, **Standard** o **Premium**.

#### <a name="sign-in-to-azure"></a>Accedere ad Azure

1. Accedere al [portale di Azure](https://portal.azure.com/) e andare all'app Web.

2. Nel menu a sinistra selezionare **Servizi app** e quindi selezionare il nome dell'app Web.

![Selezionare l'app Web nel portale di Azure](media/solution-deployment-guide-geo-distributed/image33.png)

#### <a name="check-the-pricing-tier"></a>Scegliere il piano tariffario

1. Nel riquadro di spostamento a sinistra della pagina dell'app Web scorrere fino alla sezione **Impostazioni** e selezionare **Aumenta prestazioni (piano di servizio app)** .

    ![Menu Aumenta prestazioni nell'app Web](media/solution-deployment-guide-geo-distributed/image34.png)

1. Assicurarsi che l'app Web non appartenga al livello **Gratis** o **Condiviso**. Il livello corrente dell'app Web è evidenziato in una casella blu scuro.

    ![Controllare il piano tariffario nell'app Web](media/solution-deployment-guide-geo-distributed/image35.png)

Il certificato SSL personalizzato non è supportato nei livelli **Gratuito** o **Condiviso**. Per passare a un piano di servizio superiore, seguire i passaggi nella sezione successiva o nella pagina **Scegliere il piano tariffario** e passare a [Caricare il certificato SSL e Associare il certificato SSL](/azure/app-service/app-service-web-tutorial-custom-ssl).

#### <a name="scale-up-your-app-service-plan"></a>Passare a un piano di servizio app superiore

1. Selezionare uno tra i livelli **Basic**, **Standard** o **Premium**.

2. Scegliere **Seleziona**.

![Scegliere il piano tariffario per l'app Web.](media/solution-deployment-guide-geo-distributed/image36.png)

L'operazione di passaggio a un livello superiore viene completata quando viene visualizzata la notifica.

![Notifica di passaggio al livello superiore](media/solution-deployment-guide-geo-distributed/image37.png)

#### <a name="bind-your-ssl-certificate-and-merge-intermediate-certificates"></a>Associare il certificato SSL e unire i certificati intermedi

Unire più certificati nella catena.

1. **Aprire ogni certificato** ricevuto in un editor di testo.

2. Creare un file per il certificato unito denominato *mergedcertificate.crt*. In un editor di testo copiare il contenuto di ogni certificato nel file. L'ordine dei certificati deve corrispondere all'ordine nella catena di certificati, che inizia con il certificato dell'utente e termina con il certificato radice. Sarà simile a quanto indicato nell'esempio seguente:

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

Esportare il certificato SSL unito con la chiave privata generata dal certificato.

Un file di chiave privata viene creato tramite OpenSSL. Per esportare il certificato in un file con estensione pfx, eseguire il comando seguente e sostituire i segnaposto `<private-key-file>` e `<merged-certificate-file>` con il percorso della chiave privata e il file del certificato unito:

```powershell
openssl pkcs12 -export -out myserver.pfx -inkey <private-key-file> -in <merged-certificate-file>
```

Quando richiesto, definire una password di esportazione per il caricamento del certificato SSL nel servizio app in un secondo momento.

Quando si usa IIS o **Certreq.exe** per generare la richiesta di certificato, installare il certificato in un computer locale e quindi [esportarlo in un file con estensione pfx](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc754329(v=ws.11)).

#### <a name="upload-the-ssl-certificate"></a>Caricare il certificato SSL

1. Selezionare **Impostazioni SSL** nel riquadro di spostamento a sinistra dell'app Web.

2. Selezionare **Carica certificato**.

3. In **File del certificato PFX** selezionare il file con estensione pfx.

4. In **Password certificato** digitare la password creata durante l'esportazione del file con estensione pfx.

5. Selezionare **Carica**.

    ![Caricare il certificato SSL](media/solution-deployment-guide-geo-distributed/image38.png)

Dopo che il Servizio app ha terminato il caricamento del certificato, viene visualizzata la pagina **Impostazioni SSL**.

![Impostazioni SSL](media/solution-deployment-guide-geo-distributed/image39.png)

#### <a name="bind-your-ssl-certificate"></a>Associare il certificato SSL

1. Nella sezione **Associazioni SSL** selezionare **Aggiungi binding**.

    > [!Note]  
    >  Se il certificato è stato caricato ma non viene visualizzato nei nomi di dominio nell'elenco a discesa **Nome host**, provare ad aggiornare la pagina del browser.

2. Nel pannello **Aggiungi associazione SSL** usare gli elenchi a discesa per selezionare il nome di dominio da proteggere e il certificato da usare.

3. In **Tipo SSL** selezionare se usare l'SSL basato su [**indicazione nome server (SNI)** ](https://en.wikipedia.org/wiki/Server_Name_Indication) o basato su IP.

    - **SSL basato su SNI**: è possibile aggiungere più associazioni SSL basate su SNI. Questa opzione consente di usare più certificati SSL per proteggere più domini nello stesso indirizzo IP. La maggior parte dei browser moderni (tra cui Internet Explorer, Chrome, Firefox e Opera) supporta SNI. Per altre informazioni sul supporto dei browser, vedere [Indicazione nome server](https://wikipedia.org/wiki/Server_Name_Indication).

    - **SSL basato su IP**: è possibile aggiungere una sola associazione SSL basata su IP. Questa opzione consente di usare solo un certificato SSL per proteggere un indirizzo IP pubblico dedicato. Se si proteggono più domini, proteggerli tutti usando lo stesso certificato SSL. SSL basato su IP è l'opzione standard per il binding SSL.

4. Selezionare **Aggiungi binding**.

    ![Aggiungere il binding SSL](media/solution-deployment-guide-geo-distributed/image40.png)

Al termine del caricamento da parte del Servizio app, il certificato viene visualizzato nella sezione **Binding SSL**.

![Caricamento dei binding SSL completato](media/solution-deployment-guide-geo-distributed/image41.png)

#### <a name="remap-the-a-record-for-ip-ssl"></a>Nuovo mapping del record A per IP SSL

Se non si usa SSL basato su IP nell'app Web, passare a [Testare HTTPS per il dominio personalizzato](/azure/app-service/app-service-web-tutorial-custom-ssl).

Per impostazione predefinita l'app Web usa un indirizzo IP pubblico condiviso. Quando il certificato è associato a SSL basato su IP, il servizio app crea un nuovo indirizzo IP dedicato per l'app Web.

Quando viene eseguito il mapping di un record A all'app Web, è necessario aggiornare il registro del dominio con l'indirizzo IP dedicato.

La pagina **Dominio personalizzato** viene aggiornata con il nuovo indirizzo IP dedicato. Copiare questo [indirizzo IP](/azure/app-service/app-service-web-tutorial-custom-domain), quindi eseguire nuovamente il mapping del [record A](/azure/app-service/app-service-web-tutorial-custom-domain) su questo indirizzo IP.

#### <a name="test-https"></a>Testare HTTPS

In diversi browser, passare a `https://<your.custom.domain>` per assicurarsi che l'app Web funzioni correttamente.

![Passare all'app Web](media/solution-deployment-guide-geo-distributed/image42.png)

> [!Note]  
> Se si verificano errori di convalida del certificato, la causa può essere un certificato autofirmato oppure l'omissione di certificati intermedi durante l'esportazione nel file con estensione pfx.

#### <a name="enforce-https"></a>Applicare HTTPS

Per impostazione predefinita, chiunque può accedere all'app Web tramite HTTP. Tutte le richieste HTTP alla porta HTTPS possono essere reindirizzate.

Nella pagina dell'app Web selezionare **Impostazioni SSL**. Quindi in **Solo HTTPS**, selezionare **Attiva**.

![Applicare HTTPS](media/solution-deployment-guide-geo-distributed/image43.png)

Al termine dell'operazione, passare a uno degli URL HTTP che fanno riferimento all'applicazione. Ad esempio:

- https://<nome_app>.azurewebsites.net
- `https://northwindcloud.com`
- `https://www.northwindcloud.com`

#### <a name="enforce-tls-1112"></a>Applicare TLS 1.1/1.2

Per impostazione predefinita l'app supporta [TLS](https://wikipedia.org/wiki/Transport_Layer_Security) 1.0, che non è più considerato sicuro in base agli standard di settore come [PCI DSS](https://wikipedia.org/wiki/Payment_Card_Industry_Data_Security_Standard). Per applicare versioni di TLS più recenti, seguire questa procedura:

1. Nel riquadro di spostamento a sinistra della pagina dell'app Web selezionare **Impostazioni SSL**.

2. In **Versione TLS** selezionare la versione minima di TLS.

    ![Applicare TLS 1.1 o 1.2](media/solution-deployment-guide-geo-distributed/image44.png)

### <a name="create-a-traffic-manager-profile"></a>Creare un profilo di Gestione traffico

1. Selezionare **Crea una risorsa** > **Rete** > **Profilo di Gestione traffico** > **Crea**.

2. In **Crea profilo di Gestione traffico** procedere come segue:

    1. In **Nome** specificare un nome per il profilo. Questo nome deve essere univoco all'interno della zona trafficmanager.net e determina il nome DNS, trafficmanager.net, usato per accedere al profilo di Gestione traffico.

    2. In **Metodo di routing** selezionare il metodo di routing **Geografico**.

    3. In **Sottoscrizione** selezionare la sottoscrizione in cui si vuole creare il profilo.

    4. In **Gruppo di risorse** creare un nuovo gruppo di risorse in cui aggiungere il profilo.

    5. In **Località del gruppo di risorse** selezionare la località del gruppo di risorse. Questa impostazione indica la località del gruppo di risorse e non ha alcun impatto sul profilo di Gestione traffico distribuito a livello globale.

    6. Selezionare **Crea**.

    7. Dopo il completamento della distribuzione globale del profilo di Gestione traffico, il profilo viene elencato come risorsa nel rispettivo gruppo di risorse.

        ![Gruppi di risorse in Crea profilo di Gestione traffico](media/solution-deployment-guide-geo-distributed/image45.png)

### <a name="add-traffic-manager-endpoints"></a>Aggiungere endpoint di Gestione traffico

1. Nella barra di ricerca del portale cercare il nome del **profilo di Gestione traffico** creato nella sezione precedente e selezionarlo nei risultati visualizzati.

2. In **Profilo di Gestione traffico** nella sezione **Impostazioni** selezionare **Endpoint**.

3. Selezionare **Aggiungi**.

4. Aggiunta dell'endpoint dell'hub di Azure Stack.

5. In **Tipo** selezionare **Endpoint esterno**.

6. Specificare un **Nome** per l'endpoint, idealmente il nome dell'hub di Azure Stack.

7. Come nome di dominio completo (**FQDN**) usare l'URL esterno dell'app Web hub di Azure Stack.

8. In Mapping geografico selezionare l'area geografica/continente in cui si trova la risorsa. Ad esempio, **Europa**.

9. Nell'elenco a discesa Paese/area geografica visualizzato selezionare il paese corrispondente a questo endpoint. Ad esempio, **Germania**.

10. Mantenere deselezionata l'opzione **Aggiungi come disabilitato**.

11. Selezionare **OK**.

12. Aggiunta dell'endpoint di Azure:

    1. In **Tipo** selezionare **Endpoint Azure**.

    2. In **Nome** immettere un nome per l'endpoint.

    3. In **Tipo di risorsa di destinazione** fare clic su **Servizio app**.

    4. In **Risorsa di destinazione** selezionare **Scegliere un servizio app** per visualizzare l'elenco delle App Web nella stessa sottoscrizione. In **Risorsa** selezionare il servizio app usato come primo endpoint.

13. In Mapping geografico selezionare l'area geografica/continente in cui si trova la risorsa. Ad esempio **America del Nord/America centrale/Caraibi.**

14. Nell'elenco a discesa Paese/area geografica visualizzato non immettere nessun valore, per selezionare l'intero gruppo di area precedente.

15. Mantenere deselezionata l'opzione **Aggiungi come disabilitato**.

16. Selezionare **OK**.

    > [!Note]  
    >  Creare almeno un endpoint con ambito geografico Tutte (mondo) che svolga la funzione di endpoint predefinito per la risorsa.

17. Dopo il completamento dell'aggiunta di entrambi gli endpoint, questi vengono visualizzati in **Profilo di Gestione traffico** insieme al relativo stato di monitoraggio **Online**.

    ![Stato dell'endpoint in Profilo di Gestione traffico](media/solution-deployment-guide-geo-distributed/image46.png)

#### <a name="global-enterprise-relies-on-azure-geo-distribution-capabilities"></a>Le aziende globali fanno affidamento sulle funzionalità di distribuzione geografica di Azure

L'indirizzamento del traffico di dati tramite Gestione traffico di Azure e gli endpoint geografici specifici consente alle aziende globali di rispettare le normative locali e di garantire la sicurezza e la conformità dei dati. Queste caratteristiche sono fondamentali per il funzionamento corretto di sedi operative locali e remote.

## <a name="next-steps"></a>Passaggi successivi

- Per altre informazioni sui modelli cloud di Azure, vedere [Modelli di progettazione cloud](/azure/architecture/patterns).
