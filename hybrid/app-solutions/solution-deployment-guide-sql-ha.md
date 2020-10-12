---
title: Distribuire un gruppo di disponibilità di SQL Server 2016 in Azure e nell'hub di Azure Stack
description: Informazioni su come distribuire un gruppo di disponibilità di SQL Server 2016 in Azure e nell'hub di Azure Stack.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 2c20d621247ec8e1278feb092586232cc08d5480
ms.sourcegitcommit: 485a1f97fa1579364e2be1755cadfc5ea89db50e
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 10/08/2020
ms.locfileid: "91852474"
---
# <a name="deploy-a-sql-server-2016-availability-group-to-azure-and-azure-stack-hub"></a><span data-ttu-id="331cf-103">Distribuire un gruppo di disponibilità di SQL Server 2016 in Azure e nell'hub di Azure Stack</span><span class="sxs-lookup"><span data-stu-id="331cf-103">Deploy a SQL Server 2016 availability group to Azure and Azure Stack Hub</span></span>

<span data-ttu-id="331cf-104">Questo articolo illustra una distribuzione automatizzata di un cluster di SQL Server 2016 Enterprise a disponibilità elevata di base con un sito di ripristino di emergenza asincrono tra due ambienti dell'hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="331cf-104">This article will step you through an automated deployment of a basic highly available (HA) SQL Server 2016 Enterprise cluster with an asynchronous disaster recovery (DR) site across two Azure Stack Hub environments.</span></span> <span data-ttu-id="331cf-105">Per altre informazioni su SQL Server 2016 e sulla disponibilità elevata, vedere [Gruppi di disponibilità Always On: una soluzione per la disponibilità elevata e il ripristino di emergenza](/sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server?view=sql-server-2016).</span><span class="sxs-lookup"><span data-stu-id="331cf-105">To learn more about SQL Server 2016 and high availability, see [Always On availability groups: a high-availability and disaster-recovery solution](/sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server?view=sql-server-2016).</span></span>

<span data-ttu-id="331cf-106">In questa soluzione si compilerà un ambiente di esempio per:</span><span class="sxs-lookup"><span data-stu-id="331cf-106">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="331cf-107">Orchestrare una distribuzione tra due hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="331cf-107">Orchestrate a deployment across two Azure Stack Hubs.</span></span>
> - <span data-ttu-id="331cf-108">Usare Docker per ridurre al minimo i problemi di dipendenza con i profili API di Azure.</span><span class="sxs-lookup"><span data-stu-id="331cf-108">Use Docker to minimize dependency issues with Azure API profiles.</span></span>
> - <span data-ttu-id="331cf-109">Distribuire un cluster di SQL Server 2016 Enterprise a disponibilità elevata di base con un sito di ripristino di emergenza.</span><span class="sxs-lookup"><span data-stu-id="331cf-109">Deploy a basic highly available SQL Server 2016 Enterprise cluster with a disaster recovery site.</span></span>

> [!Tip]  
> <span data-ttu-id="331cf-110">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="331cf-110">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="331cf-111">L'hub di Microsoft Azure Stack è un'estensione di Azure,</span><span class="sxs-lookup"><span data-stu-id="331cf-111">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="331cf-112">che offre all'ambiente locale l'agilità e l'innovazione del cloud computing, abilitando l'unico cloud ibrido che consente di creare e distribuire ovunque app ibride.</span><span class="sxs-lookup"><span data-stu-id="331cf-112">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="331cf-113">L'articolo [Considerazioni per la progettazione di app ibride](overview-app-design-considerations.md) esamina i concetti fondamentali per la qualità del software (posizionamento, scalabilità, disponibilità, resilienza, gestibilità e sicurezza) per la progettazione, la distribuzione e la gestione di app ibride.</span><span class="sxs-lookup"><span data-stu-id="331cf-113">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="331cf-114">Le considerazioni per la progettazione consentono di ottimizzare la progettazione delle app ibride, riducendo al minimo i rischi negli ambienti di produzione.</span><span class="sxs-lookup"><span data-stu-id="331cf-114">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="architecture-for-sql-server-2016"></a><span data-ttu-id="331cf-115">Architettura di SQL Server 2016</span><span class="sxs-lookup"><span data-stu-id="331cf-115">Architecture for SQL Server 2016</span></span>

