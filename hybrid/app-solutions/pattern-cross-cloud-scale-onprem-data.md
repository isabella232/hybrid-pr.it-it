---
title: Modello di scalabilità tra cloud (dati locali) nell'hub Azure Stack
description: Informazioni su come creare un'app multicloud scalabile che usa dati locali in Azure e Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: edbb608fbf8e5288f29572bfe4cca98ffb3cb8fc
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911129"
---
# <a name="cross-cloud-scaling-on-premises-data-pattern"></a>Modello di scalabilità tra cloud (dati locali)

Informazioni su come creare un'app ibrida che si estende su Azure e hub Azure Stack. Questo modello mostra anche come usare una singola origine dati locale per la conformità.

## <a name="context-and-problem"></a>Contesto e problema

Molte organizzazioni raccolgono e archiviano grandi quantità di dati sensibili dei clienti. Spesso non è possibile archiviare i dati sensibili nel cloud pubblico a causa delle normative aziendali o dei criteri governativi. Queste organizzazioni vogliono anche sfruttare la scalabilità del cloud pubblico. Il cloud pubblico è in grado di gestire picchi stagionali nel traffico, consentendo ai clienti di pagare esattamente l'hardware necessario, quando necessario.

## <a name="solution"></a>Soluzione

La soluzione sfrutta i vantaggi della conformità del cloud privato, combinando questi ultimi con la scalabilità del cloud pubblico. Il cloud ibrido di Azure e Azure Stack Hub offre un'esperienza coerente per gli sviluppatori. Questa coerenza consente di applicare le proprie competenze sia al cloud pubblico che agli ambienti locali.

La guida alla distribuzione della soluzione consente di distribuire un'app Web identica a un cloud pubblico e privato. È anche possibile accedere a una rete instradabile non Internet ospitata nel cloud privato. Le app Web vengono monitorate per il caricamento. In seguito a un aumento significativo del traffico, un programma modifica i record DNS per reindirizzare il traffico al cloud pubblico. Quando il traffico non è più significativo, i record DNS vengono aggiornati per indirizzare il traffico al cloud privato.

[![Scalabilità tra cloud con modello di dati locale](media/pattern-cross-cloud-scale-onprem-data/solution-architecture.png)](media/pattern-cross-cloud-scale-onprem-data/solution-architecture.png)

## <a name="components"></a>Componenti

Questa soluzione USA i componenti seguenti:

| Livello | Componente | Descrizione |
|----------|-----------|-------------|
| Azure | Servizio app di Azure | [App Azure servizio](/azure/app-service/) ti permette di creare e ospitare app Web, app per le API RESTful e funzioni di Azure. Tutto nel linguaggio di programmazione preferito, senza dover gestire l'infrastruttura. |
| | Rete virtuale di Azure| [Rete virtuale di Azure (VNet)](/azure/virtual-network/virtual-networks-overview) è il blocco predefinito fondamentale per le reti private in Azure. VNet consente a più tipi di risorse di Azure, ad esempio macchine virtuali (VM), di comunicare in modo sicuro tra loro, con Internet e con le reti locali. La soluzione illustra anche l'uso di componenti di rete aggiuntivi:<br>-subnet app e gateway.<br>-un gateway di rete locale.<br>: un gateway di rete virtuale, che funge da connessione gateway VPN da sito a sito.<br>-un indirizzo IP pubblico.<br>: una connessione VPN da punto a sito.<br>-DNS di Azure per l'hosting di domini DNS e la risoluzione dei nomi. |
| | Gestione traffico di Azure | [Gestione traffico di Azure](/azure/traffic-manager/traffic-manager-overview) è un servizio di bilanciamento del carico del traffico basato su DNS. Consente di controllare la distribuzione del traffico utente per gli endpoint di servizio in diversi Data Center. |
| | Azure Application Insights | [Application Insights](/azure/azure-monitor/app/app-insights-overview) è un servizio estendibile di gestione delle prestazioni delle applicazioni per sviluppatori Web che compilano e gestiscono app su più piattaforme.|
| | Funzioni di Azure | [Funzioni di Azure](/azure/azure-functions/) consente di eseguire il codice in un ambiente senza server senza dover prima creare una macchina virtuale o pubblicare un'app Web. |
| | Scalabilità automatica di Azure | La [scalabilità](/azure/azure-monitor/platform/autoscale-overview) automatica è una funzionalità predefinita di servizi cloud, macchine virtuali e app Web. La funzionalità consente alle app di eseguire le prestazioni migliori quando la richiesta cambia. Le app si adatteranno ai picchi di traffico, notificando quando le metriche cambiano e ridimensionando in base alle esigenze. |
| Hub Azure Stack | Calcolo IaaS | Azure Stack Hub consente di usare lo stesso modello di app, il portale self-service e le API abilitate da Azure. Azure Stack Hub IaaS offre un'ampia gamma di tecnologie open source per distribuzioni cloud ibride coerenti. L'esempio di soluzione USA una macchina virtuale Windows Server per SQL Server, ad esempio.|
| | Servizio app di Azure | Proprio come l'app Web di Azure, la soluzione USA [app Azure servizio nell'Hub Azure stack](/azure-stack/operator/azure-stack-app-service-overview) per ospitare l'app Web. |
| | Rete | La rete virtuale dell'hub Azure Stack funziona esattamente come la rete virtuale di Azure. USA molti degli stessi componenti di rete, inclusi i nomi host personalizzati.
| Azure DevOps Services | Iscrizione | Configura rapidamente l'integrazione continua per la compilazione, il test e la distribuzione. Per altre informazioni, vedere [iscrizione, accesso ad Azure DevOps](/azure/devops/user-guide/sign-up-invite-teammates?view=azure-devops). |
| | Azure Pipelines | Usare [Azure Pipelines](/azure/devops/pipelines/agents/agents?view=azure-devops) per l'integrazione continua/recapito continuo. Azure Pipelines consente di gestire gli agenti di compilazione e rilascio ospitati e le definizioni. |
| | Repository di codice | Utilizzare più repository di codice per semplificare la pipeline di sviluppo. USA repository di codice esistenti in GitHub, Bitbucket, Dropbox, OneDrive e Azure Repos. |

