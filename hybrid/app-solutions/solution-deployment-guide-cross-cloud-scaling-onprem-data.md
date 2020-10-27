---
title: Distribuire un'app ibrida con dati locali con scalabilità tra cloud
description: Informazioni su come distribuire un'app che usa dati locali con scalabilità tra cloud tramite Azure e l'hub di Azure Stack.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: ecc42a94e2c59531b2a2e933772b0d8ce8c58609
ms.sourcegitcommit: 0d5b5336bdb969588d0b92e04393e74b8f682c3b
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 10/22/2020
ms.locfileid: "92353479"
---
# <a name="deploy-hybrid-app-with-on-premises-data-that-scales-cross-cloud"></a>Distribuire un'app ibrida con dati locali con scalabilità tra cloud

Questa guida alla soluzione illustra come distribuire un'app ibrida che si estende in Azure e nell'hub di Azure Stack e usa un'unica origine dati locale.

Usando una soluzione di cloud ibrido è possibile usufruire dei vantaggi di conformità di un cloud privato e della scalabilità del cloud pubblico. Gli sviluppatori possono inoltre usufruire dell'ecosistema di sviluppatori Microsoft e applicare le loro competenze al cloud e ad ambienti locali.

## <a name="overview-and-assumptions"></a>Panoramica e presupposti

Seguire questa esercitazione per configurare un flusso di lavoro che consenta agli sviluppatori di distribuire un'app Web identica in un cloud pubblico e in un cloud privato. Questa app può accedere a una rete instradabile non Internet ospitata nel cloud privato. Queste app Web vengono monitorate e quando si verifica un picco nel traffico, un programma modifica i record DNS per reindirizzare il traffico al cloud pubblico. Quando il traffico torna al livello precedente il picco, il traffico viene indirizzato di nuovo al cloud privato.

Questa esercitazione illustra le attività seguenti:

> [!div class="checklist"]
> - Distribuire un server di database SQL Server con connessione ibrida.
> - Connettere un'app Web di Azure globale a una rete ibrida.
> - Configurare il DNS per la scalabilità tra cloud.
> - Configurare i certificati SSL per la scalabilità tra cloud.
> - Configurare e distribuire l'app Web.
> - Creare un profilo di Gestione traffico e configurarlo per la scalabilità tra cloud.
> - Configurare il monitoraggio e gli avvisi di Application Insights per un aumento del traffico.
> - Configurare il passaggio automatico del traffico tra Azure globale e l'hub di Azure Stack.

