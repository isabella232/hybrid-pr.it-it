---
title: Distribuire un gruppo di disponibilità SQL Server 2016 in Azure e nell'hub Azure Stack
description: Informazioni su come distribuire un gruppo di disponibilità SQL Server 2016 in Azure e Azure Stack Hub.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: ff6d5b9667e63a6b8d232b6dd93db2d8b12fd46d
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911045"
---
# <a name="deploy-a-sql-server-2016-availability-group-to-azure-and-azure-stack-hub"></a><span data-ttu-id="58244-103">Distribuire un gruppo di disponibilità SQL Server 2016 in Azure e nell'hub Azure Stack</span><span class="sxs-lookup"><span data-stu-id="58244-103">Deploy a SQL Server 2016 availability group to Azure and Azure Stack Hub</span></span>

<span data-ttu-id="58244-104">Questo articolo illustra in modo dettagliato la distribuzione automatica di un cluster Enterprise a disponibilità elevata SQL Server 2016 con un sito di ripristino di emergenza asincrono in due ambienti hub Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="58244-104">This article will step you through an automated deployment of a basic highly available (HA) SQL Server 2016 Enterprise cluster with an asynchronous disaster recovery (DR) site across two Azure Stack Hub environments.</span></span> <span data-ttu-id="58244-105">Per altre informazioni su SQL Server 2016 e sulla disponibilità elevata, vedere [gruppi di disponibilità always on: una soluzione di disponibilità elevata e ripristino di emergenza](https://docs.microsoft.com/sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server?view=sql-server-2016).</span><span class="sxs-lookup"><span data-stu-id="58244-105">To learn more about SQL Server 2016 and high availability, see [Always On availability groups: a high-availability and disaster-recovery solution](https://docs.microsoft.com/sql/database-engine/availability-groups/windows/always-on-availability-groups-sql-server?view=sql-server-2016).</span></span>

<span data-ttu-id="58244-106">In questa soluzione verrà compilato un ambiente di esempio per:</span><span class="sxs-lookup"><span data-stu-id="58244-106">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="58244-107">Orchestrare una distribuzione tra due hub Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="58244-107">Orchestrate a deployment across two Azure Stack Hubs.</span></span>
> - <span data-ttu-id="58244-108">Usare Docker per ridurre al minimo i problemi di dipendenza con i profili API di Azure.</span><span class="sxs-lookup"><span data-stu-id="58244-108">Use Docker to minimize dependency issues with Azure API profiles.</span></span>
> - <span data-ttu-id="58244-109">Distribuire un cluster aziendale di base a disponibilità elevata SQL Server 2016 con un sito di ripristino di emergenza.</span><span class="sxs-lookup"><span data-stu-id="58244-109">Deploy a basic highly available SQL Server 2016 Enterprise cluster with a disaster recovery site.</span></span>

> [!Tip]  
> <span data-ttu-id="58244-110">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="58244-110">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="58244-111">Microsoft Azure Stack Hub è un'estensione di Azure.</span><span class="sxs-lookup"><span data-stu-id="58244-111">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="58244-112">Azure Stack Hub offre l'agilità e l'innovazione di cloud computing all'ambiente locale, abilitando l'unico Cloud ibrido che consente di creare e distribuire app ibride ovunque.</span><span class="sxs-lookup"><span data-stu-id="58244-112">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="58244-113">L'articolo [considerazioni sulla progettazione di app ibride](overview-app-design-considerations.md) esamina i pilastri della qualità del software (posizionamento, scalabilità, disponibilità, resilienza, gestibilità e sicurezza) per la progettazione, la distribuzione e la gestione di app ibride.</span><span class="sxs-lookup"><span data-stu-id="58244-113">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="58244-114">Le considerazioni di progettazione consentono di ottimizzare la progettazione delle app ibride, riducendo al minimo le esigenze negli ambienti di produzione.</span><span class="sxs-lookup"><span data-stu-id="58244-114">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="architecture-for-sql-server-2016"></a><span data-ttu-id="58244-115">Architettura per SQL Server 2016</span><span class="sxs-lookup"><span data-stu-id="58244-115">Architecture for SQL Server 2016</span></span>

![SQL Server 2016 Hub Azure Stack a disponibilità elevata SQL](media/solution-deployment-guide-sql-ha/image1.png)

## <a name="prerequisites-for-sql-server-2016"></a><span data-ttu-id="58244-117">Prerequisiti per SQL Server 2016</span><span class="sxs-lookup"><span data-stu-id="58244-117">Prerequisites for SQL Server 2016</span></span>

- <span data-ttu-id="58244-118">Due sistemi integrati di Azure Stack hub collegati (hub Azure Stack).</span><span class="sxs-lookup"><span data-stu-id="58244-118">Two connected Azure Stack Hub integrated systems (Azure Stack Hub).</span></span> <span data-ttu-id="58244-119">Questa distribuzione non funziona nel Azure Stack Development Kit (Gabriele).</span><span class="sxs-lookup"><span data-stu-id="58244-119">This deployment doesn't work on the Azure Stack Development Kit (ASDK).</span></span> <span data-ttu-id="58244-120">Per altre informazioni sull'hub Azure Stack, vedere la [Panoramica di Azure stack](https://azure.microsoft.com/overview/azure-stack/).</span><span class="sxs-lookup"><span data-stu-id="58244-120">To learn more about Azure Stack Hub, see the [Azure Stack overview](https://azure.microsoft.com/overview/azure-stack/).</span></span>
- <span data-ttu-id="58244-121">Una sottoscrizione tenant in ogni hub Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="58244-121">A tenant subscription on each Azure Stack Hub.</span></span>
  - <span data-ttu-id="58244-122">**Prendere nota di ogni ID sottoscrizione e dell'endpoint Azure Resource Manager per ogni hub Azure Stack.**</span><span class="sxs-lookup"><span data-stu-id="58244-122">**Make a note of each subscription ID and the Azure Resource Manager endpoint for each Azure Stack Hub.**</span></span>
- <span data-ttu-id="58244-123">Un'entità servizio Azure Active Directory (Azure AD) che dispone delle autorizzazioni per la sottoscrizione tenant in ogni hub Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="58244-123">An Azure Active Directory (Azure AD) service principal that has permissions to the tenant subscription on each Azure Stack Hub.</span></span> <span data-ttu-id="58244-124">Potrebbe essere necessario creare due entità servizio se gli hub Azure Stack vengono distribuiti in tenant Azure AD diversi.</span><span class="sxs-lookup"><span data-stu-id="58244-124">You may need to create two service principals if the Azure Stack Hubs are deployed against different Azure AD tenants.</span></span> <span data-ttu-id="58244-125">Per informazioni su come creare un'entità servizio per Azure Stack Hub, vedere [creare entità servizio per consentire alle app di accedere alle risorse dell'hub Azure stack](https://docs.microsoft.com/azure-stack/user/azure-stack-create-service-principals).</span><span class="sxs-lookup"><span data-stu-id="58244-125">To learn how to create a service principal for Azure Stack Hub, see [Create service principals to give apps access to Azure Stack Hub resources](https://docs.microsoft.com/azure-stack/user/azure-stack-create-service-principals).</span></span>
  - <span data-ttu-id="58244-126">**Prendere nota dell'ID applicazione di ogni entità servizio, del segreto client e del nome del tenant (xxxxx.onmicrosoft.com).**</span><span class="sxs-lookup"><span data-stu-id="58244-126">**Make a note of each service principal's application ID, client secret, and tenant name (xxxxx.onmicrosoft.com).**</span></span>
- <span data-ttu-id="58244-127">SQL Server 2016 Enterprise syndicated per ogni Marketplace di Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="58244-127">SQL Server 2016 Enterprise syndicated to each Azure Stack Hub's Marketplace.</span></span> <span data-ttu-id="58244-128">Per altre informazioni sulla diffusione del Marketplace, vedere [scaricare gli elementi del Marketplace nell'Hub Azure stack](https://docs.microsoft.com/azure-stack/operator/azure-stack-download-azure-marketplace-item).</span><span class="sxs-lookup"><span data-stu-id="58244-128">To learn more about marketplace syndication, see [Download Marketplace items to Azure Stack Hub](https://docs.microsoft.com/azure-stack/operator/azure-stack-download-azure-marketplace-item).</span></span>
    <span data-ttu-id="58244-129">**Assicurarsi che l'organizzazione disponga delle licenze SQL appropriate.**</span><span class="sxs-lookup"><span data-stu-id="58244-129">**Make sure that your organization has the appropriate SQL licenses.**</span></span>
- <span data-ttu-id="58244-130">[Docker per Windows](https://docs.docker.com/docker-for-windows/) installato nel computer locale.</span><span class="sxs-lookup"><span data-stu-id="58244-130">[Docker for Windows](https://docs.docker.com/docker-for-windows/) installed on your local machine.</span></span>

## <a name="get-the-docker-image"></a><span data-ttu-id="58244-131">Ottenere l'immagine Docker</span><span class="sxs-lookup"><span data-stu-id="58244-131">Get the Docker image</span></span>

<span data-ttu-id="58244-132">Le immagini Docker per ogni distribuzione eliminano i problemi di dipendenza tra versioni diverse di Azure PowerShell.</span><span class="sxs-lookup"><span data-stu-id="58244-132">Docker images for each deployment eliminate dependency issues between different versions of Azure PowerShell.</span></span>

1. <span data-ttu-id="58244-133">Assicurarsi che Docker per Windows usi i contenitori di Windows.</span><span class="sxs-lookup"><span data-stu-id="58244-133">Make sure that Docker for Windows is using Windows containers.</span></span>
2. <span data-ttu-id="58244-134">Eseguire lo script seguente in un prompt dei comandi con privilegi elevati per ottenere il contenitore Docker con gli script di distribuzione.</span><span class="sxs-lookup"><span data-stu-id="58244-134">Run the following script in an elevated command prompt to get the Docker container with the deployment scripts.</span></span>

    ```powershell  
    docker pull intelligentedge/sqlserver2016-hadr:1.0.0
    ```

## <a name="deploy-the-availability-group"></a><span data-ttu-id="58244-135">Distribuire il gruppo di disponibilità</span><span class="sxs-lookup"><span data-stu-id="58244-135">Deploy the availability group</span></span>

1. <span data-ttu-id="58244-136">Una volta eseguito il pull dell'immagine del contenitore, avviare l'immagine.</span><span class="sxs-lookup"><span data-stu-id="58244-136">Once the container image has been successfully pulled, start the image.</span></span>

      ```powershell  
      docker run -it intelligentedge/sqlserver2016-hadr:1.0.0 powershell
      ```

2. <span data-ttu-id="58244-137">Una volta avviato il contenitore, verrà assegnato un terminale di PowerShell con privilegi elevati nel contenitore.</span><span class="sxs-lookup"><span data-stu-id="58244-137">Once the container has started, you'll be given an elevated PowerShell terminal in the container.</span></span> <span data-ttu-id="58244-138">Modificare le directory per ottenere lo script di distribuzione.</span><span class="sxs-lookup"><span data-stu-id="58244-138">Change directories to get to the deployment script.</span></span>

      ```powershell  
      cd .\SQLHADRDemo\
      ```

3. <span data-ttu-id="58244-139">Eseguire la distribuzione.</span><span class="sxs-lookup"><span data-stu-id="58244-139">Run the deployment.</span></span> <span data-ttu-id="58244-140">Fornire le credenziali e i nomi delle risorse laddove necessario.</span><span class="sxs-lookup"><span data-stu-id="58244-140">Provide credentials and resource names where needed.</span></span> <span data-ttu-id="58244-141">HA si riferisce all'hub Azure Stack in cui verrà distribuito il cluster a disponibilità elevata.</span><span class="sxs-lookup"><span data-stu-id="58244-141">HA refers to the Azure Stack Hub where the HA cluster will be deployed.</span></span> <span data-ttu-id="58244-142">Il ripristino di emergenza fa riferimento all'hub Azure Stack in cui verrà distribuito il cluster di ripristino di emergenza.</span><span class="sxs-lookup"><span data-stu-id="58244-142">DR refers to the Azure Stack Hub where the DR cluster will be deployed.</span></span>

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

4. <span data-ttu-id="58244-143">Digitare `Y` per consentire l'installazione del provider NuGet, che consente di avviare i moduli del profilo API "2018-03-01-Hybrid" da installare.</span><span class="sxs-lookup"><span data-stu-id="58244-143">Type `Y` to allow the NuGet provider to be installed, which will kick off the API Profile "2018-03-01-hybrid" modules to be installed.</span></span>

5. <span data-ttu-id="58244-144">Attendere il completamento della distribuzione delle risorse.</span><span class="sxs-lookup"><span data-stu-id="58244-144">Wait for resource deployment to complete.</span></span>

6. <span data-ttu-id="58244-145">Una volta completata la distribuzione delle risorse di ripristino di emergenza, uscire dal contenitore.</span><span class="sxs-lookup"><span data-stu-id="58244-145">Once DR resource deployment has completed, exit the container.</span></span>

      ```powershell
      exit
      ```

7. <span data-ttu-id="58244-146">Esaminare la distribuzione visualizzando le risorse nel portale di ogni hub Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="58244-146">Inspect the deployment by viewing the resources in each Azure Stack Hub's portal.</span></span> <span data-ttu-id="58244-147">Connettersi a una delle istanze di SQL nell'ambiente a disponibilità elevata e ispezionare il gruppo di disponibilità tramite SQL Server Management Studio (SSMS).</span><span class="sxs-lookup"><span data-stu-id="58244-147">Connect to one of the SQL instances on the HA environment and inspect the Availability Group through SQL Server Management Studio (SSMS).</span></span>

    ![SQL Server 2016 SQL HA](media/solution-deployment-guide-sql-ha/image2.png)

## <a name="next-steps"></a><span data-ttu-id="58244-149">Passaggi successivi</span><span class="sxs-lookup"><span data-stu-id="58244-149">Next steps</span></span>

- <span data-ttu-id="58244-150">Usare SQL Server Management Studio per eseguire manualmente il failover del cluster.</span><span class="sxs-lookup"><span data-stu-id="58244-150">Use SQL Server Management Studio to manually fail over the cluster.</span></span> <span data-ttu-id="58244-151">Vedere [eseguire un failover manuale forzato di un gruppo di disponibilità always on (SQL Server)](https://docs.microsoft.com/sql/database-engine/availability-groups/windows/perform-a-forced-manual-failover-of-an-availability-group-sql-server?view=sql-server-2017)</span><span class="sxs-lookup"><span data-stu-id="58244-151">See [Perform a Forced Manual Failover of an Always On Availability Group (SQL Server)](https://docs.microsoft.com/sql/database-engine/availability-groups/windows/perform-a-forced-manual-failover-of-an-availability-group-sql-server?view=sql-server-2017)</span></span>
- <span data-ttu-id="58244-152">Scopri di più sulle app cloud ibride.</span><span class="sxs-lookup"><span data-stu-id="58244-152">Learn more about hybrid cloud apps.</span></span> <span data-ttu-id="58244-153">Vedere [soluzioni cloud ibride.](https://aka.ms/azsdevtutorials)</span><span class="sxs-lookup"><span data-stu-id="58244-153">See [Hybrid Cloud Solutions.](https://aka.ms/azsdevtutorials)</span></span>
- <span data-ttu-id="58244-154">Usare i propri dati o modificare il codice in questo esempio su [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).</span><span class="sxs-lookup"><span data-stu-id="58244-154">Use your own data or modify the code to this sample on [GitHub](https://github.com/Azure-Samples/azure-intelligent-edge-patterns).</span></span>
