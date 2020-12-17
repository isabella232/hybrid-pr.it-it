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
# <a name="deploy-a-high-availability-kubernetes-cluster-on-azure-stack-hub"></a><span data-ttu-id="2c591-103">Distribuire un cluster Kubernetes a disponibilità elevata nell'hub di Azure Stack</span><span class="sxs-lookup"><span data-stu-id="2c591-103">Deploy a high availability Kubernetes cluster on Azure Stack Hub</span></span>

<span data-ttu-id="2c591-104">Questo articolo illustra come sviluppare un ambiente cluster Kubernetes a disponibilità elevata, distribuito in più istanze dell'hub di Azure Stack, in posizioni fisiche diverse.</span><span class="sxs-lookup"><span data-stu-id="2c591-104">This article will show you how to build a highly available Kubernetes cluster environment, deployed on multiple Azure Stack Hub instances, in different physical locations.</span></span>

<span data-ttu-id="2c591-105">Questa guida alla distribuzione della soluzione descrive come:</span><span class="sxs-lookup"><span data-stu-id="2c591-105">In this solution deployment guide, you learn how to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="2c591-106">Scaricare e preparare il motore del servizio Azure Kubernetes</span><span class="sxs-lookup"><span data-stu-id="2c591-106">Download and prepare the AKS Engine</span></span>
> - <span data-ttu-id="2c591-107">Connettersi alla VM helper del motore del servizio Azure Kubernetes</span><span class="sxs-lookup"><span data-stu-id="2c591-107">Connect to the AKS Engine Helper VM</span></span>
> - <span data-ttu-id="2c591-108">Distribuire un cluster Kubernetes</span><span class="sxs-lookup"><span data-stu-id="2c591-108">Deploy a Kubernetes cluster</span></span>
> - <span data-ttu-id="2c591-109">Connettersi al cluster Kubernetes</span><span class="sxs-lookup"><span data-stu-id="2c591-109">Connect to the Kubernetes cluster</span></span>
> - <span data-ttu-id="2c591-110">Connettere Azure Pipelines al cluster Kubernetes</span><span class="sxs-lookup"><span data-stu-id="2c591-110">Connect Azure Pipelines to Kubernetes cluster</span></span>
> - <span data-ttu-id="2c591-111">Configurare il monitoraggio</span><span class="sxs-lookup"><span data-stu-id="2c591-111">Configure monitoring</span></span>
> - <span data-ttu-id="2c591-112">Distribuire un'applicazione</span><span class="sxs-lookup"><span data-stu-id="2c591-112">Deploy application</span></span>
> - <span data-ttu-id="2c591-113">Dimensionare automaticamente l'applicazione</span><span class="sxs-lookup"><span data-stu-id="2c591-113">Autoscale application</span></span>
> - <span data-ttu-id="2c591-114">Configurare Gestione traffico</span><span class="sxs-lookup"><span data-stu-id="2c591-114">Configure Traffic Manager</span></span>
> - <span data-ttu-id="2c591-115">Aggiornare Kubernetes</span><span class="sxs-lookup"><span data-stu-id="2c591-115">Upgrade Kubernetes</span></span>
> - <span data-ttu-id="2c591-116">Dimensionare Kubernetes</span><span class="sxs-lookup"><span data-stu-id="2c591-116">Scale Kubernetes</span></span>

