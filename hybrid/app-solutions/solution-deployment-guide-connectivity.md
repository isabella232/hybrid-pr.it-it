---
title: Configurare la connettività di cloud ibrido in Azure e nell'hub di Azure Stack
description: Informazioni su come configurare la connettività di cloud ibrido usando Azure e l'hub di Azure Stack.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 16c5d7820e8c865a9f88cb00da5cc7c854379414
ms.sourcegitcommit: d2def847937178f68177507be151df2aa8e25d53
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 07/20/2020
ms.locfileid: "86477287"
---
# <a name="configure-hybrid-cloud-connectivity-using-azure-and-azure-stack-hub"></a>Configurare la connettività di cloud ibrido usando Azure e l'hub di Azure Stack

È possibile accedere alle risorse con sicurezza in Azure globale e nell'hub di Azure Stack usando il modello di connettività ibrida.

In questa soluzione si compilerà un ambiente di esempio per:

> [!div class="checklist"]
> - Mantenere i dati in locale per soddisfare i requisiti normativi o relativi alla privacy, mantenendo comunque l'accesso alle risorse globali di Azure.
> - Gestire un sistema legacy usando funzioni di distribuzione di app e risorse a livello di cloud in Azure globale.

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> L'hub di Microsoft Azure Stack è un'estensione di Azure che offre all'ambiente locale l'agilità e l'innovazione del cloud computing, abilitando l'unico cloud ibrido che consente di creare e distribuire ovunque app ibride.  
> 
> L'articolo [Considerazioni per la progettazione di app ibride](overview-app-design-considerations.md) esamina i concetti fondamentali di qualità del software (posizionamento, scalabilità, disponibilità, resilienza, gestibilità e sicurezza) per la progettazione, la distribuzione e la gestione di app ibride. Le considerazioni di progettazione consentono di ottimizzare la progettazione delle app ibride, riducendo al minimo i rischi negli ambienti di produzione.

## <a name="prerequisites"></a>Prerequisiti

Per creare una distribuzione di connettività ibrida sono necessari alcuni componenti. La preparazione di alcuni di questi richiede tempo ed è quindi necessario pianificare di conseguenza.

### <a name="azure"></a>Azure

- Se non si ha una sottoscrizione di Azure, creare un [account gratuito](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) prima di iniziare.
- Creare un'[app Web](/vsts/build-release/apps/cd/azure/aspnet-core-to-azure-webapp?tabs=vsts&view=vsts) in Azure. Prendere nota dell'URL dell'app Web perché sarà necessaria nella soluzione.

### <a name="azure-stack-hub"></a>Hub di Azure Stack

Un partner OEM/hardware di Azure può distribuire un hub di Azure Stack di produzione e tutti gli utenti possono distribuire un Azure Stack Development Kit (ASDK).

- Usare l'hub di Azure Stack di produzione o distribuire l'ASDK.
   >[!Note]
   >La distribuzione dell'ASDK può richiedere fino a 7 ore. È quindi necessario pianificare di conseguenza.

- Distribuire i servizi PaaS del [servizio app](/azure-stack/operator/azure-stack-app-service-deploy.md) nell'hub di Azure Stack.
- [Creare piani e offerte](/azure-stack/operator/service-plan-offer-subscription-overview.md) nell'ambiente dell'hub di Azure Stack.
- [Creare una sottoscrizione tenant](/azure-stack/operator/azure-stack-subscribe-plan-provision-vm.md) all'interno dell'ambiente dell'hub di Azure Stack.

### <a name="azure-stack-hub-components"></a>Componenti dell'hub di Azure Stack

Un operatore dell'hub di Azure Stack deve distribuire il servizio app, creare piani e offerte, creare una sottoscrizione tenant e aggiungere l'immagine di Windows Server 2016. Se questi componenti sono già disponibili, prima di avviare questa soluzione assicurarsi che soddisfino i requisiti.

Questa soluzione di esempio presuppone che l'utente abbia una certa conoscenza di base di Azure e dell'hub di Azure Stack. Per altre informazioni prima di avviare la soluzione, vedere gli articoli seguenti:

