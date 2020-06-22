---
title: Distribuisci app ibride con dati locali che scalano tra cloud
description: Informazioni su come distribuire un'app che usa dati locali e scalabilità tra cloud usando Azure e l'hub Azure Stack.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 75289eae902c5363862e345bdedb97cbcee0476e
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 06/17/2020
ms.locfileid: "84910891"
---
# <a name="deploy-hybrid-app-with-on-premises-data-that-scales-cross-cloud"></a><span data-ttu-id="3a862-103">Distribuisci app ibride con dati locali che scalano tra cloud</span><span class="sxs-lookup"><span data-stu-id="3a862-103">Deploy hybrid app with on-premises data that scales cross-cloud</span></span>

<span data-ttu-id="3a862-104">Questa guida alla soluzione illustra come distribuire un'app ibrida che si estende sia per Azure che per hub Azure Stack e usa un'unica origine dati locale.</span><span class="sxs-lookup"><span data-stu-id="3a862-104">This solution guide shows you how to deploy a hybrid app that spans both Azure and Azure Stack Hub and uses a single on-premises data source.</span></span>

<span data-ttu-id="3a862-105">Grazie a una soluzione cloud ibrida, è possibile combinare i vantaggi di conformità di un cloud privato con la scalabilità del cloud pubblico.</span><span class="sxs-lookup"><span data-stu-id="3a862-105">By using a hybrid cloud solution, you can combine the compliance benefits of a private cloud with the scalability of the public cloud.</span></span> <span data-ttu-id="3a862-106">Gli sviluppatori possono inoltre trarre vantaggio dall'ecosistema di sviluppatori Microsoft e applicare le proprie competenze al cloud e agli ambienti locali.</span><span class="sxs-lookup"><span data-stu-id="3a862-106">Your developers can also take advantage of the Microsoft developer ecosystem and apply their skills to the cloud and on-premises environments.</span></span>

## <a name="overview-and-assumptions"></a><span data-ttu-id="3a862-107">Panoramica e presupposti</span><span class="sxs-lookup"><span data-stu-id="3a862-107">Overview and assumptions</span></span>

<span data-ttu-id="3a862-108">Seguire questa esercitazione per configurare un flusso di lavoro che consente agli sviluppatori di distribuire un'app Web identica a un cloud pubblico e a un cloud privato.</span><span class="sxs-lookup"><span data-stu-id="3a862-108">Follow this tutorial to set up a workflow that lets developers deploy an identical web app to a public cloud and a private cloud.</span></span> <span data-ttu-id="3a862-109">Questa app può accedere a una rete instradabile non Internet ospitata nel cloud privato.</span><span class="sxs-lookup"><span data-stu-id="3a862-109">This app can access a non-internet routable network hosted on the private cloud.</span></span> <span data-ttu-id="3a862-110">Queste app Web vengono monitorate e quando si verifica un picco nel traffico, un programma modifica i record DNS per reindirizzare il traffico al cloud pubblico.</span><span class="sxs-lookup"><span data-stu-id="3a862-110">These web apps are monitored and when there's a spike in traffic, a program modifies the DNS records to redirect traffic to the public cloud.</span></span> <span data-ttu-id="3a862-111">Quando il traffico scende al livello prima del picco, il traffico viene indirizzato di nuovo al cloud privato.</span><span class="sxs-lookup"><span data-stu-id="3a862-111">When traffic drops to the level before the spike, traffic is routed back to the private cloud.</span></span>

<span data-ttu-id="3a862-112">Questa esercitazione illustra le attività seguenti:</span><span class="sxs-lookup"><span data-stu-id="3a862-112">This tutorial covers the following tasks:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="3a862-113">Distribuire un server di database SQL Server con connessione ibrida.</span><span class="sxs-lookup"><span data-stu-id="3a862-113">Deploy a hybrid-connected SQL Server database server.</span></span>
> - <span data-ttu-id="3a862-114">Connettere un'app Web in Azure globale a una rete ibrida.</span><span class="sxs-lookup"><span data-stu-id="3a862-114">Connect a web app in global Azure to a hybrid network.</span></span>
> - <span data-ttu-id="3a862-115">Configurare il DNS per la scalabilità tra cloud.</span><span class="sxs-lookup"><span data-stu-id="3a862-115">Configure DNS for cross-cloud scaling.</span></span>
> - <span data-ttu-id="3a862-116">Configurare i certificati SSL per la scalabilità tra cloud.</span><span class="sxs-lookup"><span data-stu-id="3a862-116">Configure SSL certificates for cross-cloud scaling.</span></span>
> - <span data-ttu-id="3a862-117">Configurare e distribuire l'app Web.</span><span class="sxs-lookup"><span data-stu-id="3a862-117">Configure and deploy the web app.</span></span>
> - <span data-ttu-id="3a862-118">Creare un profilo di gestione traffico e configurarlo per la scalabilità tra cloud.</span><span class="sxs-lookup"><span data-stu-id="3a862-118">Create a Traffic Manager profile and configure it for cross-cloud scaling.</span></span>
> - <span data-ttu-id="3a862-119">Configurare Application Insights il monitoraggio e gli avvisi per un aumento del traffico.</span><span class="sxs-lookup"><span data-stu-id="3a862-119">Set up Application Insights monitoring and alerting for increased traffic.</span></span>
> - <span data-ttu-id="3a862-120">Configurare il cambio di traffico automatico tra Azure globale e l'hub Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="3a862-120">Configure automatic traffic switching between global Azure and Azure Stack Hub.</span></span>

> [!Tip]  
> <span data-ttu-id="3a862-121">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="3a862-121">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="3a862-122">Microsoft Azure Stack Hub è un'estensione di Azure.</span><span class="sxs-lookup"><span data-stu-id="3a862-122">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="3a862-123">Azure Stack Hub offre l'agilità e l'innovazione di cloud computing all'ambiente locale, abilitando l'unico Cloud ibrido che consente di creare e distribuire app ibride ovunque.</span><span class="sxs-lookup"><span data-stu-id="3a862-123">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="3a862-124">L'articolo [considerazioni sulla progettazione di app ibride](overview-app-design-considerations.md) esamina i pilastri della qualità del software (posizionamento, scalabilità, disponibilità, resilienza, gestibilità e sicurezza) per la progettazione, la distribuzione e la gestione di app ibride.</span><span class="sxs-lookup"><span data-stu-id="3a862-124">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="3a862-125">Le considerazioni di progettazione consentono di ottimizzare la progettazione delle app ibride, riducendo al minimo le esigenze negli ambienti di produzione.</span><span class="sxs-lookup"><span data-stu-id="3a862-125">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

### <a name="assumptions"></a><span data-ttu-id="3a862-126">Presupposti</span><span class="sxs-lookup"><span data-stu-id="3a862-126">Assumptions</span></span>

<span data-ttu-id="3a862-127">Questa esercitazione presuppone che l'utente abbia una conoscenza di base dell'hub globale di Azure e Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="3a862-127">This tutorial assumes that you have a basic knowledge of global Azure and Azure Stack Hub.</span></span> <span data-ttu-id="3a862-128">Per altre informazioni prima di iniziare l'esercitazione, vedere gli articoli seguenti:</span><span class="sxs-lookup"><span data-stu-id="3a862-128">If you want to learn more before starting the tutorial, review these articles:</span></span>