## <a name="issues-and-considerations"></a>Considerazioni e problemi

Quando si decide come implementare questa soluzione, tenere presente quanto segue:

### <a name="scalability"></a>Scalabilità

Azure e l'hub Azure Stack sono particolarmente adatti per supportare le esigenze dell'azienda attualmente distribuita a livello globale.

#### <a name="hybrid-cloud-without-the-hassle"></a>Cloud ibrido senza problemi

Microsoft offre un'integrazione senza precedenti delle risorse locali con Azure Stack Hub e Azure in un'unica soluzione unificata. Questa integrazione elimina il problema della gestione di più soluzioni di punti e di una combinazione di provider di servizi cloud. Con il ridimensionamento tra cloud, la potenza di Azure è solo pochi clic. È sufficiente connettere l'hub Azure Stack ad Azure con l'espansione del cloud e i dati e le app saranno disponibili in Azure quando necessario.

- Elimina la necessità di creare e gestire un sito di ripristino di emergenza secondario.
- Risparmia tempo e denaro eliminando il backup su nastro e ospitando fino a 99 anni di dati di backup in Azure.
- Puoi eseguire facilmente la migrazione di carichi di lavoro Hyper-V, fisici (in anteprima) e VMware (in anteprima) in Azure per sfruttare i vantaggi economici ed elastici del cloud.
- Eseguire report a elevato utilizzo di calcolo o analisi su una copia replicata dell'asset locale in Azure senza alcun effetto sui carichi di lavoro di produzione.
- Irrompi nel cloud ed Esegui carichi di lavoro locali in Azure, con modelli di calcolo più grandi, quando necessario. Ibrido ti offre la potenza che ti serve, quando ti serve.
- Crea ambienti di sviluppo a più livelli in Azure con pochi clic: puoi anche replicare i dati di produzione in tempo reale nell'ambiente di sviluppo/test per mantenerli sincronizzati quasi in tempo reale.

#### <a name="economy-of-cross-cloud-scaling-with-azure-stack-hub"></a>Economia del ridimensionamento tra cloud con Azure Stack Hub

Il vantaggio principale dell'espansione nel cloud è un risparmio economico. Paghi solo per le risorse aggiuntive in caso di richiesta di tali risorse. Non è più spesa per una capacità aggiuntiva superflua o si sta tentando di prevedere picchi e fluttuazioni della domanda.

#### <a name="reduce-high-demand-loads-into-the-cloud"></a>Ridurre i carichi di domanda elevata nel cloud

La scalabilità tra cloud può essere usata per gestire i carichi di elaborazione. Il carico viene distribuito spostando le app di base nel cloud pubblico, liberando le risorse locali per le app cruciali per l'azienda. Un'app può essere applicata al cloud privato, quindi irrompere nel cloud pubblico solo quando è necessario soddisfare le esigenze.

### <a name="availability"></a>Disponibilità

La distribuzione globale presenta le proprie esigenze, ad esempio connettività variabile e normative governative diverse in base all'area. Gli sviluppatori possono sviluppare una sola app e quindi distribuirla in diversi motivi con requisiti diversi. Distribuire l'app nel cloud pubblico di Azure, quindi distribuire istanze o componenti aggiuntivi localmente. È possibile gestire il traffico tra tutte le istanze usando Azure.

### <a name="manageability"></a>Gestione

#### <a name="a-single-consistent-development-approach"></a>Un unico approccio di sviluppo coerente

Azure e hub Azure Stack consentono di usare un set di strumenti di sviluppo coerenti nell'intera organizzazione. Questa coerenza rende più semplice implementare una procedura di integrazione continua e sviluppo continuo (CI/CD). Molte app e servizi distribuiti in Azure o Azure Stack Hub sono intercambiabili e possono essere eseguiti in qualsiasi posizione senza interruzioni.

Una pipeline di integrazione continua/distribuzione continua ibrida può essere utile:

