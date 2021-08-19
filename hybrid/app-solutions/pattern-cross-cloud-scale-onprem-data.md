---
title: Modello di scalabilità tra cloud (dati locali) in hub di Azure Stack
description: Informazioni su come creare un'app multi-cloud scalabile che usa dati in azure e hub di Azure Stack.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 5c8e3adb621ae4322bf6d60792fc307dbb24ff90
ms.sourcegitcommit: df06f598da09074d387f5f765f7c4237af98fb59
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 08/17/2021
ms.locfileid: "122281245"
---
# <a name="cross-cloud-scaling-on-premises-data-pattern"></a>Modello di scalabilità tra cloud (dati locali)

Informazioni su come creare un'app ibrida che si estende su Azure e hub di Azure Stack. Questo modello illustra anche come usare una singola origine dati locale per la conformità.

## <a name="context-and-problem"></a>Contesto e problema

Molte organizzazioni raccolgono e archiviano enormi quantità di dati sensibili dei clienti. Spesso viene impedito loro di archiviare dati sensibili nel cloud pubblico a causa di normative aziendali o criteri governativi. Queste organizzazioni vogliono anche sfruttare la scalabilità del cloud pubblico. Il cloud pubblico può gestire i picchi stagionali del traffico, consentendo ai clienti di pagare esattamente l'hardware necessario, quando necessario.

## <a name="solution"></a>Soluzione

La soluzione sfrutta i vantaggi della conformità del cloud privato, combinandoli con la scalabilità del cloud pubblico. Azure e il hub di Azure Stack cloud ibrido offrono un'esperienza coerente per gli sviluppatori. Questa coerenza consente di applicare le proprie competenze sia al cloud pubblico che agli ambienti locali.

La guida alla distribuzione della soluzione consente di distribuire un'app Web identica in un cloud pubblico e privato. È anche possibile accedere a una rete instradabile non Internet ospitata nel cloud privato. Le app Web vengono monitorate per il caricamento. In caso di aumento significativo del traffico, un programma modifica i record DNS per reindirizzare il traffico al cloud pubblico. Quando il traffico non è più significativo, i record DNS vengono aggiornati per indirizzare il traffico al cloud privato.

[![Scalabilità tra cloud con modello di dati on-prem](media/pattern-cross-cloud-scale-onprem-data/solution-architecture.png)](media/pattern-cross-cloud-scale-onprem-data/solution-architecture.png)

## <a name="components"></a>Componenti

Questa soluzione usa i componenti seguenti:

| Livello | Componente | Descrizione |
|----------|-----------|-------------|
| Azure | Servizio app di Azure | [Servizio app di Azure](/azure/app-service/) consente di creare e ospitare app Web, app per le API RESTful e Funzioni di Azure. Tutto nel linguaggio di programmazione preferito, senza gestire l'infrastruttura. |
| | Rete virtuale di Azure| [Rete virtuale di Azure è il blocco](/azure/virtual-network/virtual-networks-overview) predefinito fondamentale per le reti private in Azure. La rete virtuale consente a più tipi di risorse di Azure, ad esempio macchine virtuali (VM), di comunicare in modo sicuro tra loro, con Internet e con le reti locali. La soluzione illustra anche l'uso di componenti di rete aggiuntivi:<br>: subnet dell'app e del gateway.<br>: gateway di rete locale.<br>: un gateway di rete virtuale che funge da connessione gateway VPN da sito a sito.<br>: indirizzo IP pubblico.<br>: una connessione VPN da punto a sito.<br>: DNS di Azure per l'hosting di domini DNS e la risoluzione dei nomi. |
| | Gestione traffico di Azure | [Gestione traffico di Azure](/azure/traffic-manager/traffic-manager-overview) è un servizio di bilanciamento del carico del traffico basato su DNS. Consente di controllare la distribuzione del traffico utente per gli endpoint di servizio in data center diversi. |
| | Azure Application Insights | [Application Insights](/azure/azure-monitor/app/app-insights-overview) è un servizio estendibile di gestione delle prestazioni delle applicazioni per sviluppatori Web che compilano e gestiscono app su più piattaforme.|
| | Funzioni di Azure | [Funzioni di Azure](/azure/azure-functions/) consente di eseguire il codice in un ambiente serverless senza dover prima creare una macchina virtuale o pubblicare un'app Web. |
| | Scalabilità automatica di Azure | [La scalabilità](/azure/azure-monitor/platform/autoscale-overview) automatica è una funzionalità predefinita di Servizi cloud, macchine virtuali e app Web. La funzionalità consente alle app di ottenere prestazioni ottimali quando la domanda cambia. Le app si adattano ai picchi di traffico, inviando una notifica quando le metriche cambiano e si ridimensionano in base alle esigenze. |
| Hub di Azure Stack | Calcolo IaaS | hub di Azure Stack consente di usare lo stesso modello di app, lo stesso portale self-service e le stesse API abilitate da Azure. hub di Azure Stack IaaS offre un'ampia gamma di tecnologie open source per distribuzioni di cloud ibride coerenti. L'esempio di soluzione usa una macchina Windows Server per SQL Server, ad esempio.|
| | Servizio app di Azure | Proprio come l'app Web di Azure, la soluzione usa Servizio app di Azure [in hub di Azure Stack](/azure-stack/operator/azure-stack-app-service-overview) per ospitare l'app Web. |
| | Rete | La hub di Azure Stack virtuale funziona esattamente come la rete virtuale di Azure. Usa molti degli stessi componenti di rete, inclusi i nomi host personalizzati.
| Azure DevOps Services | Iscrizione | Configurare rapidamente l'integrazione continua per la compilazione, il test e la distribuzione. Per altre informazioni, vedere [Iscriversi e accedere a Azure DevOps](/azure/devops/user-guide/sign-up-invite-teammates?view=azure-devops). |
| | Azure Pipelines | Usare [Azure Pipelines](/azure/devops/pipelines/agents/agents?view=azure-devops) per l'integrazione continua/recapito continuo. Azure Pipelines consente di gestire gli agenti e le definizioni di compilazione e versione ospitati. |
| | Repository di codice | Sfruttare più repository di codice per semplificare la pipeline di sviluppo. Usare i repository di codice esistenti in GitHub, Bitbucket, Dropbox, OneDrive e Azure Repos. |