> [!Tip]  
> <span data-ttu-id="2c591-117">![Pilastri ibridi](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="2c591-117">![Hybrid pillars](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="2c591-118">L'hub di Microsoft Azure Stack è un'estensione di Azure.</span><span class="sxs-lookup"><span data-stu-id="2c591-118">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="2c591-119">L'hub di Azure Stack offre all'ambiente locale l'agilità e l'innovazione del cloud computing, abilitando l'unico cloud ibrido che consente di creare e distribuire ovunque app ibride.</span><span class="sxs-lookup"><span data-stu-id="2c591-119">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="2c591-120">L'articolo [Considerazioni per la progettazione di app ibride](overview-app-design-considerations.md) esamina i concetti fondamentali di qualità del software (posizionamento, scalabilità, disponibilità, resilienza, gestibilità e sicurezza) per la progettazione, la distribuzione e la gestione di app ibride.</span><span class="sxs-lookup"><span data-stu-id="2c591-120">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="2c591-121">Le considerazioni di progettazione consentono di ottimizzare la progettazione delle app ibride, riducendo al minimo i rischi negli ambienti di produzione.</span><span class="sxs-lookup"><span data-stu-id="2c591-121">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="2c591-122">Prerequisiti</span><span class="sxs-lookup"><span data-stu-id="2c591-122">Prerequisites</span></span>

<span data-ttu-id="2c591-123">Prima di iniziare a usare questa guida alla distribuzione:</span><span class="sxs-lookup"><span data-stu-id="2c591-123">Before getting started with this deployment guide, make sure you:</span></span>

- <span data-ttu-id="2c591-124">Vedere l'articolo [Modello di cluster Kubernetes a disponibilità elevata](pattern-highly-available-kubernetes.md).</span><span class="sxs-lookup"><span data-stu-id="2c591-124">Review the [High availability Kubernetes cluster pattern](pattern-highly-available-kubernetes.md) article.</span></span>
- <span data-ttu-id="2c591-125">Esaminare il contenuto del [repository GitHub associato](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub), che contiene altri asset a cui viene fatto riferimento in questo articolo.</span><span class="sxs-lookup"><span data-stu-id="2c591-125">Review the contents of the [companion GitHub repository](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub), which contains additional assets referenced in this article.</span></span>
- <span data-ttu-id="2c591-126">Avere a disposizione un account in grado di accedere al [portale utente dell'hub di Azure Stack](/azure-stack/user/azure-stack-use-portal), con almeno le [autorizzazioni di "collaboratore"](/azure-stack/user/azure-stack-manage-permissions).</span><span class="sxs-lookup"><span data-stu-id="2c591-126">Have an account that can access the [Azure Stack Hub user portal](/azure-stack/user/azure-stack-use-portal), with at least ["contributor" permissions](/azure-stack/user/azure-stack-manage-permissions).</span></span>

## <a name="download-and-prepare-aks-engine"></a><span data-ttu-id="2c591-127">Scaricare e preparare il motore del servizio Azure Kubernetes</span><span class="sxs-lookup"><span data-stu-id="2c591-127">Download and prepare AKS Engine</span></span>

<span data-ttu-id="2c591-128">Il motore del servizio Azure Kubernetes è un binario che è possibile usare da qualsiasi host Windows o Linux e che può raggiungere gli endpoint di Azure Resource Manager dell'hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="2c591-128">AKS Engine is a binary that can be used from any Windows or Linux host that can reach the Azure Stack Hub Azure Resource Manager endpoints.</span></span> <span data-ttu-id="2c591-129">Questa guida descrive la distribuzione di una nuova VM Linux (o Windows) nell'hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="2c591-129">This guide describes deploying a new Linux (or Windows) VM on Azure Stack Hub.</span></span> <span data-ttu-id="2c591-130">Verrà usata in seguito quando il motore del servizio Azure Kubernetes distribuirà i cluster Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="2c591-130">It will be used later when AKS Engine deploys the Kubernetes clusters.</span></span>

> [!NOTE]
> <span data-ttu-id="2c591-131">Per distribuire un cluster Kubernetes nell'hub di Azure Stack con il motore del servizio Azure Kubernetes, è anche possibile usare una VM Windows o Linux esistente.</span><span class="sxs-lookup"><span data-stu-id="2c591-131">You can also use an existing Windows or Linux VM to deploy a Kubernetes cluster on Azure Stack Hub using AKS Engine.</span></span>

<span data-ttu-id="2c591-132">La procedura dettagliata e i requisiti per il motore del servizio Azure Kubernetes sono documentati qui:</span><span class="sxs-lookup"><span data-stu-id="2c591-132">The step-by-step process and requirements for AKS Engine are documented here:</span></span>

* <span data-ttu-id="2c591-133">[Installare il motore del servizio Azure Kubernetes in Linux nell'hub di Azure Stack](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-linux) (oppure con [Windows](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-windows))</span><span class="sxs-lookup"><span data-stu-id="2c591-133">[Install the AKS Engine on Linux in Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-linux) (or using [Windows](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-windows))</span></span>

<span data-ttu-id="2c591-134">Il motore del servizio Azure Kubernetes è uno strumento helper per distribuire e utilizzare cluster Kubernetes non gestiti in Azure e nell'hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="2c591-134">AKS Engine is a helper tool to deploy and operate (unmanaged) Kubernetes clusters (in Azure and Azure Stack Hub).</span></span>

<span data-ttu-id="2c591-135">I dettagli e le differenze del motore del servizio Azure Kubernetes nell'hub di Azure Stack sono descritti qui:</span><span class="sxs-lookup"><span data-stu-id="2c591-135">The details and differences of AKS Engine on Azure Stack Hub are described here:</span></span>

* [<span data-ttu-id="2c591-136">Che cos'è il motore del servizio Azure Kubernetes nell'hub di Azure Stack?</span><span class="sxs-lookup"><span data-stu-id="2c591-136">What is the AKS Engine on Azure Stack Hub?</span></span>](/azure-stack/user/azure-stack-kubernetes-aks-engine-overview)
* <span data-ttu-id="2c591-137">[Motore del servizio Azure Kubernetes nell'hub di Azure Stack](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md) (in GitHub)</span><span class="sxs-lookup"><span data-stu-id="2c591-137">[AKS Engine on Azure Stack Hub](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md) (on GitHub)</span></span>

<span data-ttu-id="2c591-138">Nell'ambiente di esempio si userà Terraform per automatizzare la distribuzione della VM del motore del servizio Azure Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="2c591-138">The sample environment will use Terraform to automate the deployment of the AKS Engine VM.</span></span> <span data-ttu-id="2c591-139">È possibile trovare i [dettagli e il codice nel repository GitHub associato](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/blob/master/AKSe-on-AzStackHub/src/tf/aksengine/README.md).</span><span class="sxs-lookup"><span data-stu-id="2c591-139">You can find the [details and code in the companion GitHub repo](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/blob/master/AKSe-on-AzStackHub/src/tf/aksengine/README.md).</span></span>

<span data-ttu-id="2c591-140">Il risultato di questo passaggio è un nuovo gruppo di risorse nell'hub di Azure Stack che contiene la VM helper del motore del servizio Azure Kubernetes e le risorse correlate:</span><span class="sxs-lookup"><span data-stu-id="2c591-140">The result of this step is a new resource group on Azure Stack Hub that contains the AKS Engine helper VM and related resources:</span></span>

![Risorse della VM del motore del servizio Azure Kubernetes nell'hub di Azure Stack](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-resources-on-azure-stack.png)

> [!NOTE]
> <span data-ttu-id="2c591-142">Se è necessario distribuire il motore del servizio Azure Kubernetes in un ambiente disconnesso o in configurazione air gap, vedere [Istanze disconnesse dell'hub di Azure Stack](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#disconnected-azure-stack-hub-instances) per altre informazioni.</span><span class="sxs-lookup"><span data-stu-id="2c591-142">If you have to deploy AKS Engine in a disconnected air-gapped environment, review [Disconnected Azure Stack Hub Instances](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#disconnected-azure-stack-hub-instances) to learn more.</span></span>

<span data-ttu-id="2c591-143">Nel passaggio successivo si userà la VM del motore del servizio Azure Kubernetes appena creata per distribuire un cluster Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="2c591-143">In the next step, we'll use the newly deployed AKS Engine VM to deploy a Kubernetes cluster.</span></span>

## <a name="connect-to-the-aks-engine-helper-vm"></a><span data-ttu-id="2c591-144">Connettersi alla VM helper del motore del servizio Azure Kubernetes</span><span class="sxs-lookup"><span data-stu-id="2c591-144">Connect to the AKS Engine helper VM</span></span>

<span data-ttu-id="2c591-145">Prima di tutto è necessario connettersi alla VM helper del motore del servizio Azure Kubernetes creata in precedenza.</span><span class="sxs-lookup"><span data-stu-id="2c591-145">First you must connect to the previously created AKS Engine helper VM.</span></span>

<span data-ttu-id="2c591-146">La VM dovrà avere un indirizzo IP pubblico e dovrà essere accessibile tramite SSH (porta 22/TCP).</span><span class="sxs-lookup"><span data-stu-id="2c591-146">The VM should have a Public IP Address and should be accessible via SSH (Port 22/TCP).</span></span>

![Pagina di panoramica della VM del motore del servizio Azure Kubernetes](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-vm-overview.png)

> [!TIP]
> <span data-ttu-id="2c591-148">Per connettersi a una VM Linux tramite SSH, è possibile usare uno strumento a scelta, come MobaXterm, puTTY o PowerShell in Windows 10.</span><span class="sxs-lookup"><span data-stu-id="2c591-148">You can use a tool of your choice like MobaXterm, puTTY or PowerShell in Windows 10 to connect to a Linux VM using SSH.</span></span>

```console
ssh <username>@<ipaddress>
```

<span data-ttu-id="2c591-149">Dopo aver stabilito la connessione, eseguire il comando `aks-engine`.</span><span class="sxs-lookup"><span data-stu-id="2c591-149">After connecting, run the command `aks-engine`.</span></span> <span data-ttu-id="2c591-150">Vedere [Versioni supportate del motore del servizio Azure Kubernetes](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#supported-aks-engine-versions) per altre informazioni sulle versioni del motore e di Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="2c591-150">Go to [Supported AKS Engine Versions](https://github.com/Azure/aks-engine/blob/master/docs/topics/azure-stack.md#supported-aks-engine-versions) to learn more about the AKS Engine and Kubernetes versions.</span></span>

![Esempio di riga di comando ask-engine](media/solution-deployment-guide-highly-available-kubernetes/aks-engine-cmdline-example.png)

## <a name="deploy-a-kubernetes-cluster"></a><span data-ttu-id="2c591-152">Distribuire un cluster Kubernetes</span><span class="sxs-lookup"><span data-stu-id="2c591-152">Deploy a Kubernetes cluster</span></span>

<span data-ttu-id="2c591-153">La VM helper del motore del servizio Azure Kubernetes non ha ancora creato un cluster Kubernetes nell'hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="2c591-153">The AKS Engine helper VM itself hasn't created a Kubernetes cluster on our Azure Stack Hub, yet.</span></span> <span data-ttu-id="2c591-154">La creazione del cluster è la prima operazione da eseguire nella VM helper del motore del servizio Azure Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="2c591-154">Creating the cluster is the first action to take in the AKS Engine helper VM.</span></span>

<span data-ttu-id="2c591-155">La procedura dettagliata è documentata qui:</span><span class="sxs-lookup"><span data-stu-id="2c591-155">The step-by-step process is documented here:</span></span>

* [<span data-ttu-id="2c591-156">Distribuire un cluster Kubernetes con il motore del servizio Azure Kubernetes nell'hub di Azure Stack</span><span class="sxs-lookup"><span data-stu-id="2c591-156">Deploy a Kubernetes cluster with the AKS engine on Azure Stack Hub</span></span>](/azure-stack/user/azure-stack-kubernetes-aks-engine-deploy-cluster)

<span data-ttu-id="2c591-157">Il risultato finale del comando `aks-engine deploy` e dei preparativi dei passaggi precedenti è un cluster Kubernetes completo distribuito nello spazio del tenant della prima istanza dell'hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="2c591-157">The end result of the `aks-engine deploy` command and the preparations in the previous steps is a fully featured Kubernetes cluster deployed into the tenant space of the first Azure Stack Hub instance.</span></span> <span data-ttu-id="2c591-158">Il cluster stesso è costituito da componenti dell'infrastruttura distribuita come servizio (IaaS) di Azure come VM, servizi di bilanciamento del carico, reti virtuali, dischi e così via.</span><span class="sxs-lookup"><span data-stu-id="2c591-158">The cluster itself consists of Azure IaaS components like VMs, load balancers, VNets, disks, and so on.</span></span>

![Portale dell'hub di Azure Stack con i componenti di IaaS del cluster](media/solution-deployment-guide-highly-available-kubernetes/aks-azure-stack-iaas-components.png)

1) <span data-ttu-id="2c591-160">Azure Load Balancer (endpoint API K8s)</span><span class="sxs-lookup"><span data-stu-id="2c591-160">Azure load balancer (K8s API Endpoint)</span></span>
2) <span data-ttu-id="2c591-161">Nodi di lavoro (pool di agenti)</span><span class="sxs-lookup"><span data-stu-id="2c591-161">Worker Nodes (Agent Pool)</span></span>
3) <span data-ttu-id="2c591-162">Nodi master</span><span class="sxs-lookup"><span data-stu-id="2c591-162">Master Nodes</span></span>

<span data-ttu-id="2c591-163">Il cluster è ora attivo e operativo ed è possibile connettersi, come illustrato nel passaggio successivo.</span><span class="sxs-lookup"><span data-stu-id="2c591-163">The cluster is now up-and-running and in the next step we'll connect to it.</span></span>

## <a name="connect-to-the-kubernetes-cluster"></a><span data-ttu-id="2c591-164">Connettersi al cluster Kubernetes</span><span class="sxs-lookup"><span data-stu-id="2c591-164">Connect to the Kubernetes cluster</span></span>

<span data-ttu-id="2c591-165">È ora possibile connettersi al cluster Kubernetes creato in precedenza, tramite SSH (usando la chiave SSH specificata durante la distribuzione) o tramite `kubectl` (scelta consigliata).</span><span class="sxs-lookup"><span data-stu-id="2c591-165">You can now connect to the previously created Kubernetes cluster, either via SSH (using the SSH key specified as part of the deployment) or via `kubectl` (recommended).</span></span> <span data-ttu-id="2c591-166">Lo strumento `kubectl` della riga di comando di Kubernetes è disponibile per Windows, Linux e macOS [qui](https://kubernetes.io/docs/tasks/tools/install-kubectl/).</span><span class="sxs-lookup"><span data-stu-id="2c591-166">The Kubernetes command-line tool `kubectl` is available for Windows, Linux, and macOS [here](https://kubernetes.io/docs/tasks/tools/install-kubectl/).</span></span> <span data-ttu-id="2c591-167">È già preinstallato e configurato nei nodi master del cluster.</span><span class="sxs-lookup"><span data-stu-id="2c591-167">It's already pre-installed and configured on the master nodes of our cluster.</span></span>

```console
ssh azureuser@<k8s-master-lb-ip>
```

![Eseguire kubectl nel nodo master](media/solution-deployment-guide-highly-available-kubernetes/k8s-kubectl-on-master-node.png)

<span data-ttu-id="2c591-169">Non è consigliabile usare il nodo master come jumpbox per attività amministrative.</span><span class="sxs-lookup"><span data-stu-id="2c591-169">It's not recommended to use the master node as a jumpbox for administrative tasks.</span></span> <span data-ttu-id="2c591-170">La configurazione di `kubectl` è archiviata in `.kube/config` nei nodi master, oltre che nella VM del motore del servizio Azure Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="2c591-170">The `kubectl` configuration is stored in `.kube/config` on the master node(s) as well as on the AKS Engine VM.</span></span> <span data-ttu-id="2c591-171">È possibile copiare la configurazione e usare il comando `kubectl` in un computer di amministrazione dotato di connettività con il cluster Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="2c591-171">You can copy the configuration to an admin machine with connectivity to the Kubernetes cluster and use the `kubectl` command there.</span></span> <span data-ttu-id="2c591-172">Il file `.kube/config` viene anche usato in seguito per configurare una connessione del servizio in Azure Pipelines.</span><span class="sxs-lookup"><span data-stu-id="2c591-172">The `.kube/config` file is also used later to configure a service connection in Azure Pipelines.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="2c591-173">Mantenere questi file al sicuro perché contengono le credenziali per il cluster Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="2c591-173">Keep these files secure because they contain the credentials for your Kubernetes cluster.</span></span> <span data-ttu-id="2c591-174">Un utente malintenzionato con accesso al file ha informazioni sufficienti per acquisire l'accesso amministratore.</span><span class="sxs-lookup"><span data-stu-id="2c591-174">An attacker with access to the file has enough information to gain administrator access to it.</span></span> <span data-ttu-id="2c591-175">Per tutte le azioni eseguite con il file `.kube/config` iniziale viene usato un account amministratore del cluster.</span><span class="sxs-lookup"><span data-stu-id="2c591-175">All actions that are done using the initial `.kube/config` file are done using a cluster-admin account.</span></span>

<span data-ttu-id="2c591-176">È ora possibile provare vari comandi usando `kubectl` per verificare lo stato del cluster.</span><span class="sxs-lookup"><span data-stu-id="2c591-176">You can now try various commands using `kubectl` to check the status of your cluster.</span></span>

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
> <span data-ttu-id="2c591-177">Kubernetes include un proprio modello di _ *controllo degli accessi in base al ruolo*\* che consente di creare definizioni di ruolo con granularità fine e binding di ruoli.</span><span class="sxs-lookup"><span data-stu-id="2c591-177">Kubernetes has its own _ *Role-based Access Control (RBAC)*\* model that allows you to create fine-grained role definitions and role bindings.</span></span> <span data-ttu-id="2c591-178">Questo è il metodo consigliato per controllare l'accesso al cluster al posto dell'assegnazione di autorizzazioni di amministratore del cluster.</span><span class="sxs-lookup"><span data-stu-id="2c591-178">This is the preferable way to control access to the cluster instead of handing out cluster-admin permissions.</span></span>

## <a name="connect-azure-pipelines-to-kubernetes-clusters"></a><span data-ttu-id="2c591-179">Connettere Azure Pipelines ai cluster Kubernetes</span><span class="sxs-lookup"><span data-stu-id="2c591-179">Connect Azure Pipelines to Kubernetes clusters</span></span>

<span data-ttu-id="2c591-180">Per connettere Azure Pipelines al cluster Kubernetes appena distribuito, è necessario il file di configurazione kube (`.kube/config`), come descritto nel passaggio precedente.</span><span class="sxs-lookup"><span data-stu-id="2c591-180">To connect Azure Pipelines to the newly deployed Kubernetes cluster, we need its kube config (`.kube/config`) file as explained in the previous step.</span></span>

* <span data-ttu-id="2c591-181">Connettersi a uno dei nodi master del cluster Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="2c591-181">Connect to one of the master nodes of your Kubernetes cluster.</span></span>
* <span data-ttu-id="2c591-182">Copiare il contenuto del file `.kube/config`.</span><span class="sxs-lookup"><span data-stu-id="2c591-182">Copy the content of the `.kube/config` file.</span></span>
* <span data-ttu-id="2c591-183">Passare a Azure DevOps > Impostazioni progetto > Connessioni al servizio per creare una nuova connessione al servizio "Kubernetes" (usare KubeConfig come metodo di autenticazione)</span><span class="sxs-lookup"><span data-stu-id="2c591-183">Go to Azure DevOps > Project Settings > Service Connections to create a new "Kubernetes" service connection (use KubeConfig as Authentication method)</span></span>

> [!IMPORTANT]
> <span data-ttu-id="2c591-184">Azure Pipelines (o i relativi agenti di compilazione) devono avere accesso all'API Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="2c591-184">Azure Pipelines (or its build agents) must have access to the Kubernetes API.</span></span> <span data-ttu-id="2c591-185">Se è presente una connessione Internet da Azure Pipelines al cluster Kubernetes nell'hub di Azure Stack, è necessario distribuire un agente di compilazione self-hosted di Azure Pipelines.</span><span class="sxs-lookup"><span data-stu-id="2c591-185">If there is an Internet connection from Azure Pipelines to the Azure Stack Hub Kubernetes clusetr, you'll need to deploy a self-hosted Azure Pipelines Build Agent.</span></span>

<span data-ttu-id="2c591-186">Gli agenti self-hosted di Azure Pipelines possono essere distribuiti nell'hub di Azure Stack o in un computer con connettività internet a tutti gli endpoint di gestione necessari.</span><span class="sxs-lookup"><span data-stu-id="2c591-186">When deploying self-hosted Agents for Azure Pipelines, you may deploy either on Azure Stack Hub, or on a machine with network connectivity to all required management endpoints.</span></span> <span data-ttu-id="2c591-187">Vedere i dettagli qui:</span><span class="sxs-lookup"><span data-stu-id="2c591-187">See the details here:</span></span>

* <span data-ttu-id="2c591-188">[Agenti di Azure Pipelines](/azure/devops/pipelines/agents/agents) in [Windows](/azure/devops/pipelines/agents/v2-windows) o [Linux](/azure/devops/pipelines/agents/v2-linux)</span><span class="sxs-lookup"><span data-stu-id="2c591-188">[Azure Pipelines agents](/azure/devops/pipelines/agents/agents) on [Windows](/azure/devops/pipelines/agents/v2-windows) or [Linux](/azure/devops/pipelines/agents/v2-linux)</span></span>

<span data-ttu-id="2c591-189">La sezione [Considerazioni sulla distribuzione (CI/CD)](pattern-highly-available-kubernetes.md#deployment-cicd-considerations) contiene un flusso decisionale che consente di stabilire se usare agenti ospitati da Microsoft o self-hosted:</span><span class="sxs-lookup"><span data-stu-id="2c591-189">The pattern [Deployment (CI/CD) considerations](pattern-highly-available-kubernetes.md#deployment-cicd-considerations) section contains a decision flow that helps you to understand whether to use Microsoft-hosted agents or self-hosted agents:</span></span>

<span data-ttu-id="2c591-190">[![Flusso decisionale per agenti self-hosted](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png)](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="2c591-190">[![decision flow self hosted agents](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png)](media/solution-deployment-guide-highly-available-kubernetes/aks-on-stack-self-hosted-build-agents-yes-or-no.png#lightbox)</span></span>

<span data-ttu-id="2c591-191">In questa soluzione di esempio la topologia include un agente di compilazione self-hosted in ogni istanza dell'hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="2c591-191">In this sample solution, the topology includes a self-hosted build agent on each Azure Stack Hub instance.</span></span> <span data-ttu-id="2c591-192">L'agente può accedere agli endpoint di gestione dell'hub di Azure Stack e agli endpoint API del cluster Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="2c591-192">The agent can access the Azure Stack Hub Management Endpoints and the Kubernetes cluster API endpoints.</span></span>

<span data-ttu-id="2c591-193">[![Solo traffico in uscita](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="2c591-193">[![only outbound traffic](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-stack-architecture-only-outbound-traffic.png#lightbox)</span></span>

<span data-ttu-id="2c591-194">Questa architettura soddisfa un comune requisito normativo, ovvero che devono essere presenti solo connessioni in uscita dalla soluzione dell'applicazione.</span><span class="sxs-lookup"><span data-stu-id="2c591-194">This design fulfills a common regulatory requirement, which is to have only outbound connections from the application solution.</span></span>

## <a name="configure-monitoring"></a><span data-ttu-id="2c591-195">Configurare il monitoraggio</span><span class="sxs-lookup"><span data-stu-id="2c591-195">Configure monitoring</span></span>

<span data-ttu-id="2c591-196">È possibile usare [Monitoraggio di Azure](/azure/azure-monitor/) per i contenitori per monitorare i contenitori nella soluzione.</span><span class="sxs-lookup"><span data-stu-id="2c591-196">You can use [Azure Monitor](/azure/azure-monitor/) for containers to monitor the containers in the solution.</span></span> <span data-ttu-id="2c591-197">In questo modo Monitoraggio di Azure punta al cluster Kubernetes distribuito con il motore del servizio Azure Kubernetes nell'hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="2c591-197">This points Azure Monitor to the AKS Engine-deployed Kubernetes cluster on Azure Stack Hub.</span></span>

<span data-ttu-id="2c591-198">È possibile abilitare Monitoraggio di Azure in due modi nel cluster.</span><span class="sxs-lookup"><span data-stu-id="2c591-198">There are two ways to enable Azure Monitor on your cluster.</span></span> <span data-ttu-id="2c591-199">In entrambi i casi è necessario configurare un'area di lavoro Log Analytics in Azure.</span><span class="sxs-lookup"><span data-stu-id="2c591-199">Both ways require you to set up a Log Analytics workspace in Azure.</span></span>

* <span data-ttu-id="2c591-200">Il [metodo uno](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-one) usa un grafico Helm</span><span class="sxs-lookup"><span data-stu-id="2c591-200">[Method one](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-one) uses a Helm Chart</span></span>
* <span data-ttu-id="2c591-201">Il [metodo due](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-two) fa parte della specifica del cluster del motore del servizio Azure Kubernetes</span><span class="sxs-lookup"><span data-stu-id="2c591-201">[Method two](/azure-stack/user/kubernetes-aks-engine-azure-monitor#method-two) as part of the AKS Engine cluster specification</span></span>

<span data-ttu-id="2c591-202">Nella topologia di esempio viene usato il metodo uno, che consente di automatizzare il processo e di installare più facilmente gli aggiornamenti.</span><span class="sxs-lookup"><span data-stu-id="2c591-202">In the sample topology, "Method one" is used, which allows automation of the process and updates can be installed more easily.</span></span>

<span data-ttu-id="2c591-203">Per il passaggio successivo, è necessario avere un'area di lavoro LogAnalytics di Azure (ID e chiave), `Helm` (versione 3) e `kubectl` nel computer.</span><span class="sxs-lookup"><span data-stu-id="2c591-203">For the next step, you need an Azure LogAnalytics Workspace (ID and Key), `Helm` (version 3), and `kubectl` on your machine.</span></span>

<span data-ttu-id="2c591-204">Helm è un'utilità di gestione pacchetti di Kubernetes, disponibile come binario eseguibile in macOS, Windows e Linux.</span><span class="sxs-lookup"><span data-stu-id="2c591-204">Helm is a Kubernetes package manager, available as a binary that is runs on macOS, Windows, and Linux.</span></span> <span data-ttu-id="2c591-205">Può essere scaricato qui: [helm.sh](https://helm.sh/docs/intro/quickstart/). Helm si basa sul file di configurazione di Kubernetes usato per il comando `kubectl`.</span><span class="sxs-lookup"><span data-stu-id="2c591-205">It can be downloaded here: [helm.sh](https://helm.sh/docs/intro/quickstart/) Helm relies on the Kubernetes configuration file used for the `kubectl` command.</span></span>

```bash
helm repo add incubator https://kubernetes-charts-incubator.storage.googleapis.com/
helm repo update

helm install incubator/azuremonitor-containers \
--set omsagent.secret.wsid=<your_workspace_id> \
--set omsagent.secret.key=<your_workspace_key> \
--set omsagent.env.clusterName=<my_prod_cluster> \
--generate-name
```

<span data-ttu-id="2c591-206">Questo comando installerà l'agente di Monitoraggio di Azure nel cluster Kubernetes:</span><span class="sxs-lookup"><span data-stu-id="2c591-206">This command will install the Azure Monitor agent on your Kubernetes cluster:</span></span>

```bash
kubectl get pods -n kube-system
```

```console
NAME                                       READY   STATUS
omsagent-8qdm6                             1/1     Running
omsagent-r6ppm                             1/1     Running
omsagent-rs-76c45758f5-lmc4l               1/1     Running
```

<span data-ttu-id="2c591-207">L'agente OMS (Operations Management Suite) nel cluster Kubernetes invierà i dati di monitoraggio all'area di lavoro Log Analytics di Azure tramite HTTPS in uscita.</span><span class="sxs-lookup"><span data-stu-id="2c591-207">The Operations Management Suite (OMS) Agent on your Kubernetes cluster will send monitoring data to your Azure Log Analytics Workspace (using outbound HTTPS).</span></span> <span data-ttu-id="2c591-208">È ora possibile usare Monitoraggio di Azure per ottenere informazioni approfondite sui cluster Kubernetes nell'hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="2c591-208">You can now use Azure Monitor to get deeper insights about your Kubernetes clusters on Azure Stack Hub.</span></span> <span data-ttu-id="2c591-209">Questa architettura dimostra efficacemente la potenza dell'analisi che è possibile distribuire automaticamente con i cluster dell'applicazione.</span><span class="sxs-lookup"><span data-stu-id="2c591-209">This design is a powerful way to demonstrate the power of analytics that can be automatically deployed with your application's clusters.</span></span>

<span data-ttu-id="2c591-210">[![Cluster dell'hub di Azure Stack in Monitoraggio di Azure](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="2c591-210">[![Azure Stack Hub clusters in Azure monitor](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-1.png#lightbox)</span></span>

<span data-ttu-id="2c591-211">[![Dettagli del cluster di Monitoraggio di Azure](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="2c591-211">[![Azure Monitor cluster details](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png)](media/solution-deployment-guide-highly-available-kubernetes/azure-monitor-on-stack-2.png#lightbox)</span></span>

> [!IMPORTANT]
> <span data-ttu-id="2c591-212">Se Monitoraggio di Azure non mostra dati dell'hub di Azure Stack, assicurarsi di aver seguito attentamente le istruzioni su [come aggiungere la soluzione Monitoraggio di Azure per i contenitori in un'area di lavoro Log Analytics di Azure](https://github.com/Microsoft/OMS-docker/blob/ci_feature_prod/docs/solution-onboarding.md).</span><span class="sxs-lookup"><span data-stu-id="2c591-212">If Azure Monitor does not show any Azure Stack Hub data, please make sure that you have followed the instructions on [how to add AzureMonitor-Containers solution to a Azure Loganalytics workspace](https://github.com/Microsoft/OMS-docker/blob/ci_feature_prod/docs/solution-onboarding.md) carefully.</span></span>

## <a name="deploy-the-application"></a><span data-ttu-id="2c591-213">Distribuire l'applicazione</span><span class="sxs-lookup"><span data-stu-id="2c591-213">Deploy the application</span></span>

<span data-ttu-id="2c591-214">Prima di installare l'applicazione di esempio, è necessario completare un altro passaggio per configurare il controller in ingresso basato su nginx nel cluster Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="2c591-214">Before installing our sample application, there's another step to configure the nginx-based Ingress controller on our Kubernetes cluster.</span></span> <span data-ttu-id="2c591-215">Il controller in ingresso viene usato come servizio di bilanciamento del carico di livello 7 per instradare in traffico nel cluster in base a host, percorso o protocollo.</span><span class="sxs-lookup"><span data-stu-id="2c591-215">The Ingress controller is used as a layer 7 load balancer to route traffic in our cluster based on host, path, or protocol.</span></span> <span data-ttu-id="2c591-216">Il controller Nginx in ingresso è disponibile come grafico Helm.</span><span class="sxs-lookup"><span data-stu-id="2c591-216">Nginx-ingress is available as a Helm Chart.</span></span> <span data-ttu-id="2c591-217">Per le istruzioni dettagliate, vedere il [repository GitHub del grafico Helm](https://github.com/helm/charts/tree/master/stable/nginx-ingress).</span><span class="sxs-lookup"><span data-stu-id="2c591-217">For detailed instructions, refer to the [Helm Chart GitHub repository](https://github.com/helm/charts/tree/master/stable/nginx-ingress).</span></span>

<span data-ttu-id="2c591-218">Anche l'applicazione di esempio è assemblata come grafico Helm, così come l'[agente di Monitoraggio di Azure](#configure-monitoring) nel passaggio precedente.</span><span class="sxs-lookup"><span data-stu-id="2c591-218">Our sample application is also packaged as a Helm Chart, like the [Azure Monitoring Agent](#configure-monitoring) in the previous step.</span></span> <span data-ttu-id="2c591-219">Di conseguenza, la distribuzione dell'applicazione nel cluster Kubernetes è semplice.</span><span class="sxs-lookup"><span data-stu-id="2c591-219">As such, it's straightforward to deploy the application onto our Kubernetes cluster.</span></span> <span data-ttu-id="2c591-220">È possibile trovare i [file del grafico Helm nel repository GitHub associato](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub/application/helm)</span><span class="sxs-lookup"><span data-stu-id="2c591-220">You can find the [Helm Chart files in the companion GitHub repo](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub/application/helm)</span></span>

<span data-ttu-id="2c591-221">L'applicazione di esempio è a tre livelli e viene distribuita in un cluster Kubernetes in ognuna delle due istanze dell'hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="2c591-221">The sample application is a three tier application, deployed onto a Kubernetes cluster on each of two Azure Stack Hub instances.</span></span> <span data-ttu-id="2c591-222">L'applicazione usa un database MongoDB.</span><span class="sxs-lookup"><span data-stu-id="2c591-222">The application uses a MongoDB database.</span></span> <span data-ttu-id="2c591-223">Per altre informazioni su come ottenere i dati replicati tra più istanze, vedere [Considerazioni su dati e archiviazione](pattern-highly-available-kubernetes.md#data-and-storage-considerations) del modello.</span><span class="sxs-lookup"><span data-stu-id="2c591-223">You can learn more about how to get the data replicated across multiple instances in the pattern [Data and Storage considerations](pattern-highly-available-kubernetes.md#data-and-storage-considerations).</span></span>

<span data-ttu-id="2c591-224">Dopo aver distribuito il grafico Helm per l'applicazione, verranno visualizzati tutti e tre i relativi livelli rappresentati come distribuzioni e set con stato (per il database) con un singolo pod:</span><span class="sxs-lookup"><span data-stu-id="2c591-224">After deploying the Helm Chart for the application, you'll see all three tiers of your application represented as deployments and stateful sets (for the database) with a single pod:</span></span>

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

<span data-ttu-id="2c591-225">Sul lato servizi si trovano il controller in ingresso basato su nginx e il relativo indirizzo IP pubblico:</span><span class="sxs-lookup"><span data-stu-id="2c591-225">On the services, side you'll find the nginx-based Ingress Controller and its public IP address:</span></span>

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

<span data-ttu-id="2c591-226">L'indirizzo IP esterno corrisponde all'endpoint applicazione.</span><span class="sxs-lookup"><span data-stu-id="2c591-226">The "External IP" address is our "application endpoint".</span></span> <span data-ttu-id="2c591-227">È il modo in cui gli utenti si connetteranno per aprire l'applicazione e verrà anche usato come endpoint per il passaggio successivo, [Configurare Gestione traffico](#configure-traffic-manager).</span><span class="sxs-lookup"><span data-stu-id="2c591-227">It's how users will connect to open the application and will also be used as the endpoint for our next step [Configure Traffic Manager](#configure-traffic-manager).</span></span>

## <a name="autoscale-the-application"></a><span data-ttu-id="2c591-228">Dimensionare automaticamente l'applicazione</span><span class="sxs-lookup"><span data-stu-id="2c591-228">Autoscale the application</span></span>
<span data-ttu-id="2c591-229">Facoltativamente è possibile configurare [Horizontal Pod Autoscaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/) per dimensionare l'applicazione in base a specifiche metriche come l'utilizzo della CPU.</span><span class="sxs-lookup"><span data-stu-id="2c591-229">You can optionally configure the [Horizontal Pod Autoscaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/) to scale up or down based on certain metrics like CPU utilization.</span></span> <span data-ttu-id="2c591-230">Il comando seguente creerà un'istanza di Horizontal Pod Autoscaler (HPA) che mantiene da 1 a 10 repliche dei pod controllate dalla distribuzione ratings-web.</span><span class="sxs-lookup"><span data-stu-id="2c591-230">The following command will create a Horizontal Pod Autoscaler that maintains 1 to 10 replicas of the Pods controlled by the ratings-web deployment.</span></span> <span data-ttu-id="2c591-231">HPA aumenterà e diminuirà il numero di repliche, tramite la distribuzione, per mantenere un utilizzo medio della CPU dell'80% tra tutti i pod.</span><span class="sxs-lookup"><span data-stu-id="2c591-231">HPA will increase and decrease the number of replicas (via the deployment) to maintain an average CPU utilization across all Pods of 80%.</span></span>

```kubectl
kubectl autoscale deployment ratings-web --cpu-percent=80 --min=1 --max=10
```
<span data-ttu-id="2c591-232">Per verificare lo stato corrente di HPA, è possibile eseguire:</span><span class="sxs-lookup"><span data-stu-id="2c591-232">You may check the current status of autoscaler by running:</span></span>

```kubectl
kubectl get hpa
```
```
NAME         REFERENCE                     TARGET    MINPODS   MAXPODS   REPLICAS   AGE
ratings-web   Deployment/ratings-web/scale   0% / 80%  1         10        1          18s
```
## <a name="configure-traffic-manager"></a><span data-ttu-id="2c591-233">Configurare Gestione traffico</span><span class="sxs-lookup"><span data-stu-id="2c591-233">Configure Traffic Manager</span></span>

<span data-ttu-id="2c591-234">Per distribuire il traffico tra due o più distribuzioni dell'applicazione, si userà [Gestione traffico di Azure](/azure/traffic-manager/traffic-manager-overview).</span><span class="sxs-lookup"><span data-stu-id="2c591-234">To distribute traffic between two (or more) deployments of the application, we'll use [Azure Traffic Manager](/azure/traffic-manager/traffic-manager-overview).</span></span> <span data-ttu-id="2c591-235">Gestione traffico di Azure è un servizio di bilanciamento del carico basato su DNS disponibile in Azure.</span><span class="sxs-lookup"><span data-stu-id="2c591-235">Azure Traffic Manager is a DNS-based traffic load balancer in Azure.</span></span>

> [!NOTE]
> <span data-ttu-id="2c591-236">Gestione traffico usa DNS per indirizzare le richieste client all'endpoint di servizio più appropriato, in base a un metodo di routing del traffico e all'integrità degli endpoint.</span><span class="sxs-lookup"><span data-stu-id="2c591-236">Traffic Manager uses DNS to direct client requests to the most appropriate service endpoint, based on a traffic-routing method and the health of the endpoints.</span></span>

<span data-ttu-id="2c591-237">Invece di Gestione traffico di Azure, è anche possibile usare altre soluzioni globali di bilanciamento del carico ospitate in locale.</span><span class="sxs-lookup"><span data-stu-id="2c591-237">Instead of using Azure Traffic Manager you can also use other global load-balancing solutions hosted on-premises.</span></span> <span data-ttu-id="2c591-238">Nello scenario di esempio si userà Gestione traffico di Azure per distribuire il traffico tra due istanze dell'applicazione.</span><span class="sxs-lookup"><span data-stu-id="2c591-238">In the sample scenario, we'll use Azure Traffic Manager to distribute traffic between two instances of our application.</span></span> <span data-ttu-id="2c591-239">Per l'esecuzione è possibile scegliere istanze dell'hub di Azure Stack nella stessa posizione o in posizioni diverse:</span><span class="sxs-lookup"><span data-stu-id="2c591-239">They can run on Azure Stack Hub instances in the same or different locations:</span></span>

![Gestione traffico in locale](media/solution-deployment-guide-highly-available-kubernetes/aks-azure-traffic-manager-on-premises.png)

<span data-ttu-id="2c591-241">In Azure configurare Gestione traffico in modo che punti alle due istanze diverse dell'applicazione:</span><span class="sxs-lookup"><span data-stu-id="2c591-241">In Azure, we configure Traffic Manager to point to the two different instances of our application:</span></span>

<span data-ttu-id="2c591-242">[![Profilo dell'endpoint di Gestione traffico](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png)](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png#lightbox)</span><span class="sxs-lookup"><span data-stu-id="2c591-242">[![TM endpoint profile](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png)](media/solution-deployment-guide-highly-available-kubernetes/traffic-manager-endpoint-profile-1.png#lightbox)</span></span>

<span data-ttu-id="2c591-243">Come si può notare, i due endpoint puntano alle due istanze dell'applicazione distribuita nella [sezione precedente](#deploy-the-application).</span><span class="sxs-lookup"><span data-stu-id="2c591-243">As you can see, the two endpoints point to the two instances of the deployed application from the [previous section](#deploy-the-application).</span></span>

<span data-ttu-id="2c591-244">A questo punto:</span><span class="sxs-lookup"><span data-stu-id="2c591-244">At this point:</span></span>
- <span data-ttu-id="2c591-245">L'infrastruttura di Kubernetes è stata creata, incluso un controller in ingresso.</span><span class="sxs-lookup"><span data-stu-id="2c591-245">The Kubernetes infrastructure has been created, including an Ingress Controller.</span></span>
- <span data-ttu-id="2c591-246">I cluster sono stati distribuiti tra due istanze dell'hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="2c591-246">Clusters have been deployed across two Azure Stack Hub instances.</span></span>
- <span data-ttu-id="2c591-247">Il monitoraggio è stato configurato.</span><span class="sxs-lookup"><span data-stu-id="2c591-247">Monitoring has been configured.</span></span>
- <span data-ttu-id="2c591-248">Gestione traffico di Azure bilancerà il traffico tra le due istanze dell'hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="2c591-248">Azure Traffic Manager will load balance traffic across the two Azure Stack Hub instances.</span></span>
- <span data-ttu-id="2c591-249">In questa infrastruttura l'applicazione di esempio a tre livelli è stata distribuita in modo automatizzato tramite grafici Helm.</span><span class="sxs-lookup"><span data-stu-id="2c591-249">On top of this infrastructure, the sample three-tier application has been deployed in an automated way using Helm Charts.</span></span> 

<span data-ttu-id="2c591-250">La soluzione dovrebbe ora essere attiva e accessibile agli utenti.</span><span class="sxs-lookup"><span data-stu-id="2c591-250">The solution should now be up and accessible to users!</span></span>

<span data-ttu-id="2c591-251">Esistono anche alcuni aspetti operativi post-distribuzione da considerare, che verranno descritti nelle due sezioni successive.</span><span class="sxs-lookup"><span data-stu-id="2c591-251">There are also some post-deployment operational considerations worth discussing, which are covered in the next two sections.</span></span>

## <a name="upgrade-kubernetes"></a><span data-ttu-id="2c591-252">Aggiornare Kubernetes</span><span class="sxs-lookup"><span data-stu-id="2c591-252">Upgrade Kubernetes</span></span>

<span data-ttu-id="2c591-253">Considerare gli argomenti seguenti per l'aggiornamento del cluster Kubernetes:</span><span class="sxs-lookup"><span data-stu-id="2c591-253">Consider the following topics when upgrading the Kubernetes cluster:</span></span>

- <span data-ttu-id="2c591-254">L'aggiornamento di un cluster Kubernetes è una procedura di manutenzione complessa che è possibile eseguire con il motore del servizio Azure Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="2c591-254">Upgrading a Kubernetes cluster is a complex Day 2 operation that can be done using AKS Engine.</span></span> <span data-ttu-id="2c591-255">Per altre informazioni, vedere [Aggiornare un cluster Kubernetes nell'hub di Azure Stack](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade).</span><span class="sxs-lookup"><span data-stu-id="2c591-255">For more information, see [Upgrade a Kubernetes cluster on Azure Stack Hub](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade).</span></span>
- <span data-ttu-id="2c591-256">Il motore del servizio Azure Kubernetes consente di aggiornare i cluster alle nuove versioni di Kubernetes e dell'immagine del sistema operativo di base.</span><span class="sxs-lookup"><span data-stu-id="2c591-256">AKS Engine allows you to upgrade clusters to newer Kubernetes and base OS image versions.</span></span> <span data-ttu-id="2c591-257">Per altre informazioni, vedere [Procedura per l'aggiornamento a una nuova versione di Kubernetes](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-upgrade-to-a-newer-kubernetes-version).</span><span class="sxs-lookup"><span data-stu-id="2c591-257">For more information, see [Steps to upgrade to a newer Kubernetes version](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-upgrade-to-a-newer-kubernetes-version).</span></span> 
- <span data-ttu-id="2c591-258">È anche possibile aggiornare solo i nodi sottostanti alle nuove versioni dell'immagine del sistema operativo di base.</span><span class="sxs-lookup"><span data-stu-id="2c591-258">You can also upgrade only the underlaying nodes to newer base OS image versions.</span></span> <span data-ttu-id="2c591-259">Per altre informazioni, vedere [Procedura per aggiornare solo l'immagine del sistema operativo di base](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-only-upgrade-the-os-image).</span><span class="sxs-lookup"><span data-stu-id="2c591-259">For more information, see [Steps to only upgrade the OS image](/azure-stack/user/azure-stack-kubernetes-aks-engine-upgrade#steps-to-only-upgrade-the-os-image).</span></span>

<span data-ttu-id="2c591-260">Le nuove immagini del sistema operativo di base contengono aggiornamenti di sicurezza e del kernel.</span><span class="sxs-lookup"><span data-stu-id="2c591-260">Newer base OS images contain security and kernel updates.</span></span> <span data-ttu-id="2c591-261">È responsabilità dell'operatore del cluster monitorare la disponibilità di nuove versioni di Kubernetes e delle immagini del sistema operativo.</span><span class="sxs-lookup"><span data-stu-id="2c591-261">It's the cluster operator's responsibility to monitor the availability of newer Kubernetes Versions and OS Images.</span></span> <span data-ttu-id="2c591-262">L'operatore dovrà pianificare ed eseguire questi aggiornamenti usando il motore del servizio Azure Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="2c591-262">The operator should plan and execute these upgrades using AKS Engine.</span></span> <span data-ttu-id="2c591-263">Le immagini del sistema operativo di base devono essere scaricate dal marketplace dell'hub di Azure Stack dall'operatore dell'hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="2c591-263">The base OS images must be downloaded from the Azure Stack Hub Marketplace by the Azure Stack Hub Operator.</span></span>

## <a name="scale-kubernetes"></a><span data-ttu-id="2c591-264">Dimensionare Kubernetes</span><span class="sxs-lookup"><span data-stu-id="2c591-264">Scale Kubernetes</span></span>

<span data-ttu-id="2c591-265">Il dimensionamento è un'altra operazione di manutenzione che può essere orchestrata con il motore del servizio Azure Kubernetes.</span><span class="sxs-lookup"><span data-stu-id="2c591-265">Scale is another Day 2 operation that can be orchestrated using AKS Engine.</span></span>

<span data-ttu-id="2c591-266">Il comando scale riutilizza il file di configurazione del cluster (apimodel.json) disponibile nella directory di output come input per una nuova distribuzione di Azure Resource Manager.</span><span class="sxs-lookup"><span data-stu-id="2c591-266">The scale command reuses your cluster configuration file (apimodel.json) in the output directory, as input for a new Azure Resource Manager deployment.</span></span> <span data-ttu-id="2c591-267">Il motore del servizio Azure Kubernetes esegue l'operazione di dimensionamento con uno specifico pool di agenti.</span><span class="sxs-lookup"><span data-stu-id="2c591-267">AKS Engine executes the scale operation against a specific agent pool.</span></span> <span data-ttu-id="2c591-268">Al termine dell'operazione, il motore del servizio Azure Kubernetes aggiorna la definizione del cluster nello stesso file apimodel.json.</span><span class="sxs-lookup"><span data-stu-id="2c591-268">When the scale operation is complete, AKS Engine updates the cluster definition in that same apimodel.json file.</span></span> <span data-ttu-id="2c591-269">La definizione del cluster riflette il nuovo numero di nodi in base alla configurazione del cluster corrente aggiornata.</span><span class="sxs-lookup"><span data-stu-id="2c591-269">The cluster definition reflects the new node count in order to reflect the updated, current cluster configuration.</span></span>

- [<span data-ttu-id="2c591-270">Dimensionare un cluster Kubernetes nell'hub di Azure Stack</span><span class="sxs-lookup"><span data-stu-id="2c591-270">Scale a Kubernetes cluster on Azure Stack Hub</span></span>](/azure-stack/user/azure-stack-kubernetes-aks-engine-scale)

## <a name="next-steps"></a><span data-ttu-id="2c591-271">Passaggi successivi</span><span class="sxs-lookup"><span data-stu-id="2c591-271">Next steps</span></span>

- <span data-ttu-id="2c591-272">Altre informazioni sulle [considerazioni per la progettazione di app ibride](overview-app-design-considerations.md)</span><span class="sxs-lookup"><span data-stu-id="2c591-272">Learn more about [Hybrid app design considerations](overview-app-design-considerations.md)</span></span>
- <span data-ttu-id="2c591-273">Esaminare e proporre miglioramenti del [codice di questo esempio in GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub).</span><span class="sxs-lookup"><span data-stu-id="2c591-273">Review and propose improvements to [the code for this sample on GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns/tree/master/AKSe-on-AzStackHub).</span></span>