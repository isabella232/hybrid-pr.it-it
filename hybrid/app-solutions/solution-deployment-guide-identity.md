---
title: Configurare l'identità del cloud ibrido per le app di Azure e dell'hub di Azure Stack
description: Informazioni su come configurare l'identità del cloud ibrido per le app di Azure e dell'hub di Azure Stack.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: cfe2001fcbf91f3ec0d94a7ee257b23ba89065ee
ms.sourcegitcommit: 962334135b63ac99c715e7bc8fb9282648ba63c9
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 03/23/2021
ms.locfileid: "104895347"
---
# <a name="configure-hybrid-cloud-identity-for-azure-and-azure-stack-hub-apps"></a><span data-ttu-id="16dd9-103">Configurare l'identità del cloud ibrido per le app di Azure e dell'hub di Azure Stack</span><span class="sxs-lookup"><span data-stu-id="16dd9-103">Configure hybrid cloud identity for Azure and Azure Stack Hub apps</span></span>

<span data-ttu-id="16dd9-104">Informazioni su come configurare un'identità del cloud ibrido per le app di Azure e dell'hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="16dd9-104">Learn how to configure a hybrid cloud identity for your Azure and Azure Stack Hub apps.</span></span>

<span data-ttu-id="16dd9-105">Sono disponibili due opzioni per concedere l'accesso alle app in Azure globale e nell'hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="16dd9-105">You have two options for granting access to your apps in both global Azure and Azure Stack Hub.</span></span>

 * <span data-ttu-id="16dd9-106">Quando l'hub di Azure Stack ha una connessione continua a Internet, è possibile usare Azure Active Directory (Azure AD).</span><span class="sxs-lookup"><span data-stu-id="16dd9-106">When Azure Stack Hub has a continuous connection to the internet, you can use Azure Active Directory (Azure AD).</span></span>
 * <span data-ttu-id="16dd9-107">Quando l'hub di Azure Stack è disconnesso da Internet, è possibile usare Azure Directory Federated Services (AD FS).</span><span class="sxs-lookup"><span data-stu-id="16dd9-107">When Azure Stack Hub is disconnected from the internet, you can use Azure Directory Federated Services (AD FS).</span></span>

<span data-ttu-id="16dd9-108">Con le entità servizio è possibile concedere l'accesso alle app dell'hub di Azure Stack per la distribuzione o la configurazione nell'hub di Azure Stack tramite Azure Resource Manager.</span><span class="sxs-lookup"><span data-stu-id="16dd9-108">You use service principals to grant access to your Azure Stack Hub apps for deployment or configuration using the Azure Resource Manager in Azure Stack Hub.</span></span>

<span data-ttu-id="16dd9-109">In questa soluzione verrà compilato un ambiente di esempio per:</span><span class="sxs-lookup"><span data-stu-id="16dd9-109">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="16dd9-110">Definire un'identità ibrida in Azure globale e nell'hub di Azure Stack</span><span class="sxs-lookup"><span data-stu-id="16dd9-110">Establish a hybrid identity in global Azure and Azure Stack Hub</span></span>
> - <span data-ttu-id="16dd9-111">Recuperare un token per accedere all'API dell'hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="16dd9-111">Retrieve a token to access the Azure Stack Hub API.</span></span>

<span data-ttu-id="16dd9-112">Per i passaggi di questa soluzione sono necessarie le autorizzazioni dell'operatore dell'hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="16dd9-112">You must have Azure Stack Hub operator permissions for the steps in this solution.</span></span>