![Disponibilità elevata di SQL Server 2016 nell'hub di Azure Stack](media/solution-deployment-guide-sql-ha/image1.png)

## <a name="prerequisites-for-sql-server-2016"></a><span data-ttu-id="331cf-117">Prerequisiti di SQL Server 2016</span><span class="sxs-lookup"><span data-stu-id="331cf-117">Prerequisites for SQL Server 2016</span></span>

- <span data-ttu-id="331cf-118">Due sistemi integrati dell'hub di Azure Stack connessi (hub di Azure Stack).</span><span class="sxs-lookup"><span data-stu-id="331cf-118">Two connected Azure Stack Hub integrated systems (Azure Stack Hub).</span></span> <span data-ttu-id="331cf-119">Questa distribuzione non funziona nell'Azure Stack Development Kit (ASDK).</span><span class="sxs-lookup"><span data-stu-id="331cf-119">This deployment doesn't work on the Azure Stack Development Kit (ASDK).</span></span> <span data-ttu-id="331cf-120">Per altre informazioni sull'hub di Azure Stack, vedere la [panoramica di Azure Stack](https://azure.microsoft.com/overview/azure-stack/).</span><span class="sxs-lookup"><span data-stu-id="331cf-120">To learn more about Azure Stack Hub, see the [Azure Stack overview](https://azure.microsoft.com/overview/azure-stack/).</span></span>
- <span data-ttu-id="331cf-121">Una sottoscrizione tenant in ogni hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="331cf-121">A tenant subscription on each Azure Stack Hub.</span></span>
  - <span data-ttu-id="331cf-122">**Prendere nota di ogni ID sottoscrizione e dell'endpoint Azure Resource Manager per ogni hub di Azure Stack.**</span><span class="sxs-lookup"><span data-stu-id="331cf-122">**Make a note of each subscription ID and the Azure Resource Manager endpoint for each Azure Stack Hub.**</span></span>
- <span data-ttu-id="331cf-123">Un'entità servizio Azure Active Directory (Azure AD) in possesso delle autorizzazioni per la sottoscrizione tenant in ogni hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="331cf-123">An Azure Active Directory (Azure AD) service principal that has permissions to the tenant subscription on each Azure Stack Hub.</span></span> <span data-ttu-id="331cf-124">Potrebbe essere necessario creare due entità servizio se gli hub di Azure Stack vengono distribuiti in tenant di Azure AD diversi.</span><span class="sxs-lookup"><span data-stu-id="331cf-124">You may need to create two service principals if the Azure Stack Hubs are deployed against different Azure AD tenants.</span></span> <span data-ttu-id="331cf-125">Per informazioni su come creare un'entità servizio per l'hub di Azure Stack, vedere [Creare entità servizio per consentire alle app l'accesso alle risorse dell'hub di Azure Stack](/azure-stack/user/azure-stack-create-service-principals).</span><span class="sxs-lookup"><span data-stu-id="331cf-125">To learn how to create a service principal for Azure Stack Hub, see [Create service principals to give apps access to Azure Stack Hub resources](/azure-stack/user/azure-stack-create-service-principals).</span></span>
  - <span data-ttu-id="331cf-126">**Prendere nota di ID applicazione, segreto client e nome tenant di ogni entità servizio (xxxxx.onmicrosoft.com).**</span><span class="sxs-lookup"><span data-stu-id="331cf-126">**Make a note of each service principal's application ID, client secret, and tenant name (xxxxx.onmicrosoft.com).**</span></span>
- <span data-ttu-id="331cf-127">SQL Server 2016 Enterprise è stato diffuso in ogni marketplace dell'hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="331cf-127">SQL Server 2016 Enterprise syndicated to each Azure Stack Hub's Marketplace.</span></span> <span data-ttu-id="331cf-128">Per altre informazioni sulla diffusione nel marketplace, vedere [Scaricare gli elementi del marketplace nell'hub di Azure Stack](/azure-stack/operator/azure-stack-download-azure-marketplace-item).</span><span class="sxs-lookup"><span data-stu-id="331cf-128">To learn more about marketplace syndication, see [Download Marketplace items to Azure Stack Hub](/azure-stack/operator/azure-stack-download-azure-marketplace-item).</span></span>
    <span data-ttu-id="331cf-129">**Assicurarsi che l'organizzazione disponga delle licenze SQL appropriate.**</span><span class="sxs-lookup"><span data-stu-id="331cf-129">**Make sure that your organization has the appropriate SQL licenses.**</span></span>
- <span data-ttu-id="331cf-130">[Docker per Windows](https://docs.docker.com/docker-for-windows/) installato nel computer locale.</span><span class="sxs-lookup"><span data-stu-id="331cf-130">[Docker for Windows](https://docs.docker.com/docker-for-windows/) installed on your local machine.</span></span>

## <a name="get-the-docker-image"></a><span data-ttu-id="331cf-131">Ottenere l'immagine Docker</span><span class="sxs-lookup"><span data-stu-id="331cf-131">Get the Docker image</span></span>

<span data-ttu-id="331cf-132">Le immagini Docker per ogni distribuzione eliminano i problemi di dipendenza tra versioni diverse di Azure PowerShell.</span><span class="sxs-lookup"><span data-stu-id="331cf-132">Docker images for each deployment eliminate dependency issues between different versions of Azure PowerShell.</span></span>

1. <span data-ttu-id="331cf-133">Verificare che Docker per Windows usi i contenitori di Windows.</span><span class="sxs-lookup"><span data-stu-id="331cf-133">Make sure that Docker for Windows is using Windows containers.</span></span>
2. <span data-ttu-id="331cf-134">Eseguire lo script seguente in un prompt dei comandi con privilegi elevati per ottenere il contenitore Docker con gli script di distribuzione.</span><span class="sxs-lookup"><span data-stu-id="331cf-134">Run the following script in an elevated command prompt to get the Docker container with the deployment scripts.</span></span>

    ```powershell  
    docker pull intelligentedge/sqlserver2016-hadr:1.0.0
    ```

## <a name="deploy-the-availability-group"></a><span data-ttu-id="331cf-135">Distribuire il gruppo di disponibilità</span><span class="sxs-lookup"><span data-stu-id="331cf-135">Deploy the availability group</span></span>

1. <span data-ttu-id="331cf-136">Dopo aver eseguito correttamente il pull dell'immagine del contenitore, avviare l'immagine.</span><span class="sxs-lookup"><span data-stu-id="331cf-136">Once the container image has been successfully pulled, start the image.</span></span>

      ```powershell  
      docker run -it intelligentedge/sqlserver2016-hadr:1.0.0 powershell
      ```

2. <span data-ttu-id="331cf-137">Avviato il contenitore, gli verrà assegnato un terminale di PowerShell con privilegi elevati.</span><span class="sxs-lookup"><span data-stu-id="331cf-137">Once the container has started, you'll be given an elevated PowerShell terminal in the container.</span></span> <span data-ttu-id="331cf-138">Modificare le directory per ottenere lo script di distribuzione.</span><span class="sxs-lookup"><span data-stu-id="331cf-138">Change directories to get to the deployment script.</span></span>

      ```powershell  
      cd .\SQLHADRDemo\
      ```

3. <span data-ttu-id="331cf-139">Eseguire la distribuzione.</span><span class="sxs-lookup"><span data-stu-id="331cf-139">Run the deployment.</span></span> <span data-ttu-id="331cf-140">Specificare le credenziali e i nomi delle risorse, dove necessario.</span><span class="sxs-lookup"><span data-stu-id="331cf-140">Provide credentials and resource names where needed.</span></span> <span data-ttu-id="331cf-141">HA si riferisce all'hub di Azure Stack in cui verrà distribuito il cluster a disponibilità elevata.</span><span class="sxs-lookup"><span data-stu-id="331cf-141">HA refers to the Azure Stack Hub where the HA cluster will be deployed.</span></span> <span data-ttu-id="331cf-142">DR si riferisce all'hub di Azure Stack in cui verrà distribuito il cluster di ripristino di emergenza.</span><span class="sxs-lookup"><span data-stu-id="331cf-142">DR refers to the Azure Stack Hub where the DR cluster will be deployed.</span></span>

      ```powershell
      > .\Deploy-AzureResourceGroup.ps1 `
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

4. <span data-ttu-id="331cf-143">Digitare `Y` per consentire l'installazione del provider NuGet, che avvierà i moduli del profilo di API "2018-03-01-hybrid" da installare.</span><span class="sxs-lookup"><span data-stu-id="331cf-143">Type `Y` to allow the NuGet provider to be installed, which will kick off the API Profile "2018-03-01-hybrid" modules to be installed.</span></span>

5. <span data-ttu-id="331cf-144">Attendere il completamento della distribuzione delle risorse.</span><span class="sxs-lookup"><span data-stu-id="331cf-144">Wait for resource deployment to complete.</span></span>

6. <span data-ttu-id="331cf-145">Al completamento della distribuzione della risorsa di ripristino di emergenza, uscire dal contenitore.</span><span class="sxs-lookup"><span data-stu-id="331cf-145">Once DR resource deployment has completed, exit the container.</span></span>

      ```powershell
      exit
      ```

7. <span data-ttu-id="331cf-146">Esaminare la distribuzione visualizzando le risorse nel portale di ogni hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="331cf-146">Inspect the deployment by viewing the resources in each Azure Stack Hub's portal.</span></span> <span data-ttu-id="331cf-147">Connettersi a una delle istanze di SQL nell'ambiente a disponibilità elevata ed esaminare il gruppo di disponibilità tramite SQL Server Management Studio (SSMS).</span><span class="sxs-lookup"><span data-stu-id="331cf-147">Connect to one of the SQL instances on the HA environment and inspect the Availability Group through SQL Server Management Studio (SSMS).</span></span>

    ![Disponibilità elevata SQL - SQL Server 2016](media/solution-deployment-guide-sql-ha/image2.png)

## <a name="next-steps"></a><span data-ttu-id="331cf-149">Passaggi successivi</span><span class="sxs-lookup"><span data-stu-id="331cf-149">Next steps</span></span>

- <span data-ttu-id="331cf-150">Usare SQL Server Management Studio per eseguire manualmente il failover del cluster.</span><span class="sxs-lookup"><span data-stu-id="331cf-150">Use SQL Server Management Studio to manually fail over the cluster.</span></span> <span data-ttu-id="331cf-151">Vedere [Eseguire un failover manuale forzato di un gruppo di disponibilità Always On (SQL Server)](/sql/database-engine/availability-groups/windows/perform-a-forced-manual-failover-of-an-availability-group-sql-server?view=sql-server-2017)</span><span class="sxs-lookup"><span data-stu-id="331cf-151">See [Perform a Forced Manual Failover of an Always On Availability Group (SQL Server)](/sql/database-engine/availability-groups/windows/perform-a-forced-manual-failover-of-an-availability-group-sql-server?view=sql-server-2017)</span></span>
- <span data-ttu-id="331cf-152">Altre informazioni sulle app cloud ibride.</span><span class="sxs-lookup"><span data-stu-id="331cf-152">Learn more about hybrid cloud apps.</span></span> <span data-ttu-id="331cf-153">Vedere [Soluzioni per il cloud ibrido.](/azure-stack/user/)</span><span class="sxs-lookup"><span data-stu-id="331cf-153">See [Hybrid Cloud Solutions.](/azure-stack/user/)</span></span>
- <span data-ttu-id="331cf-154">Usare i dati personalizzati o modificare il codice in questo esempio in [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).</span><span class="sxs-lookup"><span data-stu-id="331cf-154">Use your own data or modify the code to this sample on [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).</span></span>