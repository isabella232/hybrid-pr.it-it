---
title: Distribuire un cluster Kubernetes a disponibilità elevata nell'hub di Azure Stack
description: Informazioni su come distribuire una soluzione di cluster Kubernetes per la disponibilità elevata usando Azure e l'hub di Azure Stack.
author: BryanLa
ms.topic: article
ms.date: 12/03/2020
ms.author: bryanla
ms.reviewer: bryanla
ms.lastreviewed: 12/03/2020
ms.openlocfilehash: 91f5856aa670bf3810baa5e5f07dbb7dafc9e3f3
ms.sourcegitcommit: df7e3e6423c3d4e8a42dae3d1acfba1d55057258
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 12/09/2020
ms.locfileid: "96911922"
---
# <a name="deploy-a-high-availability-kubernetes-cluster-on-azure-stack-hub"></a>Distribuire un cluster Kubernetes a disponibilità elevata nell'hub di Azure Stack

Questo articolo illustra come sviluppare un ambiente cluster Kubernetes a disponibilità elevata, distribuito in più istanze dell'hub di Azure Stack, in posizioni fisiche diverse.

Questa guida alla distribuzione della soluzione descrive come:

> [!div class="checklist"]
> - Scaricare e preparare il motore del servizio Azure Kubernetes
> - Connettersi alla VM helper del motore del servizio Azure Kubernetes
> - Distribuire un cluster Kubernetes
> - Connettersi al cluster Kubernetes
> - Connettere Azure Pipelines al cluster Kubernetes
> - Configurare il monitoraggio
> - Distribuire un'applicazione
> - Dimensionare automaticamente l'applicazione
> - Configurare Gestione traffico
> - Aggiornare Kubernetes
> - Dimensionare Kubernetes

