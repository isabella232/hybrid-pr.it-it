---
title: Configurare l'identità del cloud ibrido per le app di Azure e Azure Stack Hub
description: Informazioni su come configurare l'identità del cloud ibrido per Azure e le app hub Azure Stack.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 1b3c683dd3e4a68413f83fd3cc129d6e6f594e1b
ms.sourcegitcommit: bb3e40b210f86173568a47ba18c3cc50d4a40607
ms.translationtype: MT
ms.contentlocale: it-IT
ms.lasthandoff: 06/17/2020
ms.locfileid: "84911314"
---
# <a name="configure-hybrid-cloud-identity-for-azure-and-azure-stack-hub-apps"></a><span data-ttu-id="2021f-103">Configurare l'identità del cloud ibrido per le app di Azure e Azure Stack Hub</span><span class="sxs-lookup"><span data-stu-id="2021f-103">Configure hybrid cloud identity for Azure and Azure Stack Hub apps</span></span>

<span data-ttu-id="2021f-104">Informazioni su come configurare un'identità cloud ibrida per le app di Azure e dell'hub Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="2021f-104">Learn how to configure a hybrid cloud identity for your Azure and Azure Stack Hub apps.</span></span>

<span data-ttu-id="2021f-105">Sono disponibili due opzioni per concedere l'accesso alle app in Azure globale e nell'hub Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="2021f-105">You have two options for granting access to your apps in both global Azure and Azure Stack Hub.</span></span>

 * <span data-ttu-id="2021f-106">Quando Azure Stack Hub dispone di una connessione continua a Internet, è possibile usare Azure Active Directory (Azure AD).</span><span class="sxs-lookup"><span data-stu-id="2021f-106">When Azure Stack Hub has a continuous connection to the internet, you can use Azure Active Directory (Azure AD).</span></span>
 * <span data-ttu-id="2021f-107">Quando Azure Stack Hub è disconnesso da Internet, è possibile usare i servizi federativi di Azure directory (AD FS).</span><span class="sxs-lookup"><span data-stu-id="2021f-107">When Azure Stack Hub is disconnected from the internet, you can use Azure Directory Federated Services (AD FS).</span></span>

<span data-ttu-id="2021f-108">Usare le entità servizio per concedere l'accesso alle app hub Azure Stack per la distribuzione o la configurazione usando il Azure Resource Manager nell'hub Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="2021f-108">You use service principals to grant access to your Azure Stack Hub apps for deployment or configuration using the Azure Resource Manager in Azure Stack Hub.</span></span>

<span data-ttu-id="2021f-109">In questa soluzione verrà compilato un ambiente di esempio per:</span><span class="sxs-lookup"><span data-stu-id="2021f-109">In this solution, you'll build a sample environment to:</span></span>

> [!div class="checklist"]
> - <span data-ttu-id="2021f-110">Definire un'identità ibrida nell'hub globale di Azure e Azure Stack</span><span class="sxs-lookup"><span data-stu-id="2021f-110">Establish a hybrid identity in global Azure and Azure Stack Hub</span></span>
> - <span data-ttu-id="2021f-111">Recuperare un token per accedere all'API dell'hub Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="2021f-111">Retrieve a token to access the Azure Stack Hub API.</span></span>

<span data-ttu-id="2021f-112">È necessario disporre delle autorizzazioni dell'operatore Azure Stack hub per i passaggi di questa soluzione.</span><span class="sxs-lookup"><span data-stu-id="2021f-112">You must have Azure Stack Hub operator permissions for the steps in this solution.</span></span>

> [!Tip]  
> <span data-ttu-id="2021f-113">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span><span class="sxs-lookup"><span data-stu-id="2021f-113">![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)</span></span>  
> <span data-ttu-id="2021f-114">Microsoft Azure Stack Hub è un'estensione di Azure.</span><span class="sxs-lookup"><span data-stu-id="2021f-114">Microsoft Azure Stack Hub is an extension of Azure.</span></span> <span data-ttu-id="2021f-115">Azure Stack Hub offre l'agilità e l'innovazione di cloud computing all'ambiente locale, abilitando l'unico Cloud ibrido che consente di creare e distribuire app ibride ovunque.</span><span class="sxs-lookup"><span data-stu-id="2021f-115">Azure Stack Hub brings the agility and innovation of cloud computing to your on-premises environment, enabling the only hybrid cloud that lets you build and deploy hybrid apps anywhere.</span></span>  
> 
> <span data-ttu-id="2021f-116">L'articolo [considerazioni sulla progettazione di app ibride](overview-app-design-considerations.md) esamina i pilastri della qualità del software (posizionamento, scalabilità, disponibilità, resilienza, gestibilità e sicurezza) per la progettazione, la distribuzione e la gestione di app ibride.</span><span class="sxs-lookup"><span data-stu-id="2021f-116">The article [Hybrid app design considerations](overview-app-design-considerations.md) reviews pillars of software quality (placement, scalability, availability, resiliency, manageability, and security) for designing, deploying, and operating hybrid apps.</span></span> <span data-ttu-id="2021f-117">Le considerazioni di progettazione consentono di ottimizzare la progettazione delle app ibride, riducendo al minimo le esigenze negli ambienti di produzione.</span><span class="sxs-lookup"><span data-stu-id="2021f-117">The design considerations assist in optimizing hybrid app design, minimizing challenges in production environments.</span></span>