- [<span data-ttu-id="3a862-129">Introduzione ad Azure</span><span class="sxs-lookup"><span data-stu-id="3a862-129">Introduction to Azure</span></span>](https://azure.microsoft.com/overview/what-is-azure/)
- [<span data-ttu-id="3a862-130">Concetti chiave dell'hub Azure Stack</span><span class="sxs-lookup"><span data-stu-id="3a862-130">Azure Stack Hub Key Concepts</span></span>](/azure-stack/operator/azure-stack-overview.md)

<span data-ttu-id="3a862-131">Questa esercitazione presuppone anche che si disponga di una sottoscrizione di Azure.</span><span class="sxs-lookup"><span data-stu-id="3a862-131">This tutorial also assumes that you have an Azure subscription.</span></span> <span data-ttu-id="3a862-132">Se non si ha una sottoscrizione, [creare un account gratuito prima di](https://azure.microsoft.com/free/) iniziare.</span><span class="sxs-lookup"><span data-stu-id="3a862-132">If you don't have a subscription, [create a free account](https://azure.microsoft.com/free/) before you begin.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="3a862-133">Prerequisiti</span><span class="sxs-lookup"><span data-stu-id="3a862-133">Prerequisites</span></span>

<span data-ttu-id="3a862-134">Prima di iniziare questa soluzione, assicurarsi di soddisfare i requisiti seguenti:</span><span class="sxs-lookup"><span data-stu-id="3a862-134">Before you start this solution, make sure you meet the following requirements:</span></span>

- <span data-ttu-id="3a862-135">Un Azure Stack Development Kit (Gabriele) o una sottoscrizione in un sistema integrato Azure Stack Hub.</span><span class="sxs-lookup"><span data-stu-id="3a862-135">An Azure Stack Development Kit (ASDK) or a subscription on an Azure Stack Hub Integrated System.</span></span> <span data-ttu-id="3a862-136">Per distribuire il Gabriele, seguire le istruzioni in [distribuire Gabriele usando il programma di installazione](/azure-stack/asdk/asdk-install.md).</span><span class="sxs-lookup"><span data-stu-id="3a862-136">To deploy the ASDK, follow the instructions in [Deploy the ASDK using the installer](/azure-stack/asdk/asdk-install.md).</span></span>
- <span data-ttu-id="3a862-137">L'installazione dell'hub Azure Stack deve avere installato quanto segue:</span><span class="sxs-lookup"><span data-stu-id="3a862-137">Your Azure Stack Hub installation should have the following installed:</span></span>
  - <span data-ttu-id="3a862-138">Il servizio app Azure.</span><span class="sxs-lookup"><span data-stu-id="3a862-138">The Azure App Service.</span></span> <span data-ttu-id="3a862-139">Collaborare con l'operatore Azure Stack hub per distribuire e configurare il servizio app Azure nell'ambiente in uso.</span><span class="sxs-lookup"><span data-stu-id="3a862-139">Work with your Azure Stack Hub Operator to deploy and configure the Azure App Service on your environment.</span></span> <span data-ttu-id="3a862-140">Per questa esercitazione è necessario che il servizio app disponga di almeno un ruolo di lavoro dedicato disponibile (1).</span><span class="sxs-lookup"><span data-stu-id="3a862-140">This tutorial requires the App Service to have at least one (1) available dedicated worker role.</span></span>
  - <span data-ttu-id="3a862-141">Un'immagine di Windows Server 2016.</span><span class="sxs-lookup"><span data-stu-id="3a862-141">A Windows Server 2016 image.</span></span>
  - <span data-ttu-id="3a862-142">Windows Server 2016 con un'immagine Microsoft SQL Server.</span><span class="sxs-lookup"><span data-stu-id="3a862-142">A Windows Server 2016 with a Microsoft SQL Server image.</span></span>
  - <span data-ttu-id="3a862-143">Piani e offerte appropriati.</span><span class="sxs-lookup"><span data-stu-id="3a862-143">The appropriate plans and offers.</span></span>
  - <span data-ttu-id="3a862-144">Nome di dominio per l'app Web.</span><span class="sxs-lookup"><span data-stu-id="3a862-144">A domain name for your web app.</span></span> <span data-ttu-id="3a862-145">Se non si dispone di un nome di dominio, è possibile acquistarne uno da un provider di dominio, ad esempio GoDaddy, Bluehost e inMotion.</span><span class="sxs-lookup"><span data-stu-id="3a862-145">If you don't have a domain name, you can buy one from a domain provider like GoDaddy, Bluehost, and InMotion.</span></span>
- <span data-ttu-id="3a862-146">Un certificato SSL per il dominio da un'autorità di certificazione attendibile, ad esempio LetsEncrypt.</span><span class="sxs-lookup"><span data-stu-id="3a862-146">An SSL certificate for your domain from a trusted certificate authority like LetsEncrypt.</span></span>
- <span data-ttu-id="3a862-147">Un'app Web che comunica con un database di SQL Server e supporta Application Insights.</span><span class="sxs-lookup"><span data-stu-id="3a862-147">A web app that communicates with a SQL Server database and supports Application Insights.</span></span> <span data-ttu-id="3a862-148">È possibile scaricare l'app di esempio [dotnetcore-SQLDB-tutorial](https://github.com/Azure-Samples/dotnetcore-sqldb-tutorial) da GitHub.</span><span class="sxs-lookup"><span data-stu-id="3a862-148">You can download the [dotnetcore-sqldb-tutorial](https://github.com/Azure-Samples/dotnetcore-sqldb-tutorial) sample app from GitHub.</span></span>
- <span data-ttu-id="3a862-149">Una rete ibrida tra una rete virtuale di Azure e una rete virtuale Hub Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="3a862-149">A hybrid network between an Azure virtual network and Azure Stack Hub virtual network.</span></span> <span data-ttu-id="3a862-150">Per istruzioni dettagliate, vedere [configurare la connettività cloud ibrida con Azure e l'Hub Azure stack](solution-deployment-guide-connectivity.md).</span><span class="sxs-lookup"><span data-stu-id="3a862-150">For detailed instructions, see [Configure hybrid cloud connectivity with Azure and Azure Stack Hub](solution-deployment-guide-connectivity.md).</span></span>

- <span data-ttu-id="3a862-151">Una pipeline di integrazione continua/distribuzione continua (CI/CD) ibrida con un agente di compilazione privato nell'hub Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="3a862-151">A hybrid continuous integration/continuous deployment (CI/CD) pipeline with a private build agent on Azure Stack Hub.</span></span> <span data-ttu-id="3a862-152">Per istruzioni dettagliate, vedere [configurare l'identità del cloud ibrido con Azure e App Hub Azure stack](solution-deployment-guide-identity.md).</span><span class="sxs-lookup"><span data-stu-id="3a862-152">For detailed instructions, see [Configure hybrid cloud identity with Azure and Azure Stack Hub apps](solution-deployment-guide-identity.md).</span></span>

## <a name="deploy-a-hybrid-connected-sql-server-database-server"></a><span data-ttu-id="3a862-153">Distribuire un server di database SQL Server con connessione ibrida</span><span class="sxs-lookup"><span data-stu-id="3a862-153">Deploy a hybrid-connected SQL Server database server</span></span>

1. <span data-ttu-id="3a862-154">Accedere al portale utenti dell'hub Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="3a862-154">Sign to the Azure Stack Hub user portal.</span></span>

2. <span data-ttu-id="3a862-155">Nel **Dashboard**selezionare **Marketplace**.</span><span class="sxs-lookup"><span data-stu-id="3a862-155">On the **Dashboard**, select **Marketplace**.</span></span>

    ![Marketplace Azure Stack Hub](media/solution-deployment-guide-hybrid/image1.png)

3. <span data-ttu-id="3a862-157">In **Marketplace**selezionare **calcolo**, quindi scegliere **altro**.</span><span class="sxs-lookup"><span data-stu-id="3a862-157">In **Marketplace**, select **Compute**, and then choose **More**.</span></span> <span data-ttu-id="3a862-158">In **altro**selezionare la **licenza gratuita SQL Server: SQL Server 2017 Developer nell'immagine di Windows Server** .</span><span class="sxs-lookup"><span data-stu-id="3a862-158">Under **More**, select the **Free SQL Server License: SQL Server 2017 Developer on Windows Server** image.</span></span>

    ![Selezionare un'immagine di macchina virtuale nel portale per gli utenti di Azure Stack Hub](media/solution-deployment-guide-hybrid/image2.png)

4. <span data-ttu-id="3a862-160">In **licenza gratuita SQL Server: SQL Server 2017 Developer su Windows Server**, selezionare **Crea**.</span><span class="sxs-lookup"><span data-stu-id="3a862-160">On **Free SQL Server License: SQL Server 2017 Developer on Windows Server**, select **Create**.</span></span>

5. <span data-ttu-id="3a862-161">In **nozioni di base > configurare le impostazioni di base**, specificare un **nome** per la macchina virtuale (VM), un **nome utente** per il SQL Server SA e una password per l'associazione di **accesso** .</span><span class="sxs-lookup"><span data-stu-id="3a862-161">On **Basics > Configure basic settings**, provide a **Name** for the virtual machine (VM), a **User name** for the SQL Server SA, and a **Password** for the SA.</span></span>  <span data-ttu-id="3a862-162">Dall'elenco a discesa **sottoscrizione** selezionare la sottoscrizione in cui si sta eseguendo la distribuzione.</span><span class="sxs-lookup"><span data-stu-id="3a862-162">From the **Subscription** drop-down list, select the subscription that you're deploying to.</span></span> <span data-ttu-id="3a862-163">Per **gruppo di risorse**, usare **scegliere esistente** e inserire la macchina virtuale nello stesso gruppo di risorse dell'app Web dell'hub Azure stack.</span><span class="sxs-lookup"><span data-stu-id="3a862-163">For **Resource group**, use **Choose existing** and put the VM in the same resource group as your Azure Stack Hub web app.</span></span>

    ![Configurare le impostazioni di base per la macchina virtuale nel portale per gli utenti di Azure Stack Hub](media/solution-deployment-guide-hybrid/image3.png)

6. <span data-ttu-id="3a862-165">In **dimensioni**selezionare una dimensione per la macchina virtuale.</span><span class="sxs-lookup"><span data-stu-id="3a862-165">Under **Size**, pick a size for your VM.</span></span> <span data-ttu-id="3a862-166">Per questa esercitazione, è consigliabile A2_Standard o una DS2_V2_Standard.</span><span class="sxs-lookup"><span data-stu-id="3a862-166">For this tutorial, we recommend A2_Standard or a DS2_V2_Standard.</span></span>

7. <span data-ttu-id="3a862-167">In **impostazioni > configurare le funzionalità facoltative**, configurare le impostazioni seguenti:</span><span class="sxs-lookup"><span data-stu-id="3a862-167">Under **Settings > Configure optional features**, configure the following settings:</span></span>

   - <span data-ttu-id="3a862-168">**Account di archiviazione**: se necessario, creare un nuovo account.</span><span class="sxs-lookup"><span data-stu-id="3a862-168">**Storage account**: Create a new account if you need one.</span></span>
   - <span data-ttu-id="3a862-169">**Rete virtuale**:</span><span class="sxs-lookup"><span data-stu-id="3a862-169">**Virtual network**:</span></span>

     > [!Important]  
     > <span data-ttu-id="3a862-170">Assicurarsi che la macchina virtuale SQL Server sia distribuita nella stessa rete virtuale dei gateway VPN.</span><span class="sxs-lookup"><span data-stu-id="3a862-170">Make sure your SQL Server VM is deployed on the same  virtual network as the VPN gateways.</span></span>

   - <span data-ttu-id="3a862-171">**Indirizzo IP pubblico**: usare le impostazioni predefinite.</span><span class="sxs-lookup"><span data-stu-id="3a862-171">**Public IP address**: Use the default settings.</span></span>
   - <span data-ttu-id="3a862-172">**Gruppo di sicurezza di rete**: (NSG).</span><span class="sxs-lookup"><span data-stu-id="3a862-172">**Network security group**: (NSG).</span></span> <span data-ttu-id="3a862-173">Creare un nuovo NSG.</span><span class="sxs-lookup"><span data-stu-id="3a862-173">Create a new NSG.</span></span>
   - <span data-ttu-id="3a862-174">**Estensioni e monitoraggio**: Mantieni le impostazioni predefinite.</span><span class="sxs-lookup"><span data-stu-id="3a862-174">**Extensions and Monitoring**: Keep the default settings.</span></span>
   - <span data-ttu-id="3a862-175">**Account di archiviazione di diagnostica**: se necessario, creare un nuovo account.</span><span class="sxs-lookup"><span data-stu-id="3a862-175">**Diagnostics storage account**: Create a new account if you need one.</span></span>
   - <span data-ttu-id="3a862-176">Selezionare **OK** per salvare la configurazione.</span><span class="sxs-lookup"><span data-stu-id="3a862-176">Select **OK** to save your configuration.</span></span>

     ![Configurare le funzionalità di VM facoltative nel portale per gli utenti dell'hub Azure Stack](media/solution-deployment-guide-hybrid/image4.png)

8. <span data-ttu-id="3a862-178">In **impostazioni SQL Server**configurare le impostazioni seguenti:</span><span class="sxs-lookup"><span data-stu-id="3a862-178">Under **SQL Server settings**, configure the following settings:</span></span>

   - <span data-ttu-id="3a862-179">Per la **connettività SQL**selezionare **pubblico (Internet)**.</span><span class="sxs-lookup"><span data-stu-id="3a862-179">For **SQL connectivity**, select **Public (Internet)**.</span></span>
   - <span data-ttu-id="3a862-180">Per **porta**, Mantieni il valore predefinito **1433**.</span><span class="sxs-lookup"><span data-stu-id="3a862-180">For **Port**, keep the default, **1433**.</span></span>
   - <span data-ttu-id="3a862-181">Per **autenticazione SQL**selezionare **Abilita**.</span><span class="sxs-lookup"><span data-stu-id="3a862-181">For **SQL authentication**, select **Enable**.</span></span>

     > [!Note]  
     > <span data-ttu-id="3a862-182">Quando si Abilita l'autenticazione SQL, è necessario inserire automaticamente le informazioni "SQLAdmin" configurate in **nozioni di base**.</span><span class="sxs-lookup"><span data-stu-id="3a862-182">When you enable SQL authentication, it should auto-populate with the "SQLAdmin" information that you configured in **Basics**.</span></span>

   - <span data-ttu-id="3a862-183">Per le altre impostazioni, Mantieni le impostazioni predefinite.</span><span class="sxs-lookup"><span data-stu-id="3a862-183">For the rest of the settings, keep the defaults.</span></span> <span data-ttu-id="3a862-184">Selezionare **OK**.</span><span class="sxs-lookup"><span data-stu-id="3a862-184">Select **OK**.</span></span>

     ![Configurare le impostazioni di SQL Server nel portale per gli utenti di Azure Stack Hub](media/solution-deployment-guide-hybrid/image5.png)

9. <span data-ttu-id="3a862-186">In **Riepilogo**verificare la configurazione della macchina virtuale e quindi selezionare **OK** per avviare la distribuzione.</span><span class="sxs-lookup"><span data-stu-id="3a862-186">On **Summary**, review the VM configuration and then select **OK** to start the deployment.</span></span>

    ![Riepilogo della configurazione nel portale per gli utenti dell'hub Azure Stack](media/solution-deployment-guide-hybrid/image6.png)

10. <span data-ttu-id="3a862-188">La creazione della nuova macchina virtuale richiede tempo.</span><span class="sxs-lookup"><span data-stu-id="3a862-188">It takes some time to create the new VM.</span></span> <span data-ttu-id="3a862-189">È possibile visualizzare lo stato delle VM in **macchine virtuali**.</span><span class="sxs-lookup"><span data-stu-id="3a862-189">You can view the STATUS of your VMs in **Virtual machines**.</span></span>

    ![Stato delle macchine virtuali nel portale per gli utenti di Azure Stack Hub](media/solution-deployment-guide-hybrid/image7.png)

## <a name="create-web-apps-in-azure-and-azure-stack-hub"></a><span data-ttu-id="3a862-191">Creare app Web in Azure e Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="3a862-191">Create web apps in Azure and Azure Stack Hub</span></span>

<span data-ttu-id="3a862-192">Il servizio app Azure semplifica l'esecuzione e la gestione di un'app Web.</span><span class="sxs-lookup"><span data-stu-id="3a862-192">The Azure App Service simplifies running and managing a web app.</span></span> <span data-ttu-id="3a862-193">Poiché Azure Stack Hub è coerente con Azure, il servizio app può essere eseguito in entrambi gli ambienti.</span><span class="sxs-lookup"><span data-stu-id="3a862-193">Because Azure Stack Hub is consistent with Azure,  the App Service can run in both environments.</span></span> <span data-ttu-id="3a862-194">Il servizio app verrà usato per ospitare l'app.</span><span class="sxs-lookup"><span data-stu-id="3a862-194">You'll use the App Service to host your app.</span></span>

### <a name="create-web-apps"></a><span data-ttu-id="3a862-195">Creare app Web</span><span class="sxs-lookup"><span data-stu-id="3a862-195">Create web apps</span></span>

1. <span data-ttu-id="3a862-196">Creare un'app Web in Azure seguendo le istruzioni riportate in [gestire un piano di servizio app in Azure](https://docs.microsoft.com/azure/app-service/app-service-plan-manage#create-an-app-service-plan).</span><span class="sxs-lookup"><span data-stu-id="3a862-196">Create a web app in Azure by following the instructions in [Manage an App Service plan in Azure](https://docs.microsoft.com/azure/app-service/app-service-plan-manage#create-an-app-service-plan).</span></span> <span data-ttu-id="3a862-197">Assicurarsi di inserire l'app Web nella stessa sottoscrizione e nello stesso gruppo di risorse della rete ibrida.</span><span class="sxs-lookup"><span data-stu-id="3a862-197">Make sure you put the web app in the same subscription and resource group as your hybrid network.</span></span>

2. <span data-ttu-id="3a862-198">Ripetere il passaggio precedente (1) nell'hub Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="3a862-198">Repeat the previous step (1) in Azure Stack Hub.</span></span>

### <a name="add-route-for-azure-stack-hub"></a><span data-ttu-id="3a862-199">Aggiungi route per hub Azure Stack</span><span class="sxs-lookup"><span data-stu-id="3a862-199">Add route for Azure Stack Hub</span></span>

<span data-ttu-id="3a862-200">Il servizio app nell'hub Azure Stack deve essere instradabile dalla rete Internet pubblica per consentire agli utenti di accedere all'app.</span><span class="sxs-lookup"><span data-stu-id="3a862-200">The App Service on Azure Stack Hub must be routable from the public internet to let users access your app.</span></span> <span data-ttu-id="3a862-201">Se l'hub Azure Stack è accessibile da Internet, prendere nota dell'indirizzo IP pubblico o dell'URL per l'app Web Hub Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="3a862-201">If your Azure Stack Hub is accessible from the internet, make a note of the public-facing IP address or URL for the Azure Stack Hub web app.</span></span>

<span data-ttu-id="3a862-202">Se si usa un Gabriele, è possibile [configurare un mapping NAT statico](/azure-stack/operator/azure-stack-create-vpn-connection-one-node.md#configure-the-nat-vm-on-each-asdk-for-gateway-traversal) per esporre il servizio app al di fuori dell'ambiente virtuale.</span><span class="sxs-lookup"><span data-stu-id="3a862-202">If you're using an ASDK, you can [configure a static NAT mapping](/azure-stack/operator/azure-stack-create-vpn-connection-one-node.md#configure-the-nat-vm-on-each-asdk-for-gateway-traversal) to expose App Service outside the virtual environment.</span></span>

### <a name="connect-a-web-app-in-azure-to-a-hybrid-network"></a><span data-ttu-id="3a862-203">Connettere un'app Web in Azure a una rete ibrida</span><span class="sxs-lookup"><span data-stu-id="3a862-203">Connect a web app in Azure to a hybrid network</span></span>

<span data-ttu-id="3a862-204">Per fornire connettività tra il front-end Web in Azure e il database SQL Server nell'hub Azure Stack, l'app Web deve essere connessa alla rete ibrida tra Azure e l'hub Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="3a862-204">To provide connectivity between the web front end in Azure and the SQL Server database in Azure Stack Hub, the web app must be connected to the hybrid network between Azure and Azure Stack Hub.</span></span> <span data-ttu-id="3a862-205">Per abilitare la connettività, è necessario:</span><span class="sxs-lookup"><span data-stu-id="3a862-205">To enable connectivity, you'll have to:</span></span>

- <span data-ttu-id="3a862-206">Configurare la connettività da punto a sito.</span><span class="sxs-lookup"><span data-stu-id="3a862-206">Configure point-to-site connectivity.</span></span>
- <span data-ttu-id="3a862-207">Configurare l'app Web.</span><span class="sxs-lookup"><span data-stu-id="3a862-207">Configure the web app.</span></span>
- <span data-ttu-id="3a862-208">Modificare il gateway di rete locale nell'hub Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="3a862-208">Modify the local network gateway in Azure Stack Hub.</span></span>

### <a name="configure-the-azure-virtual-network-for-point-to-site-connectivity"></a><span data-ttu-id="3a862-209">Configurare la rete virtuale di Azure per la connettività da punto a sito</span><span class="sxs-lookup"><span data-stu-id="3a862-209">Configure the Azure virtual network for point-to-site connectivity</span></span>

<span data-ttu-id="3a862-210">Il gateway di rete virtuale nel lato Azure della rete ibrida deve consentire le connessioni da punto a sito per l'integrazione con app Azure servizio.</span><span class="sxs-lookup"><span data-stu-id="3a862-210">The virtual network gateway in the Azure side of the hybrid network must allow point-to-site connections to integrate with Azure App Service.</span></span>

1. <span data-ttu-id="3a862-211">In Azure passare alla pagina del gateway di rete virtuale.</span><span class="sxs-lookup"><span data-stu-id="3a862-211">In Azure, go to the virtual network gateway page.</span></span> <span data-ttu-id="3a862-212">In **Impostazioni**selezionare **configurazione da punto a sito**.</span><span class="sxs-lookup"><span data-stu-id="3a862-212">Under **Settings**, select **Point-to-site configuration**.</span></span>

    ![Opzione da punto a sito nel gateway di rete virtuale di Azure](media/solution-deployment-guide-hybrid/image8.png)

2. <span data-ttu-id="3a862-214">Selezionare **Configura ora** per configurare la configurazione da punto a sito.</span><span class="sxs-lookup"><span data-stu-id="3a862-214">Select **Configure now** to configure point-to-site.</span></span>

    ![Avviare la configurazione da punto a sito nel gateway di rete virtuale di Azure](media/solution-deployment-guide-hybrid/image9.png)

3. <span data-ttu-id="3a862-216">Nella pagina configurazione **da punto a sito** immettere l'intervallo di indirizzi IP privati che si vuole usare nel **pool di indirizzi**.</span><span class="sxs-lookup"><span data-stu-id="3a862-216">On the **Point-to-site** configuration page, enter the private IP address range that you want to use in **Address pool**.</span></span>

   > [!Note]  
   > <span data-ttu-id="3a862-217">Verificare che l'intervallo specificato non si sovrappongano con gli intervalli di indirizzi già usati dalle subnet nei componenti globali di Azure o hub Azure Stack della rete ibrida.</span><span class="sxs-lookup"><span data-stu-id="3a862-217">Make sure that the range you specify doesn't overlap with any of the address ranges already used by subnets in the global Azure or Azure Stack Hub components of the hybrid network.</span></span>

   <span data-ttu-id="3a862-218">In **tipo di tunnel**deselezionare la **VPN IKEv2**.</span><span class="sxs-lookup"><span data-stu-id="3a862-218">Under **Tunnel Type**, uncheck the **IKEv2 VPN**.</span></span> <span data-ttu-id="3a862-219">Selezionare **Save (Salva** ) per completare la configurazione da punto a sito.</span><span class="sxs-lookup"><span data-stu-id="3a862-219">Select **Save** to finish configuring point-to-site.</span></span>

   ![Impostazioni da punto a sito nel gateway di rete virtuale di Azure](media/solution-deployment-guide-hybrid/image10.png)

### <a name="integrate-the-azure-app-service-app-with-the-hybrid-network"></a><span data-ttu-id="3a862-221">Integrare l'app di servizio app Azure con la rete ibrida</span><span class="sxs-lookup"><span data-stu-id="3a862-221">Integrate the Azure App Service app with the hybrid network</span></span>

1. <span data-ttu-id="3a862-222">Per connettere l'app alla VNet di Azure, seguire le istruzioni in [gateway required VNet Integration](https://docs.microsoft.com/azure/app-service/web-sites-integrate-with-vnet#gateway-required-vnet-integration).</span><span class="sxs-lookup"><span data-stu-id="3a862-222">To connect the app to the Azure VNet, follow the instructions in [Gateway required VNet integration](https://docs.microsoft.com/azure/app-service/web-sites-integrate-with-vnet#gateway-required-vnet-integration).</span></span>

2. <span data-ttu-id="3a862-223">Passare a **Impostazioni** per il piano di servizio app che ospita l'app Web.</span><span class="sxs-lookup"><span data-stu-id="3a862-223">Go to **Settings** for the App Service plan hosting the web app.</span></span> <span data-ttu-id="3a862-224">In **Impostazioni** selezionare **Rete**.</span><span class="sxs-lookup"><span data-stu-id="3a862-224">In **Settings**, select **Networking**.</span></span>

    ![Configurare la rete per il piano di servizio app](media/solution-deployment-guide-hybrid/image11.png)

3. <span data-ttu-id="3a862-226">In **integrazione rete virtuale**selezionare **fare clic qui per gestire**.</span><span class="sxs-lookup"><span data-stu-id="3a862-226">In **VNET Integration**, select **Click here to manage**.</span></span>

    ![Gestire l'integrazione di VNET per il piano di servizio app](media/solution-deployment-guide-hybrid/image12.png)

4. <span data-ttu-id="3a862-228">Selezionare il VNET che si desidera configurare.</span><span class="sxs-lookup"><span data-stu-id="3a862-228">Select the VNET that you want to configure.</span></span> <span data-ttu-id="3a862-229">In **indirizzi IP indirizzati a VNET**, immettere l'intervallo di indirizzi IP per Azure VNET, il Azure stack Hub VNET e gli spazi di indirizzi da punto a sito.</span><span class="sxs-lookup"><span data-stu-id="3a862-229">Under **IP ADDRESSES ROUTED TO VNET**, enter the IP address range for the Azure VNet, the Azure Stack Hub VNet, and the point-to-site address spaces.</span></span> <span data-ttu-id="3a862-230">Selezionare **Save (Salva** ) per convalidare e salvare queste impostazioni.</span><span class="sxs-lookup"><span data-stu-id="3a862-230">Select **Save** to validate and save these settings.</span></span>

    ![Intervalli di indirizzi IP da instradare nell'integrazione della rete virtuale](media/solution-deployment-guide-hybrid/image13.png)

<span data-ttu-id="3a862-232">Per altre informazioni sull'integrazione del servizio app con Azure reti virtuali, vedere [integrare l'app con una rete virtuale di Azure](https://docs.microsoft.com/azure/app-service/web-sites-integrate-with-vnet).</span><span class="sxs-lookup"><span data-stu-id="3a862-232">To learn more about how App Service integrates with Azure VNets, see [Integrate your app with an Azure Virtual Network](https://docs.microsoft.com/azure/app-service/web-sites-integrate-with-vnet).</span></span>

### <a name="configure-the-azure-stack-hub-virtual-network"></a><span data-ttu-id="3a862-233">Configurare la rete virtuale dell'hub Azure Stack</span><span class="sxs-lookup"><span data-stu-id="3a862-233">Configure the Azure Stack Hub virtual network</span></span>

<span data-ttu-id="3a862-234">Il gateway di rete locale nella rete virtuale dell'hub Azure Stack deve essere configurato in modo da instradare il traffico dall'intervallo di indirizzi da punto a sito del servizio app.</span><span class="sxs-lookup"><span data-stu-id="3a862-234">The local network gateway in the Azure Stack Hub virtual network needs to be configured to route traffic from the App Service point-to-site address range.</span></span>

1. <span data-ttu-id="3a862-235">Nell'hub Azure Stack passare a **gateway di rete locale**.</span><span class="sxs-lookup"><span data-stu-id="3a862-235">In Azure Stack Hub, go to **Local network gateway**.</span></span> <span data-ttu-id="3a862-236">In **Impostazioni** selezionare **Configurazione**.</span><span class="sxs-lookup"><span data-stu-id="3a862-236">Under **Settings**, select **Configuration**.</span></span>

    ![Opzione di configurazione del gateway nel gateway di rete locale dell'hub Azure Stack](media/solution-deployment-guide-hybrid/image14.png)

2. <span data-ttu-id="3a862-238">In **spazio di indirizzi**immettere l'intervallo di indirizzi da punto a sito per il gateway di rete virtuale in Azure.</span><span class="sxs-lookup"><span data-stu-id="3a862-238">In **Address space**, enter the point-to-site address range for the virtual network gateway in Azure.</span></span>

    ![Spazio di indirizzi da punto a sito nel gateway di rete locale dell'hub Azure Stack](media/solution-deployment-guide-hybrid/image15.png)

3. <span data-ttu-id="3a862-240">Selezionare **Save (Salva** ) per convalidare e salvare la configurazione.</span><span class="sxs-lookup"><span data-stu-id="3a862-240">Select **Save** to validate and save the configuration.</span></span>

## <a name="configure-dns-for-cross-cloud-scaling"></a><span data-ttu-id="3a862-241">Configurare DNS per la scalabilità tra cloud</span><span class="sxs-lookup"><span data-stu-id="3a862-241">Configure DNS for cross-cloud scaling</span></span>

<span data-ttu-id="3a862-242">Con la configurazione corretta del DNS per le app tra cloud, gli utenti possono accedere alle istanze globali di Azure e Azure Stack Hub dell'app Web.</span><span class="sxs-lookup"><span data-stu-id="3a862-242">By properly configuring DNS for cross-cloud apps, users can access the global Azure and Azure Stack Hub instances of your web app.</span></span> <span data-ttu-id="3a862-243">La configurazione DNS per questa esercitazione consente anche a gestione traffico di Azure di instradare il traffico quando il carico aumenta o diminuisce.</span><span class="sxs-lookup"><span data-stu-id="3a862-243">The DNS configuration for this tutorial also lets Azure Traffic Manager route traffic when the load increases or decreases.</span></span>

<span data-ttu-id="3a862-244">Questa esercitazione USA DNS di Azure per gestire il DNS perché i domini del servizio app non funzionano.</span><span class="sxs-lookup"><span data-stu-id="3a862-244">This tutorial uses Azure DNS to manage the DNS because App Service domains won't work.</span></span>

### <a name="create-subdomains"></a><span data-ttu-id="3a862-245">Creazione di sottodomini</span><span class="sxs-lookup"><span data-stu-id="3a862-245">Create subdomains</span></span>

<span data-ttu-id="3a862-246">Poiché Gestione traffico si basa su CNAME DNS, è necessario un sottodominio per instradare correttamente il traffico agli endpoint.</span><span class="sxs-lookup"><span data-stu-id="3a862-246">Because Traffic Manager relies on DNS CNAMEs, a subdomain is needed to properly route traffic to endpoints.</span></span> <span data-ttu-id="3a862-247">Per altre informazioni sui record DNS e sul mapping del dominio, vedere eseguire il mapping di [domini con gestione traffico](https://docs.microsoft.com/azure/app-service/web-sites-traffic-manager-custom-domain-name).</span><span class="sxs-lookup"><span data-stu-id="3a862-247">For more information about DNS records and domain mapping, see [map domains with Traffic Manager](https://docs.microsoft.com/azure/app-service/web-sites-traffic-manager-custom-domain-name).</span></span>

<span data-ttu-id="3a862-248">Per l'endpoint di Azure verrà creato un sottodominio che gli utenti possono usare per accedere all'app Web.</span><span class="sxs-lookup"><span data-stu-id="3a862-248">For the Azure endpoint, you'll create a subdomain that users can use to access your web app.</span></span> <span data-ttu-id="3a862-249">Per questa esercitazione, può usare **app.Northwind.com**, ma è necessario personalizzare questo valore in base al proprio dominio.</span><span class="sxs-lookup"><span data-stu-id="3a862-249">For this tutorial, can use **app.northwind.com**, but you should customize this value based on your own domain.</span></span>

<span data-ttu-id="3a862-250">Sarà inoltre necessario creare un sottodominio con un record A per l'endpoint dell'hub Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="3a862-250">You'll also need to create a subdomain with an A record for the Azure Stack Hub endpoint.</span></span> <span data-ttu-id="3a862-251">È possibile usare **azurestack.Northwind.com**.</span><span class="sxs-lookup"><span data-stu-id="3a862-251">You can use **azurestack.northwind.com**.</span></span>

### <a name="configure-a-custom-domain-in-azure"></a><span data-ttu-id="3a862-252">Configurare un dominio personalizzato in Azure</span><span class="sxs-lookup"><span data-stu-id="3a862-252">Configure a custom domain in Azure</span></span>

1. <span data-ttu-id="3a862-253">Aggiungere il nome host **app.Northwind.com** all'app Web di Azure eseguendo il [mapping di un record CNAME al servizio app Azure](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).</span><span class="sxs-lookup"><span data-stu-id="3a862-253">Add the **app.northwind.com** hostname to the Azure web app by [mapping a CNAME to Azure App Service](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).</span></span>

### <a name="configure-custom-domains-in-azure-stack-hub"></a><span data-ttu-id="3a862-254">Configurare domini personalizzati nell'hub Azure Stack</span><span class="sxs-lookup"><span data-stu-id="3a862-254">Configure custom domains in Azure Stack Hub</span></span>

1. <span data-ttu-id="3a862-255">Aggiungere il nome host **azurestack.Northwind.com** all'app web Hub Azure stack eseguendo il mapping di un [record a per app Azure servizio](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-domain#map-an-a-record).</span><span class="sxs-lookup"><span data-stu-id="3a862-255">Add the **azurestack.northwind.com** hostname to the Azure Stack Hub web app by [mapping an A record to Azure App Service](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-domain#map-an-a-record).</span></span> <span data-ttu-id="3a862-256">Usare l'indirizzo IP instradabile su Internet per l'app del servizio app.</span><span class="sxs-lookup"><span data-stu-id="3a862-256">Use the internet-routable IP address for the App Service app.</span></span>

2. <span data-ttu-id="3a862-257">Aggiungere il nome host **app.Northwind.com** all'app Web dell'hub Azure stack eseguendo il [mapping di un record CNAME al servizio app Azure](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).</span><span class="sxs-lookup"><span data-stu-id="3a862-257">Add the **app.northwind.com** hostname to the Azure Stack Hub web app by [mapping a CNAME to Azure App Service](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).</span></span> <span data-ttu-id="3a862-258">Usare il nome host configurato nel passaggio precedente (1) come destinazione per il record CNAME.</span><span class="sxs-lookup"><span data-stu-id="3a862-258">Use the hostname you configured in the previous step (1) as the target for the CNAME.</span></span>

## <a name="configure-ssl-certificates-for-cross-cloud-scaling"></a><span data-ttu-id="3a862-259">Configurare i certificati SSL per il ridimensionamento tra cloud</span><span class="sxs-lookup"><span data-stu-id="3a862-259">Configure SSL certificates for cross-cloud scaling</span></span>

<span data-ttu-id="3a862-260">È importante assicurarsi che i dati sensibili raccolti dall'app Web siano protetti in transito e archiviati nel database SQL.</span><span class="sxs-lookup"><span data-stu-id="3a862-260">It's important to ensure sensitive data collected by your web app is secure in transit to and when stored on the SQL database.</span></span>

<span data-ttu-id="3a862-261">Si configureranno le app Web di Azure e Azure Stack hub per usare i certificati SSL per tutto il traffico in ingresso.</span><span class="sxs-lookup"><span data-stu-id="3a862-261">You'll configure your Azure and Azure Stack Hub web apps to use SSL certificates for all incoming traffic.</span></span>

### <a name="add-ssl-to-azure-and-azure-stack-hub"></a><span data-ttu-id="3a862-262">Aggiungere SSL ad Azure e all'hub Azure Stack</span><span class="sxs-lookup"><span data-stu-id="3a862-262">Add SSL to Azure and Azure Stack Hub</span></span>

<span data-ttu-id="3a862-263">Per aggiungere SSL ad Azure:</span><span class="sxs-lookup"><span data-stu-id="3a862-263">To add SSL to Azure:</span></span>

1. <span data-ttu-id="3a862-264">Verificare che il certificato SSL ottenuto sia valido per il sottodominio creato.</span><span class="sxs-lookup"><span data-stu-id="3a862-264">Make sure that the SSL certificate you get is valid for the subdomain you created.</span></span> <span data-ttu-id="3a862-265">(È accettabile usare i certificati con caratteri jolly).</span><span class="sxs-lookup"><span data-stu-id="3a862-265">(It's okay to use wildcard certificates.)</span></span>

2. <span data-ttu-id="3a862-266">In Azure seguire le istruzioni riportate nelle sezioni **preparare l'app Web** e **associare il certificato SSL** dell'articolo [associare un certificato SSL personalizzato esistente ad app Web di Azure](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-ssl) .</span><span class="sxs-lookup"><span data-stu-id="3a862-266">In Azure, follow the instructions in the **Prepare your web app** and **Bind your SSL certificate** sections of the [Bind an existing custom SSL certificate to Azure Web Apps](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-ssl) article.</span></span> <span data-ttu-id="3a862-267">Selezionare **SSL basato su SNI** come **tipo SSL**.</span><span class="sxs-lookup"><span data-stu-id="3a862-267">Select **SNI-based SSL** as the **SSL Type**.</span></span>

3. <span data-ttu-id="3a862-268">Reindirizza tutto il traffico alla porta HTTPS.</span><span class="sxs-lookup"><span data-stu-id="3a862-268">Redirect all traffic to the HTTPS port.</span></span> <span data-ttu-id="3a862-269">Seguire le istruzioni nella sezione **applicare HTTPS** dell'articolo [associare un certificato SSL personalizzato esistente ad app Web di Azure](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-ssl) .</span><span class="sxs-lookup"><span data-stu-id="3a862-269">Follow the instructions in the   **Enforce HTTPS** section of the [Bind an existing custom SSL certificate to Azure Web Apps](https://docs.microsoft.com/Azure/app-service/app-service-web-tutorial-custom-ssl) article.</span></span>

<span data-ttu-id="3a862-270">Per aggiungere SSL a Azure Stack Hub:</span><span class="sxs-lookup"><span data-stu-id="3a862-270">To add SSL to Azure Stack Hub:</span></span>

1. <span data-ttu-id="3a862-271">Ripetere i passaggi 1-3 usati per Azure.</span><span class="sxs-lookup"><span data-stu-id="3a862-271">Repeat steps 1-3 that you used for Azure.</span></span>

## <a name="configure-and-deploy-the-web-app"></a><span data-ttu-id="3a862-272">Configurare e distribuire l'app Web</span><span class="sxs-lookup"><span data-stu-id="3a862-272">Configure and deploy the web app</span></span>

<span data-ttu-id="3a862-273">Il codice dell'app verrà configurato per segnalare i dati di telemetria all'istanza corretta di Application Insights e configurare le app Web con le stringhe di connessione corrette.</span><span class="sxs-lookup"><span data-stu-id="3a862-273">You'll configure the app code to report telemetry to the correct Application Insights instance and configure the web apps with the right connection strings.</span></span> <span data-ttu-id="3a862-274">Per ulteriori informazioni su Application Insights, vedere [che cos'è Application Insights?](https://docs.microsoft.com/azure/application-insights/app-insights-overview)</span><span class="sxs-lookup"><span data-stu-id="3a862-274">To learn more about Application Insights, see [What is Application Insights?](https://docs.microsoft.com/azure/application-insights/app-insights-overview)</span></span>

### <a name="add-application-insights"></a><span data-ttu-id="3a862-275">Aggiungi Application Insights</span><span class="sxs-lookup"><span data-stu-id="3a862-275">Add Application Insights</span></span>

1. <span data-ttu-id="3a862-276">Aprire l'app Web in Microsoft Visual Studio.</span><span class="sxs-lookup"><span data-stu-id="3a862-276">Open your web app in Microsoft Visual Studio.</span></span>

2. <span data-ttu-id="3a862-277">[Aggiungere Application Insights](https://docs.microsoft.com/azure/azure-monitor/app/asp-net-core#enable-client-side-telemetry-for-web-applications) al progetto per trasmettere i dati di telemetria usati da Application Insights per creare avvisi quando il traffico Web aumenta o diminuisce.</span><span class="sxs-lookup"><span data-stu-id="3a862-277">[Add Application Insights](https://docs.microsoft.com/azure/azure-monitor/app/asp-net-core#enable-client-side-telemetry-for-web-applications) to your project to transmit the telemetry that Application Insights uses to create alerts when web traffic increases or decreases.</span></span>

### <a name="configure-dynamic-connection-strings"></a><span data-ttu-id="3a862-278">Configurare le stringhe di connessione dinamiche</span><span class="sxs-lookup"><span data-stu-id="3a862-278">Configure dynamic connection strings</span></span>

<span data-ttu-id="3a862-279">Ogni istanza dell'app Web userà un metodo diverso per la connessione al database SQL.</span><span class="sxs-lookup"><span data-stu-id="3a862-279">Each instance of the web app will use a different method to connect to the SQL database.</span></span> <span data-ttu-id="3a862-280">L'app in Azure usa l'indirizzo IP privato della macchina virtuale SQL Server e l'app nell'hub Azure Stack usa l'indirizzo IP pubblico della macchina virtuale SQL Server.</span><span class="sxs-lookup"><span data-stu-id="3a862-280">The app in Azure uses the private IP address of the SQL Server VM and the app in Azure Stack Hub uses the public IP address of the SQL Server VM.</span></span>

> [!Note]  
> <span data-ttu-id="3a862-281">In un sistema integrato Azure Stack Hub, l'indirizzo IP pubblico non deve essere instradabile su Internet.</span><span class="sxs-lookup"><span data-stu-id="3a862-281">On an Azure Stack Hub integrated system, the public IP address shouldn't be internet-routable.</span></span> <span data-ttu-id="3a862-282">In un Gabriele, l'indirizzo IP pubblico non è instradabile all'esterno di Gabriele.</span><span class="sxs-lookup"><span data-stu-id="3a862-282">On an ASDK, the public IP address isn't routable outside the ASDK.</span></span>

<span data-ttu-id="3a862-283">È possibile usare le variabili di ambiente del servizio app per passare una stringa di connessione diversa a ogni istanza dell'app.</span><span class="sxs-lookup"><span data-stu-id="3a862-283">You can use App Service environment variables to pass a different connection string to each instance of the app.</span></span>

1. <span data-ttu-id="3a862-284">Aprire l'app in Visual Studio.</span><span class="sxs-lookup"><span data-stu-id="3a862-284">Open the app in Visual Studio.</span></span>

2. <span data-ttu-id="3a862-285">Aprire Startup.cs e individuare il blocco di codice seguente:</span><span class="sxs-lookup"><span data-stu-id="3a862-285">Open Startup.cs and find the following code block:</span></span>

    ```C#
    services.AddDbContext<MyDatabaseContext>(options =>
        options.UseSqlite("Data Source=localdatabase.db"));
    ```

3. <span data-ttu-id="3a862-286">Sostituire il blocco di codice precedente con il codice seguente, che usa una stringa di connessione definita nella *appsettings.jssu* file:</span><span class="sxs-lookup"><span data-stu-id="3a862-286">Replace the previous code block with the following code, which uses a connection string defined in the *appsettings.json* file:</span></span>

    ```C#
    services.AddDbContext<MyDatabaseContext>(options =>
        options.UseSqlServer(Configuration.GetConnectionString("MyDbConnection")));
     // Automatically perform database migration
     services.BuildServiceProvider().GetService<MyDatabaseContext>().Database.Migrate();
    ```

### <a name="configure-app-service-app-settings"></a><span data-ttu-id="3a862-287">Configurare le impostazioni dell'app del servizio app</span><span class="sxs-lookup"><span data-stu-id="3a862-287">Configure App Service app settings</span></span>

1. <span data-ttu-id="3a862-288">Creare stringhe di connessione per Azure e hub Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="3a862-288">Create connection strings for Azure and Azure Stack Hub.</span></span> <span data-ttu-id="3a862-289">Le stringhe devono essere uguali, ad eccezione degli indirizzi IP utilizzati.</span><span class="sxs-lookup"><span data-stu-id="3a862-289">The strings should be the same, except for the IP addresses that are used.</span></span>

2. <span data-ttu-id="3a862-290">In Azure e Azure Stack Hub aggiungere la stringa di connessione appropriata [come impostazione dell'app](https://docs.microsoft.com/azure/app-service/web-sites-configure) nell'app Web, usando `SQLCONNSTR\_` come prefisso nel nome.</span><span class="sxs-lookup"><span data-stu-id="3a862-290">In Azure and Azure Stack Hub, add the appropriate connection string [as an app setting](https://docs.microsoft.com/azure/app-service/web-sites-configure) in the web app, using `SQLCONNSTR\_` as a prefix in the name.</span></span>

3. <span data-ttu-id="3a862-291">**Salvare** le impostazioni dell'app Web e riavviare l'app.</span><span class="sxs-lookup"><span data-stu-id="3a862-291">**Save** the web app settings and restart the app.</span></span>

## <a name="enable-automatic-scaling-in-global-azure"></a><span data-ttu-id="3a862-292">Abilitare la scalabilità automatica in Azure globale</span><span class="sxs-lookup"><span data-stu-id="3a862-292">Enable automatic scaling in global Azure</span></span>

<span data-ttu-id="3a862-293">Quando si crea l'app Web in un ambiente del servizio app, questa inizia con un'istanza.</span><span class="sxs-lookup"><span data-stu-id="3a862-293">When you create your web app in an App Service environment, it starts with one instance.</span></span> <span data-ttu-id="3a862-294">È possibile scalare automaticamente per aggiungere istanze per fornire più risorse di calcolo per l'app.</span><span class="sxs-lookup"><span data-stu-id="3a862-294">You can automatically scale out to add instances to provide more compute resources for your app.</span></span> <span data-ttu-id="3a862-295">Analogamente, è possibile ridimensionare automaticamente e ridurre il numero di istanze necessarie per l'app.</span><span class="sxs-lookup"><span data-stu-id="3a862-295">Similarly, you can automatically scale in and reduce the number of instances your app needs.</span></span>

> [!Note]  
> <span data-ttu-id="3a862-296">È necessario disporre di un piano di servizio app per configurare la scalabilità orizzontale e il ridimensionamento.</span><span class="sxs-lookup"><span data-stu-id="3a862-296">You need to have an App Service plan to configure scale out and scale in.</span></span> <span data-ttu-id="3a862-297">Se non si dispone di un piano, crearne uno prima di iniziare i passaggi successivi.</span><span class="sxs-lookup"><span data-stu-id="3a862-297">If you don't have a plan, create one before starting the next steps.</span></span>

### <a name="enable-automatic-scale-out"></a><span data-ttu-id="3a862-298">Abilitare la scalabilità orizzontale automatica</span><span class="sxs-lookup"><span data-stu-id="3a862-298">Enable automatic scale-out</span></span>

1. <span data-ttu-id="3a862-299">In Azure trovare il piano di servizio app per i siti da scalare in orizzontale e quindi selezionare **scale-out (piano di servizio app)**.</span><span class="sxs-lookup"><span data-stu-id="3a862-299">In Azure, find the App Service plan for the sites you want to scale out, and then select **Scale-out (App Service plan)**.</span></span>

    ![Servizio app Azure di scalabilità orizzontale](media/solution-deployment-guide-hybrid/image16.png)

2. <span data-ttu-id="3a862-301">Selezionare **Abilita ridimensionamento**automatico.</span><span class="sxs-lookup"><span data-stu-id="3a862-301">Select **Enable autoscale**.</span></span>

    ![Abilitare la scalabilità automatica nel servizio app Azure](media/solution-deployment-guide-hybrid/image17.png)

3. <span data-ttu-id="3a862-303">Immettere un nome per il **nome dell'impostazione di scalabilità**automatica.</span><span class="sxs-lookup"><span data-stu-id="3a862-303">Enter a name for **Autoscale Setting Name**.</span></span> <span data-ttu-id="3a862-304">Per la regola di scalabilità automatica **predefinita** , selezionare **Ridimensiona in base a una metrica**.</span><span class="sxs-lookup"><span data-stu-id="3a862-304">For the **Default** auto scale rule, select **Scale based on a metric**.</span></span> <span data-ttu-id="3a862-305">Impostare i **limiti dell'istanza** su **minimo: 1**, **massimo: 10**e **valore predefinito: 1**.</span><span class="sxs-lookup"><span data-stu-id="3a862-305">Set the **Instance limits** to **Minimum: 1**, **Maximum: 10**, and **Default: 1**.</span></span>

    ![Configurare la scalabilità automatica nel servizio app Azure](media/solution-deployment-guide-hybrid/image18.png)

4. <span data-ttu-id="3a862-307">Selezionare **+ Aggiungi una regola**.</span><span class="sxs-lookup"><span data-stu-id="3a862-307">Select **+Add a rule**.</span></span>

5. <span data-ttu-id="3a862-308">In **origine metrica**selezionare **risorsa corrente**.</span><span class="sxs-lookup"><span data-stu-id="3a862-308">In **Metric Source**, select **Current Resource**.</span></span> <span data-ttu-id="3a862-309">Usare i criteri e le azioni seguenti per la regola.</span><span class="sxs-lookup"><span data-stu-id="3a862-309">Use the following Criteria and Actions for the rule.</span></span>

#### <a name="criteria"></a><span data-ttu-id="3a862-310">Criteri</span><span class="sxs-lookup"><span data-stu-id="3a862-310">Criteria</span></span>

1. <span data-ttu-id="3a862-311">In **aggregazione temporale** selezionare **media**.</span><span class="sxs-lookup"><span data-stu-id="3a862-311">Under **Time Aggregation,** select **Average**.</span></span>

2. <span data-ttu-id="3a862-312">In **nome metrica**selezionare **percentuale CPU**.</span><span class="sxs-lookup"><span data-stu-id="3a862-312">Under **Metric Name**, select **CPU Percentage**.</span></span>

3. <span data-ttu-id="3a862-313">In **operatore**selezionare **maggiore di**.</span><span class="sxs-lookup"><span data-stu-id="3a862-313">Under **Operator**, select **Greater than**.</span></span>

   - <span data-ttu-id="3a862-314">Impostare la **soglia** su **50**.</span><span class="sxs-lookup"><span data-stu-id="3a862-314">Set the **Threshold** to **50**.</span></span>
   - <span data-ttu-id="3a862-315">Impostare la **durata** su **10**.</span><span class="sxs-lookup"><span data-stu-id="3a862-315">Set the **Duration** to **10**.</span></span>

#### <a name="action"></a><span data-ttu-id="3a862-316">Azione</span><span class="sxs-lookup"><span data-stu-id="3a862-316">Action</span></span>

1. <span data-ttu-id="3a862-317">In **operazione**selezionare **aumenta numero per**.</span><span class="sxs-lookup"><span data-stu-id="3a862-317">Under **Operation**, select **Increase Count by**.</span></span>

2. <span data-ttu-id="3a862-318">Impostare il **numero di istanze** su **2**.</span><span class="sxs-lookup"><span data-stu-id="3a862-318">Set the **Instance Count** to **2**.</span></span>

3. <span data-ttu-id="3a862-319">Impostare il livello di **raffreddamento** su **5**.</span><span class="sxs-lookup"><span data-stu-id="3a862-319">Set the **Cool down** to **5**.</span></span>

4. <span data-ttu-id="3a862-320">Selezionare **Aggiungi**.</span><span class="sxs-lookup"><span data-stu-id="3a862-320">Select **Add**.</span></span>

5. <span data-ttu-id="3a862-321">Selezionare **+ Aggiungi una regola**.</span><span class="sxs-lookup"><span data-stu-id="3a862-321">Select the **+ Add a rule**.</span></span>

6. <span data-ttu-id="3a862-322">In **origine metrica**selezionare **risorsa corrente.**</span><span class="sxs-lookup"><span data-stu-id="3a862-322">In **Metric Source**, select **Current Resource.**</span></span>

   > [!Note]  
   > <span data-ttu-id="3a862-323">La risorsa corrente conterrà il nome/GUID del piano di servizio app e gli elenchi a discesa **tipo di risorsa** e **risorsa** non saranno disponibili.</span><span class="sxs-lookup"><span data-stu-id="3a862-323">The current resource will contain your App Service plan's name/GUID and the **Resource Type** and **Resource** drop-down lists will be unavailable.</span></span>

### <a name="enable-automatic-scale-in"></a><span data-ttu-id="3a862-324">Abilitare la scalabilità automatica</span><span class="sxs-lookup"><span data-stu-id="3a862-324">Enable automatic scale in</span></span>

<span data-ttu-id="3a862-325">Quando il traffico diminuisce, l'app Web di Azure può ridurre automaticamente il numero di istanze attive per ridurre i costi.</span><span class="sxs-lookup"><span data-stu-id="3a862-325">When traffic decreases, the Azure web app can automatically reduce the number of active instances to reduce costs.</span></span> <span data-ttu-id="3a862-326">Questa azione è meno aggressiva rispetto alla scalabilità orizzontale e riduce al minimo l'effetto sugli utenti dell'app.</span><span class="sxs-lookup"><span data-stu-id="3a862-326">This action is less aggressive than scale-out and minimizes the impact on app users.</span></span>

1. <span data-ttu-id="3a862-327">Passare alla condizione di scalabilità orizzontale **predefinita** , quindi selezionare **+ Aggiungi una regola**.</span><span class="sxs-lookup"><span data-stu-id="3a862-327">Go to the **Default** scale out condition, then select **+ Add a rule**.</span></span> <span data-ttu-id="3a862-328">Usare i criteri e le azioni seguenti per la regola.</span><span class="sxs-lookup"><span data-stu-id="3a862-328">Use the following Criteria and Actions for the rule.</span></span>

#### <a name="criteria"></a><span data-ttu-id="3a862-329">Criteri</span><span class="sxs-lookup"><span data-stu-id="3a862-329">Criteria</span></span>

1. <span data-ttu-id="3a862-330">In **aggregazione temporale** selezionare **media**.</span><span class="sxs-lookup"><span data-stu-id="3a862-330">Under **Time Aggregation,** select **Average**.</span></span>

2. <span data-ttu-id="3a862-331">In **nome metrica**selezionare **percentuale CPU**.</span><span class="sxs-lookup"><span data-stu-id="3a862-331">Under **Metric Name**, select **CPU Percentage**.</span></span>

3. <span data-ttu-id="3a862-332">In **operatore**selezionare **minore di**.</span><span class="sxs-lookup"><span data-stu-id="3a862-332">Under **Operator**, select **Less than**.</span></span>

   - <span data-ttu-id="3a862-333">Impostare la **soglia** su **30**.</span><span class="sxs-lookup"><span data-stu-id="3a862-333">Set the **Threshold** to **30**.</span></span>
   - <span data-ttu-id="3a862-334">Impostare la **durata** su **10**.</span><span class="sxs-lookup"><span data-stu-id="3a862-334">Set the **Duration** to **10**.</span></span>

#### <a name="action"></a><span data-ttu-id="3a862-335">Azione</span><span class="sxs-lookup"><span data-stu-id="3a862-335">Action</span></span>

1. <span data-ttu-id="3a862-336">In **operazione**selezionare **Diminuisci conteggio per**.</span><span class="sxs-lookup"><span data-stu-id="3a862-336">Under **Operation**, select **Decrease Count by**.</span></span>

   - <span data-ttu-id="3a862-337">Impostare il **numero di istanze** su **1**.</span><span class="sxs-lookup"><span data-stu-id="3a862-337">Set the **Instance Count** to **1**.</span></span>
   - <span data-ttu-id="3a862-338">Impostare il livello di **raffreddamento** su **5**.</span><span class="sxs-lookup"><span data-stu-id="3a862-338">Set the **Cool down** to **5**.</span></span>

2. <span data-ttu-id="3a862-339">Selezionare **Aggiungi**.</span><span class="sxs-lookup"><span data-stu-id="3a862-339">Select **Add**.</span></span>

## <a name="create-a-traffic-manager-profile-and-configure-cross-cloud-scaling"></a><span data-ttu-id="3a862-340">Creare un profilo di gestione traffico e configurare la scalabilità tra cloud</span><span class="sxs-lookup"><span data-stu-id="3a862-340">Create a Traffic Manager profile and configure cross-cloud scaling</span></span>

<span data-ttu-id="3a862-341">Creare un profilo di gestione traffico in Azure e quindi configurare gli endpoint per abilitare il ridimensionamento tra cloud.</span><span class="sxs-lookup"><span data-stu-id="3a862-341">Create a Traffic Manager profile in Azure and then configure endpoints to enable cross-cloud scaling.</span></span>

### <a name="create-traffic-manager-profile"></a><span data-ttu-id="3a862-342">Crea profilo di gestione traffico</span><span class="sxs-lookup"><span data-stu-id="3a862-342">Create Traffic Manager profile</span></span>

1. <span data-ttu-id="3a862-343">Selezionare **Crea una risorsa**.</span><span class="sxs-lookup"><span data-stu-id="3a862-343">Select **Create a resource**.</span></span>
2. <span data-ttu-id="3a862-344">Selezionare **rete**.</span><span class="sxs-lookup"><span data-stu-id="3a862-344">Select **Networking**.</span></span>
3. <span data-ttu-id="3a862-345">Selezionare **profilo di gestione traffico** e configurare le impostazioni seguenti:</span><span class="sxs-lookup"><span data-stu-id="3a862-345">Select **Traffic Manager profile** and configure the following settings:</span></span>

   - <span data-ttu-id="3a862-346">In **nome**immettere un nome per il profilo.</span><span class="sxs-lookup"><span data-stu-id="3a862-346">In **Name**, enter a name for your profile.</span></span> <span data-ttu-id="3a862-347">Questo nome **deve** essere univoco nella zona trafficmanager.NET e viene usato per creare un nuovo nome DNS (ad esempio, northwindstore.trafficmanager.NET).</span><span class="sxs-lookup"><span data-stu-id="3a862-347">This name **must** be unique in the trafficmanager.net zone and is used to create a new DNS name (for example, northwindstore.trafficmanager.net).</span></span>
   - <span data-ttu-id="3a862-348">Per **metodo di routing**selezionare il valore **ponderato**.</span><span class="sxs-lookup"><span data-stu-id="3a862-348">For **Routing method**, select the **Weighted**.</span></span>
   - <span data-ttu-id="3a862-349">Per **Subscription (sottoscrizione**) selezionare la sottoscrizione in cui si vuole creare il profilo.</span><span class="sxs-lookup"><span data-stu-id="3a862-349">For **Subscription**, select the subscription you want to create  this profile in.</span></span>
   - <span data-ttu-id="3a862-350">In **gruppo di risorse**creare un nuovo gruppo di risorse per questo profilo.</span><span class="sxs-lookup"><span data-stu-id="3a862-350">In **Resource Group**, create a new resource group for this profile.</span></span>
   - <span data-ttu-id="3a862-351">In **Località del gruppo di risorse** selezionare la località del gruppo di risorse.</span><span class="sxs-lookup"><span data-stu-id="3a862-351">In **Resource group location**, select the location of the resource group.</span></span> <span data-ttu-id="3a862-352">Questa impostazione si riferisce al percorso del gruppo di risorse e non ha alcun effetto sul profilo di gestione traffico distribuito a livello globale.</span><span class="sxs-lookup"><span data-stu-id="3a862-352">This setting refers to the location of the resource group and has no impact on the Traffic Manager profile that's deployed globally.</span></span>

4. <span data-ttu-id="3a862-353">Selezionare **Crea**.</span><span class="sxs-lookup"><span data-stu-id="3a862-353">Select **Create**.</span></span>

    ![Crea profilo di gestione traffico](media/solution-deployment-guide-hybrid/image19.png)

   <span data-ttu-id="3a862-355">Quando la distribuzione globale del profilo di gestione traffico è completa, viene visualizzata nell'elenco di risorse per il gruppo di risorse in cui è stato creato.</span><span class="sxs-lookup"><span data-stu-id="3a862-355">When the global deployment of your Traffic Manager profile is complete, it's shown in the list of resources for the resource group you created it under.</span></span>

### <a name="add-traffic-manager-endpoints"></a><span data-ttu-id="3a862-356">Aggiungere endpoint di Gestione traffico</span><span class="sxs-lookup"><span data-stu-id="3a862-356">Add Traffic Manager endpoints</span></span>

1. <span data-ttu-id="3a862-357">Cercare il profilo di gestione traffico creato.</span><span class="sxs-lookup"><span data-stu-id="3a862-357">Search for the Traffic Manager profile you created.</span></span> <span data-ttu-id="3a862-358">Se si è passati al gruppo di risorse per il profilo, selezionare il profilo.</span><span class="sxs-lookup"><span data-stu-id="3a862-358">If you navigated to the resource group for the profile, select the profile.</span></span>

2. <span data-ttu-id="3a862-359">In **profilo di gestione traffico**selezionare **endpoint**in **Impostazioni**.</span><span class="sxs-lookup"><span data-stu-id="3a862-359">In **Traffic Manager profile**, under **SETTINGS**, select **Endpoints**.</span></span>

3. <span data-ttu-id="3a862-360">Selezionare **Aggiungi**.</span><span class="sxs-lookup"><span data-stu-id="3a862-360">Select **Add**.</span></span>

4. <span data-ttu-id="3a862-361">In **Aggiungi endpoint**usare le impostazioni seguenti per Azure stack Hub:</span><span class="sxs-lookup"><span data-stu-id="3a862-361">In **Add endpoint**, use the following settings for Azure Stack Hub:</span></span>

   - <span data-ttu-id="3a862-362">Per **tipo**selezionare **endpoint esterno**.</span><span class="sxs-lookup"><span data-stu-id="3a862-362">For **Type**, select **External endpoint**.</span></span>
   - <span data-ttu-id="3a862-363">Immettere un **nome** per l'endpoint.</span><span class="sxs-lookup"><span data-stu-id="3a862-363">Enter a **Name** for the endpoint.</span></span>
   - <span data-ttu-id="3a862-364">Per **nome di dominio completo (FQDN) o IP**, immettere l'URL esterno per l'app Web dell'hub Azure stack.</span><span class="sxs-lookup"><span data-stu-id="3a862-364">For **Fully qualified domain name (FQDN) or IP**, enter the external URL for your Azure Stack Hub web app.</span></span>
   - <span data-ttu-id="3a862-365">Per **peso**, Mantieni il valore predefinito **1**.</span><span class="sxs-lookup"><span data-stu-id="3a862-365">For **Weight**, keep the default, **1**.</span></span> <span data-ttu-id="3a862-366">Questo peso comporta che tutto il traffico passa a questo endpoint se è integro.</span><span class="sxs-lookup"><span data-stu-id="3a862-366">This weight results in all traffic going to this endpoint if it's healthy.</span></span>
   - <span data-ttu-id="3a862-367">Lasciare deselezionata l'opzione **Aggiungi come disabilitata** .</span><span class="sxs-lookup"><span data-stu-id="3a862-367">Leave **Add as disabled** unchecked.</span></span>

5. <span data-ttu-id="3a862-368">Selezionare **OK** per salvare l'endpoint dell'hub Azure stack.</span><span class="sxs-lookup"><span data-stu-id="3a862-368">Select **OK** to save the Azure Stack Hub endpoint.</span></span>

<span data-ttu-id="3a862-369">L'endpoint di Azure verrà configurato successivamente.</span><span class="sxs-lookup"><span data-stu-id="3a862-369">You'll configure the Azure endpoint next.</span></span>

1. <span data-ttu-id="3a862-370">In **profilo di gestione traffico**selezionare **endpoint**.</span><span class="sxs-lookup"><span data-stu-id="3a862-370">On **Traffic Manager profile**, select **Endpoints**.</span></span>
2. <span data-ttu-id="3a862-371">Selezionare **+ Aggiungi**.</span><span class="sxs-lookup"><span data-stu-id="3a862-371">Select **+Add**.</span></span>
3. <span data-ttu-id="3a862-372">In **Aggiungi endpoint**usare le impostazioni seguenti per Azure:</span><span class="sxs-lookup"><span data-stu-id="3a862-372">On **Add endpoint**, use the following settings for Azure:</span></span>

   - <span data-ttu-id="3a862-373">Per **tipo**selezionare **endpoint di Azure**.</span><span class="sxs-lookup"><span data-stu-id="3a862-373">For **Type**, select **Azure endpoint**.</span></span>
   - <span data-ttu-id="3a862-374">Immettere un **nome** per l'endpoint.</span><span class="sxs-lookup"><span data-stu-id="3a862-374">Enter a **Name** for the endpoint.</span></span>
   - <span data-ttu-id="3a862-375">Per **tipo di risorsa di destinazione**selezionare **servizio app**.</span><span class="sxs-lookup"><span data-stu-id="3a862-375">For **Target resource type**, select **App Service**.</span></span>
   - <span data-ttu-id="3a862-376">Per **risorsa di destinazione**selezionare **Scegli un servizio app** per visualizzare un elenco di app Web nella stessa sottoscrizione.</span><span class="sxs-lookup"><span data-stu-id="3a862-376">For **Target resource**, select **Choose an app service** to see a list of Web Apps in the same subscription.</span></span>
   - <span data-ttu-id="3a862-377">In **Risorsa** selezionare il servizio App che si vuole aggiungere come primo endpoint.</span><span class="sxs-lookup"><span data-stu-id="3a862-377">In **Resource**, pick the App service that you want to add as the first endpoint.</span></span>
   - <span data-ttu-id="3a862-378">Per **peso**selezionare **2**.</span><span class="sxs-lookup"><span data-stu-id="3a862-378">For **Weight**, select **2**.</span></span> <span data-ttu-id="3a862-379">Questa impostazione determina l'indirizzamento del traffico a questo endpoint se l'endpoint primario non è integro o se si dispone di una regola/avviso che reindirizza il traffico quando viene attivato.</span><span class="sxs-lookup"><span data-stu-id="3a862-379">This setting results in all traffic going to this endpoint if the primary endpoint is unhealthy, or if you have a rule/alert that redirects traffic when triggered.</span></span>
   - <span data-ttu-id="3a862-380">Lasciare deselezionata l'opzione **Aggiungi come disabilitata** .</span><span class="sxs-lookup"><span data-stu-id="3a862-380">Leave **Add as disabled** unchecked.</span></span>

4. <span data-ttu-id="3a862-381">Selezionare **OK** per salvare l'endpoint di Azure.</span><span class="sxs-lookup"><span data-stu-id="3a862-381">Select **OK** to save the Azure endpoint.</span></span>

<span data-ttu-id="3a862-382">Una volta configurati entrambi gli endpoint, questi vengono elencati in **profilo di gestione traffico** quando si selezionano gli **endpoint**.</span><span class="sxs-lookup"><span data-stu-id="3a862-382">After both endpoints are configured, they're listed in **Traffic Manager profile** when you select **Endpoints**.</span></span> <span data-ttu-id="3a862-383">Nell'esempio riportato di seguito vengono illustrati due endpoint, con informazioni sullo stato e sulla configurazione per ciascuno di essi.</span><span class="sxs-lookup"><span data-stu-id="3a862-383">The example in the following screen capture shows two endpoints, with status and configuration information for each one.</span></span>

![Endpoint nel profilo di gestione traffico](media/solution-deployment-guide-hybrid/image20.png)

## <a name="set-up-application-insights-monitoring-and-alerting"></a><span data-ttu-id="3a862-385">Configurare Application Insights monitoraggio e avvisi</span><span class="sxs-lookup"><span data-stu-id="3a862-385">Set up Application Insights monitoring and alerting</span></span>

<span data-ttu-id="3a862-386">Applicazione Azure Insights consente di monitorare l'app e inviare avvisi in base alle condizioni configurate.</span><span class="sxs-lookup"><span data-stu-id="3a862-386">Azure Application Insights lets you monitor your app and send alerts based on conditions you configure.</span></span> <span data-ttu-id="3a862-387">Alcuni esempi sono: l'app non è disponibile, sta riscontrando errori o presenta problemi di prestazioni.</span><span class="sxs-lookup"><span data-stu-id="3a862-387">Some examples are: the app is unavailable, is experiencing failures, or is showing performance issues.</span></span>

<span data-ttu-id="3a862-388">Per creare avvisi si useranno le metriche di Application Insights.</span><span class="sxs-lookup"><span data-stu-id="3a862-388">You'll use Application Insights metrics to create alerts.</span></span> <span data-ttu-id="3a862-389">Quando si attivano questi avvisi, l'istanza dell'app Web passa automaticamente da Azure Stack Hub ad Azure per la scalabilità orizzontale e quindi torna all'hub Azure Stack per la scalabilità in.</span><span class="sxs-lookup"><span data-stu-id="3a862-389">When these alerts trigger, your web app's instance will automatically switch from Azure Stack Hub to Azure to scale out, and then back to Azure Stack Hub to scale in.</span></span>

### <a name="create-an-alert-from-metrics"></a><span data-ttu-id="3a862-390">Creare un avviso dalle metriche</span><span class="sxs-lookup"><span data-stu-id="3a862-390">Create an alert from metrics</span></span>

<span data-ttu-id="3a862-391">Andare al gruppo di risorse per questa esercitazione, quindi selezionare l'istanza di Application Insights per aprire **Application Insights**.</span><span class="sxs-lookup"><span data-stu-id="3a862-391">Go to the resource group for this tutorial and then select the Application Insights instance to open **Application Insights**.</span></span>

![Application Insights](media/solution-deployment-guide-hybrid/image21.png)

<span data-ttu-id="3a862-393">Questa visualizzazione verrà utilizzata per creare un avviso di scalabilità orizzontale e un avviso di riduzione delle prestazioni.</span><span class="sxs-lookup"><span data-stu-id="3a862-393">You'll use this view to create a scale-out alert and a scale-in alert.</span></span>

### <a name="create-the-scale-out-alert"></a><span data-ttu-id="3a862-394">Creare l'avviso di scalabilità orizzontale</span><span class="sxs-lookup"><span data-stu-id="3a862-394">Create the scale-out alert</span></span>

1. <span data-ttu-id="3a862-395">In **Configura**selezionare **avvisi (versione classica)**.</span><span class="sxs-lookup"><span data-stu-id="3a862-395">Under **CONFIGURE**, select **Alerts (classic)**.</span></span>
2. <span data-ttu-id="3a862-396">Selezionare **Aggiungi avviso per la metrica (versione classica)**.</span><span class="sxs-lookup"><span data-stu-id="3a862-396">Select **Add metric alert (classic)**.</span></span>
3. <span data-ttu-id="3a862-397">In **Aggiungi regola**configurare le impostazioni seguenti:</span><span class="sxs-lookup"><span data-stu-id="3a862-397">In **Add rule**, configure the following settings:</span></span>

   - <span data-ttu-id="3a862-398">Per **nome**immettere espansione **nel cloud di Azure**.</span><span class="sxs-lookup"><span data-stu-id="3a862-398">For **Name**, enter **Burst into Azure Cloud**.</span></span>
   - <span data-ttu-id="3a862-399">Una **Descrizione** è facoltativa.</span><span class="sxs-lookup"><span data-stu-id="3a862-399">A **Description** is optional.</span></span>
   - <span data-ttu-id="3a862-400">In **origine**  >  **avviso su**selezionare **metrica**.</span><span class="sxs-lookup"><span data-stu-id="3a862-400">Under **Source** > **Alert on**, select **Metrics**.</span></span>
   - <span data-ttu-id="3a862-401">In **criteri**selezionare la sottoscrizione, il gruppo di risorse per il profilo di gestione traffico e il nome del profilo di gestione traffico per la risorsa.</span><span class="sxs-lookup"><span data-stu-id="3a862-401">Under **Criteria**, select your subscription, the resource group for your Traffic Manager profile, and the name of the Traffic Manager profile for the resource.</span></span>

4. <span data-ttu-id="3a862-402">Per **metrica**selezionare **frequenza richieste**.</span><span class="sxs-lookup"><span data-stu-id="3a862-402">For **Metric**, select **Request Rate**.</span></span>
5. <span data-ttu-id="3a862-403">Per **condizione**selezionare **maggiore di**.</span><span class="sxs-lookup"><span data-stu-id="3a862-403">For **Condition**, select **Greater than**.</span></span>
6. <span data-ttu-id="3a862-404">In **soglia**immettere **2**.</span><span class="sxs-lookup"><span data-stu-id="3a862-404">For **Threshold**, enter **2**.</span></span>
7. <span data-ttu-id="3a862-405">Per **periodo**selezionare **negli ultimi 5 minuti**.</span><span class="sxs-lookup"><span data-stu-id="3a862-405">For **Period**, select **Over the last 5 minutes**.</span></span>
8. <span data-ttu-id="3a862-406">In **notifica tramite**:</span><span class="sxs-lookup"><span data-stu-id="3a862-406">Under **Notify via**:</span></span>
   - <span data-ttu-id="3a862-407">Selezionare la casella di controllo per **inviare messaggi di posta elettronica a proprietari, collaboratori e lettori**.</span><span class="sxs-lookup"><span data-stu-id="3a862-407">Check the checkbox for **Email owners, contributors, and readers**.</span></span>
   - <span data-ttu-id="3a862-408">Immettere l'indirizzo di posta elettronica per altri indirizzi di **posta elettronica dell'amministratore**.</span><span class="sxs-lookup"><span data-stu-id="3a862-408">Enter your email address for **Additional administrator email(s)**.</span></span>

9. <span data-ttu-id="3a862-409">Nella barra dei menu selezionare **Salva**.</span><span class="sxs-lookup"><span data-stu-id="3a862-409">On the menu bar, select **Save**.</span></span>

### <a name="create-the-scale-in-alert"></a><span data-ttu-id="3a862-410">Creare l'avviso di riduzione del livello</span><span class="sxs-lookup"><span data-stu-id="3a862-410">Create the scale-in alert</span></span>

1. <span data-ttu-id="3a862-411">In **Configura**selezionare **avvisi (versione classica)**.</span><span class="sxs-lookup"><span data-stu-id="3a862-411">Under **CONFIGURE**, select **Alerts (classic)**.</span></span>
2. <span data-ttu-id="3a862-412">Selezionare **Aggiungi avviso per la metrica (versione classica)**.</span><span class="sxs-lookup"><span data-stu-id="3a862-412">Select **Add metric alert (classic)**.</span></span>
3. <span data-ttu-id="3a862-413">In **Aggiungi regola**configurare le impostazioni seguenti:</span><span class="sxs-lookup"><span data-stu-id="3a862-413">In **Add rule**, configure the following settings:</span></span>

   - <span data-ttu-id="3a862-414">Per **nome**immettere di **nuovo la scalabilità nell'hub Azure stack**.</span><span class="sxs-lookup"><span data-stu-id="3a862-414">For **Name**, enter **Scale back into Azure Stack Hub**.</span></span>
   - <span data-ttu-id="3a862-415">Una **Descrizione** è facoltativa.</span><span class="sxs-lookup"><span data-stu-id="3a862-415">A **Description** is optional.</span></span>
   - <span data-ttu-id="3a862-416">In **origine**  >  **avviso su**selezionare **metrica**.</span><span class="sxs-lookup"><span data-stu-id="3a862-416">Under **Source** > **Alert on**, select **Metrics**.</span></span>
   - <span data-ttu-id="3a862-417">In **criteri**selezionare la sottoscrizione, il gruppo di risorse per il profilo di gestione traffico e il nome del profilo di gestione traffico per la risorsa.</span><span class="sxs-lookup"><span data-stu-id="3a862-417">Under **Criteria**, select your subscription, the resource group for your Traffic Manager profile, and the name of the Traffic Manager profile for the resource.</span></span>

4. <span data-ttu-id="3a862-418">Per **metrica**selezionare **frequenza richieste**.</span><span class="sxs-lookup"><span data-stu-id="3a862-418">For **Metric**, select **Request Rate**.</span></span>
5. <span data-ttu-id="3a862-419">Per **condizione**selezionare **minore di**.</span><span class="sxs-lookup"><span data-stu-id="3a862-419">For **Condition**, select **Less than**.</span></span>
6. <span data-ttu-id="3a862-420">In **soglia**immettere **2**.</span><span class="sxs-lookup"><span data-stu-id="3a862-420">For **Threshold**, enter **2**.</span></span>
7. <span data-ttu-id="3a862-421">Per **periodo**selezionare **negli ultimi 5 minuti**.</span><span class="sxs-lookup"><span data-stu-id="3a862-421">For **Period**, select **Over the last 5 minutes**.</span></span>
8. <span data-ttu-id="3a862-422">In **notifica tramite**:</span><span class="sxs-lookup"><span data-stu-id="3a862-422">Under **Notify via**:</span></span>
   - <span data-ttu-id="3a862-423">Selezionare la casella di controllo per **inviare messaggi di posta elettronica a proprietari, collaboratori e lettori**.</span><span class="sxs-lookup"><span data-stu-id="3a862-423">Check the checkbox for **Email owners, contributors, and readers**.</span></span>
   - <span data-ttu-id="3a862-424">Immettere l'indirizzo di posta elettronica per altri indirizzi di **posta elettronica dell'amministratore**.</span><span class="sxs-lookup"><span data-stu-id="3a862-424">Enter your email address for **Additional administrator email(s)**.</span></span>

9. <span data-ttu-id="3a862-425">Nella barra dei menu selezionare **Salva**.</span><span class="sxs-lookup"><span data-stu-id="3a862-425">On the menu bar, select **Save**.</span></span>

<span data-ttu-id="3a862-426">Lo screenshot seguente Mostra gli avvisi per la scalabilità orizzontale e il ridimensionamento.</span><span class="sxs-lookup"><span data-stu-id="3a862-426">The following screenshot shows the alerts for scale-out and scale-in.</span></span>

   ![Avvisi di Application Insights (versione classica)](media/solution-deployment-guide-hybrid/image22.png)

## <a name="redirect-traffic-between-azure-and-azure-stack-hub"></a><span data-ttu-id="3a862-428">Reindirizza il traffico tra Azure e l'hub Azure Stack</span><span class="sxs-lookup"><span data-stu-id="3a862-428">Redirect traffic between Azure and Azure Stack Hub</span></span>

<span data-ttu-id="3a862-429">È possibile configurare il cambio manuale o automatico del traffico delle app Web tra Azure e l'hub Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="3a862-429">You can configure manual or automatic switching of your web app traffic between Azure and Azure Stack Hub.</span></span>

### <a name="configure-manual-switching-between-azure-and-azure-stack-hub"></a><span data-ttu-id="3a862-430">Configurare il cambio manuale tra Azure e l'hub Azure Stack</span><span class="sxs-lookup"><span data-stu-id="3a862-430">Configure manual switching between Azure and Azure Stack Hub</span></span>

<span data-ttu-id="3a862-431">Quando il sito Web raggiunge le soglie configurate, si riceverà un avviso.</span><span class="sxs-lookup"><span data-stu-id="3a862-431">When your web site reaches the thresholds that you configure, you'll receive an alert.</span></span> <span data-ttu-id="3a862-432">Per reindirizzare manualmente il traffico ad Azure, seguire questa procedura.</span><span class="sxs-lookup"><span data-stu-id="3a862-432">Use the following steps to manually redirect traffic to Azure.</span></span>

1. <span data-ttu-id="3a862-433">Nella portale di Azure selezionare il profilo di gestione traffico.</span><span class="sxs-lookup"><span data-stu-id="3a862-433">In the Azure portal, select your Traffic Manager profile.</span></span>

    ![Endpoint di gestione traffico in portale di Azure](media/solution-deployment-guide-hybrid/image20.png)

2. <span data-ttu-id="3a862-435">Selezionare **Endpoint**.</span><span class="sxs-lookup"><span data-stu-id="3a862-435">Select **Endpoints**.</span></span>
3. <span data-ttu-id="3a862-436">Selezionare l' **endpoint di Azure**.</span><span class="sxs-lookup"><span data-stu-id="3a862-436">Select the **Azure endpoint**.</span></span>
4. <span data-ttu-id="3a862-437">In **stato**selezionare **abilitato**e quindi fare clic su **Salva**.</span><span class="sxs-lookup"><span data-stu-id="3a862-437">Under **Status**, select **Enabled**, and then select **Save**.</span></span>

    ![Abilitare l'endpoint di Azure in portale di Azure](media/solution-deployment-guide-hybrid/image23.png)

5. <span data-ttu-id="3a862-439">In **endpoint** per il profilo di gestione traffico selezionare **endpoint esterno**.</span><span class="sxs-lookup"><span data-stu-id="3a862-439">On **Endpoints** for the Traffic Manager profile, select **External endpoint**.</span></span>
6. <span data-ttu-id="3a862-440">In **stato**selezionare **disabilitato**e quindi fare clic su **Salva**.</span><span class="sxs-lookup"><span data-stu-id="3a862-440">Under **Status**, select **Disabled**, and then select **Save**.</span></span>

    ![Disabilitare Azure Stack endpoint Hub nell'portale di Azure](media/solution-deployment-guide-hybrid/image24.png)

<span data-ttu-id="3a862-442">Una volta configurati gli endpoint, il traffico delle app viene indirizzato all'app Web di Azure con scalabilità orizzontale anziché all'app Web Hub Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="3a862-442">After the endpoints are configured, app traffic goes to your Azure scale-out web app instead of the Azure Stack Hub web app.</span></span>

 ![Endpoint modificati nel traffico delle app Web di Azure](media/solution-deployment-guide-hybrid/image25.png)

<span data-ttu-id="3a862-444">Per invertire di nuovo il flusso all'hub Azure Stack, usare i passaggi precedenti per:</span><span class="sxs-lookup"><span data-stu-id="3a862-444">To reverse the flow back to Azure Stack Hub, use the previous steps to:</span></span>

- <span data-ttu-id="3a862-445">Abilitare l'endpoint dell'hub Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="3a862-445">Enable the Azure Stack Hub endpoint.</span></span>
- <span data-ttu-id="3a862-446">Disabilitare l'endpoint di Azure.</span><span class="sxs-lookup"><span data-stu-id="3a862-446">Disable the Azure endpoint.</span></span>

### <a name="configure-automatic-switching-between-azure-and-azure-stack-hub"></a><span data-ttu-id="3a862-447">Configurare il cambio automatico tra Azure e l'hub Azure Stack</span><span class="sxs-lookup"><span data-stu-id="3a862-447">Configure automatic switching between Azure and Azure Stack Hub</span></span>

<span data-ttu-id="3a862-448">È anche possibile usare Application Insights monitoraggio se l'app viene eseguita in un ambiente senza [Server](https://azure.microsoft.com/overview/serverless-computing/) fornito da funzioni di Azure.</span><span class="sxs-lookup"><span data-stu-id="3a862-448">You can also use Application Insights monitoring if your app runs in a [serverless](https://azure.microsoft.com/overview/serverless-computing/) environment provided by Azure Functions.</span></span>

<span data-ttu-id="3a862-449">In questo scenario è possibile configurare Application Insights per l'uso di un webhook che chiama un'app per le funzioni.</span><span class="sxs-lookup"><span data-stu-id="3a862-449">In this scenario, you can configure Application Insights to use a webhook that calls a function app.</span></span> <span data-ttu-id="3a862-450">Questa app Abilita o Disabilita automaticamente un endpoint in risposta a un avviso.</span><span class="sxs-lookup"><span data-stu-id="3a862-450">This app automatically enables or disables an endpoint in response to an alert.</span></span>

<span data-ttu-id="3a862-451">Usare i passaggi seguenti come guida per configurare il cambio automatico del traffico.</span><span class="sxs-lookup"><span data-stu-id="3a862-451">Use the following steps as a guide to configure automatic traffic switching.</span></span>

1. <span data-ttu-id="3a862-452">Creare un'app per le funzioni di Azure.</span><span class="sxs-lookup"><span data-stu-id="3a862-452">Create an Azure Function app.</span></span>
2. <span data-ttu-id="3a862-453">Creare una funzione attivata tramite HTTP.</span><span class="sxs-lookup"><span data-stu-id="3a862-453">Create an HTTP-triggered function.</span></span>
3. <span data-ttu-id="3a862-454">Importare gli SDK di Azure per Gestione risorse, app Web e gestione traffico.</span><span class="sxs-lookup"><span data-stu-id="3a862-454">Import the Azure SDKs for Resource Manager, Web Apps, and Traffic Manager.</span></span>
4. <span data-ttu-id="3a862-455">Sviluppare codice per:</span><span class="sxs-lookup"><span data-stu-id="3a862-455">Develop code to:</span></span>

   - <span data-ttu-id="3a862-456">Eseguire l'autenticazione alla sottoscrizione di Azure.</span><span class="sxs-lookup"><span data-stu-id="3a862-456">Authenticate to your Azure subscription.</span></span>
   - <span data-ttu-id="3a862-457">Usare un parametro che commuta gli endpoint di gestione traffico per indirizzare il traffico ad Azure o all'hub Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="3a862-457">Use a parameter that toggles the Traffic Manager endpoints to direct traffic to Azure or Azure Stack Hub.</span></span>

5. <span data-ttu-id="3a862-458">Salvare il codice e aggiungere l'URL dell'app per le funzioni con i parametri appropriati alla sezione **webhook** delle impostazioni della regola di avviso Application Insights.</span><span class="sxs-lookup"><span data-stu-id="3a862-458">Save your code and add the function app's URL with the appropriate parameters to the **Webhook** section of the Application Insights alert rule settings.</span></span>
6. <span data-ttu-id="3a862-459">Il traffico viene reindirizzato automaticamente quando viene attivato un avviso Application Insights.</span><span class="sxs-lookup"><span data-stu-id="3a862-459">Traffic is automatically redirected when an Application Insights alert fires.</span></span>

## <a name="next-steps"></a><span data-ttu-id="3a862-460">Passaggi successivi</span><span class="sxs-lookup"><span data-stu-id="3a862-460">Next steps</span></span>

- <span data-ttu-id="3a862-461">Per altre informazioni sui modelli cloud di Azure, vedere [modelli di progettazione cloud](https://docs.microsoft.com/azure/architecture/patterns).</span><span class="sxs-lookup"><span data-stu-id="3a862-461">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](https://docs.microsoft.com/azure/architecture/patterns).</span></span>