> [!Tip]  
> <span data-ttu-id="16dd9-113">![Diagramma dei concetti sulle app ibride](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="16dd9-113">![Hybrid pillars diagram](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="16dd9-114">L'hub di Microsoft Azure Stack è un'estensione di Azure,</span><span class="sxs-lookup"><span data-stu-id="16dd9-114">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="16dd9-115">che offre all'ambiente locale l'agilità e l'innovazione del cloud computing, abilitando l'unico cloud ibrido che consente di creare e distribuire ovunque app ibride.</span><span class="sxs-lookup"><span data-stu-id="16dd9-115">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="16dd9-116">L'articolo [Considerazioni per la progettazione di app ibride](overview-app-design-considerations.md) esamina i concetti fondamentali per la qualità del software (posizionamento, scalabilità, disponibilità, resilienza, gestibilità e sicurezza) per la progettazione, la distribuzione e la gestione di app ibride.</span><span class="sxs-lookup"><span data-stu-id="16dd9-116">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="16dd9-117">Le considerazioni per la progettazione consentono di ottimizzare la progettazione delle app ibride, riducendo al minimo i rischi negli ambienti di produzione.</span><span class="sxs-lookup"><span data-stu-id="16dd9-117">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="create-a-service-principal-for-azure-ad-in-the-portal"></a><span data-ttu-id="16dd9-118">Creare un'entità servizio per Azure AD nel portale</span><span class="sxs-lookup"><span data-stu-id="16dd9-118">Create a service principal for Azure AD in the portal</span></span>

<span data-ttu-id="16dd9-119">Se l'hub di Azure Stack è stato distribuito usando Azure AD come archivio di identità, è possibile creare entità servizio esattamente come per Azure.</span><span class="sxs-lookup"><span data-stu-id="16dd9-119">If you deployed Azure Stack Hub using Azure AD as the identity store, you can create service principals just like you do for Azure.</span></span> <span data-ttu-id="16dd9-120">L'articolo [Usare un'identità dell'app per accedere alle risorse](/azure-stack/operator/azure-stack-create-service-principals#manage-an-azure-ad-app-identity) illustra come eseguire la procedura tramite il portale.</span><span class="sxs-lookup"><span data-stu-id="16dd9-120">[Use an app identity to access resources](/azure-stack/operator/azure-stack-create-service-principals#manage-an-azure-ad-app-identity) shows you how to perform the steps through the portal.</span></span> <span data-ttu-id="16dd9-121">Prima di iniziare, verificare di avere le [autorizzazioni di Azure AD necessarie](/azure/azure-resource-manager/resource-group-create-service-principal-portal#required-permissions).</span><span class="sxs-lookup"><span data-stu-id="16dd9-121">Be sure you have the [required Azure AD permissions](/azure/azure-resource-manager/resource-group-create-service-principal-portal#required-permissions) before beginning.</span></span>

## <a name="create-a-service-principal-for-ad-fs-using-powershell"></a><span data-ttu-id="16dd9-122">Creare un'entità servizio per AD FS usando PowerShell</span><span class="sxs-lookup"><span data-stu-id="16dd9-122">Create a service principal for AD FS using PowerShell</span></span>

<span data-ttu-id="16dd9-123">Se l'hub di Azure Stack è stato distribuito con AD FS, è possibile usare PowerShell per creare un'entità servizio, assegnare un ruolo per l'accesso e accedere da PowerShell usando tale identità.</span><span class="sxs-lookup"><span data-stu-id="16dd9-123">If you deployed Azure Stack Hub with AD FS, you can use PowerShell to create a service principal, assign a role for access, and sign in from PowerShell using that identity.</span></span> <span data-ttu-id="16dd9-124">L'articolo [Usare un'identità dell'app per accedere alle risorse](/azure-stack/operator/azure-stack-create-service-principals#manage-an-ad-fs-app-identity) illustra come eseguire la procedura necessaria usando PowerShell.</span><span class="sxs-lookup"><span data-stu-id="16dd9-124">[Use an app identity to access resources](/azure-stack/operator/azure-stack-create-service-principals#manage-an-ad-fs-app-identity) shows you how to perform the required steps using PowerShell.</span></span>

## <a name="using-the-azure-stack-hub-api"></a><span data-ttu-id="16dd9-125">Uso dell'API dell'hub di Azure Stack</span><span class="sxs-lookup"><span data-stu-id="16dd9-125">Using the Azure Stack Hub API</span></span>

<span data-ttu-id="16dd9-126">La soluzione [API dell'hub di Azure Stack](/azure-stack/user/azure-stack-rest-api-use) illustra il processo di recupero di un token per accedere all'API dell'hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="16dd9-126">The [Azure Stack Hub API](/azure-stack/user/azure-stack-rest-api-use)  solution walks you through the process of retrieving a token to access the Azure Stack Hub API.</span></span>

## <a name="connect-to-azure-stack-hub-using-powershell"></a><span data-ttu-id="16dd9-127">Connettersi all'hub di Azure Stack con PowerShell</span><span class="sxs-lookup"><span data-stu-id="16dd9-127">Connect to Azure Stack Hub using PowerShell</span></span>

<span data-ttu-id="16dd9-128">La guida introduttiva per [iniziare a usare PowerShell nell'hub di Azure Stack](/azure-stack/operator/azure-stack-powershell-install) illustra la procedura necessaria per installare Azure PowerShell e connettersi all'installazione dell'hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="16dd9-128">The quickstart [to get up and running with PowerShell in Azure Stack Hub](/azure-stack/operator/azure-stack-powershell-install) walks you through the steps needed to install Azure PowerShell and connect to your Azure Stack Hub installation.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="16dd9-129">Prerequisiti</span><span class="sxs-lookup"><span data-stu-id="16dd9-129">Prerequisites</span></span>

<span data-ttu-id="16dd9-130">È necessaria un'installazione dell'hub di Azure Stack connessa ad Azure AD con una sottoscrizione a cui è possibile accedere.</span><span class="sxs-lookup"><span data-stu-id="16dd9-130">You need an Azure Stack Hub installation connected to Azure AD with a subscription you can access.</span></span> <span data-ttu-id="16dd9-131">Se non si ha un'installazione dell'hub di Azure Stack, seguire queste istruzioni per configurare un [Azure Stack Development Kit (ASDK)](/azure-stack/asdk/asdk-install).</span><span class="sxs-lookup"><span data-stu-id="16dd9-131">If you don't have an Azure Stack Hub installation, you can use these instructions to set up an [Azure Stack Development Kit (ASDK)](/azure-stack/asdk/asdk-install).</span></span>

#### <a name="connect-to-azure-stack-hub-using-code"></a><span data-ttu-id="16dd9-132">Connettersi all'hub di Azure Stack tramite codice</span><span class="sxs-lookup"><span data-stu-id="16dd9-132">Connect to Azure Stack Hub using code</span></span>

<span data-ttu-id="16dd9-133">Per connettersi all'hub di Azure Stack tramite codice, usare l'API degli endpoint di Azure Resource Manager per ottenere gli endpoint di autenticazione e di Graph per l'installazione dell'hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="16dd9-133">To connect to Azure Stack Hub using code, use the Azure Resource Manager endpoints API to get the authentication and graph endpoints for your Azure Stack Hub installation.</span></span> <span data-ttu-id="16dd9-134">Eseguire quindi l'autenticazione tramite le richieste REST.</span><span class="sxs-lookup"><span data-stu-id="16dd9-134">Then authenticate using REST requests.</span></span> <span data-ttu-id="16dd9-135">Un'applicazione client di esempio è disponibile in [GitHub](https://github.com/shriramnat/HybridARMApplication).</span><span class="sxs-lookup"><span data-stu-id="16dd9-135">You can find a sample client application on [GitHub](https://github.com/shriramnat/HybridARMApplication).</span></span>

>[!Note]
><span data-ttu-id="16dd9-136">A meno che Azure SDK per il linguaggio scelto non supporti i profili API di Azure, l'SDK potrebbe non funzionare con l'hub di Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="16dd9-136">Unless the Azure SDK for your language of choice supports Azure API Profiles, the SDK may not work with Azure Stack Hub.</span></span> <span data-ttu-id="16dd9-137">Per altre informazioni sui profili API di Azure, vedere l'articolo [Gestire i profili delle versioni dell'API](/azure-stack/user/azure-stack-version-profiles).</span><span class="sxs-lookup"><span data-stu-id="16dd9-137">To learn more about Azure API Profiles, see the [manage API version profiles](/azure-stack/user/azure-stack-version-profiles) article.</span></span>

## <a name="next-steps"></a><span data-ttu-id="16dd9-138">Passaggi successivi</span><span class="sxs-lookup"><span data-stu-id="16dd9-138">Next steps</span></span>

- <span data-ttu-id="16dd9-139">Per altre informazioni su come viene gestita l'identità nell'hub di Azure Stack, vedere [Architettura dell'identità per l'hub di Azure Stack](/azure-stack/operator/azure-stack-identity-architecture).</span><span class="sxs-lookup"><span data-stu-id="16dd9-139">To learn more about how identity is handled in Azure Stack Hub, see [Identity architecture for Azure Stack Hub](/azure-stack/operator/azure-stack-identity-architecture).</span></span>
- <span data-ttu-id="16dd9-140">Per altre informazioni sui modelli cloud di Azure, vedere [Modelli di progettazione cloud](/azure/architecture/patterns).</span><span class="sxs-lookup"><span data-stu-id="16dd9-140">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](/azure/architecture/patterns).</span></span>