## <a name="create-a-service-principal-for-azure-ad-in-the-portal"></a><span data-ttu-id="2021f-118">Creare un'entità servizio per Azure AD nel portale</span><span class="sxs-lookup"><span data-stu-id="2021f-118">Create a service principal for Azure AD in the portal</span></span>

<span data-ttu-id="2021f-119">Se è stato distribuito Azure Stack hub usando Azure AD come archivio identità, è possibile creare entità servizio esattamente come per Azure.</span><span class="sxs-lookup"><span data-stu-id="2021f-119">If you deployed Azure Stack Hub using Azure AD as the identity store, you can create service principals just like you do for Azure.</span></span> <span data-ttu-id="2021f-120">[Usare un'identità dell'app per accedere alle risorse](/azure-stack/operator/azure-stack-create-service-principals.md#manage-an-azure-ad-app-identity) Mostra come eseguire i passaggi tramite il portale.</span><span class="sxs-lookup"><span data-stu-id="2021f-120">[Use an app identity to access resources](/azure-stack/operator/azure-stack-create-service-principals.md#manage-an-azure-ad-app-identity) shows you how to perform the steps through the portal.</span></span> <span data-ttu-id="2021f-121">Prima di iniziare, assicurarsi di disporre delle [autorizzazioni Azure ad necessarie](/azure/azure-resource-manager/resource-group-create-service-principal-portal#required-permissions) .</span><span class="sxs-lookup"><span data-stu-id="2021f-121">Be sure you have the [required Azure AD permissions](/azure/azure-resource-manager/resource-group-create-service-principal-portal#required-permissions) before beginning.</span></span>

## <a name="create-a-service-principal-for-ad-fs-using-powershell"></a><span data-ttu-id="2021f-122">Creare un'entità servizio per AD FS usando PowerShell</span><span class="sxs-lookup"><span data-stu-id="2021f-122">Create a service principal for AD FS using PowerShell</span></span>

<span data-ttu-id="2021f-123">Se è stato distribuito Azure Stack Hub con AD FS, è possibile usare PowerShell per creare un'entità servizio, assegnare un ruolo per l'accesso e accedere da PowerShell usando tale identità.</span><span class="sxs-lookup"><span data-stu-id="2021f-123">If you deployed Azure Stack Hub with AD FS, you can use PowerShell to create a service principal, assign a role for access, and sign in from PowerShell using that identity.</span></span> <span data-ttu-id="2021f-124">[Usare un'identità dell'app per accedere alle risorse](/azure-stack/operator/azure-stack-create-service-principals.md#manage-an-ad-fs-app-identity) Mostra come eseguire i passaggi necessari usando PowerShell.</span><span class="sxs-lookup"><span data-stu-id="2021f-124">[Use an app identity to access resources](/azure-stack/operator/azure-stack-create-service-principals.md#manage-an-ad-fs-app-identity) shows you how to perform the required steps using PowerShell.</span></span>

## <a name="using-the-azure-stack-hub-api"></a><span data-ttu-id="2021f-125">Uso dell'API dell'hub Azure Stack</span><span class="sxs-lookup"><span data-stu-id="2021f-125">Using the Azure Stack Hub API</span></span>

<span data-ttu-id="2021f-126">La soluzione [API hub Azure stack](/azure-stack/user/azure-stack-rest-api-use.md) illustra il processo di recupero di un token per accedere all'api Hub Azure stack.</span><span class="sxs-lookup"><span data-stu-id="2021f-126">The [Azure Stack Hub API](/azure-stack/user/azure-stack-rest-api-use.md)  solution walks you through the process of retrieving a token to access the Azure Stack Hub API.</span></span>

## <a name="connect-to-azure-stack-hub-using-powershell"></a><span data-ttu-id="2021f-127">Connettersi all'hub Azure Stack usando PowerShell</span><span class="sxs-lookup"><span data-stu-id="2021f-127">Connect to Azure Stack Hub using PowerShell</span></span>

<span data-ttu-id="2021f-128">La Guida introduttiva [per iniziare a usare PowerShell nell'Hub Azure stack](/azure-stack/operator/azure-stack-powershell-install.md) illustra i passaggi necessari per installare Azure PowerShell e connettersi all'installazione dell'hub di Azure stack.</span><span class="sxs-lookup"><span data-stu-id="2021f-128">The quickstart [to get up and running with PowerShell in Azure Stack Hub](/azure-stack/operator/azure-stack-powershell-install.md) walks you through the steps needed to install Azure PowerShell and connect to your Azure Stack Hub installation.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="2021f-129">Prerequisiti</span><span class="sxs-lookup"><span data-stu-id="2021f-129">Prerequisites</span></span>

<span data-ttu-id="2021f-130">È necessaria un'installazione dell'hub Azure Stack connessa a Azure AD con una sottoscrizione a cui è possibile accedere.</span><span class="sxs-lookup"><span data-stu-id="2021f-130">You need an Azure Stack Hub installation connected to Azure AD with a subscription you can access.</span></span> <span data-ttu-id="2021f-131">Se non si dispone di un'installazione dell'hub Azure Stack, è possibile utilizzare queste istruzioni per configurare una [Azure stack Development Kit (Gabriele)](/azure-stack/asdk/asdk-install.md).</span><span class="sxs-lookup"><span data-stu-id="2021f-131">If you don't have an Azure Stack Hub installation, you can use these instructions to set up an [Azure Stack Development Kit (ASDK)](/azure-stack/asdk/asdk-install.md).</span></span>

#### <a name="connect-to-azure-stack-hub-using-code"></a><span data-ttu-id="2021f-132">Connettersi all'hub Azure Stack usando il codice</span><span class="sxs-lookup"><span data-stu-id="2021f-132">Connect to Azure Stack Hub using code</span></span>

<span data-ttu-id="2021f-133">Per connettersi all'hub Azure Stack usando il codice, usare l'API Azure Resource Manager Endpoints per ottenere gli endpoint di autenticazione e Graph per l'installazione dell'hub Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="2021f-133">To connect to Azure Stack Hub using code, use the Azure Resource Manager endpoints API to get the authentication and graph endpoints for your Azure Stack Hub installation.</span></span> <span data-ttu-id="2021f-134">Eseguire quindi l'autenticazione tramite richieste REST.</span><span class="sxs-lookup"><span data-stu-id="2021f-134">Then authenticate using REST requests.</span></span> <span data-ttu-id="2021f-135">È possibile trovare un'applicazione client di esempio in [GitHub](https://github.com/shriramnat/HybridARMApplication).</span><span class="sxs-lookup"><span data-stu-id="2021f-135">You can find a sample client application on [GitHub](https://github.com/shriramnat/HybridARMApplication).</span></span>

>[!Note]
><span data-ttu-id="2021f-136">A meno che Azure SDK per il linguaggio preferito non supporti i profili API di Azure, è possibile che l'SDK non funzioni con l'hub Azure Stack.</span><span class="sxs-lookup"><span data-stu-id="2021f-136">Unless the Azure SDK for your language of choice supports Azure API Profiles, the SDK may not work with Azure Stack Hub.</span></span> <span data-ttu-id="2021f-137">Per altre informazioni sui profili API di Azure, vedere l'articolo [gestire i profili di versione dell'API](/azure-stack/user/azure-stack-version-profiles.md) .</span><span class="sxs-lookup"><span data-stu-id="2021f-137">To learn more about Azure API Profiles, see the [manage API version profiles](/azure-stack/user/azure-stack-version-profiles.md) article.</span></span>

## <a name="next-steps"></a><span data-ttu-id="2021f-138">Passaggi successivi</span><span class="sxs-lookup"><span data-stu-id="2021f-138">Next steps</span></span>

- <span data-ttu-id="2021f-139">Per altre informazioni sul modo in cui viene gestita l'identità nell'hub Azure Stack, vedere [architettura delle identità per l'hub Azure stack](/azure-stack/operator/azure-stack-identity-architecture.md).</span><span class="sxs-lookup"><span data-stu-id="2021f-139">To learn more about how identity is handled in Azure Stack Hub, see [Identity architecture for Azure Stack Hub](/azure-stack/operator/azure-stack-identity-architecture.md).</span></span>
- <span data-ttu-id="2021f-140">Per altre informazioni sui modelli cloud di Azure, vedere [modelli di progettazione cloud](https://docs.microsoft.com/azure/architecture/patterns).</span><span class="sxs-lookup"><span data-stu-id="2021f-140">To learn more about Azure Cloud Patterns, see [Cloud Design Patterns](https://docs.microsoft.com/azure/architecture/patterns).</span></span>