## <a name="issues-and-considerations"></a>Considerazioni e problemi

Quando si decide come implementare questa soluzione, tenere presente quanto segue:

### <a name="scalability"></a>Scalabilità

Azure e hub di Azure Stack sono particolarmente adatti a supportare le esigenze dell'attuale azienda distribuita a livello globale.

#### <a name="hybrid-cloud-without-the-hassle"></a>Cloud ibrido senza problemi

Microsoft offre un'integrazione senza rivali di asset locali con hub di Azure Stack e Azure in un'unica soluzione unificata. Questa integrazione elimina il problema della gestione di soluzioni a più punti e di una combinazione di provider di servizi cloud. Con il ridimensionamento tra cloud, la potenza di Azure è a pochi clic. È sufficiente connettere i hub di Azure Stack ad Azure con burst nel cloud e i dati e le app saranno disponibili in Azure quando necessario.

- Eliminare la necessità di creare e gestire un sito di ripristino di emergenza secondario.
- Risparmiare tempo e denaro eliminando il backup su nastro e ospitando fino a 99 anni di dati di backup in Azure.
- Eseguire facilmente la migrazione dei carichi di lavoro Hyper-V, fisici (in anteprima) e VMware (in anteprima) in Azure per sfruttare i vantaggi economici e l'elasticità del cloud.
- Eseguire analisi o report a elevato utilizzo di calcolo in una copia replicata dell'asset locale in Azure senza alcun impatto sui carichi di lavoro di produzione.
- Eseguire burst nel cloud ed eseguire carichi di lavoro locali in Azure, con modelli di calcolo più grandi quando necessario. Hybrid offre la potenza necessaria, quando necessario.
- Creare ambienti di sviluppo multilivello in Azure con pochi clic: è anche possibile replicare i dati di produzione in tempo reale nell'ambiente di sviluppo/test per mantenerli near real-time sincronizzazione.

#### <a name="economy-of-cross-cloud-scaling-with-azure-stack-hub"></a>Economia della scalabilità tra cloud con hub di Azure Stack

Il vantaggio principale del bursting nel cloud è un risparmio economico. Si paga per le risorse aggiuntive solo in caso di richiesta di tali risorse. Non sono più necessarie spese per capacità aggiuntiva superflua o per prevedere picchi e fluttuazioni della domanda.

#### <a name="reduce-high-demand-loads-into-the-cloud"></a>Ridurre i carichi di lavoro elevati nel cloud

Il ridimensionamento tra cloud può essere usato per ridurre il carico di lavoro di elaborazione. Il carico viene distribuito spostando le app di base nel cloud pubblico, liberando le risorse locali per le app business critical. Un'app può essere applicata al cloud privato, quindi eseguire il burst nel cloud pubblico solo quando necessario per soddisfare le esigenze.

### <a name="availability"></a>Disponibilità

La distribuzione globale presenta problematiche specifiche, ad esempio connettività variabile e normative governative diverse in base all'area. Gli sviluppatori possono sviluppare una sola app e quindi distribuirla per diversi motivi con requisiti diversi. Distribuire l'app nel cloud pubblico di Azure, quindi distribuire istanze o componenti aggiuntivi in locale. È possibile gestire il traffico tra tutte le istanze usando Azure.

### <a name="manageability"></a>Gestione

#### <a name="a-single-consistent-development-approach"></a>Un unico approccio di sviluppo coerente

Azure e hub di Azure Stack consentono di usare un set coerente di strumenti di sviluppo all'interno dell'organizzazione. Questa coerenza semplifica l'implementazione di una pratica di integrazione continua e sviluppo continuo (CI/CD). Molte app e servizi distribuiti in Azure o hub di Azure Stack sono intercambiabili e possono essere eseguiti in entrambe le località senza problemi.

