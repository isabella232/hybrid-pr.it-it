---
title: Distribuire una soluzione MongoDB a disponibilità elevata in Azure e nell'hub di Azure Stack
description: Informazioni su come distribuire una soluzione MongoDB a disponibilità elevata in Azure e nell'hub di Azure Stack
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: def9abaa2a7231648f11453f66119399be015a4d
ms.sourcegitcommit: 485a1f97fa1579364e2be1755cadfc5ea89db50e
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 10/08/2020
ms.locfileid: "91852508"
---
# <a name="deploy-a-highly-available-mongodb-solution-to-azure-and-azure-stack-hub"></a><span data-ttu-id="2557e-103">Distribuire una soluzione MongoDB a disponibilità elevata in Azure e nell'hub di Azure Stack</span><span class="sxs-lookup"><span data-stu-id="2557e-103">Deploy a highly available MongoDB solution to Azure and Azure Stack Hub</span></span>

<span data-ttu-id="2557e-104">Questo articolo illustra una distribuzione automatizzata di un cluster MongoDB a disponibilità elevata di base con un sito di ripristino di emergenza tra due ambienti dell'hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="2557e-104">This article will step you through an automated deployment of a basic highly available (HA) MongoDB cluster with a disaster recovery (DR) site across two Azure Stack Hub environments.</span></span> <span data-ttu-id="2557e-105">Per altre informazioni su MongoDB e sulla disponibilità elevata, vedere [Membri del set di repliche](https://docs.mongodb.com/manual/core/replica-set-members/).</span><span class="sxs-lookup"><span data-stu-id="2557e-105">To learn more about MongoDB and high availability, see [Replica Set Members](https://docs.mongodb.com/manual/core/replica-set-members/).</span></span>

<span data-ttu-id="2557e-106">In questa soluzione verrà creato un ambiente di esempio per:</span><span class="sxs-lookup"><span data-stu-id="2557e-106">In this solution, you'll create a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="2557e-107">Orchestrare una distribuzione tra due hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="2557e-107">Orchestrate a deployment across two Azure Stack Hubs.</span></span>
> - <span data-ttu-id="2557e-108">Usare Docker per ridurre al minimo i problemi di dipendenza con i profili API di Azure.</span><span class="sxs-lookup"><span data-stu-id="2557e-108">Use Docker to minimize dependency issues with Azure API profiles.</span></span>
> - <span data-ttu-id="2557e-109">Distribuire un cluster MongoDB a disponibilità elevata di base con un sito di ripristino di emergenza.</span><span class="sxs-lookup"><span data-stu-id="2557e-109">Deploy a basic highly available MongoDB cluster with a disaster recovery site.</span></span>

> [!Tip]  
> <span data-ttu-id="2557e-110">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="2557e-110">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="2557e-111">L'hub di Microsoft Azure Stack è un'estensione di Azure</span><span class="sxs-lookup"><span data-stu-id="2557e-111">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="2557e-112">che offre all'ambiente locale l'agilità e l'innovazione del cloud computing, abilitando l'unico cloud ibrido che consente di creare e distribuire ovunque app ibride.</span><span class="sxs-lookup"><span data-stu-id="2557e-112">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="2557e-113">L'articolo [Considerazioni per la progettazione di app ibride](overview-app-design-considerations.md) esamina i concetti fondamentali per la qualità del software (posizionamento, scalabilità, disponibilità, resilienza, gestibilità e sicurezza) per la progettazione, la distribuzione e la gestione di app ibride.</span><span class="sxs-lookup"><span data-stu-id="2557e-113">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="2557e-114">Le considerazioni per la progettazione consentono di ottimizzare la progettazione delle app ibride, riducendo al minimo i rischi negli ambienti di produzione.</span><span class="sxs-lookup"><span data-stu-id="2557e-114">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="architecture-for-mongodb-with-azure-stack-hub"></a><span data-ttu-id="2557e-115">Architettura per MongoDB con l'hub di Azure Stack</span><span class="sxs-lookup"><span data-stu-id="2557e-115">Architecture for MongoDB with Azure Stack Hub</span></span>