- [Introduzione ad Azure](https://azure.microsoft.com/overview/what-is-azure/)
- [Concetti chiave dell'hub di Azure Stack](/azure-stack/operator/azure-stack-overview.md)

### <a name="before-you-begin"></a>Prima di iniziare

Prima di iniziare la configurazione della connettività di cloud ibrido, verificare che siano soddisfatti i criteri seguenti:

- Disponibilità di un indirizzo IPv4 pubblico esterno per il dispositivo VPN. Questo indirizzo IP non può trovarsi dietro un dispositivo NAT (Network Address Translation).
- Distribuzione di tutte le risorse nella stessa area/posizione.

#### <a name="solution-example-values"></a>Valori degli esempi della soluzione

Gli esempi in questa soluzione usano i valori seguenti. È possibile usare questi valori per creare un ambiente di test o farvi riferimento per comprendere meglio gli esempi. Per altre informazioni sulle impostazioni di un gateway VPN, vedere [Informazioni sulle impostazioni del gateway VPN](/azure/vpn-gateway/vpn-gateway-about-vpn-gateway-settings).

Specifiche della connessione:

- **Tipo VPN**: Basato su route
- **Tipo di connessione**: Da sito a sito (IPSec)
- **Tipo di gateway**: VPN
- **Nome connessione di Azure**: Azure-Gateway-AzureStack-S2SGateway (il portale inserisce questo valore automaticamente)
- **Nome connessione dell'hub di Azure Stack**: AzureStack-Gateway-Azure-S2SGateway (il portale inserisce questo valore automaticamente)
- **Chiave condivisa**: qualsiasi valore compatibile con l'hardware VPN, con valori corrispondenti in entrambi i lati della connessione
- **Sottoscrizione**: qualsiasi sottoscrizione preferita
- **Gruppo di risorse**: Test-Infra

Indirizzi IP di rete e subnet:

| Connessione di Azure/dell'hub di Azure Stack | Nome | Subnet | Indirizzo IP |
|---|---|---|---|
| Rete virtuale di Azure | ApplicationvNet<br>10.100.102.9/23 | ApplicationSubnet<br>10.100.102.0/24 |  |
|  |  | GatewaySubnet<br>10.100.103.0/24 |  |
| Rete virtuale dell'hub di Azure Stack | ApplicationvNet<br>10.100.100.0/23 | ApplicationSubnet <br>10.100.100.0/24 |  |
|  |  | GatewaySubnet <br>10.100101.0/24 |  |
| Gateway di rete virtuale di Azure | Azure-Gateway |  |  |
| Gateway di rete virtuale dell'hub di Azure Stack | AzureStack-Gateway |  |  |
| IP pubblico di Azure | Azure-GatewayPublicIP |  | Determinato al momento della creazione |
| IP pubblico dell'hub di Azure Stack | AzureStack-GatewayPublicIP |  | Determinato al momento della creazione |
| Gateway di rete locale di Azure | AzureStack-S2SGateway<br>   10.100.100.0/23 |  | Valore dell'IP pubblico dell'hub di Azure Stack |
| Gateway di rete locale dell'hub di Azure Stack | Azure-S2SGateway<br>10.100.102.0/23 |  | Valore dell'IP pubblico di Azure |

## <a name="create-a-virtual-network-in-global-azure-and-azure-stack-hub"></a>Creare una rete virtuale in Azure globale e nell'hub di Azure Stack

Usare la procedura seguente per creare una rete virtuale tramite il portale. È possibile usare questi [valori di esempio](/azure/vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal#values) se si usa questo articolo come soluzione singola. Se si usa questo articolo per configurare un ambiente di produzione, sostituire le impostazioni di esempio con i propri valori.

> [!IMPORTANT]
> È necessario assicurarsi che non si verifichi una sovrapposizione di indirizzi IP negli spazi indirizzi della rete virtuale di Azure o dell'hub di Azure Stack.

Per creare una rete virtuale in Azure:

1. Tramite il browser connettersi al [portale di Azure](https://portal.azure.com/) e accedere con l'account Azure.
2. Selezionare **Crea una risorsa**. Nel campo **Cerca nel Marketplace** immettere 'rete virtuale'. Selezionare **Rete virtuale** nei risultati.
3. Nell'elenco **Selezionare un modello di distribuzione** selezionare **Resource Manager** e quindi selezionare **Crea**.
4. In **Crea rete virtuale** configurare le impostazioni della rete virtuale. I nomi dei campi obbligatori sono preceduti da un asterisco rosso.  Quando si immette un valore valido, l'asterisco diventa un segno di spunta verde.

Per creare una rete virtuale nell'hub di Azure Stack:

1. Ripetere i passaggi precedenti (1-4) usando il **portale del tenant** dell'hub di Azure Stack.

## <a name="add-a-gateway-subnet"></a>Aggiungere una subnet del gateway

Prima di connettere la rete virtuale a un gateway, è necessario creare la subnet del gateway per la rete virtuale con cui si vuole stabilire la connessione. I servizi del gateway usano gli indirizzi IP specificati nella subnet del gateway.

Nel [portale di Azure](https://portal.azure.com/) passare alla rete virtuale di Resource Manager in cui si vuole creare un gateway di rete virtuale.

1. Selezionare la rete virtuale per aprire la pagina **Rete virtuale**.
2. In **IMPOSTAZIONI** selezionare **Subnet**.
3. Nella pagina **Subnet** selezionare **+ Subnet del gateway** per aprire la pagina **Aggiungi subnet**.

    ![Aggiungere la subnet del gateway](media/solution-deployment-guide-connectivity/image4.png)

4. Il **nome** della subnet viene compilato automaticamente con il valore 'GatewaySubnet'. Questo valore è obbligatorio per consentire ad Azure di riconoscere la subnet come subnet del gateway.
5. Modificare i valori di **Intervallo indirizzi**  specificati in modo che corrispondano ai requisiti di configurazione e quindi selezionare **OK**.

## <a name="create-a-virtual-network-gateway-in-azure-and-azure-stack"></a>Creare un gateway di rete virtuale in Azure e nell'hub di Azure Stack

Usare la procedura seguente per creare un gateway di rete virtuale in Azure.

1. Nel lato sinistro della pagina del portale selezionare **+** e immettere 'gateway di rete virtuale' nel campo di ricerca.
2. In **Risultati** selezionare **Gateway di rete virtuale**.
3. In **Gateway di rete virtuale** selezionare **Crea** per aprire la pagina **Crea gateway di rete virtuale**.
4. In **Crea gateway di rete virtuale** specificare i valori per il gateway di rete virtuale usando i **valori di esempio dell'esercitazione**. Includere i valori aggiuntivi seguenti:

   - **SKU**: Basic
   - **Rete virtuale**: selezionare la rete virtuale creata in precedenza. La subnet del gateway creata viene selezionata automaticamente.
   - **Prima configurazione IP**:  IP pubblico del gateway.
     - Selezionare **Crea configurazione IP gateway**. Si passerà alla pagina **Scegli indirizzo IP pubblico**.
     - Selezionare **+ Crea nuovo** per aprire la pagina **Crea indirizzo IP pubblico**.
     - Immettere un **nome** per l'indirizzo IP pubblico. Mantenere lo SKU **Basic** e quindi selezionare **OK** per salvare le modifiche.

       > [!Note]
       > Gateway VPN supporta attualmente solo l'allocazione degli indirizzi IP pubblici dinamici. Questo tuttavia non significa che l'indirizzo IP cambi dopo l'assegnazione al gateway VPN. L'unica volta in cui l'indirizzo IP pubblico viene modificato è quando il gateway viene eliminato e creato di nuovo. In caso di ridimensionamento, reimpostazione o altre operazioni di manutenzione o aggiornamento del gateway VPN, l'indirizzo IP non cambia.

5. Verificare le impostazioni del gateway.
6. Selezionare **Crea** per creare il gateway VPN. Le impostazioni del gateway vengono convalidate e nel dashboard viene visualizzato il riquadro "Distribuzione di Gateway di rete virtuale".

   >[!Note]
   >La creazione di un gateway può richiedere fino a 45 minuti. Potrebbe essere necessario aggiornare la pagina del portale per visualizzare lo stato di completamento.

    Dopo la creazione del gateway, è possibile visualizzare l'indirizzo IP assegnato a questo esaminando la rete virtuale nel portale. Il gateway viene visualizzato come un dispositivo connesso. Per altre informazioni sul gateway, selezionare il dispositivo.

7. Ripetere i passaggi precedenti (1-5) nella distribuzione dell'hub di Azure Stack.

## <a name="create-the-local-network-gateway-in-azure-and-azure-stack-hub"></a>Creare il gateway di rete locale in Azure e nell'hub di Azure Stack

Il gateway di rete locale in genere fa riferimento al percorso locale. Assegnare al sito un nome a cui Azure o l'hub di Azure Stack possa fare riferimento e quindi specificare:

- L'indirizzo IP del dispositivo VPN locale per cui si sta creando una connessione.
- I prefissi degli indirizzi IP che verranno instradati tramite il gateway VPN al dispositivo VPN. I prefissi degli indirizzi specificati sono quelli disponibili nella rete locale.

  >[!Note]
  >Se la rete locale cambia o è necessario modificare l'indirizzo IP pubblico del dispositivo VPN, è possibile aggiornare questi valori in un secondo momento.

1. Nel portale selezionare **+ Crea una risorsa**.
2. Nella casella di ricerca immettere **Gateway di rete locale** e quindi selezionare **Invio** per eseguire la ricerca. Verrà visualizzato un elenco di risultati.
3. Selezionare **Gateway di rete locale** e quindi selezionare **Crea** per aprire la pagina **Crea un gateway di rete locale**.
4. In **Crea un gateway di rete locale** specificare i valori per il gateway di rete locale usando i **valori di esempio dell'esercitazione**. Includere i valori aggiuntivi seguenti:

    - **Indirizzo IP**: l'indirizzo IP pubblico del dispositivo VPN a cui si vuole connettere Azure o l'hub di Azure Stack. Specificare un indirizzo IP pubblico valido che non si trovi dietro un dispositivo NAT, in modo che Azure possa raggiungere l'indirizzo. Se non si ha ancora a disposizione l'indirizzo IP, è possibile usare un valore dell'esempio come segnaposto. Sarà necessario tornare indietro e sostituire il segnaposto con l'indirizzo IP pubblico del dispositivo VPN. Azure sarà in grado di connettersi al dispositivo solo quando verrà specificato un indirizzo valido.
    - **Spazio di indirizzi**: intervallo di indirizzi per la rete rappresentata da questa rete locale. È possibile aggiungere più intervalli di spazi indirizzi. Assicurarsi che gli intervalli specificati non si sovrappongano con gli intervalli di altre reti a cui ci si vuole connettere. Azure indirizzerà l'intervallo di indirizzi specificato all'indirizzo IP del dispositivo VPN locale. Usare valori personalizzati, non un valore di esempio, se ci si vuole connettere al sito locale.
    - **Configura le impostazioni BGP:** usare solo quando si configura BGP. In caso contrario, non selezionare questa opzione.
    - **Sottoscrizione** verificare che sia visualizzata la sottoscrizione corretta.
    - **Gruppo di risorse**: selezionare il gruppo di risorse che si vuole usare. È possibile creare un nuovo gruppo di risorse o selezionarne uno già creato.
    - **Località**: selezionare la località in cui verrà creato questo oggetto. È possibile, ma non necessario, selezionare la stessa località in cui risiede la rete virtuale.
5. Dopo aver specificato i valori obbligatori, selezionare **Crea** per creare il gateway di rete locale.
6. Ripetere questi passaggi (1-5) nella distribuzione dell'hub di Azure Stack.

## <a name="configure-your-connection"></a>Configurare la connessione

Le connessioni da sito a sito verso una rete locale richiedono un dispositivo VPN. Il dispositivo VPN configurato viene detto connessione. Per configurare la connessione, sono necessari:

- Chiave condivisa. Si tratta della stessa chiave condivisa specificata durante la creazione della connessione VPN da sito a sito. In questi esempi viene usata una chiave condivisa semplice. È consigliabile generare una chiave più complessa per l'uso effettivo.
- Indirizzo IP pubblico del gateway di rete virtuale. È possibile visualizzare l'indirizzo IP pubblico usando il portale di Azure, PowerShell o l'interfaccia della riga di comando. Per trovare l'indirizzo IP pubblico del gateway VPN usando il portale di Azure, passare a Gateway di rete virtuale e quindi selezionare il nome del gateway.

Usare la procedura seguente per creare una connessione VPN da sito a sito tra il gateway di rete virtuale e il dispositivo VPN locale.

1. Nel portale di Azure selezionare **+ Crea una risorsa**.
2. Cercare **connessioni**.
3. In **Risultati**  selezionare **Connessioni**.
4. In **Connessione** selezionare **Crea**.
5. In **Crea connessione** configurare le impostazioni seguenti:

    - **Tipo di connessione**: selezionare Da sito a sito (IPSec).
    - **Gruppo di risorse**: selezionare il gruppo di risorse di test.
    - **Gateway di rete virtuale**: selezionare il gateway di rete virtuale creato.
    - **Gateway di rete locale**: selezionare il gateway di rete locale creato.
    - **Nome connessione**: questo nome viene popolato automaticamente con i valori dei due gateway.
    - **Chiave condivisa**: questo valore deve corrispondere a quello usato per il dispositivo VPN locale. L'esempio dell'esercitazione usa 'abc123', ma è consigliabile usare un valore più complesso. L'aspetto importante è che questo valore *deve* corrispondere al valore specificato durante la configurazione del dispositivo VPN.
    - I valori di **Sottoscrizione**, **Gruppo di risorse** e **Località** sono fissi.

6. Selezionare **OK** per creare la connessione.

È possibile visualizzare la connessione nella pagina **Connessioni** relativa al gateway di rete virtuale. Lo stato passa da *Sconosciuto* a *Connessione* e quindi a *Operazione completata*.

## <a name="next-steps"></a>Passaggi successivi

- Per altre informazioni sui modelli cloud di Azure, vedere [Modelli di progettazione cloud](/azure/architecture/patterns).
