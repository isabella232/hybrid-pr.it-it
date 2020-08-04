---
title: Configurare l'identità del cloud ibrido per le app di Azure e dell'hub di Azure Stack
description: Informazioni su come configurare l'identità del cloud ibrido per le app di Azure e dell'hub di Azure Stack.
author: BryanLa
ms.topic: article
ms.date: 11/05/2019
ms.author: bryanla
ms.reviewer: anajod
ms.lastreviewed: 11/05/2019
ms.openlocfilehash: 650eef0f144ecafab4586d93f72e1defdf4a61ce
ms.sourcegitcommit: d2def847937178f68177507be151df2aa8e25d53
ms.translationtype: HT
ms.contentlocale: it-IT
ms.lasthandoff: 07/20/2020
ms.locfileid: "86477253"
---
# <a name="configure-hybrid-cloud-identity-for-azure-and-azure-stack-hub-apps"></a>Configurare l'identità del cloud ibrido per le app di Azure e dell'hub di Azure Stack

Informazioni su come configurare un'identità del cloud ibrido per le app di Azure e dell'hub di Azure Stack.

Sono disponibili due opzioni per concedere l'accesso alle app in Azure globale e nell'hub di Azure Stack.

 * Quando l'hub di Azure Stack ha una connessione continua a Internet, è possibile usare Azure Active Directory (Azure AD).
 * Quando l'hub di Azure Stack è disconnesso da Internet, è possibile usare Azure Directory Federated Services (AD FS).

Con le entità servizio è possibile concedere l'accesso alle app dell'hub di Azure Stack per la distribuzione o la configurazione nell'hub di Azure Stack tramite Azure Resource Manager.

In questa soluzione verrà compilato un ambiente di esempio per:

> [!div class="checklist"]
> - Definire un'identità ibrida in Azure globale e nell'hub di Azure Stack
> - Recuperare un token per accedere all'API dell'hub di Azure Stack.

Per i passaggi di questa soluzione sono necessarie le autorizzazioni dell'operatore dell'hub di Azure Stack.

> [!Tip]  
> ![hybrid-pillars.png](./media/solution-deployment-guide-cross-cloud-scaling/hybrid-pillars.png)  
> L'hub di Microsoft Azure Stack è un'estensione di Azure che offre all'ambiente locale l'agilità e l'innovazione del cloud computing, abilitando l'unico cloud ibrido che consente di creare e distribuire ovunque app ibride.  
> 
> L'articolo [Considerazioni per la progettazione di app ibride](overview-app-design-considerations.md) esamina i concetti fondamentali per la qualità del software (posizionamento, scalabilità, disponibilità, resilienza, gestibilità e sicurezza) per la progettazione, la distribuzione e la gestione di app ibride. Le considerazioni per la progettazione consentono di ottimizzare la progettazione delle app ibride, riducendo al minimo i rischi negli ambienti di produzione.

## <a name="create-a-service-principal-for-azure-ad-in-the-portal"></a>Creare un'entità servizio per Azure AD nel portale

Se l'hub di Azure Stack è stato distribuito usando Azure AD come archivio di identità, è possibile creare entità servizio esattamente come per Azure. L'articolo [Usare un'identità dell'app per accedere alle risorse](/azure-stack/operator/azure-stack-create-service-principals.md#manage-an-azure-ad-app-identity) illustra come eseguire la procedura tramite il portale. Prima di iniziare, verificare di avere le [autorizzazioni di Azure AD necessarie](/azure/azure-resource-manager/resource-group-create-service-principal-portal#required-permissions).

## <a name="create-a-service-principal-for-ad-fs-using-powershell"></a>Creare un'entità servizio per AD FS usando PowerShell

Se l'hub di Azure Stack è stato distribuito con AD FS, è possibile usare PowerShell per creare un'entità servizio, assegnare un ruolo per l'accesso e accedere da PowerShell usando tale identità. L'articolo [Usare un'identità dell'app per accedere alle risorse](/azure-stack/operator/azure-stack-create-service-principals.md#manage-an-ad-fs-app-identity) illustra come eseguire la procedura necessaria usando PowerShell.

## <a name="using-the-azure-stack-hub-api"></a>Uso dell'API dell'hub di Azure Stack

La soluzione [API dell'hub di Azure Stack](/azure-stack/user/azure-stack-rest-api-use.md) illustra il processo di recupero di un token per accedere all'API dell'hub di Azure Stack.

## <a name="connect-to-azure-stack-hub-using-powershell"></a>Connettersi all'hub di Azure Stack con PowerShell

La guida introduttiva per [iniziare a usare PowerShell nell'hub di Azure Stack](/azure-stack/operator/azure-stack-powershell-install.md) illustra la procedura necessaria per installare Azure PowerShell e connettersi all'installazione dell'hub di Azure Stack.

### <a name="prerequisites"></a>Prerequisiti

È necessaria un'installazione dell'hub di Azure Stack connessa ad Azure AD con una sottoscrizione a cui è possibile accedere. Se non si ha un'installazione dell'hub di Azure Stack, seguire queste istruzioni per configurare un [Azure Stack Development Kit (ASDK)](/azure-stack/asdk/asdk-install.md).

#### <a name="connect-to-azure-stack-hub-using-code"></a>Connettersi all'hub di Azure Stack tramite codice

Per connettersi all'hub di Azure Stack tramite codice, usare l'API degli endpoint di Azure Resource Manager per ottenere gli endpoint di autenticazione e di Graph per l'installazione dell'hub di Azure Stack. Eseguire quindi l'autenticazione tramite le richieste REST. Un'applicazione client di esempio è disponibile in [GitHub](https://github.com/shriramnat/HybridARMApplication).

>[!Note]
>A meno che Azure SDK per il linguaggio scelto non supporti i profili API di Azure, l'SDK potrebbe non funzionare con l'hub di Azure Stack. Per altre informazioni sui profili API di Azure, vedere l'articolo [Gestire i profili delle versioni dell'API](/azure-stack/user/azure-stack-version-profiles.md).

## <a name="next-steps"></a>Passaggi successivi

- Per altre informazioni su come viene gestita l'identità nell'hub di Azure Stack, vedere [Architettura dell'identità per l'hub di Azure Stack](/azure-stack/operator/azure-stack-identity-architecture.md).
- Per altre informazioni sui modelli cloud di Azure, vedere [Modelli di progettazione cloud](/azure/architecture/patterns).
