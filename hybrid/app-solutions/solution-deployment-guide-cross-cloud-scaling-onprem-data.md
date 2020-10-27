---
title: Distribuire un'app ibrida con dati locali con scalabilità tra cloud
description: Informazioni su come distribuire un'app che usa dati locali con scalabilità tra cloud tramite Azure e l'hub di Azure Stack.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: ecc42a94e2c59531b2a2e933772b0d8ce8c58609
ms.sourcegitcommit: 0d5b5336bdb969588d0b92e04393e74b8f682c3b
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 10/22/2020
ms.locfileid: "92353479"
---
# <a name="deploy-hybrid-app-with-on-premises-data-that-scales-cross-cloud"></a><span data-ttu-id="bfcf2-103">Distribuire un'app ibrida con dati locali con scalabilità tra cloud</span><span class="sxs-lookup"><span data-stu-id="bfcf2-103">Deploy hybrid app with on-premises data that scales cross-cloud</span></span>

<span data-ttu-id="bfcf2-104">Questa guida alla soluzione illustra come distribuire un'app ibrida che si estende in Azure e nell'hub di Azure Stack e usa un'unica origine dati locale.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-104">This solution guide shows you how to deploy a hybrid app that spans both Azure and Azure Stack Hub and uses a single on-premises data source.</span></span>

<span data-ttu-id="bfcf2-105">Usando una soluzione di cloud ibrido è possibile usufruire dei vantaggi di conformità di un cloud privato e della scalabilità del cloud pubblico.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-105">By using a hybrid cloud solution, you can combine the compliance benefits of a private cloud with the scalability of the public cloud.</span></span> <span data-ttu-id="bfcf2-106">Gli sviluppatori possono inoltre usufruire dell'ecosistema di sviluppatori Microsoft e applicare le loro competenze al cloud e ad ambienti locali.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-106">Your developers can also take advantage of the Microsoft developer ecosystem and apply their skills to the cloud and on-premises environments.</span></span>

## <a name="overview-and-assumptions"></a><span data-ttu-id="bfcf2-107">Panoramica e presupposti</span><span class="sxs-lookup"><span data-stu-id="bfcf2-107">Overview and assumptions</span></span>

<span data-ttu-id="bfcf2-108">Seguire questa esercitazione per configurare un flusso di lavoro che consenta agli sviluppatori di distribuire un'app Web identica in un cloud pubblico e in un cloud privato.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-108">Follow this tutorial to set up a workflow that lets developers deploy an identical web app to a public cloud and a private cloud.</span></span> <span data-ttu-id="bfcf2-109">Questa app può accedere a una rete instradabile non Internet ospitata nel cloud privato.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-109">This app can access a non-internet routable network hosted on the private cloud.</span></span> <span data-ttu-id="bfcf2-110">Queste app Web vengono monitorate e quando si verifica un picco nel traffico, un programma modifica i record DNS per reindirizzare il traffico al cloud pubblico.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-110">These web apps are monitored and when there's a spike in traffic, a program modifies the DNS records to redirect traffic to the public cloud.</span></span> <span data-ttu-id="bfcf2-111">Quando il traffico torna al livello precedente il picco, il traffico viene indirizzato di nuovo al cloud privato.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-111">When traffic drops to the level before the spike, traffic is routed back to the private cloud.</span></span>

<span data-ttu-id="bfcf2-112">Questa esercitazione illustra le attività seguenti:</span><span class="sxs-lookup"><span data-stu-id="bfcf2-112">This tutorial covers the following tasks:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="bfcf2-113">Distribuire un server di database SQL Server con connessione ibrida.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-113">Deploy a hybrid-connected SQL Server database server.</span></span>
> - <span data-ttu-id="bfcf2-114">Connettere un'app Web di Azure globale a una rete ibrida.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-114">Connect a web app in global Azure to a hybrid network.</span></span>
> - <span data-ttu-id="bfcf2-115">Configurare il DNS per la scalabilità tra cloud.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-115">Configure DNS for cross-cloud scaling.</span></span>
> - <span data-ttu-id="bfcf2-116">Configurare i certificati SSL per la scalabilità tra cloud.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-116">Configure SSL certificates for cross-cloud scaling.</span></span>
> - <span data-ttu-id="bfcf2-117">Configurare e distribuire l'app Web.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-117">Configure and deploy the web app.</span></span>
> - <span data-ttu-id="bfcf2-118">Creare un profilo di Gestione traffico e configurarlo per la scalabilità tra cloud.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-118">Create a Traffic Manager profile and configure it for cross-cloud scaling.</span></span>
> - <span data-ttu-id="bfcf2-119">Configurare il monitoraggio e gli avvisi di Application Insights per un aumento del traffico.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-119">Set up Application Insights monitoring and alerting for increased traffic.</span></span>
> - <span data-ttu-id="bfcf2-120">Configurare il passaggio automatico del traffico tra Azure globale e l'hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-120">Configure automatic traffic switching between global Azure and Azure Stack Hub.</span></span>

