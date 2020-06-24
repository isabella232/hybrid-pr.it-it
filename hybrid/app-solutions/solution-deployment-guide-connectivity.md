---
title: Configurare la connettività cloud ibrida in Azure e Azure Stack Hub
description: Informazioni su come configurare la connettività cloud ibrida con Azure e l'hub Azure Stack.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 0e1a0fc4fb4110fdb406d4b4b2e72abb8f5412c9
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910866"
---
# <a name="configure-hybrid-cloud-connectivity-using-azure-and-azure-stack-hub"></a>Configurare la connettività cloud ibrida con Azure e l'hub Azure Stack

È possibile accedere alle risorse con sicurezza nell'hub globale di Azure e Azure Stack usando il modello di connettività ibrida.

In questa soluzione verrà compilato un ambiente di esempio per:

> [!div class="checklist"]
> - Mantieni i dati locali per soddisfare i requisiti di privacy o normativi, ma Mantieni l'accesso alle risorse globali di Azure.
> - Gestione di un sistema legacy con la distribuzione di app con scalabilità cloud e risorse in Azure globale.

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub è un'estensione di Azure. Azure Stack Hub offre l'agilità e l'innovazione di cloud computing all'ambiente locale, abilitando l'unico Cloud ibrido che consente di creare e distribuire app ibride ovunque.  
> 
> L'articolo [considerazioni sulla progettazione di app ibride](overview-app-design-considerations.md) esamina i pilastri della qualità del software (posizionamento, scalabilità, disponibilità, resilienza, gestibilità e sicurezza) per la progettazione, la distribuzione e la gestione di app ibride. Le considerazioni di progettazione consentono di ottimizzare la progettazione delle app ibride, riducendo al minimo le esigenze negli ambienti di produzione.

## <a name="prerequisites"></a>Prerequisiti

Per creare una distribuzione di connettività ibrida sono necessari alcuni componenti. Alcuni di questi componenti hanno tempo per prepararsi, quindi pianificare di conseguenza.

### <a name="azure"></a>Azure

- Se non si ha una sottoscrizione di Azure, creare un [account gratuito](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) prima di iniziare.
- Creare un' [app Web](https://docs.microsoft.com/vsts/build-release/apps/cd/azure/aspnet-core-to-azure-webapp?view=vsts&tabs=vsts) in Azure. Prendere nota dell'URL dell'app Web perché sarà necessario nella soluzione.

### <a name="azure-stack-hub"></a>Hub di Azure Stack

Un partner Azure OEM/hardware può distribuire un hub Azure Stack di produzione e tutti gli utenti possono distribuire una Azure Stack Development Kit (Gabriele).

- Usare l'hub Azure Stack di produzione o distribuire Gabriele.
   >[!Note]
   >La distribuzione di Gabriele può richiedere fino a 7 ore, quindi pianificare di conseguenza.

- Distribuire i servizi PaaS del [servizio app](/azure-stack/operator/azure-stack-app-service-deploy.md) nell'hub Azure stack.
- [Creare piani e offerte](/azure-stack/operator/service-plan-offer-subscription-overview.md) nell'ambiente dell'hub Azure stack.
- [Creare una sottoscrizione tenant](/azure-stack/operator/azure-stack-subscribe-plan-provision-vm.md) all'interno dell'ambiente Azure stack Hub.

### <a name="azure-stack-hub-components"></a>Componenti dell'hub Azure Stack

Un operatore Azure Stack hub deve distribuire il servizio app, creare piani e offerte, creare una sottoscrizione tenant e aggiungere l'immagine di Windows Server 2016. Se si hanno già questi componenti, assicurarsi che soddisfino i requisiti prima di avviare questa soluzione.

Questo esempio di soluzione presuppone che si disponga di una conoscenza di base di Azure e dell'hub Azure Stack. Per altre informazioni prima di avviare la soluzione, vedere gli articoli seguenti:

