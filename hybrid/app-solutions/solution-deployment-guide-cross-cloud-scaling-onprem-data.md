---
title: Distribuisci app ibride con dati locali che scalano tra cloud
description: Informazioni su come distribuire un'app che usa dati locali e scalabilità tra cloud usando Azure e l'hub Azure Stack.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 75289eae902c5363862e345bdedb97cbcee0476e
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910891"
---
# <a name="deploy-hybrid-app-with-on-premises-data-that-scales-cross-cloud"></a>Distribuisci app ibride con dati locali che scalano tra cloud

Questa guida alla soluzione illustra come distribuire un'app ibrida che si estende sia per Azure che per hub Azure Stack e usa un'unica origine dati locale.

Grazie a una soluzione cloud ibrida, è possibile combinare i vantaggi di conformità di un cloud privato con la scalabilità del cloud pubblico. Gli sviluppatori possono inoltre trarre vantaggio dall'ecosistema di sviluppatori Microsoft e applicare le proprie competenze al cloud e agli ambienti locali.

## <a name="overview-and-assumptions"></a>Panoramica e presupposti

Seguire questa esercitazione per configurare un flusso di lavoro che consente agli sviluppatori di distribuire un'app Web identica a un cloud pubblico e a un cloud privato. Questa app può accedere a una rete instradabile non Internet ospitata nel cloud privato. Queste app Web vengono monitorate e quando si verifica un picco nel traffico, un programma modifica i record DNS per reindirizzare il traffico al cloud pubblico. Quando il traffico scende al livello prima del picco, il traffico viene indirizzato di nuovo al cloud privato.

Questa esercitazione illustra le attività seguenti:

> [!div class="checklist"]
> - Distribuire un server di database SQL Server con connessione ibrida.
> - Connettere un'app Web in Azure globale a una rete ibrida.
> - Configurare il DNS per la scalabilità tra cloud.
> - Configurare i certificati SSL per la scalabilità tra cloud.
> - Configurare e distribuire l'app Web.
> - Creare un profilo di gestione traffico e configurarlo per la scalabilità tra cloud.
> - Configurare Application Insights il monitoraggio e gli avvisi per un aumento del traffico.
> - Configurare il cambio di traffico automatico tra Azure globale e l'hub Azure Stack.

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub è un'estensione di Azure. Azure Stack Hub offre l'agilità e l'innovazione di cloud computing all'ambiente locale, abilitando l'unico Cloud ibrido che consente di creare e distribuire app ibride ovunque.  
> 
> L'articolo [considerazioni sulla progettazione di app ibride](overview-app-design-considerations.md) esamina i pilastri della qualità del software (posizionamento, scalabilità, disponibilità, resilienza, gestibilità e sicurezza) per la progettazione, la distribuzione e la gestione di app ibride. Le considerazioni di progettazione consentono di ottimizzare la progettazione delle app ibride, riducendo al minimo le esigenze negli ambienti di produzione.

### <a name="assumptions"></a>Presupposti

Questa esercitazione presuppone che l'utente abbia una conoscenza di base dell'hub globale di Azure e Azure Stack. Per altre informazioni prima di iniziare l'esercitazione, vedere gli articoli seguenti:

- [Introduzione ad Azure](https://azure.microsoft.com/overview/what-is-azure/)
- [Concetti chiave dell'hub Azure Stack](/azure-stack/operator/azure-stack-overview.md)

Questa esercitazione presuppone anche che si disponga di una sottoscrizione di Azure. Se non si ha una sottoscrizione, [creare un account gratuito prima di](https://azure.microsoft.com/free/) iniziare.

## <a name="prerequisites"></a>Prerequisiti

Prima di iniziare questa soluzione, assicurarsi di soddisfare i requisiti seguenti:

- Un Azure Stack Development Kit (Gabriele) o una sottoscrizione in un sistema integrato Azure Stack Hub. Per distribuire il Gabriele, seguire le istruzioni in [distribuire Gabriele usando il programma di installazione](/azure-stack/asdk/asdk-install.md).
- L'installazione dell'hub Azure Stack deve avere installato quanto segue:
  - Il servizio app Azure. Collaborare con l'operatore Azure Stack hub per distribuire e configurare il servizio app Azure nell'ambiente in uso. Per questa esercitazione è necessario che il servizio app disponga di almeno un ruolo di lavoro dedicato disponibile (1).
  - Un'immagine di Windows Server 2016.
  - Windows Server 2016 con un'immagine Microsoft SQL Server.
  - Piani e offerte appropriati.
  - Nome di dominio per l'app Web. Se non si dispone di un nome di dominio, è possibile acquistarne uno da un provider di dominio, ad esempio GoDaddy, Bluehost e inMotion.
- Un certificato SSL per il dominio da un'autorità di certificazione attendibile, ad esempio LetsEncrypt.
- Un'app Web che comunica con un database di SQL Server e supporta Application Insights. È possibile scaricare l'app di esempio [dotnetcore-SQLDB-tutorial](https://github.com/Azure-Samples/dotnetcore-sqldb-tutorial) da GitHub.
- Una rete ibrida tra una rete virtuale di Azure e una rete virtuale Hub Azure Stack. Per istruzioni dettagliate, vedere [configurare la connettività cloud ibrida con Azure e l'Hub Azure stack](solution-deployment-guide-connectivity.md).

- Una pipeline di integrazione continua/distribuzione continua (CI/CD) ibrida con un agente di compilazione privato nell'hub Azure Stack. Per istruzioni dettagliate, vedere [configurare l'identità del cloud ibrido con Azure e App Hub Azure stack](solution-deployment-guide-identity.md).

## <a name="deploy-a-hybrid-connected-sql-server-database-server"></a>Distribuire un server di database SQL Server con connessione ibrida

1. Accedere al portale utenti dell'hub Azure Stack.

2. Nel **Dashboard**selezionare **Marketplace**.

    ![Marketplace Azure Stack Hub](media/solution-deployment-guide-hybrid/image1.png)

3. In **Marketplace**selezionare **calcolo**, quindi scegliere **altro**. In **altro**selezionare la **licenza gratuita SQL Server: SQL Server 2017 Developer nell'immagine di Windows Server** .

    ![Selezionare un'immagine di macchina virtuale nel portale per gli utenti di Azure Stack Hub](media/solution-deployment-guide-hybrid/image2.png)

4. In **licenza gratuita SQL Server: SQL Server 2017 Developer su Windows Server**, selezionare **Crea**.

5. In **nozioni di base > configurare le impostazioni di base**, specificare un **nome** per la macchina virtuale (VM), un **nome utente** per il SQL Server SA e una password per l'associazione di **accesso** .  Dall'elenco a discesa **sottoscrizione** selezionare la sottoscrizione in cui si sta eseguendo la distribuzione. Per **gruppo di risorse**, usare **scegliere esistente** e inserire la macchina virtuale nello stesso gruppo di risorse dell'app Web dell'hub Azure stack.

    ![Configurare le impostazioni di base per la macchina virtuale nel portale per gli utenti di Azure Stack Hub](media/solution-deployment-guide-hybrid/image3.png)

6. In **dimensioni**selezionare una dimensione per la macchina virtuale. Per questa esercitazione, è consigliabile A2_Standard o una DS2_V2_Standard.

7. In **impostazioni > configurare le funzionalità facoltative**, configurare le impostazioni seguenti:

   - **Account di archiviazione**: se necessario, creare un nuovo account.
   - **Rete virtuale**:

     > [!Important]  
     > Assicurarsi che la macchina virtuale SQL Server sia distribuita nella stessa rete virtuale dei gateway VPN.

   - **Indirizzo IP pubblico**: usare le impostazioni predefinite.
   - **Gruppo di sicurezza di rete**: (NSG). Creare un nuovo NSG.
   - **Estensioni e monitoraggio**: Mantieni le impostazioni predefinite.
   - **Account di archiviazione di diagnostica**: se necessario, creare un nuovo account.
   - Selezionare **OK** per salvare la configurazione.

     ![Configurare le funzionalità di VM facoltative nel portale per gli utenti dell'hub Azure Stack](media/solution-deployment-guide-hybrid/image4.png)

8. In **impostazioni SQL Server**configurare le impostazioni seguenti:

   - Per la **connettività SQL**selezionare **pubblico (Internet)**.
   - Per **porta**, Mantieni il valore predefinito **1433**.
   - Per **autenticazione SQL**selezionare **Abilita**.

     > [!Note]  
     > Quando si Abilita l'autenticazione SQL, è necessario inserire automaticamente le informazioni "SQLAdmin" configurate in **nozioni di base**.

   - Per le altre impostazioni, Mantieni le impostazioni predefinite. Selezionare **OK**.

     ![Configurare le impostazioni di SQL Server nel portale per gli utenti di Azure Stack Hub](media/solution-deployment-guide-hybrid/image5.png)

9. In **Riepilogo**verificare la configurazione della macchina virtuale e quindi selezionare **OK** per avviare la distribuzione.

    ![Riepilogo della configurazione nel portale per gli utenti dell'hub Azure Stack](media/solution-deployment-guide-hybrid/image6.png)

10. La creazione della nuova macchina virtuale richiede tempo. È possibile visualizzare lo stato delle VM in **macchine virtuali**.

    ![Stato delle macchine virtuali nel portale per gli utenti di Azure Stack Hub](media/solution-deployment-guide-hybrid/image7.png)

## <a name="create-web-apps-in-azure-and-azure-stack-hub"></a>Creare app Web in Azure e Azure Stack Hub

Il servizio app Azure semplifica l'esecuzione e la gestione di un'app Web. Poiché Azure Stack Hub è coerente con Azure, il servizio app può essere eseguito in entrambi gli ambienti. Il servizio app verrà usato per ospitare l'app.

### <a name="create-web-apps"></a>Creare app Web

1. Creare un'app Web in Azure seguendo le istruzioni riportate in [gestire un piano di servizio app in Azure](https://docs.microsoft.com/azure/app-service/app-service-plan-manage#create-an-app-service-plan). Assicurarsi di inserire l'app Web nella stessa sottoscrizione e nello stesso gruppo di risorse della rete ibrida.

2. Ripetere il passaggio precedente (1) nell'hub Azure Stack.

### <a name="add-route-for-azure-stack-hub"></a>Aggiungi route per hub Azure Stack

Il servizio app nell'hub Azure Stack deve essere instradabile dalla rete Internet pubblica per consentire agli utenti di accedere all'app. Se l'hub Azure Stack è accessibile da Internet, prendere nota dell'indirizzo IP pubblico o dell'URL per l'app Web Hub Azure Stack.

Se si usa un Gabriele, è possibile [configurare un mapping NAT statico](/azure-stack/operator/azure-stack-create-vpn-connection-one-node.md#configure-the-nat-vm-on-each-asdk-for-gateway-traversal) per esporre il servizio app al di fuori dell'ambiente virtuale.

### <a name="connect-a-web-app-in-azure-to-a-hybrid-network"></a>Connettere un'app Web in Azure a una rete ibrida

Per fornire connettività tra il front-end Web in Azure e il database SQL Server nell'hub Azure Stack, l'app Web deve essere connessa alla rete ibrida tra Azure e l'hub Azure Stack. Per abilitare la connettività, è necessario:

- Configurare la connettività da punto a sito.
- Configurare l'app Web.
- Modificare il gateway di rete locale nell'hub Azure Stack.

### <a name="configure-the-azure-virtual-network-for-point-to-site-connectivity"></a>Configurare la rete virtuale di Azure per la connettività da punto a sito

Il gateway di rete virtuale nel lato Azure della rete ibrida deve consentire le connessioni da punto a sito per l'integrazione con app Azure servizio.

1. In Azure passare alla pagina del gateway di rete virtuale. In **Impostazioni**selezionare **configurazione da punto a sito**.

    ![Opzione da punto a sito nel gateway di rete virtuale di Azure](media/solution-deployment-guide-hybrid/image8.png)

2. Selezionare **Configura ora** per configurare la configurazione da punto a sito.

    ![Avviare la configurazione da punto a sito nel gateway di rete virtuale di Azure](media/solution-deployment-guide-hybrid/image9.png)

3. Nella pagina configurazione **da punto a sito** immettere l'intervallo di indirizzi IP privati che si vuole usare nel **pool di indirizzi**.

   > [!Note]  
   > Verificare che l'intervallo specificato non si sovrappongano con gli intervalli di indirizzi già usati dalle subnet nei componenti globali di Azure o hub Azure Stack della rete ibrida.

   In **tipo di tunnel**deselezionare la **VPN IKEv2**. Selezionare **Save (Salva** ) per completare la configurazione da punto a sito.

   ![Impostazioni da punto a sito nel gateway di rete virtuale di Azure](media/solution-deployment-guide-hybrid/image10.png)

### <a name="integrate-the-azure-app-service-app-with-the-hybrid-network"></a>Integrare l'app di servizio app Azure con la rete ibrida

1. Per connettere l'app alla VNet di Azure, seguire le istruzioni in [gateway required VNet Integration](https://docs.microsoft.com/azure/app-service/web-sites-integrate-with-vnet#gateway-required-vnet-integration).

2. Passare a **Impostazioni** per il piano di servizio app che ospita l'app Web. In **Impostazioni** selezionare **Rete**.

    ![Configurare la rete per il piano di servizio app](media/solution-deployment-guide-hybrid/image11.png)

3. In **integrazione rete virtuale**selezionare **fare clic qui per gestire**.

    ![Gestire l'integrazione di VNET per il piano di servizio app](media/solution-deployment-guide-hybrid/image12.png)

4. Selezionare il VNET che si desidera configurare. In **indirizzi IP indirizzati a VNET**, immettere l'intervallo di indirizzi IP per Azure VNET, il Azure stack Hub VNET e gli spazi di indirizzi da punto a sito. Selezionare **Save (Salva** ) per convalidare e salvare queste impostazioni.

    ![Intervalli di indirizzi IP da instradare nell'integrazione della rete virtuale](media/solution-deployment-guide-hybrid/image13.png)

Per altre informazioni sull'integrazione del servizio app con Azure reti virtuali, vedere [integrare l'app con una rete virtuale di Azure](https://docs.microsoft.com/azure/app-service/web-sites-integrate-with-vnet).

### <a name="configure-the-azure-stack-hub-virtual-network"></a>Configurare la rete virtuale dell'hub Azure Stack

Il gateway di rete locale nella rete virtuale dell'hub Azure Stack deve essere configurato in modo da instradare il traffico dall'intervallo di indirizzi da punto a sito del servizio app.

1. Nell'hub Azure Stack passare a **gateway di rete locale**. In **Impostazioni** selezionare **Configurazione**.

    ![Opzione di configurazione del gateway nel gateway di rete locale dell'hub Azure Stack](media/solution-deployment-guide-hybrid/image14.png)

2. In **spazio di indirizzi**immettere l'intervallo di indirizzi da punto a sito per il gateway di rete virtuale in Azure.

    ![Spazio di indirizzi da punto a sito nel gateway di rete locale dell'hub Azure Stack](media/solution-deployment-guide-hybrid/image15.png)

3. Selezionare **Save (Salva** ) per convalidare e salvare la configurazione.

## <a name="configure-dns-for-cross-cloud-scaling"></a>Configurare DNS per la scalabilità tra cloud

Con la configurazione corretta del DNS per le app tra cloud, gli utenti possono accedere alle istanze globali di Azure e Azure Stack Hub dell'app Web. La configurazione DNS per questa esercitazione consente anche a gestione traffico di Azure di instradare il traffico quando il carico aumenta o diminuisce.

Questa esercitazione USA DNS di Azure per gestire il DNS perché i domini del servizio app non funzionano.

### <a name="create-subdomains"></a>Creazione di sottodomini

Poiché Gestione traffico si basa su CNAME DNS, è necessario un sottodominio per instradare correttamente il traffico agli endpoint. Per altre informazioni sui record DNS e sul mapping del dominio, vedere eseguire il mapping di [domini con gestione traffico](https://docs.microsoft.com/azure/app-service/web-sites-traffic-manager-custom-domain-name).

Per l'endpoint di Azure verrà creato un sottodominio che gli utenti possono usare per accedere all'app Web. Per questa esercitazione, può usare **app.Northwind.com**, ma è necessario personalizzare questo valore in base al proprio dominio.

Sarà inoltre necessario creare un sottodominio con un record A per l'endpoint dell'hub Azure Stack. È possibile usare **azurestack.Northwind.com**.

### <a name="configure-a-custom-domain-in-azure"></a>Configurare un dominio personalizzato in Azure

1. Aggiungere il nome host **app.Northwind.com** all'app Web di Azure eseguendo il [mapping di un record CNAME al servizio app Azure](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).

### <a name="configure-custom-domains-in-azure-stack-hub"></a>Configurare domini personalizzati nell'hub Azure Stack

1. Aggiungere il nome host **azurestack.Northwind.com** all'app web Hub Azure stack eseguendo il mapping di un [record a per app Azure servizio](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-domain#map-an-a-record). Usare l'indirizzo IP instradabile su Internet per l'app del servizio app.

2. Aggiungere il nome host **app.Northwind.com** all'app Web dell'hub Azure stack eseguendo il [mapping di un record CNAME al servizio app Azure](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record). Usare il nome host configurato nel passaggio precedente (1) come destinazione per il record CNAME.

## <a name="configure-ssl-certificates-for-cross-cloud-scaling"></a>Configurare i certificati SSL per il ridimensionamento tra cloud

È importante assicurarsi che i dati sensibili raccolti dall'app Web siano protetti in transito e archiviati nel database SQL.

Si configureranno le app Web di Azure e Azure Stack hub per usare i certificati SSL per tutto il traffico in ingresso.

### <a name="add-ssl-to-azure-and-azure-stack-hub"></a>Aggiungere SSL ad Azure e all'hub Azure Stack

Per aggiungere SSL ad Azure:

1. Verificare che il certificato SSL ottenuto sia valido per il sottodominio creato. (È accettabile usare i certificati con caratteri jolly).

2. In Azure seguire le istruzioni riportate nelle sezioni **preparare l'app Web** e **associare il certificato SSL** dell'articolo [associare un certificato SSL personalizzato esistente ad app Web di Azure](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-ssl) . Selezionare **SSL basato su SNI** come **tipo SSL**.

3. Reindirizza tutto il traffico alla porta HTTPS. Seguire le istruzioni nella sezione **applicare HTTPS** dell'articolo [associare un certificato SSL personalizzato esistente ad app Web di Azure](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-ssl) .

Per aggiungere SSL a Azure Stack Hub:

1. Ripetere i passaggi 1-3 usati per Azure.

## <a name="configure-and-deploy-the-web-app"></a>Configurare e distribuire l'app Web

Il codice dell'app verrà configurato per segnalare i dati di telemetria all'istanza corretta di Application Insights e configurare le app Web con le stringhe di connessione corrette. Per ulteriori informazioni su Application Insights, vedere [che cos'è Application Insights?](https://docs.microsoft.com/azure/application-insights/app-insights-overview)

### <a name="add-application-insights"></a>Aggiungi Application Insights

1. Aprire l'app Web in Microsoft Visual Studio.

2. [Aggiungere Application Insights](https://docs.microsoft.com/azure/azure-monitor/app/asp-net-core#enable-client-side-telemetry-for-web-applications) al progetto per trasmettere i dati di telemetria usati da Application Insights per creare avvisi quando il traffico Web aumenta o diminuisce.

### <a name="configure-dynamic-connection-strings"></a>Configurare le stringhe di connessione dinamiche

Ogni istanza dell'app Web userà un metodo diverso per la connessione al database SQL. L'app in Azure usa l'indirizzo IP privato della macchina virtuale SQL Server e l'app nell'hub Azure Stack usa l'indirizzo IP pubblico della macchina virtuale SQL Server.

> [!Note]  
> In un sistema integrato Azure Stack Hub, l'indirizzo IP pubblico non deve essere instradabile su Internet. In un Gabriele, l'indirizzo IP pubblico non è instradabile all'esterno di Gabriele.

È possibile usare le variabili di ambiente del servizio app per passare una stringa di connessione diversa a ogni istanza dell'app.

1. Aprire l'app in Visual Studio.

2. Aprire Startup.cs e individuare il blocco di codice seguente:

    ```C#
    services.AddDbContext<MyDatabaseContext>(options =>
        options.UseSqlite("Data Source=localdatabase.db"));
    ```

3. Sostituire il blocco di codice precedente con il codice seguente, che usa una stringa di connessione definita nella *appsettings.jssu* file:

    ```C#
    services.AddDbContext<MyDatabaseContext>(options =>
        options.UseSqlServer(Configuration.GetConnectionString("MyDbConnection")));
     // Automatically perform database migration
     services.BuildServiceProvider().GetService<MyDatabaseContext>().Database.Migrate();
    ```

### <a name="configure-app-service-app-settings"></a>Configurare le impostazioni dell'app del servizio app

1. Creare stringhe di connessione per Azure e hub Azure Stack. Le stringhe devono essere uguali, ad eccezione degli indirizzi IP utilizzati.

2. In Azure e Azure Stack Hub aggiungere la stringa di connessione appropriata [come impostazione dell'app](https://docs.microsoft.com/azure/app-service/web-sites-configure) nell'app Web, usando `SQLCONNSTR\_` come prefisso nel nome.

3. **Salvare** le impostazioni dell'app Web e riavviare l'app.

## <a name="enable-automatic-scaling-in-global-azure"></a>Abilitare la scalabilità automatica in Azure globale

Quando si crea l'app Web in un ambiente del servizio app, questa inizia con un'istanza. È possibile scalare automaticamente per aggiungere istanze per fornire più risorse di calcolo per l'app. Analogamente, è possibile ridimensionare automaticamente e ridurre il numero di istanze necessarie per l'app.

> [!Note]  
> È necessario disporre di un piano di servizio app per configurare la scalabilità orizzontale e il ridimensionamento. Se non si dispone di un piano, crearne uno prima di iniziare i passaggi successivi.

### <a name="enable-automatic-scale-out"></a>Abilitare la scalabilità orizzontale automatica

1. In Azure trovare il piano di servizio app per i siti da scalare in orizzontale e quindi selezionare **scale-out (piano di servizio app)**.

    ![Servizio app Azure di scalabilità orizzontale](media/solution-deployment-guide-hybrid/image16.png)

2. Selezionare **Abilita ridimensionamento**automatico.

    ![Abilitare la scalabilità automatica nel servizio app Azure](media/solution-deployment-guide-hybrid/image17.png)

3. Immettere un nome per il **nome dell'impostazione di scalabilità**automatica. Per la regola di scalabilità automatica **predefinita** , selezionare **Ridimensiona in base a una metrica**. Impostare i **limiti dell'istanza** su **minimo: 1**, **massimo: 10**e **valore predefinito: 1**.

    ![Configurare la scalabilità automatica nel servizio app Azure](media/solution-deployment-guide-hybrid/image18.png)

4. Selezionare **+ Aggiungi una regola**.

5. In **origine metrica**selezionare **risorsa corrente**. Usare i criteri e le azioni seguenti per la regola.

#### <a name="criteria"></a>Criteri

1. In **aggregazione temporale** selezionare **media**.

2. In **nome metrica**selezionare **percentuale CPU**.

3. In **operatore**selezionare **maggiore di**.

   - Impostare la **soglia** su **50**.
   - Impostare la **durata** su **10**.

#### <a name="action"></a>Azione

1. In **operazione**selezionare **aumenta numero per**.

2. Impostare il **numero di istanze** su **2**.

3. Impostare il livello di **raffreddamento** su **5**.

4. Selezionare **Aggiungi**.

5. Selezionare **+ Aggiungi una regola**.

6. In **origine metrica**selezionare **risorsa corrente.**

   > [!Note]  
   > La risorsa corrente conterrà il nome/GUID del piano di servizio app e gli elenchi a discesa **tipo di risorsa** e **risorsa** non saranno disponibili.

### <a name="enable-automatic-scale-in"></a>Abilitare la scalabilità automatica

Quando il traffico diminuisce, l'app Web di Azure può ridurre automaticamente il numero di istanze attive per ridurre i costi. Questa azione è meno aggressiva rispetto alla scalabilità orizzontale e riduce al minimo l'effetto sugli utenti dell'app.

1. Passare alla condizione di scalabilità orizzontale **predefinita** , quindi selezionare **+ Aggiungi una regola**. Usare i criteri e le azioni seguenti per la regola.

#### <a name="criteria"></a>Criteri

1. In **aggregazione temporale** selezionare **media**.

2. In **nome metrica**selezionare **percentuale CPU**.

3. In **operatore**selezionare **minore di**.

   - Impostare la **soglia** su **30**.
   - Impostare la **durata** su **10**.

#### <a name="action"></a>Azione

1. In **operazione**selezionare **Diminuisci conteggio per**.

   - Impostare il **numero di istanze** su **1**.
   - Impostare il livello di **raffreddamento** su **5**.

2. Selezionare **Aggiungi**.

## <a name="create-a-traffic-manager-profile-and-configure-cross-cloud-scaling"></a>Creare un profilo di gestione traffico e configurare la scalabilità tra cloud

Creare un profilo di gestione traffico in Azure e quindi configurare gli endpoint per abilitare il ridimensionamento tra cloud.

### <a name="create-traffic-manager-profile"></a>Crea profilo di gestione traffico

1. Selezionare **Crea una risorsa**.
2. Selezionare **rete**.
3. Selezionare **profilo di gestione traffico** e configurare le impostazioni seguenti:

   - In **nome**immettere un nome per il profilo. Questo nome **deve** essere univoco nella zona trafficmanager.NET e viene usato per creare un nuovo nome DNS (ad esempio, northwindstore.trafficmanager.NET).
   - Per **metodo di routing**selezionare il valore **ponderato**.
   - Per **Subscription (sottoscrizione**) selezionare la sottoscrizione in cui si vuole creare il profilo.
   - In **gruppo di risorse**creare un nuovo gruppo di risorse per questo profilo.
   - In **Località del gruppo di risorse** selezionare la località del gruppo di risorse. Questa impostazione si riferisce al percorso del gruppo di risorse e non ha alcun effetto sul profilo di gestione traffico distribuito a livello globale.

4. Selezionare **Crea**.

    ![Crea profilo di gestione traffico](media/solution-deployment-guide-hybrid/image19.png)

   Quando la distribuzione globale del profilo di gestione traffico è completa, viene visualizzata nell'elenco di risorse per il gruppo di risorse in cui è stato creato.

### <a name="add-traffic-manager-endpoints"></a>Aggiungere endpoint di Gestione traffico

1. Cercare il profilo di gestione traffico creato. Se si è passati al gruppo di risorse per il profilo, selezionare il profilo.

2. In **profilo di gestione traffico**selezionare **endpoint**in **Impostazioni**.

3. Selezionare **Aggiungi**.

4. In **Aggiungi endpoint**usare le impostazioni seguenti per Azure stack Hub:

   - Per **tipo**selezionare **endpoint esterno**.
   - Immettere un **nome** per l'endpoint.
   - Per **nome di dominio completo (FQDN) o IP**, immettere l'URL esterno per l'app Web dell'hub Azure stack.
   - Per **peso**, Mantieni il valore predefinito **1**. Questo peso comporta che tutto il traffico passa a questo endpoint se è integro.
   - Lasciare deselezionata l'opzione **Aggiungi come disabilitata** .

5. Selezionare **OK** per salvare l'endpoint dell'hub Azure stack.

L'endpoint di Azure verrà configurato successivamente.

1. In **profilo di gestione traffico**selezionare **endpoint**.
2. Selezionare **+ Aggiungi**.
3. In **Aggiungi endpoint**usare le impostazioni seguenti per Azure:

   - Per **tipo**selezionare **endpoint di Azure**.
   - Immettere un **nome** per l'endpoint.
   - Per **tipo di risorsa di destinazione**selezionare **servizio app**.
   - Per **risorsa di destinazione**selezionare **Scegli un servizio app** per visualizzare un elenco di app Web nella stessa sottoscrizione.
   - In **Risorsa** selezionare il servizio App che si vuole aggiungere come primo endpoint.
   - Per **peso**selezionare **2**. Questa impostazione determina l'indirizzamento del traffico a questo endpoint se l'endpoint primario non è integro o se si dispone di una regola/avviso che reindirizza il traffico quando viene attivato.
   - Lasciare deselezionata l'opzione **Aggiungi come disabilitata** .

4. Selezionare **OK** per salvare l'endpoint di Azure.

Una volta configurati entrambi gli endpoint, questi vengono elencati in **profilo di gestione traffico** quando si selezionano gli **endpoint**. Nell'esempio riportato di seguito vengono illustrati due endpoint, con informazioni sullo stato e sulla configurazione per ciascuno di essi.

![Endpoint nel profilo di gestione traffico](media/solution-deployment-guide-hybrid/image20.png)

## <a name="set-up-application-insights-monitoring-and-alerting"></a>Configurare Application Insights monitoraggio e avvisi

Applicazione Azure Insights consente di monitorare l'app e inviare avvisi in base alle condizioni configurate. Alcuni esempi sono: l'app non è disponibile, sta riscontrando errori o presenta problemi di prestazioni.

Per creare avvisi si useranno le metriche di Application Insights. Quando si attivano questi avvisi, l'istanza dell'app Web passa automaticamente da Azure Stack Hub ad Azure per la scalabilità orizzontale e quindi torna all'hub Azure Stack per la scalabilità in.

### <a name="create-an-alert-from-metrics"></a>Creare un avviso dalle metriche

Andare al gruppo di risorse per questa esercitazione, quindi selezionare l'istanza di Application Insights per aprire **Application Insights**.

![Application Insights](media/solution-deployment-guide-hybrid/image21.png)

Questa visualizzazione verrà utilizzata per creare un avviso di scalabilità orizzontale e un avviso di riduzione delle prestazioni.

### <a name="create-the-scale-out-alert"></a>Creare l'avviso di scalabilità orizzontale

1. In **Configura**selezionare **avvisi (versione classica)**.
2. Selezionare **Aggiungi avviso per la metrica (versione classica)**.
3. In **Aggiungi regola**configurare le impostazioni seguenti:

   - Per **nome**immettere espansione **nel cloud di Azure**.
   - Una **Descrizione** è facoltativa.
   - In **origine**  >  **avviso su**selezionare **metrica**.
   - In **criteri**selezionare la sottoscrizione, il gruppo di risorse per il profilo di gestione traffico e il nome del profilo di gestione traffico per la risorsa.

4. Per **metrica**selezionare **frequenza richieste**.
5. Per **condizione**selezionare **maggiore di**.
6. In **soglia**immettere **2**.
7. Per **periodo**selezionare **negli ultimi 5 minuti**.
8. In **notifica tramite**:
   - Selezionare la casella di controllo per **inviare messaggi di posta elettronica a proprietari, collaboratori e lettori**.
   - Immettere l'indirizzo di posta elettronica per altri indirizzi di **posta elettronica dell'amministratore**.

9. Nella barra dei menu selezionare **Salva**.

### <a name="create-the-scale-in-alert"></a>Creare l'avviso di riduzione del livello

1. In **Configura**selezionare **avvisi (versione classica)**.
2. Selezionare **Aggiungi avviso per la metrica (versione classica)**.
3. In **Aggiungi regola**configurare le impostazioni seguenti:

   - Per **nome**immettere di **nuovo la scalabilità nell'hub Azure stack**.
   - Una **Descrizione** è facoltativa.
   - In **origine**  >  **avviso su**selezionare **metrica**.
   - In **criteri**selezionare la sottoscrizione, il gruppo di risorse per il profilo di gestione traffico e il nome del profilo di gestione traffico per la risorsa.

4. Per **metrica**selezionare **frequenza richieste**.
5. Per **condizione**selezionare **minore di**.
6. In **soglia**immettere **2**.
7. Per **periodo**selezionare **negli ultimi 5 minuti**.
8. In **notifica tramite**:
   - Selezionare la casella di controllo per **inviare messaggi di posta elettronica a proprietari, collaboratori e lettori**.
   - Immettere l'indirizzo di posta elettronica per altri indirizzi di **posta elettronica dell'amministratore**.

9. Nella barra dei menu selezionare **Salva**.

Lo screenshot seguente Mostra gli avvisi per la scalabilità orizzontale e il ridimensionamento.

   ![Avvisi di Application Insights (versione classica)](media/solution-deployment-guide-hybrid/image22.png)

## <a name="redirect-traffic-between-azure-and-azure-stack-hub"></a>Reindirizza il traffico tra Azure e l'hub Azure Stack

È possibile configurare il cambio manuale o automatico del traffico delle app Web tra Azure e l'hub Azure Stack.

### <a name="configure-manual-switching-between-azure-and-azure-stack-hub"></a>Configurare il cambio manuale tra Azure e l'hub Azure Stack

Quando il sito Web raggiunge le soglie configurate, si riceverà un avviso. Per reindirizzare manualmente il traffico ad Azure, seguire questa procedura.

1. Nella portale di Azure selezionare il profilo di gestione traffico.

    ![Endpoint di gestione traffico in portale di Azure](media/solution-deployment-guide-hybrid/image20.png)

2. Selezionare **Endpoint**.
3. Selezionare l' **endpoint di Azure**.
4. In **stato**selezionare **abilitato**e quindi fare clic su **Salva**.

    ![Abilitare l'endpoint di Azure in portale di Azure](media/solution-deployment-guide-hybrid/image23.png)

5. In **endpoint** per il profilo di gestione traffico selezionare **endpoint esterno**.
6. In **stato**selezionare **disabilitato**e quindi fare clic su **Salva**.

    ![Disabilitare Azure Stack endpoint Hub nell'portale di Azure](media/solution-deployment-guide-hybrid/image24.png)

Una volta configurati gli endpoint, il traffico delle app viene indirizzato all'app Web di Azure con scalabilità orizzontale anziché all'app Web Hub Azure Stack.

 ![Endpoint modificati nel traffico delle app Web di Azure](media/solution-deployment-guide-hybrid/image25.png)

Per invertire di nuovo il flusso all'hub Azure Stack, usare i passaggi precedenti per:

- Abilitare l'endpoint dell'hub Azure Stack.
- Disabilitare l'endpoint di Azure.

### <a name="configure-automatic-switching-between-azure-and-azure-stack-hub"></a>Configurare il cambio automatico tra Azure e l'hub Azure Stack

È anche possibile usare Application Insights monitoraggio se l'app viene eseguita in un ambiente senza [Server](https://azure.microsoft.com/overview/serverless-computing/) fornito da funzioni di Azure.

In questo scenario è possibile configurare Application Insights per l'uso di un webhook che chiama un'app per le funzioni. Questa app Abilita o Disabilita automaticamente un endpoint in risposta a un avviso.

Usare i passaggi seguenti come guida per configurare il cambio automatico del traffico.

1. Creare un'app per le funzioni di Azure.
2. Creare una funzione attivata tramite HTTP.
3. Importare gli SDK di Azure per Gestione risorse, app Web e gestione traffico.
4. Sviluppare codice per:

   - Eseguire l'autenticazione alla sottoscrizione di Azure.
   - Usare un parametro che commuta gli endpoint di gestione traffico per indirizzare il traffico ad Azure o all'hub Azure Stack.

5. Salvare il codice e aggiungere l'URL dell'app per le funzioni con i parametri appropriati alla sezione **webhook** delle impostazioni della regola di avviso Application Insights.
6. Il traffico viene reindirizzato automaticamente quando viene attivato un avviso Application Insights.

## <a name="next-steps"></a>Passaggi successivi

- Per altre informazioni sui modelli cloud di Azure, vedere [modelli di progettazione cloud](https://docs.microsoft.com/azure/architecture/patterns).