Una pipeline CI/CD ibrida consente di:

- Avviare una nuova compilazione in base ai commit del codice nel repository del codice.
- Distribuire automaticamente il codice appena compilato in Azure per il test di accettazione dell'utente.
- Dopo che il codice ha superato il test, eseguire automaticamente la distribuzione in hub di Azure Stack.

### <a name="a-single-consistent-identity-management-solution"></a>Una singola soluzione di gestione delle identità coerente

hub di Azure Stack funziona sia con Azure Active Directory (Azure AD) che con Active Directory Federation Services (ADFS). hub di Azure Stack funziona con Azure AD in scenari connessi. Per gli ambienti che non hanno connettività, è possibile usare ADFS come soluzione disconnessa. Le entità servizio vengono usate per concedere l'accesso alle app, consentendo loro di distribuire o configurare le risorse tramite Azure Resource Manager.

### <a name="security"></a>Sicurezza

#### <a name="ensure-compliance-and-data-sovereignty"></a>Garantire la conformità e la sovranità dei dati

hub di Azure Stack consente di eseguire lo stesso servizio in più paesi come se si usasse un cloud pubblico. La distribuzione della stessa app nei data center in ogni paese consente di soddisfare i requisiti di sovranità dei dati. Questa funzionalità garantisce che i dati personali siano conservati all'interno dei confini di ogni paese.

#### <a name="azure-stack-hub---security-posture"></a>hub di Azure Stack- sicurezza

Non esiste una posizione di sicurezza senza un solido processo di manutenzione continua. Per questo motivo, Microsoft ha investito in un motore di orchestrazione che applica facilmente patch e aggiornamenti all'intera infrastruttura.

Grazie alle partnership con hub di Azure Stack partner OEM, Microsoft estende lo stesso ruolo di sicurezza ai componenti specifici di OEM, ad esempio l'host del ciclo di vita hardware e il software in esecuzione su di esso. Questa partnership garantisce hub di Azure Stack un'uniforme e solida posizione di sicurezza nell'intera infrastruttura. A sua volta, i clienti possono compilare e proteggere i carichi di lavoro delle app.

#### <a name="use-of-service-principals-via-powershell-cli-and-azure-portal"></a>Uso di entità servizio tramite PowerShell, interfaccia della riga di comando e portale di Azure

Per concedere l'accesso alle risorse a uno script o a un'app, configurare un'identità per l'app e autenticare l'app con le proprie credenziali. Questa identità è nota come entità servizio e consente di:

- Assegnare all'identità dell'app autorizzazioni diverse da quelle personali e limitate esattamente alle esigenze dell'app.
- Usare un certificato per l'autenticazione in caso di esecuzione di uno script automatico.

Per altre informazioni sulla creazione dell'entità servizio e sull'uso di un certificato per le credenziali, vedere [Usare un'identità dell'app per accedere alle risorse](/azure-stack/operator/azure-stack-create-service-principals).

## <a name="when-to-use-this-pattern"></a>Quando usare questo modello

- L'organizzazione usa un approccio DevOps o ne prevede uno per il prossimo futuro.
- Si vogliono implementare procedure ci/CD nell'implementazione hub di Azure Stack e nel cloud pubblico.
- Si vuole consolidare la pipeline CI/CD in ambienti cloud e locali.
- Voglio la possibilità di sviluppare app facilmente usando servizi cloud o locali.
- Voglio sfruttare competenze di sviluppo coerenti tra le app cloud e locali.
- Sto usando Azure, ma ho sviluppatori che lavorano in un cloud hub di Azure Stack locale.
- Le app locali hanno picchi di domanda durante fluttuazioni stagionali, cicliche o imprevedibili.
- Sono disponibili componenti locali e si vuole usare il cloud per ridimensionarli senza problemi.
- Si vuole la scalabilità del cloud, ma si vuole che l'app sia eseguita in locale il più possibile.

## <a name="next-steps"></a>Passaggi successivi

Per altre informazioni sugli argomenti introdotti in questo articolo:

- Per [una panoramica dell'uso di](https://www.youtube.com/watch?v=2lw8zOpJTn0) questo modello, vedere Ridimensionare dinamicamente le app tra data center e cloud pubblico.
- Vedere [Considerazioni sulla progettazione di app ibride](overview-app-design-considerations.md) per altre informazioni sulle procedure consigliate e per rispondere ad altre domande.
- Questo modello usa la Azure Stack di prodotti, tra cui hub di Azure Stack. Vedere la [famiglia di prodotti e le soluzioni di Azure Stack](/azure-stack) per altre informazioni sull'intero portfolio di prodotti e soluzioni.

Quando si è pronti per testare l'esempio di soluzione, continuare con la guida alla distribuzione della soluzione [cross-cloud scaling (dati locali).](/azure/architecture/hybrid/deployments/solution-deployment-guide-cross-cloud-scaling-onprem-data) Questa guida contiene istruzioni dettagliate per la distribuzione e il test dei componenti.