> [!Tip]  
> <span data-ttu-id="bfcf2-121">![Diagramma dei concetti sulle app ibride](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="bfcf2-121">![Hybrid pillars diagram](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="bfcf2-122">L'hub di Microsoft Azure Stack è un'estensione di Azure.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-122">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="bfcf2-123">L'hub di Azure Stack offre all'ambiente locale l'agilità e l'innovazione del cloud computing, abilitando l'unico cloud ibrido che consente di creare e distribuire ovunque app ibride.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-123">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that allows you to build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="bfcf2-124">L'articolo [Considerazioni per la progettazione di app ibride](overview-app-design-considerations.md) esamina i concetti fondamentali di qualità del software (posizionamento, scalabilità, disponibilità, resilienza, gestibilità e sicurezza) per la progettazione, la distribuzione e la gestione di app ibride.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-124">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="bfcf2-125">Le considerazioni per la progettazione consentono di ottimizzare la progettazione delle app ibride, riducendo al minimo i rischi negli ambienti di produzione.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-125">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

### <a name="assumptions"></a><span data-ttu-id="bfcf2-126">Presupposti</span><span class="sxs-lookup"><span data-stu-id="bfcf2-126">Assumptions</span></span>

<span data-ttu-id="bfcf2-127">Questa esercitazione presuppone che l'utente abbia una conoscenza di base di Azure globale e dell'hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-127">This tutorial assumes that you have a basic knowledge of global Azure and Azure Stack Hub.</span></span> <span data-ttu-id="bfcf2-128">Per altre informazioni prima di iniziare l'esercitazione, vedere gli articoli seguenti:</span><span class="sxs-lookup"><span data-stu-id="bfcf2-128">If you want to learn more before starting the tutorial, review these articles:</span></span>

- [<span data-ttu-id="bfcf2-129">Introduzione ad Azure</span><span class="sxs-lookup"><span data-stu-id="bfcf2-129">Introduction to Azure</span></span>](https://azure.microsoft.com/overview/what-is-azure/)
- [<span data-ttu-id="bfcf2-130">Concetti chiave dell'hub di Azure Stack</span><span class="sxs-lookup"><span data-stu-id="bfcf2-130">Azure Stack Hub Key Concepts</span></span>](/azure-stack/operator/azure-stack-overview.md)

<span data-ttu-id="bfcf2-131">In questa esercitazione si presuppone anche che l'utente abbia una sottoscrizione di Azure.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-131">This tutorial also assumes that you have an Azure subscription.</span></span> <span data-ttu-id="bfcf2-132">Se non si ha già una sottoscrizione, [creare un account gratuito](https://azure.microsoft.com/free/) prima di iniziare.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-132">If you don't have a subscription, [create a free account](https://azure.microsoft.com/free/) before you begin.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="bfcf2-133">Prerequisiti</span><span class="sxs-lookup"><span data-stu-id="bfcf2-133">Prerequisites</span></span>

<span data-ttu-id="bfcf2-134">Prima di iniziare a creare questa soluzione, verificare di soddisfare i requisiti seguenti:</span><span class="sxs-lookup"><span data-stu-id="bfcf2-134">Before you start this solution, make sure you meet the following requirements:</span></span>

- <span data-ttu-id="bfcf2-135">Un Azure Stack Development Kit (ASDK) o una sottoscrizione in un sistema integrato dell'hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-135">An Azure Stack Development Kit (ASDK) or a subscription on an Azure Stack Hub Integrated System.</span></span> <span data-ttu-id="bfcf2-136">Per distribuire un ASDK, seguire le istruzioni riportate in [Distribuire l'ASDK usando il programma di installazione](/azure-stack/asdk/asdk-install.md).</span><span class="sxs-lookup"><span data-stu-id="bfcf2-136">To deploy the ASDK, follow the instructions in [Deploy the ASDK using the installer](/azure-stack/asdk/asdk-install.md).</span></span>
- <span data-ttu-id="bfcf2-137">L'installazione dell'hub di Azure Stack deve includere quanto segue:</span><span class="sxs-lookup"><span data-stu-id="bfcf2-137">Your Azure Stack Hub installation should have the following installed:</span></span>
  - <span data-ttu-id="bfcf2-138">Il Servizio app di Azure.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-138">The Azure App Service.</span></span> <span data-ttu-id="bfcf2-139">Collaborare con l'operatore dell'hub di Azure Stack per distribuire e configurare il Servizio app di Azure nell'ambiente in uso.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-139">Work with your Azure Stack Hub Operator to deploy and configure the Azure App Service on your environment.</span></span> <span data-ttu-id="bfcf2-140">Per questa esercitazione è necessario che il servizio app includa almeno un (1) ruolo di lavoro dedicato disponibile.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-140">This tutorial requires the App Service to have at least one (1) available dedicated worker role.</span></span>
  - <span data-ttu-id="bfcf2-141">Un'immagine di Windows Server 2016.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-141">A Windows Server 2016 image.</span></span>
  - <span data-ttu-id="bfcf2-142">Un'immagine di Windows Server 2016 con Microsoft SQL Server.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-142">A Windows Server 2016 with a Microsoft SQL Server image.</span></span>
  - <span data-ttu-id="bfcf2-143">I piani e le offerte appropriati.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-143">The appropriate plans and offers.</span></span>
  - <span data-ttu-id="bfcf2-144">Un nome di dominio per l'app Web.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-144">A domain name for your web app.</span></span> <span data-ttu-id="bfcf2-145">Se non si ha un nome di dominio, è possibile acquistarne uno da un provider di domini, ad esempio GoDaddy, Bluehost e inMotion.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-145">If you don't have a domain name, you can buy one from a domain provider like GoDaddy, Bluehost, and InMotion.</span></span>
- <span data-ttu-id="bfcf2-146">Un certificato SSL per il dominio emesso da un'autorità di certificazione attendibile, ad esempio LetsEncrypt.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-146">An SSL certificate for your domain from a trusted certificate authority like LetsEncrypt.</span></span>
- <span data-ttu-id="bfcf2-147">Un'app Web che comunica con un database di SQL Server e supporta Application Insights.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-147">A web app that communicates with a SQL Server database and supports Application Insights.</span></span> <span data-ttu-id="bfcf2-148">È possibile scaricare l'app di esempio [dotnetcore-sqldb-tutorial](https://github.com/Azure-Samples/dotnetcore-sqldb-tutorial) da GitHub.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-148">You can download the [dotnetcore-sqldb-tutorial](https://github.com/Azure-Samples/dotnetcore-sqldb-tutorial) sample app from GitHub.</span></span>
- <span data-ttu-id="bfcf2-149">Una rete ibrida tra una rete virtuale di Azure e una rete virtuale dell'hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-149">A hybrid network between an Azure virtual network and Azure Stack Hub virtual network.</span></span> <span data-ttu-id="bfcf2-150">Per istruzioni dettagliate, vedere [Configurare la connettività di cloud ibrido con Azure e l'hub di Azure Stack](solution-deployment-guide-connectivity.md).</span><span class="sxs-lookup"><span data-stu-id="bfcf2-150">For detailed instructions, see [Configure hybrid cloud connectivity with Azure and Azure Stack Hub](solution-deployment-guide-connectivity.md).</span></span>

- <span data-ttu-id="bfcf2-151">Una pipeline con integrazione continua/distribuzione continua (CI/CD) ibrida con un agente di compilazione privato nell'hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-151">A hybrid continuous integration/continuous deployment (CI/CD) pipeline with a private build agent on Azure Stack Hub.</span></span> <span data-ttu-id="bfcf2-152">Per istruzioni dettagliate, vedere [Configurare l'identità del cloud ibrido con app di Azure e dell'hub di Azure Stack](solution-deployment-guide-identity.md).</span><span class="sxs-lookup"><span data-stu-id="bfcf2-152">For detailed instructions, see [Configure hybrid cloud identity with Azure and Azure Stack Hub apps](solution-deployment-guide-identity.md).</span></span>

## <a name="deploy-a-hybrid-connected-sql-server-database-server"></a><span data-ttu-id="bfcf2-153">Distribuire un server di database SQL Server con connessione ibrida</span><span class="sxs-lookup"><span data-stu-id="bfcf2-153">Deploy a hybrid-connected SQL Server database server</span></span>

1. <span data-ttu-id="bfcf2-154">Accedere al portale utenti dell'hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-154">Sign to the Azure Stack Hub user portal.</span></span>

2. <span data-ttu-id="bfcf2-155">Nel **Dashboard** selezionare **Marketplace** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-155">On the **Dashboard** , select **Marketplace** .</span></span>

    ![Marketplace dell'hub di Azure Stack](media/solution-deployment-guide-hybrid/image1.png)

3. <span data-ttu-id="bfcf2-157">In **Marketplace** selezionare **Calcola** e quindi scegliere **Altro** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-157">In **Marketplace** , select **Compute** , and then choose **More** .</span></span> <span data-ttu-id="bfcf2-158">In **Altro** selezionare l'immagine **Free SQL Server License: SQL Server 2017 Developer on Windows Server** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-158">Under **More** , select the **Free SQL Server License: SQL Server 2017 Developer on Windows Server** image.</span></span>

    ![Selezionare un'immagine di macchina virtuale nel portale utenti dell'hub di Azure Stack](media/solution-deployment-guide-hybrid/image2.png)

4. <span data-ttu-id="bfcf2-160">In **Free SQL Server License: SQL Server 2017 Developer on Windows Server** selezionare **Crea** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-160">On **Free SQL Server License: SQL Server 2017 Developer on Windows Server** , select **Create** .</span></span>

5. <span data-ttu-id="bfcf2-161">In **Generale > Configura le impostazioni di base** specificare un **Nome** per la macchina virtuale (VM), un **Nome utente** per l'amministratore di sistema di SQL Server e una **Password** per l'amministratore di sistema.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-161">On **Basics > Configure basic settings** , provide a **Name** for the virtual machine (VM), a **User name** for the SQL Server SA, and a **Password** for the SA.</span></span>  <span data-ttu-id="bfcf2-162">Nell'elenco a discesa **Sottoscrizione** selezionare la sottoscrizione in cui viene eseguita la distribuzione.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-162">From the **Subscription** drop-down list, select the subscription that you're deploying to.</span></span> <span data-ttu-id="bfcf2-163">Per **Gruppo di risorse** usare **Seleziona esistente** e inserire la macchina virtuale nello stesso gruppo di risorse dell'app Web dell'hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-163">For **Resource group** , use **Choose existing** and put the VM in the same resource group as your Azure Stack Hub web app.</span></span>

    ![Configurare le impostazioni di base per la macchina virtuale nel portale utenti dell'hub di Azure Stack](media/solution-deployment-guide-hybrid/image3.png)

6. <span data-ttu-id="bfcf2-165">In **Dimensione** selezionare una dimensione per la macchina virtuale.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-165">Under **Size** , pick a size for your VM.</span></span> <span data-ttu-id="bfcf2-166">Per questa esercitazione è consigliabile selezionare A2_Standard o DS2_V2_Standard.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-166">For this tutorial, we recommend A2_Standard or a DS2_V2_Standard.</span></span>

7. <span data-ttu-id="bfcf2-167">In **Impostazioni > Configura funzionalità facoltative** configurare le impostazioni seguenti:</span><span class="sxs-lookup"><span data-stu-id="bfcf2-167">Under **Settings > Configure optional features** , configure the following settings:</span></span>

   - <span data-ttu-id="bfcf2-168">**Account di archiviazione** : se necessario, creare un nuovo account.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-168">**Storage account** : Create a new account if you need one.</span></span>
   - <span data-ttu-id="bfcf2-169">**Rete virtuale** :</span><span class="sxs-lookup"><span data-stu-id="bfcf2-169">**Virtual network** :</span></span>

     > [!Important]  
     > <span data-ttu-id="bfcf2-170">Assicurarsi che la macchina virtuale SQL Server sia distribuita nella stessa rete virtuale dei gateway VPN.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-170">Make sure your SQL Server VM is deployed on the same  virtual network as the VPN gateways.</span></span>

   - <span data-ttu-id="bfcf2-171">**Indirizzo IP pubblico** : usare le impostazioni predefinite.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-171">**Public IP address** : Use the default settings.</span></span>
   - <span data-ttu-id="bfcf2-172">**Network security group** (Gruppo di sicurezza di rete): (NSG).</span><span class="sxs-lookup"><span data-stu-id="bfcf2-172">**Network security group** : (NSG).</span></span> <span data-ttu-id="bfcf2-173">Creare un nuovo gruppo di sicurezza di rete.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-173">Create a new NSG.</span></span>
   - <span data-ttu-id="bfcf2-174">**Estensioni e Monitoraggio** : Mantenere le impostazioni predefinite.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-174">**Extensions and Monitoring** : Keep the default settings.</span></span>
   - <span data-ttu-id="bfcf2-175">**Account di archiviazione di diagnostica** : se necessario, creare un nuovo account.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-175">**Diagnostics storage account** : Create a new account if you need one.</span></span>
   - <span data-ttu-id="bfcf2-176">Selezionare **OK** per salvare la configurazione.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-176">Select **OK** to save your configuration.</span></span>

     ![Configurare le funzionalità della macchina virtuale facoltative nel portale utenti dell'hub di Azure Stack](media/solution-deployment-guide-hybrid/image4.png)

8. <span data-ttu-id="bfcf2-178">In **Impostazioni di SQL Server** configurare le impostazioni seguenti:</span><span class="sxs-lookup"><span data-stu-id="bfcf2-178">Under **SQL Server settings** , configure the following settings:</span></span>

   - <span data-ttu-id="bfcf2-179">Per **Connettività SQL** selezionare **Pubblico (Internet)** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-179">For **SQL connectivity** , select **Public (Internet)** .</span></span>
   - <span data-ttu-id="bfcf2-180">Per **Porta** mantenere l'impostazione predefinita **1433** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-180">For **Port** , keep the default, **1433** .</span></span>
   - <span data-ttu-id="bfcf2-181">Per **Autenticazione SQL** selezionare **Abilita** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-181">For **SQL authentication** , select **Enable** .</span></span>

     > [!Note]  
     > <span data-ttu-id="bfcf2-182">Quando si abilita l'autenticazione SQL, vengono inserite automaticamente le informazioni "SQLAdmin" configurate in **Generale** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-182">When you enable SQL authentication, it should auto-populate with the "SQLAdmin" information that you configured in **Basics** .</span></span>

   - <span data-ttu-id="bfcf2-183">Per le impostazioni rimanenti, mantenere le impostazioni predefinite.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-183">For the rest of the settings, keep the defaults.</span></span> <span data-ttu-id="bfcf2-184">Selezionare **OK** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-184">Select **OK** .</span></span>

     ![Configurare le impostazioni di SQL Server nel portale utenti dell'hub di Azure Stack](media/solution-deployment-guide-hybrid/image5.png)

9. <span data-ttu-id="bfcf2-186">In **Riepilogo** rivedere la configurazione della macchina virtuale e quindi selezionare **OK** per avviare la distribuzione.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-186">On **Summary** , review the VM configuration and then select **OK** to start the deployment.</span></span>

    ![Riepilogo della configurazione nel portale utenti dell'hub di Azure Stack](media/solution-deployment-guide-hybrid/image6.png)

10. <span data-ttu-id="bfcf2-188">La creazione della nuova macchina virtuale richiede tempo.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-188">It takes some time to create the new VM.</span></span> <span data-ttu-id="bfcf2-189">È possibile visualizzare lo stato delle macchine virtuali in **Macchine virtuali** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-189">You can view the STATUS of your VMs in **Virtual machines** .</span></span>

    ![Stato delle macchine virtuali nel portale utenti dell'hub di Azure Stack](media/solution-deployment-guide-hybrid/image7.png)

## <a name="create-web-apps-in-azure-and-azure-stack-hub"></a><span data-ttu-id="bfcf2-191">Creare app Web in Azure e nell'hub di Azure Stack</span><span class="sxs-lookup"><span data-stu-id="bfcf2-191">Create web apps in Azure and Azure Stack Hub</span></span>

<span data-ttu-id="bfcf2-192">Il Servizio app di Azure semplifica l'esecuzione e la gestione di un'app Web.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-192">The Azure App Service simplifies running and managing a web app.</span></span> <span data-ttu-id="bfcf2-193">Poiché l'hub di Azure Stack è compatibile con Azure, il servizio app può essere eseguito in entrambi gli ambienti.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-193">Because Azure Stack Hub is consistent with Azure,  the App Service can run in both environments.</span></span> <span data-ttu-id="bfcf2-194">Il servizio app verrà usato per ospitare l'app.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-194">You'll use the App Service to host your app.</span></span>

### <a name="create-web-apps"></a><span data-ttu-id="bfcf2-195">Creare app Web</span><span class="sxs-lookup"><span data-stu-id="bfcf2-195">Create web apps</span></span>

1. <span data-ttu-id="bfcf2-196">Creare un'app Web in Azure seguendo le istruzioni riportate in [Gestire un piano di servizio app in Azure](/azure/app-service/app-service-plan-manage#create-an-app-service-plan).</span><span class="sxs-lookup"><span data-stu-id="bfcf2-196">Create a web app in Azure by following the instructions in [Manage an App Service plan in Azure](/azure/app-service/app-service-plan-manage#create-an-app-service-plan).</span></span> <span data-ttu-id="bfcf2-197">Assicurarsi di inserire l'app Web nella stessa sottoscrizione e nello stesso gruppo di risorse della rete ibrida.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-197">Make sure you put the web app in the same subscription and resource group as your hybrid network.</span></span>

2. <span data-ttu-id="bfcf2-198">Ripetere il passaggio precedente (1) nell'hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-198">Repeat the previous step (1) in Azure Stack Hub.</span></span>

### <a name="add-route-for-azure-stack-hub"></a><span data-ttu-id="bfcf2-199">Aggiungere la route per l'hub di Azure Stack</span><span class="sxs-lookup"><span data-stu-id="bfcf2-199">Add route for Azure Stack Hub</span></span>

<span data-ttu-id="bfcf2-200">Il servizio app nell'hub di Azure Stack deve essere instradabile dalla rete Internet pubblica per consentire agli utenti di accedere all'app.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-200">The App Service on Azure Stack Hub must be routable from the public internet to let users access your app.</span></span> <span data-ttu-id="bfcf2-201">Se l'hub di Azure Stack è accessibile da Internet, prendere nota dell'indirizzo IP pubblico o dell'URL dell'app Web dell'hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-201">If your Azure Stack Hub is accessible from the internet, make a note of the public-facing IP address or URL for the Azure Stack Hub web app.</span></span>

<span data-ttu-id="bfcf2-202">Se si usa un ASDK, è possibile [configurare un mapping NAT statico](/azure-stack/operator/azure-stack-create-vpn-connection-one-node.md#configure-the-nat-vm-on-each-asdk-for-gateway-traversal) per esporre il servizio app al di fuori dell'ambiente virtuale.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-202">If you're using an ASDK, you can [configure a static NAT mapping](/azure-stack/operator/azure-stack-create-vpn-connection-one-node.md#configure-the-nat-vm-on-each-asdk-for-gateway-traversal) to expose App Service outside the virtual environment.</span></span>

### <a name="connect-a-web-app-in-azure-to-a-hybrid-network"></a><span data-ttu-id="bfcf2-203">Connettere un'app Web di Azure a una rete ibrida</span><span class="sxs-lookup"><span data-stu-id="bfcf2-203">Connect a web app in Azure to a hybrid network</span></span>

<span data-ttu-id="bfcf2-204">Per offrire connettività tra il front-end Web in Azure e il database SQL Server nell'hub di Azure Stack, l'app Web deve essere connessa alla rete ibrida tra Azure e l'hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-204">To provide connectivity between the web front end in Azure and the SQL Server database in Azure Stack Hub, the web app must be connected to the hybrid network between Azure and Azure Stack Hub.</span></span> <span data-ttu-id="bfcf2-205">Per abilitare la connettività, è necessario:</span><span class="sxs-lookup"><span data-stu-id="bfcf2-205">To enable connectivity, you'll have to:</span></span>

- <span data-ttu-id="bfcf2-206">Configurare la connettività da punto a sito.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-206">Configure point-to-site connectivity.</span></span>
- <span data-ttu-id="bfcf2-207">Configurare l'app Web.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-207">Configure the web app.</span></span>
- <span data-ttu-id="bfcf2-208">Modificare il gateway di rete locale nell'hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-208">Modify the local network gateway in Azure Stack Hub.</span></span>

### <a name="configure-the-azure-virtual-network-for-point-to-site-connectivity"></a><span data-ttu-id="bfcf2-209">Configurare la rete virtuale di Azure per la connettività da punto a sito</span><span class="sxs-lookup"><span data-stu-id="bfcf2-209">Configure the Azure virtual network for point-to-site connectivity</span></span>

<span data-ttu-id="bfcf2-210">Il gateway di rete virtuale sul lato Azure della rete ibrida deve consentire le connessioni da punto a sito per l'integrazione con il Servizio app di Azure.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-210">The virtual network gateway in the Azure side of the hybrid network must allow point-to-site connections to integrate with Azure App Service.</span></span>

1. <span data-ttu-id="bfcf2-211">Nel portale di Azure passare alla pagina del gateway di rete virtuale.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-211">In the Azure portal, go to the virtual network gateway page.</span></span> <span data-ttu-id="bfcf2-212">In **Impostazioni** selezionare **Configurazione da punto a sito** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-212">Under **Settings** , select **Point-to-site configuration** .</span></span>

    ![Opzione da punto a sito nel gateway di rete virtuale di Azure](media/solution-deployment-guide-hybrid/image8.png)

2. <span data-ttu-id="bfcf2-214">Selezionare **Configura adesso** per configurare la connessione da punto a sito.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-214">Select **Configure now** to configure point-to-site.</span></span>

    ![Iniziare la configurazione da punto a sito nel gateway di rete virtuale di Azure](media/solution-deployment-guide-hybrid/image9.png)

3. <span data-ttu-id="bfcf2-216">Nella pagina di configurazione **Da punto a sito** immettere l'intervallo di indirizzi IP privati che si vuole usare in **Pool di indirizzi** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-216">On the **Point-to-site** configuration page, enter the private IP address range that you want to use in **Address pool** .</span></span>

   > [!Note]  
   > <span data-ttu-id="bfcf2-217">Verificare che l'intervallo specificato non si sovrapponga ad altri intervalli di indirizzi già usati dalle subnet nei componenti di Azure globale e dell'hub di Azure Stack della rete ibrida.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-217">Make sure that the range you specify doesn't overlap with any of the address ranges already used by subnets in the global Azure or Azure Stack Hub components of the hybrid network.</span></span>

   <span data-ttu-id="bfcf2-218">In **Tipo di tunnel** deselezionare **VPN IKEv2** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-218">Under **Tunnel Type** , uncheck the **IKEv2 VPN** .</span></span> <span data-ttu-id="bfcf2-219">Selezionare **Salva** per completare la configurazione da punto a sito.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-219">Select **Save** to finish configuring point-to-site.</span></span>

   ![Impostazioni da punto a sito nel gateway di rete virtuale di Azure](media/solution-deployment-guide-hybrid/image10.png)

### <a name="integrate-the-azure-app-service-app-with-the-hybrid-network"></a><span data-ttu-id="bfcf2-221">Integrare l'app del Servizio app di Azure con la rete ibrida</span><span class="sxs-lookup"><span data-stu-id="bfcf2-221">Integrate the Azure App Service app with the hybrid network</span></span>

1. <span data-ttu-id="bfcf2-222">Per connettere l'app alla VNet di Azure, seguire le istruzioni riportate in [Integrazione rete virtuale richiesta dal gateway](/azure/app-service/web-sites-integrate-with-vnet#gateway-required-vnet-integration).</span><span class="sxs-lookup"><span data-stu-id="bfcf2-222">To connect the app to the Azure VNet, follow the instructions in [Gateway required VNet integration](/azure/app-service/web-sites-integrate-with-vnet#gateway-required-vnet-integration).</span></span>

2. <span data-ttu-id="bfcf2-223">Passare a **Impostazioni** per il piano di servizio app che ospita l'app Web.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-223">Go to **Settings** for the App Service plan hosting the web app.</span></span> <span data-ttu-id="bfcf2-224">In **Impostazioni** selezionare **Rete** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-224">In **Settings** , select **Networking** .</span></span>

    ![Configurare la rete per il piano di servizio app](media/solution-deployment-guide-hybrid/image11.png)

3. <span data-ttu-id="bfcf2-226">In **Integrazione rete virtuale** selezionare **Fare clic qui per la gestione** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-226">In **VNET Integration** , select **Click here to manage** .</span></span>

    ![Gestire l'integrazione rete virtuale per il piano di servizio app](media/solution-deployment-guide-hybrid/image12.png)

4. <span data-ttu-id="bfcf2-228">Selezionare la rete virtuale da configurare.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-228">Select the VNET that you want to configure.</span></span> <span data-ttu-id="bfcf2-229">In **ROUTING DEGLI INDIRIZZI IP ALLA RETE VIRTUALE** immettere l'intervallo di indirizzi IP per la rete virtuale di Azure, la rete virtuale dell'hub di Azure Stack e gli spazi indirizzi da punto a sito.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-229">Under **IP ADDRESSES ROUTED TO VNET** , enter the IP address range for the Azure VNet, the Azure Stack Hub VNet, and the point-to-site address spaces.</span></span> <span data-ttu-id="bfcf2-230">Selezionare **Salva** per convalidare e salvare le impostazioni.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-230">Select **Save** to validate and save these settings.</span></span>

    ![Intervalli di indirizzi IP da instradare nell'integrazione della rete virtuale](media/solution-deployment-guide-hybrid/image13.png)

<span data-ttu-id="bfcf2-232">Per altre informazioni sull'integrazione del servizio app con le reti virtuali di Azure, vedere [Integrare l'app con una rete virtuale di Azure](/azure/app-service/web-sites-integrate-with-vnet).</span><span class="sxs-lookup"><span data-stu-id="bfcf2-232">To learn more about how App Service integrates with Azure VNets, see [Integrate your app with an Azure Virtual Network](/azure/app-service/web-sites-integrate-with-vnet).</span></span>

### <a name="configure-the-azure-stack-hub-virtual-network"></a><span data-ttu-id="bfcf2-233">Configurare la rete virtuale dell'hub di Azure Stack</span><span class="sxs-lookup"><span data-stu-id="bfcf2-233">Configure the Azure Stack Hub virtual network</span></span>

<span data-ttu-id="bfcf2-234">Il gateway di rete locale nella rete virtuale dell'hub di Azure Stack deve essere configurato in modo da instradare il traffico dall'intervallo di indirizzi da punto a sito del servizio app.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-234">The local network gateway in the Azure Stack Hub virtual network needs to be configured to route traffic from the App Service point-to-site address range.</span></span>

1. <span data-ttu-id="bfcf2-235">Nel portale Hub di Azure Stack passare a **Gateway di rete virtuale** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-235">In the Azure Stack Hub portal, go to **Local network gateway** .</span></span> <span data-ttu-id="bfcf2-236">In **Impostazioni** selezionare **Configurazione** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-236">Under **Settings** , select **Configuration** .</span></span>

    ![Opzione di configurazione del gateway nel gateway di rete locale dell'hub di Azure Stack](media/solution-deployment-guide-hybrid/image14.png)

2. <span data-ttu-id="bfcf2-238">In **Spazio indirizzi** immettere l'intervallo di indirizzi da punto a sito per il gateway di rete virtuale in Azure.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-238">In **Address space** , enter the point-to-site address range for the virtual network gateway in Azure.</span></span>

    ![Spazio indirizzi da punto a sito nel gateway di rete locale dell'hub di Azure Stack](media/solution-deployment-guide-hybrid/image15.png)

3. <span data-ttu-id="bfcf2-240">Selezionare **Salva** per convalidare e salvare la configurazione.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-240">Select **Save** to validate and save the configuration.</span></span>

## <a name="configure-dns-for-cross-cloud-scaling"></a><span data-ttu-id="bfcf2-241">Configurare il DNS per la scalabilità tra cloud</span><span class="sxs-lookup"><span data-stu-id="bfcf2-241">Configure DNS for cross-cloud scaling</span></span>

<span data-ttu-id="bfcf2-242">Con la configurazione corretta del DNS per le app tra cloud, gli utenti possono accedere alle istanze di Azure globale e dell'hub di Azure Stack dell'app Web.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-242">By properly configuring DNS for cross-cloud apps, users can access the global Azure and Azure Stack Hub instances of your web app.</span></span> <span data-ttu-id="bfcf2-243">La configurazione DNS per questa esercitazione consente anche a Gestione traffico di Azure di instradare il traffico quando il carico aumenta o diminuisce.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-243">The DNS configuration for this tutorial also lets Azure Traffic Manager route traffic when the load increases or decreases.</span></span>

<span data-ttu-id="bfcf2-244">Questa esercitazione usa DNS di Azure per la gestione del DNS poiché i domini del servizio app non funzionano.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-244">This tutorial uses Azure DNS to manage the DNS because App Service domains won't work.</span></span>

### <a name="create-subdomains"></a><span data-ttu-id="bfcf2-245">Creare sottodomini</span><span class="sxs-lookup"><span data-stu-id="bfcf2-245">Create subdomains</span></span>

<span data-ttu-id="bfcf2-246">Poiché Gestione traffico si basa su CNAME DNS, è necessario un sottodominio per instradare correttamente il traffico agli endpoint.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-246">Because Traffic Manager relies on DNS CNAMEs, a subdomain is needed to properly route traffic to endpoints.</span></span> <span data-ttu-id="bfcf2-247">Per altre informazioni sui record DNS e sul mapping del dominio, vedere [Mappare i domini con Gestione traffico](/azure/app-service/web-sites-traffic-manager-custom-domain-name).</span><span class="sxs-lookup"><span data-stu-id="bfcf2-247">For more information about DNS records and domain mapping, see [map domains with Traffic Manager](/azure/app-service/web-sites-traffic-manager-custom-domain-name).</span></span>

<span data-ttu-id="bfcf2-248">Per l'endpoint di Azure verrà creato un sottodominio che gli utenti possono usare per accedere all'app Web.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-248">For the Azure endpoint, you'll create a subdomain that users can use to access your web app.</span></span> <span data-ttu-id="bfcf2-249">Per questa esercitazione, è possibile usare **app.northwind.com** , ma è necessario personalizzare questo valore in base al proprio dominio.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-249">For this tutorial, can use **app.northwind.com** , but you should customize this value based on your own domain.</span></span>

<span data-ttu-id="bfcf2-250">Sarà anche necessario creare un sottodominio con un record A per l'endpoint dell'hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-250">You'll also need to create a subdomain with an A record for the Azure Stack Hub endpoint.</span></span> <span data-ttu-id="bfcf2-251">È possibile usare **azurestack.northwind.com** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-251">You can use **azurestack.northwind.com** .</span></span>

### <a name="configure-a-custom-domain-in-azure"></a><span data-ttu-id="bfcf2-252">Configurare un dominio personalizzato in Azure</span><span class="sxs-lookup"><span data-stu-id="bfcf2-252">Configure a custom domain in Azure</span></span>

1. <span data-ttu-id="bfcf2-253">Aggiungere il nome host **app.northwind.com** all'app Web di Azure eseguendo [il mapping di un CNAME al Servizio app di Azure](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).</span><span class="sxs-lookup"><span data-stu-id="bfcf2-253">Add the **app.northwind.com** hostname to the Azure web app by [mapping a CNAME to Azure App Service](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).</span></span>

### <a name="configure-custom-domains-in-azure-stack-hub"></a><span data-ttu-id="bfcf2-254">Configurare domini personalizzati nell'hub di Azure Stack</span><span class="sxs-lookup"><span data-stu-id="bfcf2-254">Configure custom domains in Azure Stack Hub</span></span>

1. <span data-ttu-id="bfcf2-255">Aggiungere il nome host **azurestack.northwind.com** all'app Web dell'hub di Azure Stack eseguendo [il mapping di un record A al Servizio app di Azure](/azure/app-service/app-service-web-tutorial-custom-domain#map-an-a-record).</span><span class="sxs-lookup"><span data-stu-id="bfcf2-255">Add the **azurestack.northwind.com** hostname to the Azure Stack Hub web app by [mapping an A record to Azure App Service](/azure/app-service/app-service-web-tutorial-custom-domain#map-an-a-record).</span></span> <span data-ttu-id="bfcf2-256">Usare l'indirizzo IP instradabile su Internet per l'app del servizio app.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-256">Use the internet-routable IP address for the App Service app.</span></span>

2. <span data-ttu-id="bfcf2-257">Aggiungere il nome host **app.northwind.com** all'app Web dell'hub di Azure Stack eseguendo [il mapping di un CNAME al Servizio app di Azure](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).</span><span class="sxs-lookup"><span data-stu-id="bfcf2-257">Add the **app.northwind.com** hostname to the Azure Stack Hub web app by [mapping a CNAME to Azure App Service](/azure/app-service/app-service-web-tutorial-custom-domain#map-a-cname-record).</span></span> <span data-ttu-id="bfcf2-258">Usare il nome host configurato nel passaggio precedente (1) come destinazione per CNAME.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-258">Use the hostname you configured in the previous step (1) as the target for the CNAME.</span></span>

## <a name="configure-ssl-certificates-for-cross-cloud-scaling"></a><span data-ttu-id="bfcf2-259">Configurare i certificati SSL per la scalabilità tra cloud</span><span class="sxs-lookup"><span data-stu-id="bfcf2-259">Configure SSL certificates for cross-cloud scaling</span></span>

<span data-ttu-id="bfcf2-260">È importante assicurarsi che i dati sensibili raccolti dall'app Web siano protetti in transito e quando vengono archiviati nel database SQL.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-260">It's important to ensure sensitive data collected by your web app is secure in transit to and when stored on the SQL database.</span></span>

<span data-ttu-id="bfcf2-261">Si configureranno le app Web di Azure e dell'hub di Azure Stack per l'uso dei certificati SSL per tutto il traffico in ingresso.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-261">You'll configure your Azure and Azure Stack Hub web apps to use SSL certificates for all incoming traffic.</span></span>

### <a name="add-ssl-to-azure-and-azure-stack-hub"></a><span data-ttu-id="bfcf2-262">Aggiungere SSL ad Azure e all'hub di Azure Stack</span><span class="sxs-lookup"><span data-stu-id="bfcf2-262">Add SSL to Azure and Azure Stack Hub</span></span>

<span data-ttu-id="bfcf2-263">Per aggiungere SSL ad Azure:</span><span class="sxs-lookup"><span data-stu-id="bfcf2-263">To add SSL to Azure:</span></span>

1. <span data-ttu-id="bfcf2-264">Verificare che il certificato SSL ottenuto sia valido per il sottodominio creato.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-264">Make sure that the SSL certificate you get is valid for the subdomain you created.</span></span> <span data-ttu-id="bfcf2-265">È possibile usare certificati con caratteri jolly.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-265">(It's okay to use wildcard certificates.)</span></span>

2. <span data-ttu-id="bfcf2-266">Nel portale di Azure seguire le istruzioni riportate nelle sezioni **Preparare l'app Web** e **Associare il certificato SSL** dell'articolo [Associare un certificato SSL personalizzato esistente ad app Web di Azure](/azure/app-service/app-service-web-tutorial-custom-ssl).</span><span class="sxs-lookup"><span data-stu-id="bfcf2-266">In the Azure portal, follow the instructions in the **Prepare your web app** and **Bind your SSL certificate** sections of the [Bind an existing custom SSL certificate to Azure Web Apps](/azure/app-service/app-service-web-tutorial-custom-ssl) article.</span></span> <span data-ttu-id="bfcf2-267">Selezionare **SSL basato su SNI** come **Tipo SSL** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-267">Select **SNI-based SSL** as the **SSL Type** .</span></span>

3. <span data-ttu-id="bfcf2-268">Reindirizzare tutto il traffico alla porta HTTPS.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-268">Redirect all traffic to the HTTPS port.</span></span> <span data-ttu-id="bfcf2-269">Seguire le istruzioni riportate nella sezione **Applicare HTTPS** dell'articolo [Associare un certificato SSL personalizzato esistente ad app Web di Azure](/azure/app-service/app-service-web-tutorial-custom-ssl).</span><span class="sxs-lookup"><span data-stu-id="bfcf2-269">Follow the instructions in the   **Enforce HTTPS** section of the [Bind an existing custom SSL certificate to Azure Web Apps](/azure/app-service/app-service-web-tutorial-custom-ssl) article.</span></span>

<span data-ttu-id="bfcf2-270">Per aggiungere SSL all'hub di Azure Stack:</span><span class="sxs-lookup"><span data-stu-id="bfcf2-270">To add SSL to Azure Stack Hub:</span></span>

1. <span data-ttu-id="bfcf2-271">Ripetere i passaggi 1-3 seguiti per Azure usando il portale Hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-271">Repeat steps 1-3 that you used for Azure, using the Azure Stack Hub portal.</span></span>

## <a name="configure-and-deploy-the-web-app"></a><span data-ttu-id="bfcf2-272">Configurare e distribuire l'app Web</span><span class="sxs-lookup"><span data-stu-id="bfcf2-272">Configure and deploy the web app</span></span>

<span data-ttu-id="bfcf2-273">Il codice dell'app verrà configurato per inviare i dati di telemetria all'istanza corretta di Application Insights e configurare le app Web con le stringhe di connessione appropriate.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-273">You'll configure the app code to report telemetry to the correct Application Insights instance and configure the web apps with the right connection strings.</span></span> <span data-ttu-id="bfcf2-274">Per altre informazioni su Application Insights, vedere [Informazioni su Azure Application Insights](/azure/application-insights/app-insights-overview).</span><span class="sxs-lookup"><span data-stu-id="bfcf2-274">To learn more about Application Insights, see [What is Application Insights?](/azure/application-insights/app-insights-overview)</span></span>

### <a name="add-application-insights"></a><span data-ttu-id="bfcf2-275">Aggiungere Application Insights</span><span class="sxs-lookup"><span data-stu-id="bfcf2-275">Add Application Insights</span></span>

1. <span data-ttu-id="bfcf2-276">Aprire l'app Web in Microsoft Visual Studio.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-276">Open your web app in Microsoft Visual Studio.</span></span>

2. <span data-ttu-id="bfcf2-277">[Aggiungere Application Insights](/azure/azure-monitor/app/asp-net-core#enable-client-side-telemetry-for-web-applications) al progetto per trasmettere i dati di telemetria che Application Insights usa per creare avvisi quando il traffico Web aumenta o diminuisce.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-277">[Add Application Insights](/azure/azure-monitor/app/asp-net-core#enable-client-side-telemetry-for-web-applications) to your project to transmit the telemetry that Application Insights uses to create alerts when web traffic increases or decreases.</span></span>

### <a name="configure-dynamic-connection-strings"></a><span data-ttu-id="bfcf2-278">Configurare le stringhe di connessione dinamiche</span><span class="sxs-lookup"><span data-stu-id="bfcf2-278">Configure dynamic connection strings</span></span>

<span data-ttu-id="bfcf2-279">Ogni istanza dell'app Web userà un metodo diverso per la connessione al database SQL.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-279">Each instance of the web app will use a different method to connect to the SQL database.</span></span> <span data-ttu-id="bfcf2-280">L'app in Azure usa l'indirizzo IP privato della macchina virtuale di SQL Server, mentre l'app nell'hub di Azure Stack usa l'indirizzo IP pubblico della macchina virtuale di SQL Server.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-280">The app in Azure uses the private IP address of the SQL Server VM and the app in Azure Stack Hub uses the public IP address of the SQL Server VM.</span></span>

> [!Note]  
> <span data-ttu-id="bfcf2-281">In un sistema integrato dell'hub di Azure Stack, l'indirizzo IP pubblico non deve essere instradabile su Internet.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-281">On an Azure Stack Hub integrated system, the public IP address shouldn't be internet-routable.</span></span> <span data-ttu-id="bfcf2-282">In un ASDK, l'indirizzo IP pubblico non è instradabile all'esterno dell'ASDK.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-282">On an ASDK, the public IP address isn't routable outside the ASDK.</span></span>

<span data-ttu-id="bfcf2-283">È possibile usare le variabili di ambiente del servizio app per passare una stringa di connessione diversa a ogni istanza dell'app.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-283">You can use App Service environment variables to pass a different connection string to each instance of the app.</span></span>

1. <span data-ttu-id="bfcf2-284">Aprire l'app in Visual Studio.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-284">Open the app in Visual Studio.</span></span>

2. <span data-ttu-id="bfcf2-285">Aprire Startup.cs e trovare il blocco di codice seguente:</span><span class="sxs-lookup"><span data-stu-id="bfcf2-285">Open Startup.cs and find the following code block:</span></span>

    ```C#
    services.AddDbContext<MyDatabaseContext>(options =>
        options.UseSqlite("Data Source=localdatabase.db"));
    ```

3. <span data-ttu-id="bfcf2-286">Sostituire il blocco di codice precedente con il codice seguente che usa una stringa di connessione definita nel file *appsettings.json* :</span><span class="sxs-lookup"><span data-stu-id="bfcf2-286">Replace the previous code block with the following code, which uses a connection string defined in the *appsettings.json* file:</span></span>

    ```C#
    services.AddDbContext<MyDatabaseContext>(options =>
        options.UseSqlServer(Configuration.GetConnectionString("MyDbConnection")));
     // Automatically perform database migration
     services.BuildServiceProvider().GetService<MyDatabaseContext>().Database.Migrate();
    ```

### <a name="configure-app-service-app-settings"></a><span data-ttu-id="bfcf2-287">Configurare le impostazioni dell'app Servizio app di Azure</span><span class="sxs-lookup"><span data-stu-id="bfcf2-287">Configure App Service app settings</span></span>

1. <span data-ttu-id="bfcf2-288">Creare stringhe di connessione per Azure e per l'hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-288">Create connection strings for Azure and Azure Stack Hub.</span></span> <span data-ttu-id="bfcf2-289">Le stringhe devono essere uguali, ad eccezione degli indirizzi IP usati.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-289">The strings should be the same, except for the IP addresses that are used.</span></span>

2. <span data-ttu-id="bfcf2-290">In Azure e nell'hub di Azure Stack aggiungere la stringa di connessione appropriata [come impostazione dell'app](/azure/app-service/web-sites-configure) nell'app Web usando `SQLCONNSTR\_` come prefisso nel nome.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-290">In Azure and Azure Stack Hub, add the appropriate connection string [as an app setting](/azure/app-service/web-sites-configure) in the web app, using `SQLCONNSTR\_` as a prefix in the name.</span></span>

3. <span data-ttu-id="bfcf2-291">**Salvare** le impostazioni dell'app Web e riavviare l'app.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-291">**Save** the web app settings and restart the app.</span></span>

## <a name="enable-automatic-scaling-in-global-azure"></a><span data-ttu-id="bfcf2-292">Abilitare la scalabilità automatica in Azure globale</span><span class="sxs-lookup"><span data-stu-id="bfcf2-292">Enable automatic scaling in global Azure</span></span>

<span data-ttu-id="bfcf2-293">Quando si crea l'app Web in un ambiente del servizio app, il servizio viene avviato con una istanza.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-293">When you create your web app in an App Service environment, it starts with one instance.</span></span> <span data-ttu-id="bfcf2-294">È quindi possibile aumentare automaticamente il numero di istanze per offrire più risorse di calcolo per l'app.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-294">You can automatically scale out to add instances to provide more compute resources for your app.</span></span> <span data-ttu-id="bfcf2-295">Analogamente, è possibile ridurre automaticamente il numero di istanze necessarie per l'app.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-295">Similarly, you can automatically scale in and reduce the number of instances your app needs.</span></span>

> [!Note]  
> <span data-ttu-id="bfcf2-296">È necessario avere un piano di servizio app per configurare l'aumento e la riduzione delle istanze.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-296">You need to have an App Service plan to configure scale out and scale in.</span></span> <span data-ttu-id="bfcf2-297">Se non si ha un piano, crearne uno prima di eseguire i passaggi successivi.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-297">If you don't have a plan, create one before starting the next steps.</span></span>

### <a name="enable-automatic-scale-out"></a><span data-ttu-id="bfcf2-298">Abilitare l'aumento automatico di istanze</span><span class="sxs-lookup"><span data-stu-id="bfcf2-298">Enable automatic scale-out</span></span>

1. <span data-ttu-id="bfcf2-299">Nel portale di Azure individuare il piano di servizio app per i siti di cui aumentare il numero di istanze, quindi selezionare **Aumenta istanze (piano di servizio app)** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-299">In the Azure portal, find the App Service plan for the sites you want to scale out, and then select **Scale-out (App Service plan)** .</span></span>

    ![Aumentare le istanze del Servizio app di Azure](media/solution-deployment-guide-hybrid/image16.png)

2. <span data-ttu-id="bfcf2-301">Selezionare **Abilita scalabilità automatica** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-301">Select **Enable autoscale** .</span></span>

    ![Abilitare la scalabilità automatica nel servizio app di Azure](media/solution-deployment-guide-hybrid/image17.png)

3. <span data-ttu-id="bfcf2-303">Immettere un nome per **Nome impostazione di scalabilità automatica** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-303">Enter a name for **Autoscale Setting Name** .</span></span> <span data-ttu-id="bfcf2-304">Per il valore della regola di scalabilità automatica **Predefinito** , selezionare **Ridimensiona in base a una metrica** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-304">For the **Default** auto scale rule, select **Scale based on a metric** .</span></span> <span data-ttu-id="bfcf2-305">Impostare **Limiti per le istanze** su **Minimo: 1** , **Massimo: 10** e **Predefinito: 1** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-305">Set the **Instance limits** to **Minimum: 1** , **Maximum: 10** , and **Default: 1** .</span></span>

    ![Configurare la scalabilità automatica nel servizio app di Azure](media/solution-deployment-guide-hybrid/image18.png)

4. <span data-ttu-id="bfcf2-307">Selezionare **+Aggiungi una regola** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-307">Select **+Add a rule** .</span></span>

5. <span data-ttu-id="bfcf2-308">In **Origine metrica** selezionare **Risorsa corrente** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-308">In **Metric Source** , select **Current Resource** .</span></span> <span data-ttu-id="bfcf2-309">Usare i criteri e le azioni seguenti per la regola.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-309">Use the following Criteria and Actions for the rule.</span></span>

#### <a name="criteria"></a><span data-ttu-id="bfcf2-310">Criteri</span><span class="sxs-lookup"><span data-stu-id="bfcf2-310">Criteria</span></span>

1. <span data-ttu-id="bfcf2-311">In **Aggregazione temporale** selezionare **Medio** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-311">Under **Time Aggregation,** select **Average** .</span></span>

2. <span data-ttu-id="bfcf2-312">In **Nome metrica** selezionare **Percentuale CPU** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-312">Under **Metric Name** , select **CPU Percentage** .</span></span>

3. <span data-ttu-id="bfcf2-313">In **Operatore** selezionare **Maggiore di** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-313">Under **Operator** , select **Greater than** .</span></span>

   - <span data-ttu-id="bfcf2-314">Impostare la **Soglia** su **50** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-314">Set the **Threshold** to **50** .</span></span>
   - <span data-ttu-id="bfcf2-315">Impostare la **Durata** su **10** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-315">Set the **Duration** to **10** .</span></span>

#### <a name="action"></a><span data-ttu-id="bfcf2-316">Azione</span><span class="sxs-lookup"><span data-stu-id="bfcf2-316">Action</span></span>

1. <span data-ttu-id="bfcf2-317">In **Operazione** selezionare **Aumenta numero di** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-317">Under **Operation** , select **Increase Count by** .</span></span>

2. <span data-ttu-id="bfcf2-318">Impostare il **Numero di istanze** su **2** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-318">Set the **Instance Count** to **2** .</span></span>

3. <span data-ttu-id="bfcf2-319">Impostare **Disattiva regole dopo** su **5** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-319">Set the **Cool down** to **5** .</span></span>

4. <span data-ttu-id="bfcf2-320">Selezionare **Aggiungi** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-320">Select **Add** .</span></span>

5. <span data-ttu-id="bfcf2-321">Selezionare **+ Aggiungi una regola** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-321">Select the **+ Add a rule** .</span></span>

6. <span data-ttu-id="bfcf2-322">In **Origine metrica** selezionare **Risorsa corrente** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-322">In **Metric Source** , select **Current Resource.**</span></span>

   > [!Note]  
   > <span data-ttu-id="bfcf2-323">La risorsa corrente conterrà il nome/GUID del piano di servizio app e gli elenchi a discesa **Tipo di risorsa** e **Risorsa** non saranno disponibili.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-323">The current resource will contain your App Service plan's name/GUID and the **Resource Type** and **Resource** drop-down lists will be unavailable.</span></span>

### <a name="enable-automatic-scale-in"></a><span data-ttu-id="bfcf2-324">Abilitare la riduzione automatica di istanze</span><span class="sxs-lookup"><span data-stu-id="bfcf2-324">Enable automatic scale in</span></span>

<span data-ttu-id="bfcf2-325">Quando il traffico diminuisce, l'app Web di Azure può ridurre automaticamente il numero di istanze attive per ridurre i costi.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-325">When traffic decreases, the Azure web app can automatically reduce the number of active instances to reduce costs.</span></span> <span data-ttu-id="bfcf2-326">Questa azione è meno aggressiva rispetto allo scale-out e riduce al minimo l'effetto sugli utenti dell'app.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-326">This action is less aggressive than scale-out and minimizes the impact on app users.</span></span>

1. <span data-ttu-id="bfcf2-327">Passare alla condizione di scale-out **Predefinito** , quindi selezionare **+ Aggiungi una regola** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-327">Go to the **Default** scale out condition, then select **+ Add a rule** .</span></span> <span data-ttu-id="bfcf2-328">Usare i criteri e le azioni seguenti per la regola.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-328">Use the following Criteria and Actions for the rule.</span></span>

#### <a name="criteria"></a><span data-ttu-id="bfcf2-329">Criteri</span><span class="sxs-lookup"><span data-stu-id="bfcf2-329">Criteria</span></span>

1. <span data-ttu-id="bfcf2-330">In **Aggregazione temporale** selezionare **Medio** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-330">Under **Time Aggregation,** select **Average** .</span></span>

2. <span data-ttu-id="bfcf2-331">In **Nome metrica** selezionare **Percentuale CPU** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-331">Under **Metric Name** , select **CPU Percentage** .</span></span>

3. <span data-ttu-id="bfcf2-332">In **Operatore** selezionare **Minore di** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-332">Under **Operator** , select **Less than** .</span></span>

   - <span data-ttu-id="bfcf2-333">Impostare la **Soglia** su **30** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-333">Set the **Threshold** to **30** .</span></span>
   - <span data-ttu-id="bfcf2-334">Impostare la **Durata** su **10** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-334">Set the **Duration** to **10** .</span></span>

#### <a name="action"></a><span data-ttu-id="bfcf2-335">Azione</span><span class="sxs-lookup"><span data-stu-id="bfcf2-335">Action</span></span>

1. <span data-ttu-id="bfcf2-336">In **Operazione** selezionare **Riduci numero di** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-336">Under **Operation** , select **Decrease Count by** .</span></span>

   - <span data-ttu-id="bfcf2-337">Impostare il **Numero di istanze** su **1** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-337">Set the **Instance Count** to **1** .</span></span>
   - <span data-ttu-id="bfcf2-338">Impostare **Disattiva regole dopo** su **5** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-338">Set the **Cool down** to **5** .</span></span>

2. <span data-ttu-id="bfcf2-339">Selezionare **Aggiungi** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-339">Select **Add** .</span></span>

## <a name="create-a-traffic-manager-profile-and-configure-cross-cloud-scaling"></a><span data-ttu-id="bfcf2-340">Creare un profilo di Gestione traffico e configurare il ridimensionamento tra cloud</span><span class="sxs-lookup"><span data-stu-id="bfcf2-340">Create a Traffic Manager profile and configure cross-cloud scaling</span></span>

<span data-ttu-id="bfcf2-341">Creare un profilo di Gestione traffico usando il portale di Azure, quindi configurare l'endpoint per abilitare il dimensionamento tra cloud.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-341">Create a Traffic Manager profile using the Azure portal, then configure endpoints to enable cross-cloud scaling.</span></span>

### <a name="create-traffic-manager-profile"></a><span data-ttu-id="bfcf2-342">Creare un profilo di Gestione traffico</span><span class="sxs-lookup"><span data-stu-id="bfcf2-342">Create Traffic Manager profile</span></span>

1. <span data-ttu-id="bfcf2-343">Selezionare **Crea una risorsa** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-343">Select **Create a resource** .</span></span>
2. <span data-ttu-id="bfcf2-344">Selezionare **Rete** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-344">Select **Networking** .</span></span>
3. <span data-ttu-id="bfcf2-345">Selezionare **Profilo di Gestione traffico** e configurare le impostazioni seguenti:</span><span class="sxs-lookup"><span data-stu-id="bfcf2-345">Select **Traffic Manager profile** and configure the following settings:</span></span>

   - <span data-ttu-id="bfcf2-346">In **Nome** immettere un nome per il profilo.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-346">In **Name** , enter a name for your profile.</span></span> <span data-ttu-id="bfcf2-347">Questo nome **deve** essere univoco nella zona trafficmanager.net e viene usato per creare un nuovo nome DNS (ad esempio, northwindstore.trafficmanager.net).</span><span class="sxs-lookup"><span data-stu-id="bfcf2-347">This name **must** be unique in the trafficmanager.net zone and is used to create a new DNS name (for example, northwindstore.trafficmanager.net).</span></span>
   - <span data-ttu-id="bfcf2-348">Per **Metodo di routing** selezionare **Ponderato** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-348">For **Routing method** , select the **Weighted** .</span></span>
   - <span data-ttu-id="bfcf2-349">Per **Sottoscrizione** selezionare la sottoscrizione in cui si vuole creare il profilo.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-349">For **Subscription** , select the subscription you want to create  this profile in.</span></span>
   - <span data-ttu-id="bfcf2-350">In **Gruppo di risorse** creare un nuovo gruppo di risorse per il profilo.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-350">In **Resource Group** , create a new resource group for this profile.</span></span>
   - <span data-ttu-id="bfcf2-351">In **Località del gruppo di risorse** selezionare la località del gruppo di risorse.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-351">In **Resource group location** , select the location of the resource group.</span></span> <span data-ttu-id="bfcf2-352">Questa impostazione indica la posizione del gruppo di risorse e non ha alcun impatto sul profilo di Gestione traffico che viene distribuito a livello globale.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-352">This setting refers to the location of the resource group and has no impact on the Traffic Manager profile that's deployed globally.</span></span>

4. <span data-ttu-id="bfcf2-353">Selezionare **Crea** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-353">Select **Create** .</span></span>

    ![Creare un profilo di Gestione traffico](media/solution-deployment-guide-hybrid/image19.png)

   <span data-ttu-id="bfcf2-355">Dopo aver completato la distribuzione globale del profilo di Gestione traffico, il profilo viene visualizzato nell'elenco delle risorse del gruppo di risorse in cui è stato creato.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-355">When the global deployment of your Traffic Manager profile is complete, it's shown in the list of resources for the resource group you created it under.</span></span>

### <a name="add-traffic-manager-endpoints"></a><span data-ttu-id="bfcf2-356">Aggiungere endpoint di Gestione traffico</span><span class="sxs-lookup"><span data-stu-id="bfcf2-356">Add Traffic Manager endpoints</span></span>

1. <span data-ttu-id="bfcf2-357">Cercare il profilo di Gestione traffico creato.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-357">Search for the Traffic Manager profile you created.</span></span> <span data-ttu-id="bfcf2-358">Se si è passati al gruppo di risorse per il profilo, selezionare il profilo.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-358">If you navigated to the resource group for the profile, select the profile.</span></span>

2. <span data-ttu-id="bfcf2-359">In **Profilo di Gestione traffico** in **IMPOSTAZIONI** selezionare **Endpoint** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-359">In **Traffic Manager profile** , under **SETTINGS** , select **Endpoints** .</span></span>

3. <span data-ttu-id="bfcf2-360">Selezionare **Aggiungi** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-360">Select **Add** .</span></span>

4. <span data-ttu-id="bfcf2-361">In **Aggiungi endpoint** usare le impostazioni seguenti per l'hub di Azure Stack:</span><span class="sxs-lookup"><span data-stu-id="bfcf2-361">In **Add endpoint** , use the following settings for Azure Stack Hub:</span></span>

   - <span data-ttu-id="bfcf2-362">Per **Tipo** selezionare **Endpoint esterno** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-362">For **Type** , select **External endpoint** .</span></span>
   - <span data-ttu-id="bfcf2-363">Immettere un **Nome** per l'endpoint.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-363">Enter a **Name** for the endpoint.</span></span>
   - <span data-ttu-id="bfcf2-364">Per **Nome di dominio completo (FQDN) o indirizzo IP** immettere l'URL esterno dell'app Web dell'hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-364">For **Fully qualified domain name (FQDN) or IP** , enter the external URL for your Azure Stack Hub web app.</span></span>
   - <span data-ttu-id="bfcf2-365">Per **Peso** mantenere l'impostazione predefinita **1** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-365">For **Weight** , keep the default, **1** .</span></span> <span data-ttu-id="bfcf2-366">In questo modo tutto il traffico viene instradato a questo endpoint se è integro.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-366">This weight results in all traffic going to this endpoint if it's healthy.</span></span>
   - <span data-ttu-id="bfcf2-367">Mantenere deselezionata l'opzione **Aggiungi come disabilitato** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-367">Leave **Add as disabled** unchecked.</span></span>

5. <span data-ttu-id="bfcf2-368">Selezionare **OK** per salvare l'endpoint dell'hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-368">Select **OK** to save the Azure Stack Hub endpoint.</span></span>

<span data-ttu-id="bfcf2-369">L'endpoint di Azure verrà configurato successivamente.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-369">You'll configure the Azure endpoint next.</span></span>

1. <span data-ttu-id="bfcf2-370">In **Profilo di Gestione traffico** selezionare **Endpoint** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-370">On **Traffic Manager profile** , select **Endpoints** .</span></span>
2. <span data-ttu-id="bfcf2-371">Selezionare **+Aggiungi** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-371">Select **+Add** .</span></span>
3. <span data-ttu-id="bfcf2-372">In **Aggiungi endpoint** usare le impostazioni seguenti per Azure:</span><span class="sxs-lookup"><span data-stu-id="bfcf2-372">On **Add endpoint** , use the following settings for Azure:</span></span>

   - <span data-ttu-id="bfcf2-373">Per **Tipo** selezionare **Endpoint Azure** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-373">For **Type** , select **Azure endpoint** .</span></span>
   - <span data-ttu-id="bfcf2-374">Immettere un **Nome** per l'endpoint.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-374">Enter a **Name** for the endpoint.</span></span>
   - <span data-ttu-id="bfcf2-375">Per **Tipo di risorsa di destinazione** selezionare **Servizio app** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-375">For **Target resource type** , select **App Service** .</span></span>
   - <span data-ttu-id="bfcf2-376">Per **Risorsa di destinazione** selezionare **Scegliere un servizio app** per visualizzare un elenco di app Web nella stessa sottoscrizione.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-376">For **Target resource** , select **Choose an app service** to see a list of Web Apps in the same subscription.</span></span>
   - <span data-ttu-id="bfcf2-377">In **Risorsa** selezionare il servizio App che si vuole aggiungere come primo endpoint.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-377">In **Resource** , pick the App service that you want to add as the first endpoint.</span></span>
   - <span data-ttu-id="bfcf2-378">Per **Peso** selezionare **2** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-378">For **Weight** , select **2** .</span></span> <span data-ttu-id="bfcf2-379">Questa impostazione determina l'indirizzamento di tutto il traffico a questo endpoint se l'endpoint primario non è integro o se è presente una regola o un avviso che reindirizza il traffico quando viene attivato.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-379">This setting results in all traffic going to this endpoint if the primary endpoint is unhealthy, or if you have a rule/alert that redirects traffic when triggered.</span></span>
   - <span data-ttu-id="bfcf2-380">Mantenere deselezionata l'opzione **Aggiungi come disabilitato** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-380">Leave **Add as disabled** unchecked.</span></span>

4. <span data-ttu-id="bfcf2-381">Selezionare **OK** per salvare l'endpoint di Azure.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-381">Select **OK** to save the Azure endpoint.</span></span>

<span data-ttu-id="bfcf2-382">Dopo aver configurato entrambi gli endpoint, gli endpoint vengono elencati in **Profilo di Gestione traffico** quando si seleziona **Endpoint** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-382">After both endpoints are configured, they're listed in **Traffic Manager profile** when you select **Endpoints** .</span></span> <span data-ttu-id="bfcf2-383">L'esempio dello screenshot seguente mostra due endpoint con le relative informazioni sullo stato e sulla configurazione.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-383">The example in the following screen capture shows two endpoints, with status and configuration information for each one.</span></span>

![Endpoint nel profilo di Gestione traffico](media/solution-deployment-guide-hybrid/image20.png)

## <a name="set-up-application-insights-monitoring-and-alerting-in-azure"></a><span data-ttu-id="bfcf2-385">Configurare il monitoraggio e gli avvisi di Application Insights in Azure</span><span class="sxs-lookup"><span data-stu-id="bfcf2-385">Set up Application Insights monitoring and alerting in Azure</span></span>

<span data-ttu-id="bfcf2-386">Azure Application Insights consente di monitorare l'app e inviare avvisi in base alle condizioni configurate.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-386">Azure Application Insights lets you monitor your app and send alerts based on conditions you configure.</span></span> <span data-ttu-id="bfcf2-387">Alcuni esempi sono: l'app non è disponibile, sta riscontrando errori o presenta problemi di prestazioni.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-387">Some examples are: the app is unavailable, is experiencing failures, or is showing performance issues.</span></span>

<span data-ttu-id="bfcf2-388">Per creare gli avvisi si useranno le metriche di Azure Application Insights.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-388">You'll use Azure Application Insights metrics to create alerts.</span></span> <span data-ttu-id="bfcf2-389">Quando si attivano questi avvisi, l'istanza dell'app Web passa automaticamente dall'hub di Azure Stack ad Azure per l'aumento delle istanze e quindi torna all'hub di Azure Stack per la riduzione.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-389">When these alerts trigger, your web app's instance will automatically switch from Azure Stack Hub to Azure to scale out, and then back to Azure Stack Hub to scale in.</span></span>

### <a name="create-an-alert-from-metrics"></a><span data-ttu-id="bfcf2-390">Creare un avviso dalle metriche</span><span class="sxs-lookup"><span data-stu-id="bfcf2-390">Create an alert from metrics</span></span>

<span data-ttu-id="bfcf2-391">Nel portale di Azure passare al gruppo di risorse di questa esercitazione e quindi selezionare l'istanza di Application Insights per aprire **Application Insights** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-391">In the Azure portal, go to the resource group for this tutorial, and select the Application Insights instance to open **Application Insights** .</span></span>

![Application Insights](media/solution-deployment-guide-hybrid/image21.png)

<span data-ttu-id="bfcf2-393">Questa visualizzazione verrà usata per creare un avviso di aumento delle istanze e un avviso di riduzione delle istanze.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-393">You'll use this view to create a scale-out alert and a scale-in alert.</span></span>

### <a name="create-the-scale-out-alert"></a><span data-ttu-id="bfcf2-394">Creare l'avviso di aumento delle istanze</span><span class="sxs-lookup"><span data-stu-id="bfcf2-394">Create the scale-out alert</span></span>

1. <span data-ttu-id="bfcf2-395">In **CONFIGURA** selezionare **Avvisi (versione classica)** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-395">Under **CONFIGURE** , select **Alerts (classic)** .</span></span>
2. <span data-ttu-id="bfcf2-396">Selezionare **Aggiungi avviso per la metrica (versione classica)** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-396">Select **Add metric alert (classic)** .</span></span>
3. <span data-ttu-id="bfcf2-397">In **Aggiungi regola** configurare le impostazioni seguenti:</span><span class="sxs-lookup"><span data-stu-id="bfcf2-397">In **Add rule** , configure the following settings:</span></span>

   - <span data-ttu-id="bfcf2-398">Per **Nome** immettere **Entra in Azure Cloud** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-398">For **Name** , enter **Burst into Azure Cloud** .</span></span>
   - <span data-ttu-id="bfcf2-399">La **Descrizione** è facoltativa.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-399">A **Description** is optional.</span></span>
   - <span data-ttu-id="bfcf2-400">In **Origine** > **Avviso per** selezionare **Metriche** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-400">Under **Source** > **Alert on** , select **Metrics** .</span></span>
   - <span data-ttu-id="bfcf2-401">In **Criteri** selezionare la sottoscrizione, il gruppo di risorse per il profilo di Gestione traffico e il nome del profilo di Gestione traffico per la risorsa.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-401">Under **Criteria** , select your subscription, the resource group for your Traffic Manager profile, and the name of the Traffic Manager profile for the resource.</span></span>

4. <span data-ttu-id="bfcf2-402">Per **Metrica** selezionare **Velocità richiesta** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-402">For **Metric** , select **Request Rate** .</span></span>
5. <span data-ttu-id="bfcf2-403">Per **Condizione** selezionare **Maggiore di** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-403">For **Condition** , select **Greater than** .</span></span>
6. <span data-ttu-id="bfcf2-404">Per **Soglia** immettere **2** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-404">For **Threshold** , enter **2** .</span></span>
7. <span data-ttu-id="bfcf2-405">Per **Periodo** selezionare **Negli ultimi 5 minuti** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-405">For **Period** , select **Over the last 5 minutes** .</span></span>
8. <span data-ttu-id="bfcf2-406">In **Notifica tramite** :</span><span class="sxs-lookup"><span data-stu-id="bfcf2-406">Under **Notify via** :</span></span>
   - <span data-ttu-id="bfcf2-407">Selezionare la casella di controllo **Invia messaggio di posta elettronica a proprietari, collaboratori e lettori** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-407">Check the checkbox for **Email owners, contributors, and readers** .</span></span>
   - <span data-ttu-id="bfcf2-408">Immettere l'indirizzo e-mail per **Indirizzi di posta elettronica aggiuntivi dell'amministratore** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-408">Enter your email address for **Additional administrator email(s)** .</span></span>

9. <span data-ttu-id="bfcf2-409">Nella barra dei menu selezionare **Salva** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-409">On the menu bar, select **Save** .</span></span>

### <a name="create-the-scale-in-alert"></a><span data-ttu-id="bfcf2-410">Creare l'avviso di riduzione delle istanze</span><span class="sxs-lookup"><span data-stu-id="bfcf2-410">Create the scale-in alert</span></span>

1. <span data-ttu-id="bfcf2-411">In **CONFIGURA** selezionare **Avvisi (versione classica)** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-411">Under **CONFIGURE** , select **Alerts (classic)** .</span></span>
2. <span data-ttu-id="bfcf2-412">Selezionare **Aggiungi avviso per la metrica (versione classica)** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-412">Select **Add metric alert (classic)** .</span></span>
3. <span data-ttu-id="bfcf2-413">In **Aggiungi regola** configurare le impostazioni seguenti:</span><span class="sxs-lookup"><span data-stu-id="bfcf2-413">In **Add rule** , configure the following settings:</span></span>

   - <span data-ttu-id="bfcf2-414">Per **Nome** immettere **Torna all'hub di Azure Stack** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-414">For **Name** , enter **Scale back into Azure Stack Hub** .</span></span>
   - <span data-ttu-id="bfcf2-415">La **Descrizione** è facoltativa.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-415">A **Description** is optional.</span></span>
   - <span data-ttu-id="bfcf2-416">In **Origine** > **Avviso per** selezionare **Metriche** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-416">Under **Source** > **Alert on** , select **Metrics** .</span></span>
   - <span data-ttu-id="bfcf2-417">In **Criteri** selezionare la sottoscrizione, il gruppo di risorse per il profilo di Gestione traffico e il nome del profilo di Gestione traffico per la risorsa.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-417">Under **Criteria** , select your subscription, the resource group for your Traffic Manager profile, and the name of the Traffic Manager profile for the resource.</span></span>

4. <span data-ttu-id="bfcf2-418">Per **Metrica** selezionare **Velocità richiesta** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-418">For **Metric** , select **Request Rate** .</span></span>
5. <span data-ttu-id="bfcf2-419">Per **Condizione** selezionare **Minore di** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-419">For **Condition** , select **Less than** .</span></span>
6. <span data-ttu-id="bfcf2-420">Per **Soglia** immettere **2** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-420">For **Threshold** , enter **2** .</span></span>
7. <span data-ttu-id="bfcf2-421">Per **Periodo** selezionare **Negli ultimi 5 minuti** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-421">For **Period** , select **Over the last 5 minutes** .</span></span>
8. <span data-ttu-id="bfcf2-422">In **Notifica tramite** :</span><span class="sxs-lookup"><span data-stu-id="bfcf2-422">Under **Notify via** :</span></span>
   - <span data-ttu-id="bfcf2-423">Selezionare la casella di controllo **Invia messaggio di posta elettronica a proprietari, collaboratori e lettori** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-423">Check the checkbox for **Email owners, contributors, and readers** .</span></span>
   - <span data-ttu-id="bfcf2-424">Immettere l'indirizzo e-mail per **Indirizzi di posta elettronica aggiuntivi dell'amministratore** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-424">Enter your email address for **Additional administrator email(s)** .</span></span>

9. <span data-ttu-id="bfcf2-425">Nella barra dei menu selezionare **Salva** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-425">On the menu bar, select **Save** .</span></span>

<span data-ttu-id="bfcf2-426">Lo screenshot seguente mostra gli avvisi per l'aumento e la riduzione di istanze.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-426">The following screenshot shows the alerts for scale-out and scale-in.</span></span>

   ![Avvisi di Application Insights (versione classica)](media/solution-deployment-guide-hybrid/image22.png)

## <a name="redirect-traffic-between-azure-and-azure-stack-hub"></a><span data-ttu-id="bfcf2-428">Reindirizzare il traffico tra Azure e l'hub di Azure Stack</span><span class="sxs-lookup"><span data-stu-id="bfcf2-428">Redirect traffic between Azure and Azure Stack Hub</span></span>

<span data-ttu-id="bfcf2-429">È possibile configurare la commutazione manuale o automatica del traffico delle app Web tra Azure e l'hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-429">You can configure manual or automatic switching of your web app traffic between Azure and Azure Stack Hub.</span></span>

### <a name="configure-manual-switching-between-azure-and-azure-stack-hub"></a><span data-ttu-id="bfcf2-430">Configurare la commutazione manuale tra Azure e l'hub di Azure Stack</span><span class="sxs-lookup"><span data-stu-id="bfcf2-430">Configure manual switching between Azure and Azure Stack Hub</span></span>

<span data-ttu-id="bfcf2-431">Quando il sito Web raggiunge le soglie configurate, si riceverà un avviso.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-431">When your web site reaches the thresholds that you configure, you'll receive an alert.</span></span> <span data-ttu-id="bfcf2-432">Per reindirizzare manualmente il traffico ad Azure, eseguire i passaggi seguenti.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-432">Use the following steps to manually redirect traffic to Azure.</span></span>

1. <span data-ttu-id="bfcf2-433">Nel portale di Azure selezionare il profilo di Gestione traffico.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-433">In the Azure portal, select your Traffic Manager profile.</span></span>

    ![Endpoint di Gestione traffico nel portale di Azure](media/solution-deployment-guide-hybrid/image20.png)

2. <span data-ttu-id="bfcf2-435">Selezionare **Endpoint** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-435">Select **Endpoints** .</span></span>
3. <span data-ttu-id="bfcf2-436">Selezionare l' **Endpoint Azure** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-436">Select the **Azure endpoint** .</span></span>
4. <span data-ttu-id="bfcf2-437">In **Stato** selezionare **Abilitato** e quindi selezionare **Salva** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-437">Under **Status** , select **Enabled** , and then select **Save** .</span></span>

    ![Abilitare l'endpoint di Azure nel portale di Azure](media/solution-deployment-guide-hybrid/image23.png)

5. <span data-ttu-id="bfcf2-439">In **Endpoint** per il profilo di Gestione traffico selezionare **Endpoint esterno** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-439">On **Endpoints** for the Traffic Manager profile, select **External endpoint** .</span></span>
6. <span data-ttu-id="bfcf2-440">In **Stato** selezionare **Disabilitato** e quindi selezionare **Salva** .</span><span class="sxs-lookup"><span data-stu-id="bfcf2-440">Under **Status** , select **Disabled** , and then select **Save** .</span></span>

    ![Disabilitare l'endpoint dell'hub di Azure Stack nel portale di Azure](media/solution-deployment-guide-hybrid/image24.png)

<span data-ttu-id="bfcf2-442">Dopo aver configurato gli endpoint, il traffico delle app viene indirizzato all'app Web di cui sono state aumentate le istanze anziché all'app Web dell'hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-442">After the endpoints are configured, app traffic goes to your Azure scale-out web app instead of the Azure Stack Hub web app.</span></span>

 ![Endpoint modificati nel traffico dell'app Web di Azure](media/solution-deployment-guide-hybrid/image25.png)

<span data-ttu-id="bfcf2-444">Per invertire di nuovo il flusso verso l'hub di Azure Stack, eseguire i passaggi precedenti per:</span><span class="sxs-lookup"><span data-stu-id="bfcf2-444">To reverse the flow back to Azure Stack Hub, use the previous steps to:</span></span>

- <span data-ttu-id="bfcf2-445">Abilitare l'endpoint dell'hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-445">Enable the Azure Stack Hub endpoint.</span></span>
- <span data-ttu-id="bfcf2-446">Disabilitare l'endpoint di Azure.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-446">Disable the Azure endpoint.</span></span>

### <a name="configure-automatic-switching-between-azure-and-azure-stack-hub"></a><span data-ttu-id="bfcf2-447">Configurare la commutazione automatica tra Azure e l'hub di Azure Stack</span><span class="sxs-lookup"><span data-stu-id="bfcf2-447">Configure automatic switching between Azure and Azure Stack Hub</span></span>

<span data-ttu-id="bfcf2-448">È anche possibile usare il monitoraggio di Application Insights se l'app è in esecuzione in un ambiente [serverless](https://azure.microsoft.com/overview/serverless-computing/) fornito da Funzioni di Azure.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-448">You can also use Application Insights monitoring if your app runs in a [serverless](https://azure.microsoft.com/overview/serverless-computing/) environment provided by Azure Functions.</span></span>

<span data-ttu-id="bfcf2-449">In questo scenario è possibile configurare Application Insights per l'uso di un webhook che chiama un'app per le funzioni.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-449">In this scenario, you can configure Application Insights to use a webhook that calls a function app.</span></span> <span data-ttu-id="bfcf2-450">Questa app abilita o disabilita automaticamente un endpoint in risposta a un avviso.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-450">This app automatically enables or disables an endpoint in response to an alert.</span></span>

<span data-ttu-id="bfcf2-451">Usare i passaggi seguenti come guida per configurare la commutazione automatica del traffico.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-451">Use the following steps as a guide to configure automatic traffic switching.</span></span>

1. <span data-ttu-id="bfcf2-452">Creare un'app per le funzioni di Azure.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-452">Create an Azure Function app.</span></span>
2. <span data-ttu-id="bfcf2-453">Creare una funzione attivata da HTTP.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-453">Create an HTTP-triggered function.</span></span>
3. <span data-ttu-id="bfcf2-454">Importare gli SDK di Azure per Resource Manager, App Web e Gestione traffico.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-454">Import the Azure SDKs for Resource Manager, Web Apps, and Traffic Manager.</span></span>
4. <span data-ttu-id="bfcf2-455">Sviluppare codice per:</span><span class="sxs-lookup"><span data-stu-id="bfcf2-455">Develop code to:</span></span>

   - <span data-ttu-id="bfcf2-456">Eseguire l'autenticazione nella sottoscrizione di Azure.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-456">Authenticate to your Azure subscription.</span></span>
   - <span data-ttu-id="bfcf2-457">Usare un parametro che commuta gli endpoint di Gestione traffico per indirizzare il traffico ad Azure o all'hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-457">Use a parameter that toggles the Traffic Manager endpoints to direct traffic to Azure or Azure Stack Hub.</span></span>

5. <span data-ttu-id="bfcf2-458">Salvare il codice e aggiungere l'URL dell'app per le funzioni con i parametri appropriati alla sezione **Webhook** delle impostazioni delle regole degli avvisi di Application Insights.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-458">Save your code and add the function app's URL with the appropriate parameters to the **Webhook** section of the Application Insights alert rule settings.</span></span>
6. <span data-ttu-id="bfcf2-459">Il traffico viene reindirizzato automaticamente quando viene attivato un avviso di Application Insights.</span><span class="sxs-lookup"><span data-stu-id="bfcf2-459">Traffic is automatically redirected when an Application Insights alert fires.</span></span>

## <a name="next-steps"></a><span data-ttu-id="bfcf2-460">Passaggi successivi</span><span class="sxs-lookup"><span data-stu-id="bfcf2-460">Next steps</span></span>

- <span data-ttu-id="bfcf2-461">Per altre informazioni sui modelli cloud di Azure, vedere [Modelli di progettazione cloud](/azure/architecture/patterns).</span><span class="sxs-lookup"><span data-stu-id="bfcf2-461">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](/azure/architecture/patterns).</span></span>