- [Introduzione ad Azure](https://azure.microsoft.com/overview/what-is-azure/)
- [Concetti chiave dell'hub Azure Stack](/azure-stack/operator/azure-stack-overview.md)

### <a name="before-you-begin"></a>Prima di iniziare

Prima di iniziare la configurazione della connettività del cloud ibrido, verificare che siano soddisfatti i criteri seguenti:

- Per il dispositivo VPN è necessario un indirizzo IPv4 pubblico con connessione esterna. Questo indirizzo IP non può trovarsi dietro un NAT (Network Address Translation).
- Tutte le risorse vengono distribuite nella stessa area/località.

#### <a name="solution-example-values"></a>Valori di esempio della soluzione

Gli esempi in questa soluzione utilizzano i valori seguenti. È possibile utilizzare questi valori per creare un ambiente di testing o farvi riferimento per comprendere meglio gli esempi. Per altre informazioni sulle impostazioni del gateway VPN, vedere [informazioni sulle impostazioni del gateway VPN](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-about-vpn-gateway-settings).

Specifiche di connessione:

- **Tipo VPN**: basato su Route
- **Tipo di connessione**: da sito a sito (IPSec)
- **Tipo di gateway**: VPN
- **Nome della connessione di Azure**: Azure-gateway-AzureStack-S2SGateway (il portale effettuerà il riempimento automatico di questo valore)
- **Nome della connessione dell'Hub Azure stack**: AzureStack-gateway-Azure-S2SGateway (il portale effettuerà il riempimento automatico di questo valore)
- **Chiave condivisa**: qualsiasi compatibile con l'hardware VPN, con valori corrispondenti su entrambi i lati della connessione
- **Sottoscrizione**: qualsiasi sottoscrizione preferita
- **Gruppo di risorse**: test-infra

Indirizzi IP di rete e subnet:

| Connessione hub Azure/Azure Stack | Nome | Subnet | Indirizzo IP |
|---|---|---|---|
| Azure vNet | ApplicationvNet<br>10.100.102.9/23 | ApplicationSubnet<br>10.100.102.0/24 |  |
|  |  | GatewaySubnet<br>10.100.103.0/24 |  |
| VNet Hub Azure Stack | ApplicationvNet<br>10.100.100.0/23 | ApplicationSubnet <br>10.100.100.0/24 |  |
|  |  | GatewaySubnet <br>10.100101.0/24 |  |
| Gateway di rete virtuale di Azure | Azure-gateway |  |  |
| Gateway di rete virtuale dell'hub Azure Stack | AzureStack-gateway |  |  |
| IP pubblico di Azure | Azure-GatewayPublicIP |  | Determinato al momento della creazione |
| IP pubblico dell'hub Azure Stack | AzureStack-GatewayPublicIP |  | Determinato al momento della creazione |
| Gateway di rete locale di Azure | AzureStack-S2SGateway<br>   10.100.100.0/23 |  | Valore IP pubblico dell'hub Azure Stack |
| Gateway di rete locale dell'hub Azure Stack | Azure-S2SGateway<br>10.100.102.0/23 |  | Valore IP pubblico di Azure |

## <a name="create-a-virtual-network-in-global-azure-and-azure-stack-hub"></a>Creare una rete virtuale in Azure globale e Azure Stack Hub

Usare la procedura seguente per creare una rete virtuale usando il portale. È possibile usare questi [valori di esempio](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal#values) se si usa questo articolo come solo soluzione. Se si usa questo articolo per configurare un ambiente di produzione, sostituire le impostazioni di esempio con i propri valori.

> [!IMPORTANT]
> È necessario assicurarsi che non esista una sovrapposizione degli indirizzi IP negli spazi di indirizzi vNet dell'hub di Azure o Azure Stack.

Per creare una vNet in Azure:

1. Usare il browser per connettersi alla [portale di Azure](https://portal.azure.com/) e accedere con l'account Azure.
2. Selezionare **Crea una risorsa**. Nel campo **Cerca nel Marketplace** immettere "rete virtuale". Selezionare **rete virtuale** dai risultati.
3. Nell'elenco **selezionare un modello di distribuzione** selezionare **Gestione risorse**e quindi fare clic su **Crea**.
4. In **Crea rete virtuale**configurare le impostazioni di VNet. I nomi dei campi obbligatori sono preceduti da un asterisco rosso.  Quando si immette un valore valido, l'asterisco diventa un segno di spunta verde.

Per creare un vNet nell'hub Azure Stack:

1. Ripetere i passaggi precedenti (1-4) usando il **portale tenant**dell'hub Azure stack.

## <a name="add-a-gateway-subnet"></a>Aggiungere una subnet del gateway

Prima di connettere la rete virtuale a un gateway, è necessario creare la subnet del gateway per la rete virtuale a cui ci si vuole connettere. I servizi gateway usano gli indirizzi IP specificati nella subnet del gateway.

Nella [portale di Azure](https://portal.azure.com/)passare alla rete virtuale Gestione risorse in cui si vuole creare un gateway di rete virtuale.

1. Selezionare il vNet per aprire la pagina **rete virtuale** .
2. In **Impostazioni**selezionare **Subnet**.
3. Nella pagina **subnet** selezionare **+ subnet gateway** per aprire la pagina **Aggiungi subnet** .

    ![Aggiungere la subnet del gateway](media/solution-deployment-guide-connectivity/image4.png)

4. Il **nome** della subnet viene compilato automaticamente con il valore ' GatewaySubnet '. Questo valore è obbligatorio per consentire ad Azure di riconoscere la subnet come subnet del gateway.
5. Modificare i valori dell' **intervallo di indirizzi** forniti in base ai requisiti di configurazione e quindi fare clic su **OK**.

## <a name="create-a-virtual-network-gateway-in-azure-and-azure-stack"></a>Creare un gateway di rete virtuale in Azure e Azure Stack

Usare la procedura seguente per creare un gateway di rete virtuale in Azure.

1. Sul lato sinistro della pagina del portale selezionare **+** e immettere "gateway di rete virtuale" nel campo di ricerca.
2. In **risultati**selezionare **gateway di rete virtuale**.
3. In **gateway di rete virtuale**selezionare **Crea** per aprire la pagina **Crea gateway di rete virtuale** .
4. In **Crea gateway di rete virtuale**specificare i valori per il gateway di rete usando i **valori di esempio dell'esercitazione**. Includere i valori aggiuntivi seguenti:

   - **SKU**: di base
   - **Rete virtuale**: selezionare la rete virtuale creata in precedenza. La subnet del gateway creata viene selezionata automaticamente.
   - **Prima configurazione IP**: indirizzo IP pubblico del gateway.
     - Selezionare **Crea configurazione IP gateway**, che consente di scegliere la pagina **Scegli indirizzo IP pubblico** .
     - Selezionare **+ Crea nuovo** per aprire la pagina **Crea indirizzo IP pubblico** .
     - Immettere un **nome** per l'indirizzo IP pubblico. Lasciare lo SKU come **Basic**, quindi fare clic su **OK** per salvare le modifiche.

       > [!Note]
       > Attualmente, il gateway VPN supporta solo l'allocazione di indirizzi IP pubblici dinamici. Questo non significa tuttavia che l'indirizzo IP cambia dopo che è stato assegnato al gateway VPN. L'unica volta in cui l'indirizzo IP pubblico viene modificato è quando il gateway viene eliminato e creato di nuovo. Il ridimensionamento, la reimpostazione o altri aggiornamenti o manutenzione interna del gateway VPN non modificano l'indirizzo IP.

5. Verificare le impostazioni del gateway.
6. Selezionare **Crea** per creare il gateway VPN. Le impostazioni del gateway vengono convalidate e nel dashboard viene visualizzato il riquadro "distribuzione del gateway di rete virtuale".

   >[!Note]
   >La creazione di un gateway può richiedere fino a 45 minuti. Potrebbe essere necessario aggiornare la pagina del portale per visualizzare lo stato di completamento.

    Dopo la creazione del gateway, è possibile visualizzare l'indirizzo IP assegnato esaminando la rete virtuale nel portale. Il gateway viene visualizzato come un dispositivo connesso. Per visualizzare altre informazioni sul gateway, selezionare il dispositivo.

7. Ripetere i passaggi precedenti (1-5) nella distribuzione dell'hub Azure Stack.

## <a name="create-the-local-network-gateway-in-azure-and-azure-stack-hub"></a>Creare il gateway di rete locale in Azure e Azure Stack Hub

Il gateway di rete locale in genere fa riferimento al percorso locale. Assegnare al sito un nome a cui può fare riferimento l'hub Azure o Azure Stack e quindi specificare:

- Indirizzo IP del dispositivo VPN locale per cui si sta creando una connessione.
- I prefissi degli indirizzi IP che verranno indirizzati tramite il gateway VPN al dispositivo VPN. I prefissi degli indirizzi specificati sono quelli disponibili nella rete locale.

  >[!Note]
  >Se la rete locale viene modificata o è necessario modificare l'indirizzo IP pubblico per il dispositivo VPN, è possibile aggiornare questi valori in un secondo momento.

1. Nel portale selezionare **+ Crea una risorsa**.
2. Nella casella di ricerca immettere **gateway di rete locale**, quindi premere **invio** per eseguire la ricerca. Viene visualizzato un elenco di risultati.
3. Selezionare **gateway di rete locale**, quindi selezionare **Crea** per aprire la pagina **Crea gateway di rete locale** .
4. In **Crea gateway di rete locale**specificare i valori per il gateway di rete locale usando i **valori di esempio dell'esercitazione**. Includere i valori aggiuntivi seguenti:

    - **Indirizzo IP**: indirizzo IP pubblico del dispositivo VPN a cui si vuole connettere Azure o hub Azure stack. Specificare un indirizzo IP pubblico valido che non si trovi dietro un NAT, in modo che Azure possa raggiungere l'indirizzo. Se non si dispone dell'indirizzo IP in questo momento, è possibile usare un valore dell'esempio come segnaposto. È necessario tornare indietro e sostituire il segnaposto con l'indirizzo IP pubblico del dispositivo VPN. Azure non è in grado di connettersi al dispositivo fino a quando non viene fornito un indirizzo valido.
    - **Spazio degli indirizzi**: l'intervallo di indirizzi per la rete rappresentata da questa rete locale. È possibile aggiungere più intervalli di spazi indirizzi. Assicurarsi che gli intervalli specificati non si sovrappongano con gli intervalli di altre reti a cui ci si vuole connettere. Azure indirizzerà l'intervallo di indirizzi specificato all'indirizzo IP del dispositivo VPN locale. Usare valori personalizzati se si vuole connettersi al sito locale, non a un valore di esempio.
    - **Configurare le impostazioni BGP**: usare solo quando si configura BGP. In caso contrario, non selezionare questa opzione.
    - **Sottoscrizione**: verificare che sia visualizzata la sottoscrizione corretta.
    - **Gruppo di risorse**: selezionare il gruppo di risorse che si vuole usare. È possibile creare un nuovo gruppo di risorse o selezionarne uno già creato.
    - **Località**: selezionare la località in cui verrà creato l'oggetto. Potrebbe essere necessario selezionare la stessa posizione in cui risiede la VNet, ma non è necessario eseguire questa operazione.
5. Al termine, selezionare **Crea** per creare il gateway di rete locale.
6. Ripetere questi passaggi (1-5) nella distribuzione dell'hub Azure Stack.

## <a name="configure-your-connection"></a>Configurare la connessione

Per le connessioni da sito a sito a una rete locale è necessario un dispositivo VPN. Il dispositivo VPN configurato viene definito connessione. Per configurare la connessione, è necessario:

- Chiave condivisa. Si tratta della stessa chiave condivisa specificata durante la creazione della connessione VPN da sito a sito. In questi esempi viene usata una chiave condivisa semplice. È consigliabile generare una chiave più complessa per l'uso effettivo.
- Indirizzo IP pubblico del gateway di rete virtuale. È possibile visualizzare l'indirizzo IP pubblico usando il portale di Azure, PowerShell o l'interfaccia della riga di comando. Per trovare l'indirizzo IP pubblico del gateway VPN usando il portale di Azure, passare a gateway di rete virtuale, quindi selezionare il nome del gateway.

Usare la procedura seguente per creare una connessione VPN da sito a sito tra il gateway di rete virtuale e il dispositivo VPN locale.

1. Nella portale di Azure selezionare **+ Crea una risorsa**.
2. Cercare le **connessioni**.
3. In **risultati**selezionare **connessioni**.
4. In **connessione**selezionare **Crea**.
5. In **Crea connessione**configurare le impostazioni seguenti:

    - **Tipo di connessione**: selezionare da sito a sito (IPSec).
    - **Gruppo di risorse**: selezionare il gruppo di risorse di test.
    - **Gateway di rete virtuale**: selezionare il gateway di rete virtuale creato.
    - **Gateway di rete locale**: selezionare il gateway di rete locale creato.
    - **Nome connessione**: questo nome viene popolato automaticamente usando i valori dei due gateway.
    - **Chiave condivisa**: questo valore deve corrispondere al valore che si sta usando per il dispositivo VPN locale. Nell'esempio di esercitazione viene usato ' abc123', ma è consigliabile usare un elemento più complesso. L'aspetto importante è che questo valore *deve* essere lo stesso valore specificato durante la configurazione del dispositivo VPN.
    - I valori di **Sottoscrizione**, **Gruppo di risorse** e **Località** sono fissi.

6. Selezionare **OK** per creare la connessione.

È possibile visualizzare la connessione nella pagina **connessioni** del gateway di rete virtuale. Lo stato passerà da *sconosciuto* a *connessione*e quindi a *riuscito*.

## <a name="next-steps"></a>Passaggi successivi

- Per altre informazioni sui modelli cloud di Azure, vedere [modelli di progettazione cloud](https://docs.microsoft.com/azure/architecture/patterns).
