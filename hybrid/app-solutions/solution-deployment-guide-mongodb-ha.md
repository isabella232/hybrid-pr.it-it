---
title: Distribuire una soluzione MongoDB a disponibilità elevata in Azure e in hub Azure Stack
description: Informazioni su come distribuire una soluzione MongoDB a disponibilità elevata in Azure e in hub Azure Stack
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: b34ba7c10ff5f658d645923ae8b6de2fb2607ccb
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911451"
---
# <a name="deploy-a-highly-available-mongodb-solution-to-azure-and-azure-stack-hub"></a><span data-ttu-id="ea425-103">Distribuire una soluzione MongoDB a disponibilità elevata in Azure e in hub Azure Stack</span><span class="sxs-lookup"><span data-stu-id="ea425-103">Deploy a highly available MongoDB solution to Azure and Azure Stack Hub</span></span>

<span data-ttu-id="ea425-104">Questo articolo illustra in modo dettagliato la distribuzione automatica di un cluster MongoDB di base a disponibilità elevata con un sito di ripristino di emergenza in due ambienti Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="ea425-104">This article will step you through an automated deployment of a basic highly available (HA) MongoDB cluster with a disaster recovery (DR) site across two Azure Stack Hub environments.</span></span> <span data-ttu-id="ea425-105">Per altre informazioni su MongoDB e sulla disponibilità elevata, vedere [membri del set di repliche](https://docs.mongodb.com/manual/core/replica-set-members/).</span><span class="sxs-lookup"><span data-stu-id="ea425-105">To learn more about MongoDB and high availability, see [Replica Set Members](https://docs.mongodb.com/manual/core/replica-set-members/).</span></span>

<span data-ttu-id="ea425-106">In questa soluzione verrà creato un ambiente di esempio per:</span><span class="sxs-lookup"><span data-stu-id="ea425-106">In this solution, you'll create a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="ea425-107">Orchestrare una distribuzione tra due hub Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="ea425-107">Orchestrate a deployment across two Azure Stack Hubs.</span></span>
> - <span data-ttu-id="ea425-108">Usare Docker per ridurre al minimo i problemi di dipendenza con i profili API di Azure.</span><span class="sxs-lookup"><span data-stu-id="ea425-108">Use Docker to minimize dependency issues with Azure API profiles.</span></span>
> - <span data-ttu-id="ea425-109">Distribuire un cluster MongoDB di base a disponibilità elevata con un sito di ripristino di emergenza.</span><span class="sxs-lookup"><span data-stu-id="ea425-109">Deploy a basic highly available MongoDB cluster with a disaster recovery site.</span></span>

> [!Tip]  
> <span data-ttu-id="ea425-110">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="ea425-110">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="ea425-111">Microsoft Azure Stack Hub è un'estensione di Azure.</span><span class="sxs-lookup"><span data-stu-id="ea425-111">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="ea425-112">Azure Stack Hub offre l'agilità e l'innovazione di cloud computing all'ambiente locale, abilitando l'unico Cloud ibrido che consente di creare e distribuire app ibride ovunque.</span><span class="sxs-lookup"><span data-stu-id="ea425-112">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="ea425-113">L'articolo [considerazioni sulla progettazione di app ibride](overview-app-design-considerations.md) esamina i pilastri della qualità del software (posizionamento, scalabilità, disponibilità, resilienza, gestibilità e sicurezza) per la progettazione, la distribuzione e la gestione di app ibride.</span><span class="sxs-lookup"><span data-stu-id="ea425-113">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="ea425-114">Le considerazioni di progettazione consentono di ottimizzare la progettazione delle app ibride, riducendo al minimo le esigenze negli ambienti di produzione.</span><span class="sxs-lookup"><span data-stu-id="ea425-114">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="architecture-for-mongodb-with-azure-stack-hub"></a><span data-ttu-id="ea425-115">Architettura per MongoDB con hub Azure Stack</span><span class="sxs-lookup"><span data-stu-id="ea425-115">Architecture for MongoDB with Azure Stack Hub</span></span>

![architettura MongoDB a disponibilità elevata nell'hub Azure Stack](media/solution-deployment-guide-mongodb-ha/image1.png)

## <a name="prerequisites-for-mongodb-with-azure-stack-hub"></a><span data-ttu-id="ea425-117">Prerequisiti per MongoDB con hub Azure Stack</span><span class="sxs-lookup"><span data-stu-id="ea425-117">Prerequisites for MongoDB with Azure Stack Hub</span></span>

- <span data-ttu-id="ea425-118">Due sistemi integrati di Azure Stack hub collegati (hub Azure Stack).</span><span class="sxs-lookup"><span data-stu-id="ea425-118">Two connected Azure Stack Hub integrated systems (Azure Stack Hub).</span></span> <span data-ttu-id="ea425-119">Questa distribuzione non funziona nel Azure Stack Development Kit (Gabriele).</span><span class="sxs-lookup"><span data-stu-id="ea425-119">This deployment doesn't work on the Azure Stack Development Kit (ASDK).</span></span> <span data-ttu-id="ea425-120">Per altre informazioni sull'hub Azure Stack, vedere [che cos'è Azure stack Hub?](https://azure.microsoft.com/products/azure-stack/hub/)</span><span class="sxs-lookup"><span data-stu-id="ea425-120">To learn more about Azure Stack Hub, see [What is Azure Stack Hub?](https://azure.microsoft.com/products/azure-stack/hub/)</span></span>
  - <span data-ttu-id="ea425-121">Una sottoscrizione tenant in ogni hub Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="ea425-121">A tenant subscription on each Azure Stack Hub.</span></span> 
  - <span data-ttu-id="ea425-122">**Prendere nota di ogni ID sottoscrizione e dell'endpoint Azure Resource Manager per ogni hub Azure Stack.**</span><span class="sxs-lookup"><span data-stu-id="ea425-122">**Make a note of each subscription ID and the Azure Resource Manager endpoint for each Azure Stack Hub.**</span></span>
- <span data-ttu-id="ea425-123">Un'entità servizio Azure Active Directory (Azure AD) che dispone delle autorizzazioni per la sottoscrizione tenant in ogni hub Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="ea425-123">An Azure Active Directory (Azure AD) service principal that has permissions to the tenant subscription on each Azure Stack Hub.</span></span> <span data-ttu-id="ea425-124">Potrebbe essere necessario creare due entità servizio se gli hub Azure Stack vengono distribuiti in tenant Azure AD diversi.</span><span class="sxs-lookup"><span data-stu-id="ea425-124">You may need to create two service principals if the Azure Stack Hubs are deployed against different Azure AD tenants.</span></span> <span data-ttu-id="ea425-125">Per informazioni su come creare un'entità servizio per Azure Stack Hub, vedere [usare un'identità dell'app per accedere alle risorse dell'hub Azure stack](https://docs.microsoft.com/azure-stack/user/azure-stack-create-service-principals).</span><span class="sxs-lookup"><span data-stu-id="ea425-125">To learn how to create a service principal for Azure Stack Hub, see [Use an app identity to access Azure Stack Hub resources](https://docs.microsoft.com/azure-stack/user/azure-stack-create-service-principals).</span></span>
  - <span data-ttu-id="ea425-126">**Prendere nota dell'ID applicazione di ogni entità servizio, del segreto client e del nome del tenant (xxxxx.onmicrosoft.com).**</span><span class="sxs-lookup"><span data-stu-id="ea425-126">**Make a note of each service principal's application ID, client secret, and tenant name (xxxxx.onmicrosoft.com).**</span></span>
- <span data-ttu-id="ea425-127">Ubuntu 16,04 è stato diffuso a ogni Marketplace dell'hub Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="ea425-127">Ubuntu 16.04 syndicated to each Azure Stack Hub's Marketplace.</span></span> <span data-ttu-id="ea425-128">Per altre informazioni sulla diffusione del Marketplace, vedere [scaricare gli elementi del Marketplace nell'Hub Azure stack](https://docs.microsoft.com/azure-stack/operator/azure-stack-download-azure-marketplace-item).</span><span class="sxs-lookup"><span data-stu-id="ea425-128">To learn more about marketplace syndication, see [Download Marketplace items to Azure Stack Hub](https://docs.microsoft.com/azure-stack/operator/azure-stack-download-azure-marketplace-item).</span></span>
- <span data-ttu-id="ea425-129">[Docker per Windows](https://docs.docker.com/docker-for-windows/) installato nel computer locale.</span><span class="sxs-lookup"><span data-stu-id="ea425-129">[Docker for Windows](https://docs.docker.com/docker-for-windows/) installed on your local machine.</span></span>

## <a name="get-the-docker-image"></a><span data-ttu-id="ea425-130">Ottenere l'immagine Docker</span><span class="sxs-lookup"><span data-stu-id="ea425-130">Get the Docker image</span></span>

<span data-ttu-id="ea425-131">Le immagini Docker per ogni distribuzione eliminano i problemi di dipendenza tra versioni diverse di Azure PowerShell.</span><span class="sxs-lookup"><span data-stu-id="ea425-131">Docker images for each deployment eliminate dependency issues between different versions of Azure PowerShell.</span></span>

1. <span data-ttu-id="ea425-132">Assicurarsi che Docker per Windows usi i contenitori di Windows.</span><span class="sxs-lookup"><span data-stu-id="ea425-132">Make sure that Docker for Windows is using Windows containers.</span></span>
2. <span data-ttu-id="ea425-133">Eseguire il comando seguente in un prompt dei comandi con privilegi elevati per ottenere il contenitore Docker con gli script di distribuzione.</span><span class="sxs-lookup"><span data-stu-id="ea425-133">Run the following command in an elevated command prompt to get the Docker container with the deployment scripts.</span></span>

    ```powershell  
    docker pull intelligentedge/mongodb-hadr:1.0.0
    ```

## <a name="deploy-the-clusters"></a><span data-ttu-id="ea425-134">Distribuire i cluster</span><span class="sxs-lookup"><span data-stu-id="ea425-134">Deploy the clusters</span></span>

1. <span data-ttu-id="ea425-135">Una volta eseguito il pull dell'immagine del contenitore, avviare l'immagine.</span><span class="sxs-lookup"><span data-stu-id="ea425-135">Once the container image has been successfully pulled, start the image.</span></span>

    ```powershell  
    docker run -it intelligentedge/mongodb-hadr:1.0.0 powershell
    ```

2. <span data-ttu-id="ea425-136">Una volta avviato il contenitore, verrà assegnato un terminale di PowerShell con privilegi elevati nel contenitore.</span><span class="sxs-lookup"><span data-stu-id="ea425-136">Once the container has started, you'll be given an elevated PowerShell terminal in the container.</span></span> <span data-ttu-id="ea425-137">Modificare le directory per ottenere lo script di distribuzione.</span><span class="sxs-lookup"><span data-stu-id="ea425-137">Change directories to get to the deployment script.</span></span>

    ```powershell  
    cd .\MongoHADRDemo\
    ```

3. <span data-ttu-id="ea425-138">Eseguire la distribuzione.</span><span class="sxs-lookup"><span data-stu-id="ea425-138">Run the deployment.</span></span> <span data-ttu-id="ea425-139">Fornire le credenziali e i nomi delle risorse laddove necessario.</span><span class="sxs-lookup"><span data-stu-id="ea425-139">Provide credentials and resource names where needed.</span></span> <span data-ttu-id="ea425-140">HA si riferisce all'hub Azure Stack in cui verrà distribuito il cluster a disponibilità elevata.</span><span class="sxs-lookup"><span data-stu-id="ea425-140">HA refers to the Azure Stack Hub where the HA cluster will be deployed.</span></span> <span data-ttu-id="ea425-141">Il ripristino di emergenza fa riferimento all'hub Azure Stack in cui verrà distribuito il cluster di ripristino di emergenza.</span><span class="sxs-lookup"><span data-stu-id="ea425-141">DR refers to the Azure Stack Hub where the DR cluster will be deployed.</span></span>

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

4. <span data-ttu-id="ea425-142">Digitare `Y` per consentire l'installazione del provider NuGet, che consente di avviare i moduli del profilo API "2018-03-01-Hybrid" da installare.</span><span class="sxs-lookup"><span data-stu-id="ea425-142">Type `Y` to allow the NuGet provider to be installed, which will kick off the API Profile "2018-03-01-hybrid" modules to be installed.</span></span>

5. <span data-ttu-id="ea425-143">Le risorse a disponibilità elevata vengono distribuite per prime.</span><span class="sxs-lookup"><span data-stu-id="ea425-143">The HA resources will deploy first.</span></span> <span data-ttu-id="ea425-144">Monitorare la distribuzione e attenderne il completamento.</span><span class="sxs-lookup"><span data-stu-id="ea425-144">Monitor the deployment and wait for it to finish.</span></span> <span data-ttu-id="ea425-145">Una volta che il messaggio indica che la distribuzione a disponibilità elevata è stata completata, è possibile controllare il portale dell'hub Azure Stack HA per visualizzare le risorse distribuite.</span><span class="sxs-lookup"><span data-stu-id="ea425-145">Once you have the message stating that the HA deployment is finished, you can check the HA Azure Stack Hub's portal to see the resources deployed.</span></span>

6. <span data-ttu-id="ea425-146">Continuare con la distribuzione delle risorse di ripristino di emergenza e decidere se si vuole abilitare una Jump box nell'hub Azure Stack di ripristino di emergenza per interagire con il cluster.</span><span class="sxs-lookup"><span data-stu-id="ea425-146">Continue with the deployment of DR resources and decide if you'd like to enable a jump box on the DR Azure Stack Hub to interact with the cluster.</span></span>

7. <span data-ttu-id="ea425-147">Attendere il completamento della distribuzione delle risorse di ripristino di emergenza.</span><span class="sxs-lookup"><span data-stu-id="ea425-147">Wait for DR resource deployment to finish.</span></span>

8. <span data-ttu-id="ea425-148">Al termine della distribuzione delle risorse di ripristino di emergenza, uscire dal contenitore.</span><span class="sxs-lookup"><span data-stu-id="ea425-148">Once DR resource deployment has finished, exit the container.</span></span>

  ```powershell
  exit
  ```

## <a name="next-steps"></a><span data-ttu-id="ea425-149">Passaggi successivi</span><span class="sxs-lookup"><span data-stu-id="ea425-149">Next steps</span></span>

- <span data-ttu-id="ea425-150">Se è stata abilitata la macchina virtuale jump box nell'hub Azure Stack di ripristino di emergenza, è possibile connettersi tramite SSH e interagire con il cluster MongoDB installando l'interfaccia della riga di comando di Mongo.</span><span class="sxs-lookup"><span data-stu-id="ea425-150">If you enabled the jump box VM on the DR Azure Stack Hub, you can connect via SSH and interact with the MongoDB cluster by installing the mongo CLI.</span></span> <span data-ttu-id="ea425-151">Per ulteriori informazioni sull'interazione con MongoDB, vedere [la shell di Mongo](https://docs.mongodb.com/manual/mongo/).</span><span class="sxs-lookup"><span data-stu-id="ea425-151">To learn more about interacting with MongoDB, see [The mongo Shell](https://docs.mongodb.com/manual/mongo/).</span></span>
- <span data-ttu-id="ea425-152">Per altre informazioni sulle app cloud ibride, vedere [soluzioni cloud ibride.](https://aka.ms/azsdevtutorials)</span><span class="sxs-lookup"><span data-stu-id="ea425-152">To learn more about hybrid cloud apps, see [Hybrid Cloud Solutions.](https://aka.ms/azsdevtutorials)</span></span>
- <span data-ttu-id="ea425-153">Modificare il codice in questo esempio su [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).</span><span class="sxs-lookup"><span data-stu-id="ea425-153">Modify the code to this sample on [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).</span></span>