- Avviare una nuova compilazione in base ai commit del codice nel repository del codice.
- Distribuisci automaticamente il codice appena compilato in Azure per i test di accettazione degli utenti.
- Una volta che il codice ha superato i test, Distribuisci automaticamente nell'hub Azure Stack.

### <a name="a-single-consistent-identity-management-solution"></a>Una singola soluzione di gestione delle identità coerente

Azure Stack hub funziona con Azure Active Directory (Azure AD) e Active Directory Federation Services (ADFS). Azure Stack hub funziona con Azure AD in scenari connessi. Per gli ambienti che non dispongono di connettività, è possibile utilizzare ADFS come soluzione disconnessa. Le entità servizio vengono usate per concedere l'accesso alle app, consentendo la distribuzione o la configurazione di risorse tramite Azure Resource Manager.

### <a name="security"></a>Sicurezza

#### <a name="ensure-compliance-and-data-sovereignty"></a>Assicurare la conformità e la sovranità dei dati

Azure Stack Hub consente di eseguire lo stesso servizio in più paesi come se si utilizzasse un cloud pubblico. La distribuzione della stessa app nei data center in ogni paese consente di soddisfare i requisiti di sovranità dei dati. Questa funzionalità garantisce che i dati personali vengano mantenuti entro i confini di ogni paese.

#### <a name="azure-stack-hub---security-posture"></a>Hub Azure Stack-postura sicurezza

Non esiste alcuna postura di sicurezza senza un processo continuo di manutenzione continuo. Per questo motivo, Microsoft ha investito in un motore di orchestrazione che applica patch e aggiornamenti agevolmente nell'intera infrastruttura.

Grazie alle partnership con i partner OEM Azure Stack Hub, Microsoft estende lo stesso comportamento di sicurezza ai componenti specifici degli OEM, ad esempio l'host del ciclo di vita hardware e il software in esecuzione su di esso. Questa partnership garantisce che Azure Stack Hub abbia una postura di sicurezza uniforme e uniforme nell'intera infrastruttura. A loro volta, i clienti possono creare e proteggere i carichi di lavoro delle app.

#### <a name="use-of-service-principals-via-powershell-cli-and-azure-portal"></a>Uso delle entità servizio tramite PowerShell, l'interfaccia della riga di comando e portale di Azure

Per concedere l'accesso alle risorse a uno script o a un'app, configurare un'identità per l'app e autenticare l'app con le proprie credenziali. Questa identità è nota come entità servizio e consente di:

- Assegnare autorizzazioni all'identità dell'app diverse dalle proprie autorizzazioni e limitate a specifiche esigenze dell'app.
- Usare un certificato per l'autenticazione in caso di esecuzione di uno script automatico.

Per altre informazioni sulla creazione dell'entità servizio e sull'uso di un certificato per le credenziali, vedere [usare un'identità dell'app per accedere alle risorse](/azure-stack/operator/azure-stack-create-service-principals).

## <a name="when-to-use-this-pattern"></a>Quando usare questo modello

- L'organizzazione usa un approccio DevOps o ne prevede uno per il prossimo futuro.
- Desidero implementare procedure CI/CD nell'implementazione dell'hub Azure Stack e nel cloud pubblico.
- Si desidera consolidare la pipeline di integrazione continua/recapito continuo tra ambienti cloud e locali.
- Desidero la possibilità di sviluppare app senza interruzioni usando i servizi cloud o locali.
- Desidero sfruttare le competenze di sviluppo coerenti in tutte le app cloud e locali.
- Sto usando Azure, ma ho sviluppatori che lavorano in un cloud di hub Azure Stack locale.
- Le app locali presentano picchi di richieste durante le fluttuazioni stagionali, cicliche o imprevedibili.
- Ho componenti locali e voglio usare il cloud per ridimensionarli senza interruzioni.
- Desidero la scalabilità cloud, ma voglio che l'app venga eseguita in locale il più possibile.

## <a name="next-steps"></a>Passaggi successivi

Per ulteriori informazioni sugli argomenti introdotti in questo articolo:

- Per una panoramica del modo in cui viene usato questo modello, vedere [ridimensionare in modo dinamico le app tra i Data Center e il cloud pubblico](https://www.youtube.com/watch?v=2lw8zOpJTn0) .
- Per ulteriori informazioni sulle procedure consigliate e per rispondere a domande aggiuntive, vedere [considerazioni sulla progettazione di app ibride](overview-app-design-considerations.md) .
- Questo modello usa la famiglia di prodotti Azure Stack, incluso l'hub Azure Stack. Per ulteriori informazioni sull'intero portfolio di prodotti e soluzioni, vedere la [famiglia di prodotti e soluzioni Azure stack](/azure-stack) .

Quando si è pronti per eseguire il test dell'esempio di soluzione, continuare con la [Guida alla distribuzione di soluzioni per la scalabilità tra cloud (dati locali)](solution-deployment-guide-cross-cloud-scaling-onprem-data.md). Nella Guida alla distribuzione vengono fornite istruzioni dettagliate per la distribuzione e il test dei componenti di.