> [!Tip]  
> ![Diagramma dei concetti sulle app ibride](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> L'hub di Microsoft Azure Stack è un'estensione di Azure. L'hub di Azure Stack offre all'ambiente locale l'agilità e l'innovazione del cloud computing, abilitando l'unico cloud ibrido che consente di creare e distribuire ovunque app ibride.  
> 
> L'articolo [Considerazioni per la progettazione di app ibride](overview-app-design-considerations.md) esamina i concetti fondamentali di qualità del software (posizionamento, scalabilità, disponibilità, resilienza, gestibilità e sicurezza) per la progettazione, la distribuzione e la gestione di app ibride. Le considerazioni per la progettazione consentono di ottimizzare la progettazione delle app ibride, riducendo al minimo i rischi negli ambienti di produzione.

### <a name="assumptions"></a>Presupposti

Questa esercitazione presuppone che l'utente abbia una conoscenza di base di Azure globale e dell'hub di Azure Stack. Per altre informazioni prima di iniziare l'esercitazione, vedere gli articoli seguenti:

- [Introduzione ad Azure](https://azure.microsoft.com/overview/what-is-azure/)
- [Concetti chiave dell'hub di Azure Stack](/azure-stack/operator/azure-stack-overview.md)

In questa esercitazione si presuppone anche che l'utente abbia una sottoscrizione di Azure. Se non si ha già una sottoscrizione, [creare un account gratuito](https://azure.microsoft.com/free/) prima di iniziare.

## <a name="prerequisites"></a>Prerequisiti

Prima di iniziare a creare questa soluzione, verificare di soddisfare i requisiti seguenti:

- Un Azure Stack Development Kit (ASDK) o una sottoscrizione in un sistema integrato dell'hub di Azure Stack. Per distribuire un ASDK, seguire le istruzioni riportate in [Distribuire l'ASDK usando il programma di installazione](/azure-stack/asdk/asdk-install.md).
- L'installazione dell'hub di Azure Stack deve includere quanto segue:
  - Il Servizio app di Azure. Collaborare con l'operatore dell'hub di Azure Stack per distribuire e configurare il Servizio app di Azure nell'ambiente in uso. Per questa esercitazione è necessario che il servizio app includa almeno un (1) ruolo di lavoro dedicato disponibile.
  - Un'immagine di Windows Server 2016.
  - Un'immagine di Windows Server 2016 con Microsoft SQL Server.
  - I piani e le offerte appropriati.
  - Un nome di dominio per l'app Web. Se non si ha un nome di dominio, è possibile acquistarne uno da un provider di domini, ad esempio GoDaddy, Bluehost e inMotion.
- Un certificato SSL per il dominio emesso da un'autorità di certificazione attendibile, ad esempio LetsEncrypt.
- Un'app Web che comunica con un database di SQL Server e supporta Application Insights. È possibile scaricare l'app di esempio [dotnetcore-sqldb-tutorial](https://github.com/Azure-Samples/dotnetcore-sqldb-tutorial) da GitHub.
- Una rete ibrida tra una rete virtuale di Azure e una rete virtuale dell'hub di Azure Stack. Per istruzioni dettagliate, vedere [Configurare la connettività di cloud ibrido con Azure e l'hub di Azure Stack](solution-deployment-guide-connectivity.md).

- Una pipeline con integrazione continua/distribuzione continua (CI/CD) ibrida con un agente di compilazione privato nell'hub di Azure Stack. Per istruzioni dettagliate, vedere [Configurare l'identità del cloud ibrido con app di Azure e dell'hub di Azure Stack](solution-deployment-guide-identity.md).

## <a name="deploy-a-hybrid-connected-sql-server-database-server"></a>Distribuire un server di database SQL Server con connessione ibrida

1. Accedere al portale utenti dell'hub di Azure Stack.

2. Nel **Dashboard** selezionare **Marketplace** .

    ![Marketplace dell'hub di Azure Stack](media/solution-deployment-guide-hybrid/image1.png)

3. In **Marketplace** selezionare **Calcola** e quindi scegliere **Altro** . In **Altro** selezionare l'immagine **Free SQL Server License: SQL Server 2017 Developer on Windows Server** .

    ![Selezionare un'immagine di macchina virtuale nel portale utenti dell'hub di Azure Stack](media/solution-deployment-guide-hybrid/image2.png)

4. In **Free SQL Server License: SQL Server 2017 Developer on Windows Server** selezionare **Crea** .

5. In **Generale > Configura le impostazioni di base** specificare un **Nome** per la macchina virtuale (VM), un **Nome utente** per l'amministratore di sistema di SQL Server e una **Password** per l'amministratore di sistema.  Nell'elenco a discesa **Sottoscrizione** selezionare la sottoscrizione in cui viene eseguita la distribuzione. Per **Gruppo di risorse** usare **Seleziona esistente** e inserire la macchina virtuale nello stesso gruppo di risorse dell'app Web dell'hub di Azure Stack.

    ![Configurare le impostazioni di base per la macchina virtuale nel portale utenti dell'hub di Azure Stack](media/solution-deployment-guide-hybrid/image3.png)

6. In **Dimensione** selezionare una dimensione per la macchina virtuale. Per questa esercitazione è consigliabile selezionare A2_Standard o DS2_V2_Standard.

7. In **Impostazioni > Configura funzionalità facoltative** configurare le impostazioni seguenti:

   - **Account di archiviazione** : se necessario, creare un nuovo account.
   - **Rete virtuale** :

     > [!Important]  
     > Assicurarsi che la macchina virtuale SQL Server sia distribuita nella stessa rete virtuale dei gateway VPN.

   - **Indirizzo IP pubblico** : usare le impostazioni predefinite.
   - **Network security group** (Gruppo di sicurezza di rete): (NSG). Creare un nuovo gruppo di sicurezza di rete.
   - **Estensioni e Monitoraggio** : Mantenere le impostazioni predefinite.
   - **Account di archiviazione di diagnostica** : se necessario, creare un nuovo account.
   - Selezionare **OK** per salvare la configurazione.

     ![Configurare le funzionalità della macchina virtuale facoltative nel portale utenti dell'hub di Azure Stack](media/solution-deployment-guide-hybrid/image4.png)

8. In **Impostazioni di SQL Server** configurare le impostazioni seguenti:

   - Per **Connettività SQL** selezionare **Pubblico (Internet)** .
   - Per **Porta** mantenere l'impostazione predefinita **1433** .
   - Per **Autenticazione SQL** selezionare **Abilita** .

     > [!Note]  
     > Quando si abilita l'autenticazione SQL, vengono inserite automaticamente le informazioni "SQLAdmin" configurate in **Generale** .

   - Per le impostazioni rimanenti, mantenere le impostazioni predefinite. Selezionare **OK** .

     ![Configurare le impostazioni di SQL Server nel portale utenti dell'hub di Azure Stack](media/solution-deployment-guide-hybrid/image5.png)

9. In **Riepilogo** rivedere la configurazione della macchina virtuale e quindi selezionare **OK** per avviare la distribuzione.

    ![Riepilogo della configurazione nel portale utenti dell'hub di Azure Stack](media/solution-deployment-guide-hybrid/image6.png)

10. La creazione della nuova macchina virtuale richiede tempo. È possibile visualizzare lo stato delle macchine virtuali in **Macchine virtuali** .

    ![Stato delle macchine virtuali nel portale utenti dell'hub di Azure Stack](media/solution-deployment-guide-hybrid/image7.png)

## <a name="create-web-apps-in-azure-and-azure-stack-hub"></a>Creare app Web in Azure e nell'hub di Azure Stack

Il Servizio app di Azure semplifica l'esecuzione e la gestione di un'app Web. Poiché l'hub di Azure Stack è compatibile con Azure, il servizio app può essere eseguito in entrambi gli ambienti. Il servizio app verrà usato per ospitare l'app.

### <a name="create-web-apps"></a>Creare app Web

1. Creare un'app Web in Azure seguendo le istruzioni riportate in [Gestire un piano di servizio app in Azure](/azure/app-service/app-service-plan-manage#create-an-app-service-plan). Assicurarsi di inserire l'app Web nella stessa sottoscrizione e nello stesso gruppo di risorse della rete ibrida.

2. Ripetere il passaggio precedente (1) nell'hub di Azure Stack.

### <a name="add-route-for-azure-stack-hub"></a>Aggiungere la route per l'hub di Azure Stack

Il servizio app nell'hub di Azure Stack deve essere instradabile dalla rete Internet pubblica per consentire agli utenti di accedere all'app. Se l'hub di Azure Stack è accessibile da Internet, prendere nota dell'indirizzo IP pubblico o dell'URL dell'app Web dell'hub di Azure Stack.

Se si usa un ASDK, è possibile [configurare un mapping NAT statico](/azure-stack/operator/azure-stack-create-vpn-connection-one-node.md#configure-the-nat-vm-on-each-asdk-for-gateway-traversal) per esporre il servizio app al di fuori dell'ambiente virtuale.

### <a name="connect-a-web-app-in-azure-to-a-hybrid-network"></a>Connettere un'app Web di Azure a una rete ibrida

Per offrire connettività tra il front-end Web in Azure e il database SQL Server nell'hub di Azure Stack, l'app Web deve essere connessa alla rete ibrida tra Azure e l'hub di Azure Stack. Per abilitare la connettività, è necessario:

- Configurare la connettività da punto a sito.
- Configurare l'app Web.
- Modificare il gateway di rete locale nell'hub di Azure Stack.

### <a name="configure-the-azure-virtual-network-for-point-to-site-connectivity"></a>Configurare la rete virtuale di Azure per la connettività da punto a sito

Il gateway di rete virtuale sul lato Azure della rete ibrida deve consentire le connessioni da punto a sito per l'integrazione con il Servizio app di Azure.

1. Nel portale di Azure passare alla pagina del gateway di rete virtuale. In **Impostazioni** selezionare **Configurazione da punto a sito** .

    ![Opzione da punto a sito nel gateway di rete virtuale di Azure](media/solution-deployment-guide-hybrid/image8.png)

2. Selezionare **Configura adesso** per configurare la connessione da punto a sito.

    ![Iniziare la configurazione da punto a sito nel gateway di rete virtuale di Azure](media/solution-deployment-guide-hybrid/image9.png)

3. Nella pagina di configurazione **Da punto a sito** immettere l'intervallo di indirizzi IP privati che si vuole usare in **Pool di indirizzi** .

   > [!Note]  
   > Verificare che l'intervallo specificato non si sovrapponga ad altri intervalli di indirizzi già usati dalle subnet nei componenti di Azure globale e dell'hub di Azure Stack della rete ibrida.

   In **Tipo di tunnel** deselezionare **VPN IKEv2** . Selezionare **Salva** per completare la configurazione da punto a sito.

   ![Impostazioni da punto a sito nel gateway di rete virtuale di Azure](media/solution-deployment-guide-hybrid/image10.png)

### <a name="integrate-the-azure-app-service-app-with-the-hybrid-network"></a>Integrare l'app del Servizio app di Azure con la rete ibrida

1. Per connettere l'app alla VNet di Azure, seguire le istruzioni riportate in [Integrazione rete virtuale richiesta dal gateway](/azure/app-service/web-sites-integrate-with-vnet#gateway-required-vnet-integration).

2. Passare a **Impostazioni** per il piano di servizio app che ospita l'app Web. In **Impostazioni** selezionare **Rete** .

    ![Configurare la rete per il piano di servizio app](media/solution-deployment-guide-hybrid/image11.png)

3. In **Integrazione rete virtuale** selezionare **Fare clic qui per la gestione** .

    ![Gestire l'integrazione rete virtuale per il piano di servizio app](media/solution-deployment-guide-hybrid/image12.png)

4. Selezionare la rete virtuale da configurare. In **ROUTING DEGLI INDIRIZZI IP ALLA RETE VIRTUALE** immettere l'intervallo di indirizzi IP per la rete virtuale di Azure, la rete virtuale dell'hub di Azure Stack e gli spazi indirizzi da punto a sito. Selezionare **Salva** per convalidare e salvare le impostazioni.

    ![Intervalli di indirizzi IP da instradare nell'integrazione della rete virtuale](media/solution-deployment-guide-hybrid/image13.png)

Per altre informazioni sull'integrazione del servizio app con le reti virtuali di Azure, vedere [Integrare l'app con una rete virtuale di Azure](/azure/app-service/web-sites-integrate-with-vnet).

### <a name="configure-the-azure-stack-hub-virtual-network"></a>Configurare la rete virtuale dell'hub di Azure Stack

Il gateway di rete locale nella rete virtuale dell'hub di Azure Stack deve essere configurato in modo da instradare il traffico dall'intervallo di indirizzi da punto a sito del servizio app.

1. Nel portale Hub di Azure Stack passare a **Gateway di rete virtuale** . In **Impostazioni** selezionare **Configurazione** .

    ![Opzione di configurazione del gateway nel gateway di rete locale dell'hub di Azure Stack](media/solution-deployment-guide-hybrid/image14.png)

2. In **Spazio indirizzi** immettere l'intervallo di indirizzi da punto a sito per il gateway di rete virtuale in Azure.

    ![Spazio indirizzi da punto a sito nel gateway di rete locale dell'hub di Azure Stack](media/solution-deployment-guide-hybrid/image15.png)

3. Selezionare **Salva** per convalidare e salvare la configurazione.

## <a name="configure-dns-for-cross-cloud-scaling"></a>Configurare il DNS per la scalabilità tra cloud

Con la configurazione corretta del DNS per le app tra cloud, gli utenti possono accedere alle istanze di Azure globale e dell'hub di Azure Stack dell'app Web. La configurazione DNS per questa esercitazione consente anche a Gestione traffico di Azure di instradare il traffico quando il carico aumenta o diminuisce.

Questa esercitazione usa DNS di Azure per la gestione del DNS poiché i domini del servizio app non funzionano.

### <a name="create-subdomains"></a>Creare sottodomini

Poiché Gestione traffico si basa su CNAME DNS, è necessario un sottodominio per instradare correttamente il traffico agli endpoint. Per altre informazioni sui record DNS e sul mapping del dominio, vedere [Mappare i domini con Gestione traffico](/azure/app-service/web-sites-traffic-manager-custom-domain-name).

Per l'endpoint di Azure verrà creato un sottodominio che gli utenti possono usare per accedere all'app Web. Per questa esercitazione, è possibile usare **app.northwind.com** , ma è necessario personalizzare questo valore in base al proprio dominio.

Sarà anche necessario creare un sottodominio con un record A per l'endpoint dell'hub di Azure Stack. È possibile usare **azurestack.northwind.com** .

### <a name="configure-a-custom-domain-in-azure"></a>Configurare un dominio personalizzato in Azure

1. Aggiungere il nome host **app.northwind.com** all'app Web di Azure eseguendo [il mapping di un CNAME al Servizio app di Azure](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).

### <a name="configure-custom-domains-in-azure-stack-hub"></a>Configurare domini personalizzati nell'hub di Azure Stack

1. Aggiungere il nome host **azurestack.northwind.com** all'app Web dell'hub di Azure Stack eseguendo [il mapping di un record A al Servizio app di Azure](/azure/app-service/app-service-web-tutorial-custom-domain#map-an-a-record). Usare l'indirizzo IP instradabile su Internet per l'app del servizio app.

2. Aggiungere il nome host **app.northwind.com** all'app Web dell'hub di Azure Stack eseguendo [il mapping di un CNAME al Servizio app di Azure](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record). Usare il nome host configurato nel passaggio precedente (1) come destinazione per CNAME.

## <a name="configure-ssl-certificates-for-cross-cloud-scaling"></a>Configurare i certificati SSL per la scalabilità tra cloud

È importante assicurarsi che i dati sensibili raccolti dall'app Web siano protetti in transito e quando vengono archiviati nel database SQL.

Si configureranno le app Web di Azure e dell'hub di Azure Stack per l'uso dei certificati SSL per tutto il traffico in ingresso.

### <a name="add-ssl-to-azure-and-azure-stack-hub"></a>Aggiungere SSL ad Azure e all'hub di Azure Stack

Per aggiungere SSL ad Azure:

1. Verificare che il certificato SSL ottenuto sia valido per il sottodominio creato. È possibile usare certificati con caratteri jolly.

2. Nel portale di Azure seguire le istruzioni riportate nelle sezioni **Preparare l'app Web** e **Associare il certificato SSL** dell'articolo [Associare un certificato SSL personalizzato esistente ad app Web di Azure](/azure/app-service/app-service-web-tutorial-custom-ssl). Selezionare **SSL basato su SNI** come **Tipo SSL** .

3. Reindirizzare tutto il traffico alla porta HTTPS. Seguire le istruzioni riportate nella sezione **Applicare HTTPS** dell'articolo [Associare un certificato SSL personalizzato esistente ad app Web di Azure](/azure/app-service/app-service-web-tutorial-custom-ssl).

Per aggiungere SSL all'hub di Azure Stack:

1. Ripetere i passaggi 1-3 seguiti per Azure usando il portale Hub di Azure Stack.

## <a name="configure-and-deploy-the-web-app"></a>Configurare e distribuire l'app Web

Il codice dell'app verrà configurato per inviare i dati di telemetria all'istanza corretta di Application Insights e configurare le app Web con le stringhe di connessione appropriate. Per altre informazioni su Application Insights, vedere [Informazioni su Azure Application Insights](/azure/application-insights/app-insights-overview).

### <a name="add-application-insights"></a>Aggiungere Application Insights

1. Aprire l'app Web in Microsoft Visual Studio.

2. [Aggiungere Application Insights](/azure/azure-monitor/app/asp-net-core#enable-client-side-telemetry-for-web-applications) al progetto per trasmettere i dati di telemetria che Application Insights usa per creare avvisi quando il traffico Web aumenta o diminuisce.

### <a name="configure-dynamic-connection-strings"></a>Configurare le stringhe di connessione dinamiche

Ogni istanza dell'app Web userà un metodo diverso per la connessione al database SQL. L'app in Azure usa l'indirizzo IP privato della macchina virtuale di SQL Server, mentre l'app nell'hub di Azure Stack usa l'indirizzo IP pubblico della macchina virtuale di SQL Server.

> [!Note]  
> In un sistema integrato dell'hub di Azure Stack, l'indirizzo IP pubblico non deve essere instradabile su Internet. In un ASDK, l'indirizzo IP pubblico non è instradabile all'esterno dell'ASDK.

È possibile usare le variabili di ambiente del servizio app per passare una stringa di connessione diversa a ogni istanza dell'app.

1. Aprire l'app in Visual Studio.

2. Aprire Startup.cs e trovare il blocco di codice seguente:

    ```C#
    services.AddDbContext<MyDatabaseContext>(options =>
        options.UseSqlite("Data Source=localdatabase.db"));
    ```

3. Sostituire il blocco di codice precedente con il codice seguente che usa una stringa di connessione definita nel file *appsettings.json* :

    ```C#
    services.AddDbContext<MyDatabaseContext>(options =>
        options.UseSqlServer(Configuration.GetConnectionString("MyDbConnection")));
     // Automatically perform database migration
     services.BuildServiceProvider().GetService<MyDatabaseContext>().Database.Migrate();
    ```

### <a name="configure-app-service-app-settings"></a>Configurare le impostazioni dell'app Servizio app di Azure

1. Creare stringhe di connessione per Azure e per l'hub di Azure Stack. Le stringhe devono essere uguali, ad eccezione degli indirizzi IP usati.

2. In Azure e nell'hub di Azure Stack aggiungere la stringa di connessione appropriata [come impostazione dell'app](/azure/app-service/web-sites-configure) nell'app Web usando `SQLCONNSTR\_` come prefisso nel nome.

3. **Salvare** le impostazioni dell'app Web e riavviare l'app.

## <a name="enable-automatic-scaling-in-global-azure"></a>Abilitare la scalabilità automatica in Azure globale

Quando si crea l'app Web in un ambiente del servizio app, il servizio viene avviato con una istanza. È quindi possibile aumentare automaticamente il numero di istanze per offrire più risorse di calcolo per l'app. Analogamente, è possibile ridurre automaticamente il numero di istanze necessarie per l'app.

> [!Note]  
> È necessario avere un piano di servizio app per configurare l'aumento e la riduzione delle istanze. Se non si ha un piano, crearne uno prima di eseguire i passaggi successivi.

### <a name="enable-automatic-scale-out"></a>Abilitare l'aumento automatico di istanze

1. Nel portale di Azure individuare il piano di servizio app per i siti di cui aumentare il numero di istanze, quindi selezionare **Aumenta istanze (piano di servizio app)** .

    ![Aumentare le istanze del Servizio app di Azure](media/solution-deployment-guide-hybrid/image16.png)

2. Selezionare **Abilita scalabilità automatica** .

    ![Abilitare la scalabilità automatica nel servizio app di Azure](media/solution-deployment-guide-hybrid/image17.png)

3. Immettere un nome per **Nome impostazione di scalabilità automatica** . Per il valore della regola di scalabilità automatica **Predefinito** , selezionare **Ridimensiona in base a una metrica** . Impostare **Limiti per le istanze** su **Minimo: 1** , **Massimo: 10** e **Predefinito: 1** .

    ![Configurare la scalabilità automatica nel servizio app di Azure](media/solution-deployment-guide-hybrid/image18.png)

4. Selezionare **+Aggiungi una regola** .

5. In **Origine metrica** selezionare **Risorsa corrente** . Usare i criteri e le azioni seguenti per la regola.

#### <a name="criteria"></a>Criteri

1. In **Aggregazione temporale** selezionare **Medio** .

2. In **Nome metrica** selezionare **Percentuale CPU** .

3. In **Operatore** selezionare **Maggiore di** .

   - Impostare la **Soglia** su **50** .
   - Impostare la **Durata** su **10** .

#### <a name="action"></a>Azione

1. In **Operazione** selezionare **Aumenta numero di** .

2. Impostare il **Numero di istanze** su **2** .

3. Impostare **Disattiva regole dopo** su **5** .

4. Selezionare **Aggiungi** .

5. Selezionare **+ Aggiungi una regola** .

6. In **Origine metrica** selezionare **Risorsa corrente** .

   > [!Note]  
   > La risorsa corrente conterrà il nome/GUID del piano di servizio app e gli elenchi a discesa **Tipo di risorsa** e **Risorsa** non saranno disponibili.

### <a name="enable-automatic-scale-in"></a>Abilitare la riduzione automatica di istanze

Quando il traffico diminuisce, l'app Web di Azure può ridurre automaticamente il numero di istanze attive per ridurre i costi. Questa azione è meno aggressiva rispetto allo scale-out e riduce al minimo l'effetto sugli utenti dell'app.

1. Passare alla condizione di scale-out **Predefinito** , quindi selezionare **+ Aggiungi una regola** . Usare i criteri e le azioni seguenti per la regola.

#### <a name="criteria"></a>Criteri

1. In **Aggregazione temporale** selezionare **Medio** .

2. In **Nome metrica** selezionare **Percentuale CPU** .

3. In **Operatore** selezionare **Minore di** .

   - Impostare la **Soglia** su **30** .
   - Impostare la **Durata** su **10** .

#### <a name="action"></a>Azione

1. In **Operazione** selezionare **Riduci numero di** .

   - Impostare il **Numero di istanze** su **1** .
   - Impostare **Disattiva regole dopo** su **5** .

2. Selezionare **Aggiungi** .

## <a name="create-a-traffic-manager-profile-and-configure-cross-cloud-scaling"></a>Creare un profilo di Gestione traffico e configurare il ridimensionamento tra cloud

Creare un profilo di Gestione traffico usando il portale di Azure, quindi configurare l'endpoint per abilitare il dimensionamento tra cloud.

### <a name="create-traffic-manager-profile"></a>Creare un profilo di Gestione traffico

1. Selezionare **Crea una risorsa** .
2. Selezionare **Rete** .
3. Selezionare **Profilo di Gestione traffico** e configurare le impostazioni seguenti:

   - In **Nome** immettere un nome per il profilo. Questo nome **deve** essere univoco nella zona trafficmanager.net e viene usato per creare un nuovo nome DNS (ad esempio, northwindstore.trafficmanager.net).
   - Per **Metodo di routing** selezionare **Ponderato** .
   - Per **Sottoscrizione** selezionare la sottoscrizione in cui si vuole creare il profilo.
   - In **Gruppo di risorse** creare un nuovo gruppo di risorse per il profilo.
   - In **Località del gruppo di risorse** selezionare la località del gruppo di risorse. Questa impostazione indica la posizione del gruppo di risorse e non ha alcun impatto sul profilo di Gestione traffico che viene distribuito a livello globale.

4. Selezionare **Crea** .

    ![Creare un profilo di Gestione traffico](media/solution-deployment-guide-hybrid/image19.png)

   Dopo aver completato la distribuzione globale del profilo di Gestione traffico, il profilo viene visualizzato nell'elenco delle risorse del gruppo di risorse in cui è stato creato.

### <a name="add-traffic-manager-endpoints"></a>Aggiungere endpoint di Gestione traffico

1. Cercare il profilo di Gestione traffico creato. Se si è passati al gruppo di risorse per il profilo, selezionare il profilo.

2. In **Profilo di Gestione traffico** in **IMPOSTAZIONI** selezionare **Endpoint** .

3. Selezionare **Aggiungi** .

4. In **Aggiungi endpoint** usare le impostazioni seguenti per l'hub di Azure Stack:

   - Per **Tipo** selezionare **Endpoint esterno** .
   - Immettere un **Nome** per l'endpoint.
   - Per **Nome di dominio completo (FQDN) o indirizzo IP** immettere l'URL esterno dell'app Web dell'hub di Azure Stack.
   - Per **Peso** mantenere l'impostazione predefinita **1** . In questo modo tutto il traffico viene instradato a questo endpoint se è integro.
   - Mantenere deselezionata l'opzione **Aggiungi come disabilitato** .

5. Selezionare **OK** per salvare l'endpoint dell'hub di Azure Stack.

L'endpoint di Azure verrà configurato successivamente.

1. In **Profilo di Gestione traffico** selezionare **Endpoint** .
2. Selezionare **+Aggiungi** .
3. In **Aggiungi endpoint** usare le impostazioni seguenti per Azure:

   - Per **Tipo** selezionare **Endpoint Azure** .
   - Immettere un **Nome** per l'endpoint.
   - Per **Tipo di risorsa di destinazione** selezionare **Servizio app** .
   - Per **Risorsa di destinazione** selezionare **Scegliere un servizio app** per visualizzare un elenco di app Web nella stessa sottoscrizione.
   - In **Risorsa** selezionare il servizio App che si vuole aggiungere come primo endpoint.
   - Per **Peso** selezionare **2** . Questa impostazione determina l'indirizzamento di tutto il traffico a questo endpoint se l'endpoint primario non è integro o se è presente una regola o un avviso che reindirizza il traffico quando viene attivato.
   - Mantenere deselezionata l'opzione **Aggiungi come disabilitato** .

4. Selezionare **OK** per salvare l'endpoint di Azure.

Dopo aver configurato entrambi gli endpoint, gli endpoint vengono elencati in **Profilo di Gestione traffico** quando si seleziona **Endpoint** . L'esempio dello screenshot seguente mostra due endpoint con le relative informazioni sullo stato e sulla configurazione.

![Endpoint nel profilo di Gestione traffico](media/solution-deployment-guide-hybrid/image20.png)

## <a name="set-up-application-insights-monitoring-and-alerting-in-azure"></a>Configurare il monitoraggio e gli avvisi di Application Insights in Azure

Azure Application Insights consente di monitorare l'app e inviare avvisi in base alle condizioni configurate. Alcuni esempi sono: l'app non è disponibile, sta riscontrando errori o presenta problemi di prestazioni.

Per creare gli avvisi si useranno le metriche di Azure Application Insights. Quando si attivano questi avvisi, l'istanza dell'app Web passa automaticamente dall'hub di Azure Stack ad Azure per l'aumento delle istanze e quindi torna all'hub di Azure Stack per la riduzione.

### <a name="create-an-alert-from-metrics"></a>Creare un avviso dalle metriche

Nel portale di Azure passare al gruppo di risorse di questa esercitazione e quindi selezionare l'istanza di Application Insights per aprire **Application Insights** .

![Application Insights](media/solution-deployment-guide-hybrid/image21.png)

Questa visualizzazione verrà usata per creare un avviso di aumento delle istanze e un avviso di riduzione delle istanze.

### <a name="create-the-scale-out-alert"></a>Creare l'avviso di aumento delle istanze

1. In **CONFIGURA** selezionare **Avvisi (versione classica)** .
2. Selezionare **Aggiungi avviso per la metrica (versione classica)** .
3. In **Aggiungi regola** configurare le impostazioni seguenti:

   - Per **Nome** immettere **Entra in Azure Cloud** .
   - La **Descrizione** è facoltativa.
   - In **Origine** > **Avviso per** selezionare **Metriche** .
   - In **Criteri** selezionare la sottoscrizione, il gruppo di risorse per il profilo di Gestione traffico e il nome del profilo di Gestione traffico per la risorsa.

4. Per **Metrica** selezionare **Velocità richiesta** .
5. Per **Condizione** selezionare **Maggiore di** .
6. Per **Soglia** immettere **2** .
7. Per **Periodo** selezionare **Negli ultimi 5 minuti** .
8. In **Notifica tramite** :
   - Selezionare la casella di controllo **Invia messaggio di posta elettronica a proprietari, collaboratori e lettori** .
   - Immettere l'indirizzo e-mail per **Indirizzi di posta elettronica aggiuntivi dell'amministratore** .

9. Nella barra dei menu selezionare **Salva** .

### <a name="create-the-scale-in-alert"></a>Creare l'avviso di riduzione delle istanze

1. In **CONFIGURA** selezionare **Avvisi (versione classica)** .
2. Selezionare **Aggiungi avviso per la metrica (versione classica)** .
3. In **Aggiungi regola** configurare le impostazioni seguenti:

   - Per **Nome** immettere **Torna all'hub di Azure Stack** .
   - La **Descrizione** è facoltativa.
   - In **Origine** > **Avviso per** selezionare **Metriche** .
   - In **Criteri** selezionare la sottoscrizione, il gruppo di risorse per il profilo di Gestione traffico e il nome del profilo di Gestione traffico per la risorsa.

4. Per **Metrica** selezionare **Velocità richiesta** .
5. Per **Condizione** selezionare **Minore di** .
6. Per **Soglia** immettere **2** .
7. Per **Periodo** selezionare **Negli ultimi 5 minuti** .
8. In **Notifica tramite** :
   - Selezionare la casella di controllo **Invia messaggio di posta elettronica a proprietari, collaboratori e lettori** .
   - Immettere l'indirizzo e-mail per **Indirizzi di posta elettronica aggiuntivi dell'amministratore** .

9. Nella barra dei menu selezionare **Salva** .

Lo screenshot seguente mostra gli avvisi per l'aumento e la riduzione di istanze.

   ![Avvisi di Application Insights (versione classica)](media/solution-deployment-guide-hybrid/image22.png)

## <a name="redirect-traffic-between-azure-and-azure-stack-hub"></a>Reindirizzare il traffico tra Azure e l'hub di Azure Stack

È possibile configurare la commutazione manuale o automatica del traffico delle app Web tra Azure e l'hub di Azure Stack.

### <a name="configure-manual-switching-between-azure-and-azure-stack-hub"></a>Configurare la commutazione manuale tra Azure e l'hub di Azure Stack

Quando il sito Web raggiunge le soglie configurate, si riceverà un avviso. Per reindirizzare manualmente il traffico ad Azure, eseguire i passaggi seguenti.

1. Nel portale di Azure selezionare il profilo di Gestione traffico.

    ![Endpoint di Gestione traffico nel portale di Azure](media/solution-deployment-guide-hybrid/image20.png)

2. Selezionare **Endpoint** .
3. Selezionare l' **Endpoint Azure** .
4. In **Stato** selezionare **Abilitato** e quindi selezionare **Salva** .

    ![Abilitare l'endpoint di Azure nel portale di Azure](media/solution-deployment-guide-hybrid/image23.png)

5. In **Endpoint** per il profilo di Gestione traffico selezionare **Endpoint esterno** .
6. In **Stato** selezionare **Disabilitato** e quindi selezionare **Salva** .

    ![Disabilitare l'endpoint dell'hub di Azure Stack nel portale di Azure](media/solution-deployment-guide-hybrid/image24.png)

Dopo aver configurato gli endpoint, il traffico delle app viene indirizzato all'app Web di cui sono state aumentate le istanze anziché all'app Web dell'hub di Azure Stack.

 ![Endpoint modificati nel traffico dell'app Web di Azure](media/solution-deployment-guide-hybrid/image25.png)

Per invertire di nuovo il flusso verso l'hub di Azure Stack, eseguire i passaggi precedenti per:

- Abilitare l'endpoint dell'hub di Azure Stack.
- Disabilitare l'endpoint di Azure.

### <a name="configure-automatic-switching-between-azure-and-azure-stack-hub"></a>Configurare la commutazione automatica tra Azure e l'hub di Azure Stack

È anche possibile usare il monitoraggio di Application Insights se l'app è in esecuzione in un ambiente [serverless](https://azure.microsoft.com/overview/serverless-computing/) fornito da Funzioni di Azure.

In questo scenario è possibile configurare Application Insights per l'uso di un webhook che chiama un'app per le funzioni. Questa app abilita o disabilita automaticamente un endpoint in risposta a un avviso.

Usare i passaggi seguenti come guida per configurare la commutazione automatica del traffico.

1. Creare un'app per le funzioni di Azure.
2. Creare una funzione attivata da HTTP.
3. Importare gli SDK di Azure per Resource Manager, App Web e Gestione traffico.
4. Sviluppare codice per:

   - Eseguire l'autenticazione nella sottoscrizione di Azure.
   - Usare un parametro che commuta gli endpoint di Gestione traffico per indirizzare il traffico ad Azure o all'hub di Azure Stack.

5. Salvare il codice e aggiungere l'URL dell'app per le funzioni con i parametri appropriati alla sezione **Webhook** delle impostazioni delle regole degli avvisi di Application Insights.
6. Il traffico viene reindirizzato automaticamente quando viene attivato un avviso di Application Insights.

## <a name="next-steps"></a>Passaggi successivi

- Per altre informazioni sui modelli cloud di Azure, vedere [Modelli di progettazione cloud](/azure/architecture/patterns).
