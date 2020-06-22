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
# <a name="configure-hybrid-cloud-identity-for-azure-and-azure-stack-hub-apps"></a>Configurare l'identità del cloud ibrido per le app di Azure e Azure Stack Hub

Informazioni su come configurare un'identità cloud ibrida per le app di Azure e dell'hub Azure Stack.

Sono disponibili due opzioni per concedere l'accesso alle app in Azure globale e nell'hub Azure Stack.

 * Quando Azure Stack Hub dispone di una connessione continua a Internet, è possibile usare Azure Active Directory (Azure AD).
 * Quando Azure Stack Hub è disconnesso da Internet, è possibile usare i servizi federativi di Azure directory (AD FS).

Usare le entità servizio per concedere l'accesso alle app hub Azure Stack per la distribuzione o la configurazione usando il Azure Resource Manager nell'hub Azure Stack.

In questa soluzione verrà compilato un ambiente di esempio per:

> [!div class="checklist"]
> - Definire un'identità ibrida nell'hub globale di Azure e Azure Stack
> - Recuperare un token per accedere all'API dell'hub Azure Stack.

È necessario disporre delle autorizzazioni dell'operatore Azure Stack hub per i passaggi di questa soluzione.

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> Microsoft Azure Stack Hub è un'estensione di Azure. Azure Stack Hub offre l'agilità e l'innovazione di cloud computing all'ambiente locale, abilitando l'unico Cloud ibrido che consente di creare e distribuire app ibride ovunque.  
> 
> L'articolo [considerazioni sulla progettazione di app ibride](overview-app-design-considerations.md) esamina i pilastri della qualità del software (posizionamento, scalabilità, disponibilità, resilienza, gestibilità e sicurezza) per la progettazione, la distribuzione e la gestione di app ibride. Le considerazioni di progettazione consentono di ottimizzare la progettazione delle app ibride, riducendo al minimo le esigenze negli ambienti di produzione.

## <a name="create-a-service-principal-for-azure-ad-in-the-portal"></a>Creare un'entità servizio per Azure AD nel portale

Se è stato distribuito Azure Stack hub usando Azure AD come archivio identità, è possibile creare entità servizio esattamente come per Azure. [Usare un'identità dell'app per accedere alle risorse](/azure-stack/operator/azure-stack-create-service-principals.md#manage-an-azure-ad-app-identity) Mostra come eseguire i passaggi tramite il portale. Prima di iniziare, assicurarsi di disporre delle [autorizzazioni Azure ad necessarie](/azure/azure-resource-manager/resource-group-create-service-principal-portal#required-permissions) .

## <a name="create-a-service-principal-for-ad-fs-using-powershell"></a>Creare un'entità servizio per AD FS usando PowerShell

Se è stato distribuito Azure Stack Hub con AD FS, è possibile usare PowerShell per creare un'entità servizio, assegnare un ruolo per l'accesso e accedere da PowerShell usando tale identità. [Usare un'identità dell'app per accedere alle risorse](/azure-stack/operator/azure-stack-create-service-principals.md#manage-an-ad-fs-app-identity) Mostra come eseguire i passaggi necessari usando PowerShell.

## <a name="using-the-azure-stack-hub-api"></a>Uso dell'API dell'hub Azure Stack

La soluzione [API hub Azure stack](/azure-stack/user/azure-stack-rest-api-use.md) illustra il processo di recupero di un token per accedere all'api Hub Azure stack.

## <a name="connect-to-azure-stack-hub-using-powershell"></a>Connettersi all'hub Azure Stack usando PowerShell

La Guida introduttiva [per iniziare a usare PowerShell nell'Hub Azure stack](/azure-stack/operator/azure-stack-powershell-install.md) illustra i passaggi necessari per installare Azure PowerShell e connettersi all'installazione dell'hub di Azure stack.

### <a name="prerequisites"></a>Prerequisiti

È necessaria un'installazione dell'hub Azure Stack connessa a Azure AD con una sottoscrizione a cui è possibile accedere. Se non si dispone di un'installazione dell'hub Azure Stack, è possibile utilizzare queste istruzioni per configurare una [Azure stack Development Kit (Gabriele)](/azure-stack/asdk/asdk-install.md).

#### <a name="connect-to-azure-stack-hub-using-code"></a>Connettersi all'hub Azure Stack usando il codice

Per connettersi all'hub Azure Stack usando il codice, usare l'API Azure Resource Manager Endpoints per ottenere gli endpoint di autenticazione e Graph per l'installazione dell'hub Azure Stack. Eseguire quindi l'autenticazione tramite richieste REST. È possibile trovare un'applicazione client di esempio in [GitHub](https://github.com/shriramnat/HybridARMApplication).

>[!Note]
>A meno che Azure SDK per il linguaggio preferito non supporti i profili API di Azure, è possibile che l'SDK non funzioni con l'hub Azure Stack. Per altre informazioni sui profili API di Azure, vedere l'articolo [gestire i profili di versione dell'API](/azure-stack/user/azure-stack-version-profiles.md) .

## <a name="next-steps"></a>Passaggi successivi

- Per altre informazioni sul modo in cui viene gestita l'identità nell'hub Azure Stack, vedere [architettura delle identità per l'hub Azure stack](/azure-stack/operator/azure-stack-identity-architecture.md).
- Per altre informazioni sui modelli cloud di Azure, vedere [modelli di progettazione cloud](https://docs.microsoft.com/azure/architecture/patterns).
