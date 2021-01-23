---
ms.openlocfilehash: 73ba9c271c9c6675ffbb22ce4f800ffb652c715d
ms.sourcegitcommit: 6341ad07cd5b03269e7fd20cd3212e48baee7c07
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 01/23/2021
ms.locfileid: "49942134"
---
<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="c7728-101">In diesem Lernprogramm erfahren Sie, wie Sie eine ASP.NET Core-Web-App erstellen, die die Microsoft Graph-API zum Abrufen von Kalenderinformationen für einen Benutzer verwendet.</span><span class="sxs-lookup"><span data-stu-id="c7728-101">This tutorial teaches you how to build an ASP.NET Core web app that uses the Microsoft Graph API to retrieve calendar information for a user.</span></span>

> [!TIP]
> <span data-ttu-id="c7728-102">Wenn Sie nur das abgeschlossene Lernprogramm herunterladen möchten, können Sie das [GitHub-Repository herunterladen oder klonen.](https://github.com/microsoftgraph/msgraph-training-aspnet-core)</span><span class="sxs-lookup"><span data-stu-id="c7728-102">If you prefer to just download the completed tutorial, you can download or clone the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-aspnet-core).</span></span> <span data-ttu-id="c7728-103">Anweisungen zum Konfigurieren der  App mit einer App-ID und einem geheimen Schlüssel finden Sie in der README-Datei im Demoordner.</span><span class="sxs-lookup"><span data-stu-id="c7728-103">See the README file in the **demo** folder for instructions on configuring the app with an app ID and secret.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="c7728-104">Voraussetzungen</span><span class="sxs-lookup"><span data-stu-id="c7728-104">Prerequisites</span></span>

<span data-ttu-id="c7728-105">Bevor Sie mit diesem Lernprogramm beginnen, sollten Sie [das .NET Core SDK](https://dotnet.microsoft.com/download) auf Ihrem Entwicklungscomputer installieren.</span><span class="sxs-lookup"><span data-stu-id="c7728-105">Before you start this tutorial, you should have the [.NET Core SDK](https://dotnet.microsoft.com/download) installed on your development machine.</span></span> <span data-ttu-id="c7728-106">Wenn Sie nicht über das SDK verfügen, finden Sie unter dem vorherigen Link Downloadoptionen.</span><span class="sxs-lookup"><span data-stu-id="c7728-106">If you do not have the SDK, visit the previous link for download options.</span></span>

<span data-ttu-id="c7728-107">Sie sollten auch über ein persönliches Microsoft-Konto mit einem Postfach auf Outlook.com oder ein Microsoft-Arbeits- oder Schulkonto verfügen.</span><span class="sxs-lookup"><span data-stu-id="c7728-107">You should also have either a personal Microsoft account with a mailbox on Outlook.com, or a Microsoft work or school account.</span></span> <span data-ttu-id="c7728-108">Wenn Sie kein Microsoft-Konto haben, gibt es mehrere Optionen, um ein kostenloses Konto zu erhalten:</span><span class="sxs-lookup"><span data-stu-id="c7728-108">If you don't have a Microsoft account, there are a couple of options to get a free account:</span></span>

- <span data-ttu-id="c7728-109">Sie können [sich für ein neues persönliches Microsoft-Konto registrieren.](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1)</span><span class="sxs-lookup"><span data-stu-id="c7728-109">You can [sign up for a new personal Microsoft account](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1).</span></span>
- <span data-ttu-id="c7728-110">Sie können [sich für das Office 365-Entwicklerprogramm](https://developer.microsoft.com/office/dev-program) registrieren, um ein kostenloses Office 365-Abonnement zu erhalten.</span><span class="sxs-lookup"><span data-stu-id="c7728-110">You can [sign up for the Office 365 Developer Program](https://developer.microsoft.com/office/dev-program) to get a free Office 365 subscription.</span></span>

> [!NOTE]
> <span data-ttu-id="c7728-111">Dieses Lernprogramm wurde mit .NET Core SDK, Version 5.0.102, geschrieben.</span><span class="sxs-lookup"><span data-stu-id="c7728-111">This tutorial was written with .NET Core SDK version 5.0.102.</span></span> <span data-ttu-id="c7728-112">Die Schritte in diesem Handbuch funktionieren möglicherweise mit anderen Versionen, wurden jedoch noch nicht getestet.</span><span class="sxs-lookup"><span data-stu-id="c7728-112">The steps in this guide may work with other versions, but that has not been tested.</span></span>

## <a name="feedback"></a><span data-ttu-id="c7728-113">Feedback</span><span class="sxs-lookup"><span data-stu-id="c7728-113">Feedback</span></span>

<span data-ttu-id="c7728-114">Bitte geben Sie Feedback zu diesem Lernprogramm im [GitHub-Repository.](https://github.com/microsoftgraph/msgraph-training-aspnet-core)</span><span class="sxs-lookup"><span data-stu-id="c7728-114">Please provide any feedback on this tutorial in the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-aspnet-core).</span></span>