![architettura MongoDB a disponibilità elevata nell'hub di Azure Stack](media/solution-deployment-guide-mongodb-ha/image1.png)

## <a name="prerequisites-for-mongodb-with-azure-stack-hub"></a><span data-ttu-id="2557e-117">Prerequisiti per MongoDB con l'hub di Azure Stack</span><span class="sxs-lookup"><span data-stu-id="2557e-117">Prerequisites for MongoDB with Azure Stack Hub</span></span>

- <span data-ttu-id="2557e-118">Due sistemi integrati dell'hub di Azure Stack connessi (hub di Azure Stack).</span><span class="sxs-lookup"><span data-stu-id="2557e-118">Two connected Azure Stack Hub integrated systems (Azure Stack Hub).</span></span> <span data-ttu-id="2557e-119">Questa distribuzione non funziona nell'Azure Stack Development Kit (ASDK).</span><span class="sxs-lookup"><span data-stu-id="2557e-119">This deployment doesn't work on the Azure Stack Development Kit (ASDK).</span></span> <span data-ttu-id="2557e-120">Per altre informazioni sull'hub di Azure Stack, vedere [Cos'è l'hub di Azure Stack?](https://azure.microsoft.com/products/azure-stack/hub/)</span><span class="sxs-lookup"><span data-stu-id="2557e-120">To learn more about Azure Stack Hub, see [What is Azure Stack Hub?](https://azure.microsoft.com/products/azure-stack/hub/)</span></span>
  - <span data-ttu-id="2557e-121">Una sottoscrizione tenant in ogni hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="2557e-121">A tenant subscription on each Azure Stack Hub.</span></span> 
  - <span data-ttu-id="2557e-122">**Prendere nota di ogni ID sottoscrizione e dell'endpoint Azure Resource Manager per ogni hub di Azure Stack.**</span><span class="sxs-lookup"><span data-stu-id="2557e-122">**Make a note of each subscription ID and the Azure Resource Manager endpoint for each Azure Stack Hub.**</span></span>
- <span data-ttu-id="2557e-123">Un'entità servizio Azure Active Directory (Azure AD) in possesso delle autorizzazioni per la sottoscrizione tenant in ogni hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="2557e-123">An Azure Active Directory (Azure AD) service principal that has permissions to the tenant subscription on each Azure Stack Hub.</span></span> <span data-ttu-id="2557e-124">Potrebbe essere necessario creare due entità servizio se gli hub di Azure Stack vengono distribuiti in tenant di Azure AD diversi.</span><span class="sxs-lookup"><span data-stu-id="2557e-124">You may need to create two service principals if the Azure Stack Hubs are deployed against different Azure AD tenants.</span></span> <span data-ttu-id="2557e-125">Per informazioni su come creare un'entità servizio per l'hub di Azure Stack, vedere [Usare un'identità dell'app per accedere alle risorse dell'hub di Azure Stack](/azure-stack/user/azure-stack-create-service-principals).</span><span class="sxs-lookup"><span data-stu-id="2557e-125">To learn how to create a service principal for Azure Stack Hub, see [Use an app identity to access Azure Stack Hub resources](/azure-stack/user/azure-stack-create-service-principals).</span></span>
  - <span data-ttu-id="2557e-126">**Prendere nota di ID applicazione, segreto client e nome tenant di ogni entità servizio (xxxxx.onmicrosoft.com).**</span><span class="sxs-lookup"><span data-stu-id="2557e-126">**Make a note of each service principal's application ID, client secret, and tenant name (xxxxx.onmicrosoft.com).**</span></span>
- <span data-ttu-id="2557e-127">Ubuntu 16.04 è stato diffuso a ogni marketplace dell'hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="2557e-127">Ubuntu 16.04 syndicated to each Azure Stack Hub's Marketplace.</span></span> <span data-ttu-id="2557e-128">Per altre informazioni sulla diffusione del marketplace, vedere [Scaricare gli elementi del marketplace nell'hub di Azure Stack](/azure-stack/operator/azure-stack-download-azure-marketplace-item).</span><span class="sxs-lookup"><span data-stu-id="2557e-128">To learn more about marketplace syndication, see [Download Marketplace items to Azure Stack Hub](/azure-stack/operator/azure-stack-download-azure-marketplace-item).</span></span>
- <span data-ttu-id="2557e-129">[Docker per Windows](https://docs.docker.com/docker-for-windows/) installato nel computer locale.</span><span class="sxs-lookup"><span data-stu-id="2557e-129">[Docker for Windows](https://docs.docker.com/docker-for-windows/) installed on your local machine.</span></span>

## <a name="get-the-docker-image"></a><span data-ttu-id="2557e-130">Ottenere l'immagine Docker</span><span class="sxs-lookup"><span data-stu-id="2557e-130">Get the Docker image</span></span>

<span data-ttu-id="2557e-131">Le immagini Docker per ogni distribuzione eliminano i problemi di dipendenza tra versioni diverse di Azure PowerShell.</span><span class="sxs-lookup"><span data-stu-id="2557e-131">Docker images for each deployment eliminate dependency issues between different versions of Azure PowerShell.</span></span>

1. <span data-ttu-id="2557e-132">Verificare che Docker per Windows usi i contenitori di Windows.</span><span class="sxs-lookup"><span data-stu-id="2557e-132">Make sure that Docker for Windows is using Windows containers.</span></span>
2. <span data-ttu-id="2557e-133">Eseguire il comando seguente in un prompt dei comandi con privilegi elevati per ottenere il contenitore Docker con gli script di distribuzione.</span><span class="sxs-lookup"><span data-stu-id="2557e-133">Run the following command in an elevated command prompt to get the Docker container with the deployment scripts.</span></span>

    ```powershell  
    docker pull intelligentedge/mongodb-hadr:1.0.0
    ```

## <a name="deploy-the-clusters"></a><span data-ttu-id="2557e-134">Distribuire i cluster</span><span class="sxs-lookup"><span data-stu-id="2557e-134">Deploy the clusters</span></span>

1. <span data-ttu-id="2557e-135">Dopo aver eseguito correttamente il pull dell'immagine del contenitore, avviare l'immagine.</span><span class="sxs-lookup"><span data-stu-id="2557e-135">Once the container image has been successfully pulled, start the image.</span></span>

    ```powershell  
    docker run -it intelligentedge/mongodb-hadr:1.0.0 powershell
    ```

2. <span data-ttu-id="2557e-136">Avviato il contenitore, gli verrà assegnato un terminale di PowerShell con privilegi elevati.</span><span class="sxs-lookup"><span data-stu-id="2557e-136">Once the container has started, you'll be given an elevated PowerShell terminal in the container.</span></span> <span data-ttu-id="2557e-137">Modificare le directory per ottenere lo script di distribuzione.</span><span class="sxs-lookup"><span data-stu-id="2557e-137">Change directories to get to the deployment script.</span></span>

    ```powershell  
    cd .\MongoHADRDemo\
    ```

3. <span data-ttu-id="2557e-138">Eseguire la distribuzione.</span><span class="sxs-lookup"><span data-stu-id="2557e-138">Run the deployment.</span></span> <span data-ttu-id="2557e-139">Specificare le credenziali e i nomi di risorse, dove necessario.</span><span class="sxs-lookup"><span data-stu-id="2557e-139">Provide credentials and resource names where needed.</span></span> <span data-ttu-id="2557e-140">HA si riferisce all'hub di Azure Stack in cui verrà distribuito il cluster a disponibilità elevata.</span><span class="sxs-lookup"><span data-stu-id="2557e-140">HA refers to the Azure Stack Hub where the HA cluster will be deployed.</span></span> <span data-ttu-id="2557e-141">DR si riferisce all'hub di Azure Stack in cui verrà distribuito il cluster di ripristino di emergenza.</span><span class="sxs-lookup"><span data-stu-id="2557e-141">DR refers to the Azure Stack Hub where the DR cluster will be deployed.</span></span>

    ```powershell
    .\Deploy-AzureResourceGroup.ps1 `
    -AzureStackApplicationId_HA "applicationIDforHAServicePrincipal" `
    -AzureStackApplicationSercet_HA "clientSecretforHAServicePrincipal" `
    -AADTenantName_HA "hatenantname.onmicrosoft.com" `
    -AzureStackResourceGroup_HA "haresourcegroupname" `
    -AzureStackArmEndpoint_HA "https://management.haazurestack.com" `
    -AzureStackSubscriptionId_HA "haSubscriptionId" `
    -AzureStackApplicationId_DR "applicationIDforDRServicePrincipal" `
    -AzureStackApplicationSercet_DR "ClientSecretforDRServicePrincipal" `
    -AADTenantName_DR "drtenantname.onmicrosoft.com" `
    -AzureStackResourceGroup_DR "drresourcegroupname" `
    -AzureStackArmEndpoint_DR "https://management.drazurestack.com" `
    -AzureStackSubscriptionId_DR "drSubscriptionId"
    ```

4. <span data-ttu-id="2557e-142">Digitare `Y` per consentire l'installazione del provider NuGet, che avvierà i moduli del profilo di API "2018-03-01-hybrid" da installare.</span><span class="sxs-lookup"><span data-stu-id="2557e-142">Type `Y` to allow the NuGet provider to be installed, which will kick off the API Profile "2018-03-01-hybrid" modules to be installed.</span></span>

5. <span data-ttu-id="2557e-143">Le risorse a disponibilità elevata verranno distribuite per prime.</span><span class="sxs-lookup"><span data-stu-id="2557e-143">The HA resources will deploy first.</span></span> <span data-ttu-id="2557e-144">Monitorare la distribuzione e attenderne il completamento.</span><span class="sxs-lookup"><span data-stu-id="2557e-144">Monitor the deployment and wait for it to finish.</span></span> <span data-ttu-id="2557e-145">Dopo aver ricevuto il messaggio che indica il completamento della distribuzione a disponibilità elevata, è possibile controllare il portale dell'hub di Azure Stack a disponibilità elevata per visualizzare le risorse distribuite.</span><span class="sxs-lookup"><span data-stu-id="2557e-145">Once you have the message stating that the HA deployment is finished, you can check the HA Azure Stack Hub's portal to see the resources deployed.</span></span>

6. <span data-ttu-id="2557e-146">Continuare con la distribuzione delle risorse di ripristino di emergenza e decidere se si vuole abilitare un jumpbox nell'hub di Azure Stack di ripristino di emergenza per interagire con il cluster.</span><span class="sxs-lookup"><span data-stu-id="2557e-146">Continue with the deployment of DR resources and decide if you'd like to enable a jump box on the DR Azure Stack Hub to interact with the cluster.</span></span>

7. <span data-ttu-id="2557e-147">Attendere il completamento della distribuzione della risorsa di ripristino di emergenza.</span><span class="sxs-lookup"><span data-stu-id="2557e-147">Wait for DR resource deployment to finish.</span></span>

8. <span data-ttu-id="2557e-148">Al termine della distribuzione della risorsa di ripristino di emergenza, uscire dal contenitore.</span><span class="sxs-lookup"><span data-stu-id="2557e-148">Once DR resource deployment has finished, exit the container.</span></span>

  ```powershell
  exit
  ```

## <a name="next-steps"></a><span data-ttu-id="2557e-149">Passaggi successivi</span><span class="sxs-lookup"><span data-stu-id="2557e-149">Next steps</span></span>

- <span data-ttu-id="2557e-150">Se è stata abilitata la VM jumpbox nell'hub di Azure Stack di ripristino di emergenza, è possibile connettersi tramite SSH e interagire con il cluster di MongoDB installando l'interfaccia della riga di comando di Mongo.</span><span class="sxs-lookup"><span data-stu-id="2557e-150">If you enabled the jump box VM on the DR Azure Stack Hub, you can connect via SSH and interact with the MongoDB cluster by installing the mongo CLI.</span></span> <span data-ttu-id="2557e-151">Per altre informazioni sull'interazione con MongoDB, vedere [Mongo Shell](https://docs.mongodb.com/manual/mongo/).</span><span class="sxs-lookup"><span data-stu-id="2557e-151">To learn more about interacting with MongoDB, see [The mongo Shell](https://docs.mongodb.com/manual/mongo/).</span></span>
- <span data-ttu-id="2557e-152">Per altre informazioni sulle app per il cloud ibrido, vedere [Soluzioni per il cloud ibrido.](/azure-stack/user/)</span><span class="sxs-lookup"><span data-stu-id="2557e-152">To learn more about hybrid cloud apps, see [Hybrid Cloud Solutions.](/azure-stack/user/)</span></span>
- <span data-ttu-id="2557e-153">Modificare il codice in questo esempio in [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).</span><span class="sxs-lookup"><span data-stu-id="2557e-153">Modify the code to this sample on [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).</span></span>