> [!Tip]  
> ![Pilastri ibridi](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> L'hub di Microsoft Azure Stack è un'estensione di Azure. L'hub di Azure Stack offre all'ambiente locale l'agilità e l'innovazione del cloud computing, abilitando l'unico cloud ibrido che consente di creare e distribuire ovunque app ibride.  
> 
> L'articolo [Considerazioni per la progettazione di app ibride](overview-app-design-considerations.md) esamina i concetti fondamentali di qualità del software (posizionamento, scalabilità, disponibilità, resilienza, gestibilità e sicurezza) per la progettazione, la distribuzione e la gestione di app ibride. Le considerazioni di progettazione consentono di ottimizzare la progettazione delle app ibride, riducendo al minimo i rischi negli ambienti di produzione.

## <a name="prerequisites"></a>Prerequisiti

Prima di iniziare a usare questa guida alla distribuzione:

- Vedere l'articolo [Modello di cluster Kubernetes a disponibilità elevata](pattern-highly-available-kubernetes.md).
- Esaminare il contenuto del [repository GitHub associato](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub), che contiene altri asset a cui viene fatto riferimento in questo articolo.
- Avere a disposizione un account in grado di accedere al [portale utente dell'hub di Azure Stack](/azure-stack/user/azure-stack-use-portal), con almeno le [autorizzazioni di "collaboratore"](/azure-stack/user/azure-stack-manage-permissions).

## <a name="download-and-prepare-aks-engine"></a>Scaricare e preparare il motore del servizio Azure Kubernetes

Il motore del servizio Azure Kubernetes è un binario che è possibile usare da qualsiasi host Windows o Linux e che può raggiungere gli endpoint di Azure Resource Manager dell'hub di Azure Stack. Questa guida descrive la distribuzione di una nuova VM Linux (o Windows) nell'hub di Azure Stack. Verrà usata in seguito quando il motore del servizio Azure Kubernetes distribuirà i cluster Kubernetes.

> [!NOTE]
> Per distribuire un cluster Kubernetes nell'hub di Azure Stack con il motore del servizio Azure Kubernetes, è anche possibile usare una VM Windows o Linux esistente.

La procedura dettagliata e i requisiti per il motore del servizio Azure Kubernetes sono documentati qui:

* [Installare il motore del servizio Azure Kubernetes in Linux nell'hub di Azure Stack](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-linux) (oppure con [Windows](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-windows))

Il motore del servizio Azure Kubernetes è uno strumento helper per distribuire e utilizzare cluster Kubernetes non gestiti in Azure e nell'hub di Azure Stack.

I dettagli e le differenze del motore del servizio Azure Kubernetes nell'hub di Azure Stack sono descritti qui:

* [Che cos'è il motore del servizio Azure Kubernetes nell'hub di Azure Stack?](/azure-stack/user/azure-stack-kubernetes-aks-engine-overview)
* [Motore del servizio Azure Kubernetes nell'hub di Azure Stack](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md) (in GitHub)

Nell'ambiente di esempio si userà Terraform per automatizzare la distribuzione della VM del motore del servizio Azure Kubernetes. È possibile trovare i [dettagli e il codice nel repository GitHub associato](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/blob/master/AKSe-on-AzStackHub/src/tf/aksengine/README.md).

Il risultato di questo passaggio è un nuovo gruppo di risorse nell'hub di Azure Stack che contiene la VM helper del motore del servizio Azure Kubernetes e le risorse correlate:

![Risorse della VM del motore del servizio Azure Kubernetes nell'hub di Azure Stack](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-resources-on-azure-stack.png)

> [!NOTE]
> Se è necessario distribuire il motore del servizio Azure Kubernetes in un ambiente disconnesso o in configurazione air gap, vedere [Istanze disconnesse dell'hub di Azure Stack](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#disconnected-azure-stack-hub-instances) per altre informazioni.

Nel passaggio successivo si userà la VM del motore del servizio Azure Kubernetes appena creata per distribuire un cluster Kubernetes.

## <a name="connect-to-the-aks-engine-helper-vm"></a>Connettersi alla VM helper del motore del servizio Azure Kubernetes

Prima di tutto è necessario connettersi alla VM helper del motore del servizio Azure Kubernetes creata in precedenza.

La VM dovrà avere un indirizzo IP pubblico e dovrà essere accessibile tramite SSH (porta 22/TCP).

![Pagina di panoramica della VM del motore del servizio Azure Kubernetes](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-vm-overview.png)

> [!TIP]
> Per connettersi a una VM Linux tramite SSH, è possibile usare uno strumento a scelta, come MobaXterm, puTTY o PowerShell in Windows 10.

```console
ssh <username>@<ipaddress>
```

Dopo aver stabilito la connessione, eseguire il comando `aks-engine`. Vedere [Versioni supportate del motore del servizio Azure Kubernetes](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#supported-aks-engine-versions) per altre informazioni sulle versioni del motore e di Kubernetes.

![Esempio di riga di comando ask-engine](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-cmdline-example.png)

## <a name="deploy-a-kubernetes-cluster"></a>Distribuire un cluster Kubernetes

La VM helper del motore del servizio Azure Kubernetes non ha ancora creato un cluster Kubernetes nell'hub di Azure Stack. La creazione del cluster è la prima operazione da eseguire nella VM helper del motore del servizio Azure Kubernetes.

La procedura dettagliata è documentata qui:

* [Distribuire un cluster Kubernetes con il motore del servizio Azure Kubernetes nell'hub di Azure Stack](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-cluster)

Il risultato finale del comando `aks-engine deploy` e dei preparativi dei passaggi precedenti è un cluster Kubernetes completo distribuito nello spazio del tenant della prima istanza dell'hub di Azure Stack. Il cluster stesso è costituito da componenti dell'infrastruttura distribuita come servizio (IaaS) di Azure come VM, servizi di bilanciamento del carico, reti virtuali, dischi e così via.

![Portale dell'hub di Azure Stack con i componenti di IaaS del cluster](media/solution-deployment-guide-highly-available-kubernetes/aks-azure-stack-iaas-components.png)

1) Azure Load Balancer (endpoint API K8s)
2) Nodi di lavoro (pool di agenti)
3) Nodi master

Il cluster è ora attivo e operativo ed è possibile connettersi, come illustrato nel passaggio successivo.

## <a name="connect-to-the-kubernetes-cluster"></a>Connettersi al cluster Kubernetes

È ora possibile connettersi al cluster Kubernetes creato in precedenza, tramite SSH (usando la chiave SSH specificata durante la distribuzione) o tramite `kubectl` (scelta consigliata). Lo strumento `kubectl` della riga di comando di Kubernetes è disponibile per Windows, Linux e macOS [qui](https://kubernetes.io/docs/tasks/tools/install-kubectl/). È già preinstallato e configurato nei nodi master del cluster.

```console
ssh azureuser@<k8s-master-lb-ip>
```

![Eseguire kubectl nel nodo master](media/solution-deployment-guide-highly-available-kubernetes/k8s-kubectl-on-master-node.png)

Non è consigliabile usare il nodo master come jumpbox per attività amministrative. La configurazione di `kubectl` è archiviata in `.kube/config` nei nodi master, oltre che nella VM del motore del servizio Azure Kubernetes. È possibile copiare la configurazione e usare il comando `kubectl` in un computer di amministrazione dotato di connettività con il cluster Kubernetes. Il file `.kube/config` viene anche usato in seguito per configurare una connessione del servizio in Azure Pipelines.

> [!IMPORTANT]
> Mantenere questi file al sicuro perché contengono le credenziali per il cluster Kubernetes. Un utente malintenzionato con accesso al file ha informazioni sufficienti per acquisire l'accesso amministratore. Per tutte le azioni eseguite con il file `.kube/config` iniziale viene usato un account amministratore del cluster.

È ora possibile provare vari comandi usando `kubectl` per verificare lo stato del cluster.

```bash
kubectl get nodes
```

```console
NAME                       STATUS   ROLE     VERSION
k8s-linuxpool-35064155-0   Ready    agent    v1.14.8
k8s-linuxpool-35064155-1   Ready    agent    v1.14.8
k8s-linuxpool-35064155-2   Ready    agent    v1.14.8
k8s-master-35064155-0      Ready    master   v1.14.8
```

```bash
kubectl cluster-info
```

```console
Kubernetes master is running at https://aks.***
CoreDNS is running at https://aks.**_/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
kubernetes-dashboard is running at https://aks._*_/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy
Metrics-server is running at https://aks._*_/api/v1/namespaces/kube-system/services/https:metrics-server:/proxy
```

> [!IMPORTANT]
> Kubernetes include un proprio modello di _ *controllo degli accessi in base al ruolo** che consente di creare definizioni di ruolo con granularità fine e binding di ruoli. Questo è il metodo consigliato per controllare l'accesso al cluster al posto dell'assegnazione di autorizzazioni di amministratore del cluster.

## <a name="connect-azure-pipelines-to-kubernetes-clusters"></a>Connettere Azure Pipelines ai cluster Kubernetes

Per connettere Azure Pipelines al cluster Kubernetes appena distribuito, è necessario il file di configurazione kube (`.kube/config`), come descritto nel passaggio precedente.

* Connettersi a uno dei nodi master del cluster Kubernetes.
* Copiare il contenuto del file `.kube/config`.
* Passare a Azure DevOps > Impostazioni progetto > Connessioni al servizio per creare una nuova connessione al servizio "Kubernetes" (usare KubeConfig come metodo di autenticazione)

> [!IMPORTANT]
> Azure Pipelines (o i relativi agenti di compilazione) devono avere accesso all'API Kubernetes. Se è presente una connessione Internet da Azure Pipelines al cluster Kubernetes nell'hub di Azure Stack, è necessario distribuire un agente di compilazione self-hosted di Azure Pipelines.

Gli agenti self-hosted di Azure Pipelines possono essere distribuiti nell'hub di Azure Stack o in un computer con connettività internet a tutti gli endpoint di gestione necessari. Vedere i dettagli qui:

* [Agenti di Azure Pipelines](/azure/devops/pipelines/agents/agents) in [Windows](/azure/devops/pipelines/agents/v2-windows) o [Linux](/azure/devops/pipelines/agents/v2-linux)

La sezione [Considerazioni sulla distribuzione (CI/CD)](pattern-highly-available-kubernetes.md#deployment-cicd-considerations) contiene un flusso decisionale che consente di stabilire se usare agenti ospitati da Microsoft o self-hosted:

[![Flusso decisionale per agenti self-hosted](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png)](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png#lightbox)

In questa soluzione di esempio la topologia include un agente di compilazione self-hosted in ogni istanza dell'hub di Azure Stack. L'agente può accedere agli endpoint di gestione dell'hub di Azure Stack e agli endpoint API del cluster Kubernetes.

[![Solo traffico in uscita](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png#lightbox)

Questa architettura soddisfa un comune requisito normativo, ovvero che devono essere presenti solo connessioni in uscita dalla soluzione dell'applicazione.

## <a name="configure-monitoring"></a>Configurare il monitoraggio

È possibile usare [Monitoraggio di Azure](/azure/azure-monitor/) per i contenitori per monitorare i contenitori nella soluzione. In questo modo Monitoraggio di Azure punta al cluster Kubernetes distribuito con il motore del servizio Azure Kubernetes nell'hub di Azure Stack.

È possibile abilitare Monitoraggio di Azure in due modi nel cluster. In entrambi i casi è necessario configurare un'area di lavoro Log Analytics in Azure.

* Il [metodo uno](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-one) usa un grafico Helm
* Il [metodo due](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-two) fa parte della specifica del cluster del motore del servizio Azure Kubernetes

Nella topologia di esempio viene usato il metodo uno, che consente di automatizzare il processo e di installare più facilmente gli aggiornamenti.

Per il passaggio successivo, è necessario avere un'area di lavoro LogAnalytics di Azure (ID e chiave), `Helm` (versione 3) e `kubectl` nel computer.

Helm è un'utilità di gestione pacchetti di Kubernetes, disponibile come binario eseguibile in macOS, Windows e Linux. Può essere scaricato qui: [helm.sh](https://helm.sh/docs/intro/quickstart/). Helm si basa sul file di configurazione di Kubernetes usato per il comando `kubectl`.

```bash
helm repo add incubator https://kubernetes-charts-incubator.storage.googleapis.com/
helm repo update

helm install incubator/azuremonitor-containers \
--set omsagent.secret.wsid=<your_workspace_id> \
--set omsagent.secret.key=<your_workspace_key> \
--set omsagent.env.clusterName=<my_prod_cluster> \
--generate-name
```

Questo comando installerà l'agente di Monitoraggio di Azure nel cluster Kubernetes:

```bash
kubectl get pods -n kube-system
```

```console
NAME                                       READY   STATUS
omsagent-8qdm6                             1/1     Running
omsagent-r6ppm                             1/1     Running
omsagent-rs-76c45758f5-lmc4l               1/1     Running
```

L'agente OMS (Operations Management Suite) nel cluster Kubernetes invierà i dati di monitoraggio all'area di lavoro Log Analytics di Azure tramite HTTPS in uscita. È ora possibile usare Monitoraggio di Azure per ottenere informazioni approfondite sui cluster Kubernetes nell'hub di Azure Stack. Questa architettura dimostra efficacemente la potenza dell'analisi che è possibile distribuire automaticamente con i cluster dell'applicazione.

[![Cluster dell'hub di Azure Stack in Monitoraggio di Azure](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png#lightbox)

[![Dettagli del cluster di Monitoraggio di Azure](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png#lightbox)

> [!IMPORTANT]
> Se Monitoraggio di Azure non mostra dati dell'hub di Azure Stack, assicurarsi di aver seguito attentamente le istruzioni su [come aggiungere la soluzione Monitoraggio di Azure per i contenitori in un'area di lavoro Log Analytics di Azure](https://github.com/Microsoft/OMS-docker/blob/ci_feature_prod/docs/solution-onboarding.md).

## <a name="deploy-the-application"></a>Distribuire l'applicazione

Prima di installare l'applicazione di esempio, è necessario completare un altro passaggio per configurare il controller in ingresso basato su nginx nel cluster Kubernetes. Il controller in ingresso viene usato come servizio di bilanciamento del carico di livello 7 per instradare in traffico nel cluster in base a host, percorso o protocollo. Il controller Nginx in ingresso è disponibile come grafico Helm. Per le istruzioni dettagliate, vedere il [repository GitHub del grafico Helm](https://github.com/helm/charts/tree/master/stable/nginx-ingress).

Anche l'applicazione di esempio è assemblata come grafico Helm, così come l'[agente di Monitoraggio di Azure](#configure-monitoring) nel passaggio precedente. Di conseguenza, la distribuzione dell'applicazione nel cluster Kubernetes è semplice. È possibile trovare i [file del grafico Helm nel repository GitHub associato](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub/application/helm)

L'applicazione di esempio è a tre livelli e viene distribuita in un cluster Kubernetes in ognuna delle due istanze dell'hub di Azure Stack. L'applicazione usa un database MongoDB. Per altre informazioni su come ottenere i dati replicati tra più istanze, vedere [Considerazioni su dati e archiviazione](pattern-highly-available-kubernetes.md#data-and-storage-considerations) del modello.

Dopo aver distribuito il grafico Helm per l'applicazione, verranno visualizzati tutti e tre i relativi livelli rappresentati come distribuzioni e set con stato (per il database) con un singolo pod:

```kubectl
kubectl get pod,deployment,statefulset
```

```console
NAME                                         READY   STATUS
pod/ratings-api-569d7f7b54-mrv5d             1/1     Running
pod/ratings-mongodb-0                        1/1     Running
pod/ratings-web-85667bfb86-l6vxz             1/1     Running

NAME                                         READY
deployment.extensions/ratings-api            1/1
deployment.extensions/ratings-web            1/1

NAME                                         READY
statefulset.apps/ratings-mongodb             1/1
```

Sul lato servizi si trovano il controller in ingresso basato su nginx e il relativo indirizzo IP pubblico:

```kubectl
kubectl get service
```

```console
NAME                                         TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)
kubernetes                                   ClusterIP      10.0.0.1       <none>        443/TCP
nginx-ingress-1588931383-controller          LoadBalancer   10.0.114.180   *public-ip*   443:30667/TCP
nginx-ingress-1588931383-default-backend     ClusterIP      10.0.76.54     <none>        80/TCP
ratings-api                                  ClusterIP      10.0.46.69     <none>        80/TCP
ratings-web                                  ClusterIP      10.0.161.124   <none>        80/TCP
```

L'indirizzo IP esterno corrisponde all'endpoint applicazione. È il modo in cui gli utenti si connetteranno per aprire l'applicazione e verrà anche usato come endpoint per il passaggio successivo, [Configurare Gestione traffico](#configure-traffic-manager).

## <a name="autoscale-the-application"></a>Dimensionare automaticamente l'applicazione
Facoltativamente è possibile configurare [Horizontal Pod Autoscaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/) per dimensionare l'applicazione in base a specifiche metriche come l'utilizzo della CPU. Il comando seguente creerà un'istanza di Horizontal Pod Autoscaler (HPA) che mantiene da 1 a 10 repliche dei pod controllate dalla distribuzione ratings-web. HPA aumenterà e diminuirà il numero di repliche, tramite la distribuzione, per mantenere un utilizzo medio della CPU dell'80% tra tutti i pod.

```kubectl
kubectl autoscale deployment ratings-web --cpu-percent=80 --min=1 --max=10
```
Per verificare lo stato corrente di HPA, è possibile eseguire:

```kubectl
kubectl get hpa
```
```
NAME         REFERENCE                     TARGET    MINPODS   MAXPODS   REPLICAS   AGE
ratings-web   Deployment/ratings-web/scale   0% / 80%  1         10        1          18s
```
## <a name="configure-traffic-manager"></a>Configurare Gestione traffico

Per distribuire il traffico tra due o più distribuzioni dell'applicazione, si userà [Gestione traffico di Azure](/azure/traffic-manager/traffic-manager-overview). Gestione traffico di Azure è un servizio di bilanciamento del carico basato su DNS disponibile in Azure.

> [!NOTE]
> Gestione traffico usa DNS per indirizzare le richieste client all'endpoint di servizio più appropriato, in base a un metodo di routing del traffico e all'integrità degli endpoint.

Invece di Gestione traffico di Azure, è anche possibile usare altre soluzioni globali di bilanciamento del carico ospitate in locale. Nello scenario di esempio si userà Gestione traffico di Azure per distribuire il traffico tra due istanze dell'applicazione. Per l'esecuzione è possibile scegliere istanze dell'hub di Azure Stack nella stessa posizione o in posizioni diverse:

![Gestione traffico in locale](media/solution-deployment-guide-highly-available-kubernetes/aks-azure-traffic-manager-on-premises.png)

In Azure configurare Gestione traffico in modo che punti alle due istanze diverse dell'applicazione:

[![Profilo dell'endpoint di Gestione traffico](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png)](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png#lightbox)

Come si può notare, i due endpoint puntano alle due istanze dell'applicazione distribuita nella [sezione precedente](#deploy-the-application).

A questo punto:
- L'infrastruttura di Kubernetes è stata creata, incluso un controller in ingresso.
- I cluster sono stati distribuiti tra due istanze dell'hub di Azure Stack.
- Il monitoraggio è stato configurato.
- Gestione traffico di Azure bilancerà il traffico tra le due istanze dell'hub di Azure Stack.
- In questa infrastruttura l'applicazione di esempio a tre livelli è stata distribuita in modo automatizzato tramite grafici Helm. 

La soluzione dovrebbe ora essere attiva e accessibile agli utenti.

Esistono anche alcuni aspetti operativi post-distribuzione da considerare, che verranno descritti nelle due sezioni successive.

## <a name="upgrade-kubernetes"></a>Aggiornare Kubernetes

Considerare gli argomenti seguenti per l'aggiornamento del cluster Kubernetes:

- L'aggiornamento di un cluster Kubernetes è una procedura di manutenzione complessa che è possibile eseguire con il motore del servizio Azure Kubernetes. Per altre informazioni, vedere [Aggiornare un cluster Kubernetes nell'hub di Azure Stack](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade).
- Il motore del servizio Azure Kubernetes consente di aggiornare i cluster alle nuove versioni di Kubernetes e dell'immagine del sistema operativo di base. Per altre informazioni, vedere [Procedura per l'aggiornamento a una nuova versione di Kubernetes](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-upgrade-to-a-newer-kubernetes-version). 
- È anche possibile aggiornare solo i nodi sottostanti alle nuove versioni dell'immagine del sistema operativo di base. Per altre informazioni, vedere [Procedura per aggiornare solo l'immagine del sistema operativo di base](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-only-upgrade-the-os-image).

Le nuove immagini del sistema operativo di base contengono aggiornamenti di sicurezza e del kernel. È responsabilità dell'operatore del cluster monitorare la disponibilità di nuove versioni di Kubernetes e delle immagini del sistema operativo. L'operatore dovrà pianificare ed eseguire questi aggiornamenti usando il motore del servizio Azure Kubernetes. Le immagini del sistema operativo di base devono essere scaricate dal marketplace dell'hub di Azure Stack dall'operatore dell'hub di Azure Stack.

## <a name="scale-kubernetes"></a>Dimensionare Kubernetes

Il dimensionamento è un'altra operazione di manutenzione che può essere orchestrata con il motore del servizio Azure Kubernetes.

Il comando scale riutilizza il file di configurazione del cluster (apimodel.json) disponibile nella directory di output come input per una nuova distribuzione di Azure Resource Manager. Il motore del servizio Azure Kubernetes esegue l'operazione di dimensionamento con uno specifico pool di agenti. Al termine dell'operazione, il motore del servizio Azure Kubernetes aggiorna la definizione del cluster nello stesso file apimodel.json. La definizione del cluster riflette il nuovo numero di nodi in base alla configurazione del cluster corrente aggiornata.

- [Dimensionare un cluster Kubernetes nell'hub di Azure Stack](/azure-stack/user/azure-stack-kubernetes-aks-engine-scale)

## <a name="next-steps"></a>Passaggi successivi

- Altre informazioni sulle [considerazioni per la progettazione di app ibride](overview-app-design-considerations.md)
- Esaminare e proporre miglioramenti del [codice di questo esempio in GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub).