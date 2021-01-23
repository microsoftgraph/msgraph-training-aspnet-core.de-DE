---
ms.openlocfilehash: 5b1a776c28b6f9218c713dde68f45e571ebfd999
ms.sourcegitcommit: 6341ad07cd5b03269e7fd20cd3212e48baee7c07
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 01/23/2021
ms.locfileid: "49942155"
---
<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="a6da1-101">Erstellen Sie zunächst eine ASP.NET Core-Web-App.</span><span class="sxs-lookup"><span data-stu-id="a6da1-101">Start by creating an ASP.NET Core web app.</span></span>

1. <span data-ttu-id="a6da1-102">Öffnen Sie die Befehlszeilenschnittstelle (CLI) in einem Verzeichnis, in dem Sie das Projekt erstellen möchten.</span><span class="sxs-lookup"><span data-stu-id="a6da1-102">Open your command-line interface (CLI) in a directory where you want to create the project.</span></span> <span data-ttu-id="a6da1-103">Führen Sie den folgenden Befehl aus.</span><span class="sxs-lookup"><span data-stu-id="a6da1-103">Run the following command.</span></span>

    ```Shell
    dotnet new mvc -o GraphTutorial
    ```

1. <span data-ttu-id="a6da1-104">Nachdem das Projekt erstellt wurde, überprüfen Sie, ob es funktioniert, indem Sie das aktuelle Verzeichnis in das Verzeichnis **"GraphTutorial"** ändern und den folgenden Befehl in Ihrer CLI ausführen.</span><span class="sxs-lookup"><span data-stu-id="a6da1-104">Once the project is created, verify that it works by changing the current directory to the **GraphTutorial** directory and running the following command in your CLI.</span></span>

    ```Shell
    dotnet run
    ```

1. <span data-ttu-id="a6da1-105">Öffnen Sie Ihren Browser, und navigieren Sie zu `https://localhost:5001` .</span><span class="sxs-lookup"><span data-stu-id="a6da1-105">Open your browser and browse to `https://localhost:5001`.</span></span> <span data-ttu-id="a6da1-106">Wenn alles funktioniert, sollte eine Standardmäßige ASP.NET A0 angezeigt werden.</span><span class="sxs-lookup"><span data-stu-id="a6da1-106">If everything is working, you should see a default ASP.NET Core page.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="a6da1-107">Wenn Eine Warnung angezeigt wird, dass das Zertifikat für **localhost** nicht vertrauenswürdig ist, können Sie das Entwicklungszertifikat mithilfe der .NET Core CLI installieren und als vertrauenswürdig einstufe.</span><span class="sxs-lookup"><span data-stu-id="a6da1-107">If you receive a warning that the certificate for **localhost** is un-trusted you can use the .NET Core CLI to install and trust the development certificate.</span></span> <span data-ttu-id="a6da1-108">Anweisungen zu bestimmten Betriebssystemen finden Sie unter "HTTPS [erzwingen" in ASP.NET Core.](/aspnet/core/security/enforcing-ssl?view=aspnetcore-5.0)</span><span class="sxs-lookup"><span data-stu-id="a6da1-108">See [Enforce HTTPS in ASP.NET Core](/aspnet/core/security/enforcing-ssl?view=aspnetcore-5.0) for instructions for specific operating systems.</span></span>

## <a name="add-nuget-packages"></a><span data-ttu-id="a6da1-109">Hinzufügen von NuGet-Paketen</span><span class="sxs-lookup"><span data-stu-id="a6da1-109">Add NuGet packages</span></span>

<span data-ttu-id="a6da1-110">Installieren Sie vor dem Wechsel einige zusätzliche NuGet-Pakete, die Sie später verwenden werden.</span><span class="sxs-lookup"><span data-stu-id="a6da1-110">Before moving on, install some additional NuGet packages that you will use later.</span></span>

- <span data-ttu-id="a6da1-111">[Microsoft.Identity.Web zum](https://www.nuget.org/packages/Microsoft.Identity.Web/) Anfordern und Verwalten von Zugriffstoken.</span><span class="sxs-lookup"><span data-stu-id="a6da1-111">[Microsoft.Identity.Web](https://www.nuget.org/packages/Microsoft.Identity.Web/) for requesting and managing access tokens.</span></span>
- <span data-ttu-id="a6da1-112">[Microsoft.Identity.Web.MicrosoftGraph zum](https://www.nuget.org/packages/Microsoft.Identity.Web.MicrosoftGraph/) Hinzufügen des Microsoft Graph SDK per Abhängigkeitsinjektion.</span><span class="sxs-lookup"><span data-stu-id="a6da1-112">[Microsoft.Identity.Web.MicrosoftGraph](https://www.nuget.org/packages/Microsoft.Identity.Web.MicrosoftGraph/) for adding the Microsoft Graph SDK via dependency injection.</span></span>
- <span data-ttu-id="a6da1-113">[Microsoft.Identity.Web.UI](https://www.nuget.org/packages/Microsoft.Identity.Web.UI/) für die Anmelde- und Abmeldebenutzeroberfläche.</span><span class="sxs-lookup"><span data-stu-id="a6da1-113">[Microsoft.Identity.Web.UI](https://www.nuget.org/packages/Microsoft.Identity.Web.UI/) for sign-in and sign-out UI.</span></span>
- <span data-ttu-id="a6da1-114">[TimeZoneConverter](https://github.com/mj1856/TimeZoneConverter) für die plattformübergreifende Behandlung von Zeitzonenbezeichnern.</span><span class="sxs-lookup"><span data-stu-id="a6da1-114">[TimeZoneConverter](https://github.com/mj1856/TimeZoneConverter) for handling time zoned identifiers cross-platform.</span></span>

1. <span data-ttu-id="a6da1-115">Führen Sie die folgenden Befehle in Ihrer CLI aus, um die Abhängigkeiten zu installieren.</span><span class="sxs-lookup"><span data-stu-id="a6da1-115">Run the following commands in your CLI to install the dependencies.</span></span>

    ```Shell
    dotnet add package Microsoft.Identity.Web --version 1.5.1
    dotnet add package Microsoft.Identity.Web.MicrosoftGraph --version 1.5.1
    dotnet add package Microsoft.Identity.Web.UI --version 1.5.1
    dotnet add package TimeZoneConverter
    ```

## <a name="design-the-app"></a><span data-ttu-id="a6da1-116">Entwerfen der App</span><span class="sxs-lookup"><span data-stu-id="a6da1-116">Design the app</span></span>

<span data-ttu-id="a6da1-117">In diesem Abschnitt erstellen Sie die grundlegende Benutzeroberflächenstruktur der Anwendung.</span><span class="sxs-lookup"><span data-stu-id="a6da1-117">In this section you will create the basic UI structure of the application.</span></span>

### <a name="implement-alert-extension-methods"></a><span data-ttu-id="a6da1-118">Implementieren von Warnungserweiterungsmethoden</span><span class="sxs-lookup"><span data-stu-id="a6da1-118">Implement alert extension methods</span></span>

<span data-ttu-id="a6da1-119">In diesem Abschnitt erstellen Sie Erweiterungsmethoden für den `IActionResult` Typ, der von Controlleransichten zurückgegeben wird.</span><span class="sxs-lookup"><span data-stu-id="a6da1-119">In this section you will create extension methods for the `IActionResult` type returned by controller views.</span></span> <span data-ttu-id="a6da1-120">Diese Erweiterung ermöglicht das Übergeben temporärer Fehler- oder Erfolgsmeldungen an die Ansicht.</span><span class="sxs-lookup"><span data-stu-id="a6da1-120">This extension will enable passing temporary error or success messages to the view.</span></span>

> [!TIP]
> <span data-ttu-id="a6da1-121">Sie können einen beliebigen Texteditor verwenden, um die Quelldateien für dieses Lernprogramm zu bearbeiten.</span><span class="sxs-lookup"><span data-stu-id="a6da1-121">You can use any text editor to edit the source files for this tutorial.</span></span> <span data-ttu-id="a6da1-122">Allerdings bietet [Visual Studio Code](https://code.visualstudio.com/) zusätzliche Features, z. B. Debugging und IntelliSense.</span><span class="sxs-lookup"><span data-stu-id="a6da1-122">However, [Visual Studio Code](https://code.visualstudio.com/) provides additional features, such as debugging and Intellisense.</span></span>

1. <span data-ttu-id="a6da1-123">Erstellen Sie ein neues Verzeichnis im **Verzeichnis GraphTutorial** mit dem Namen **Alerts**.</span><span class="sxs-lookup"><span data-stu-id="a6da1-123">Create a new directory in the **GraphTutorial** directory named **Alerts**.</span></span>

1. <span data-ttu-id="a6da1-124">Erstellen Sie eine neue Datei **namens WithAlertResult.cs** im Verzeichnis **./Alerts,** und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="a6da1-124">Create a new file named **WithAlertResult.cs** in the **./Alerts** directory and add the following code.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Alerts/WithAlertResult.cs" id="WithAlertResultSnippet":::

1. <span data-ttu-id="a6da1-125">Erstellen Sie eine neue Datei **namens AlertExtensions.cs** im **Verzeichnis ./Alerts,** und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="a6da1-125">Create a new file named **AlertExtensions.cs** in the **./Alerts** directory and add the following code.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Alerts/AlertExtensions.cs" id="AlertExtensionsSnippet":::

### <a name="implement-user-data-extension-methods"></a><span data-ttu-id="a6da1-126">Implementieren von Methoden für die Benutzerdatenerweiterung</span><span class="sxs-lookup"><span data-stu-id="a6da1-126">Implement user data extension methods</span></span>

<span data-ttu-id="a6da1-127">In diesem Abschnitt erstellen Sie Erweiterungsmethoden für das von der `ClaimsPrincipal` Microsoft Identity Platform generierte Objekt.</span><span class="sxs-lookup"><span data-stu-id="a6da1-127">In this section you will create extension methods for the `ClaimsPrincipal` object generated by the Microsoft Identity platform.</span></span> <span data-ttu-id="a6da1-128">Dadurch können Sie die vorhandene Benutzeridentität mit Daten aus Microsoft Graph erweitern.</span><span class="sxs-lookup"><span data-stu-id="a6da1-128">This will allow you to extend the existing user identity with data from Microsoft Graph.</span></span>

> [!NOTE]
> <span data-ttu-id="a6da1-129">Dieser Code ist vorerst nur ein Platzhalter, den Sie in einem späteren Abschnitt vervollständigen werden.</span><span class="sxs-lookup"><span data-stu-id="a6da1-129">This code is just a placeholder for now, you will complete it in a later section.</span></span>

1. <span data-ttu-id="a6da1-130">Erstellen Sie ein neues Verzeichnis im **GraphTutorial-Verzeichnis** mit dem Namen **Graph**.</span><span class="sxs-lookup"><span data-stu-id="a6da1-130">Create a new directory in the **GraphTutorial** directory named **Graph**.</span></span>

1. <span data-ttu-id="a6da1-131">Erstellen Sie eine neue Datei namens **GraphClaimsPrincipalExtensions.cs,** und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="a6da1-131">Create a new file named **GraphClaimsPrincipalExtensions.cs** and add the following code.</span></span>

    ```csharp
    using System.Security.Claims;

    namespace GraphTutorial
    {
        public static class GraphClaimTypes {
            public const string DisplayName ="graph_name";
            public const string Email = "graph_email";
            public const string Photo = "graph_photo";
            public const string TimeZone = "graph_timezone";
            public const string DateTimeFormat = "graph_datetimeformat";
        }

        // Helper methods to access Graph user data stored in
        // the claims principal
        public static class GraphClaimsPrincipalExtensions
        {
            public static string GetUserGraphDisplayName(this ClaimsPrincipal claimsPrincipal)
            {
                return "Adele Vance";
            }

            public static string GetUserGraphEmail(this ClaimsPrincipal claimsPrincipal)
            {
                return "adelev@contoso.com";
            }

            public static string GetUserGraphPhoto(this ClaimsPrincipal claimsPrincipal)
            {
                return "/img/no-profile-photo.png";
            }
        }
    }
    ```

### <a name="create-views"></a><span data-ttu-id="a6da1-132">Erstellen von Ansichten</span><span class="sxs-lookup"><span data-stu-id="a6da1-132">Create views</span></span>

<span data-ttu-id="a6da1-133">In diesem Abschnitt implementieren Sie die Ansichten "Ausschnitt" für die Anwendung.</span><span class="sxs-lookup"><span data-stu-id="a6da1-133">In this section you will implement the Razor views for the application.</span></span>

1. <span data-ttu-id="a6da1-134">Fügen Sie eine neue Datei namens **_LoginPartial.cshtml** im Verzeichnis **./Views/Shared** hinzu, und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="a6da1-134">Add a new file named **_LoginPartial.cshtml** in the **./Views/Shared** directory and add the following code.</span></span>

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Shared/_LoginPartial.cshtml" id="LoginPartialSnippet":::

1. <span data-ttu-id="a6da1-135">Fügen Sie eine neue Datei namens **_AlertPartial.cshtml** im Verzeichnis **./Views/Shared** hinzu, und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="a6da1-135">Add a new file named **_AlertPartial.cshtml** in the **./Views/Shared** directory and add the following code.</span></span>

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Shared/_AlertPartial.cshtml" id="AlertPartialSnippet":::

1. <span data-ttu-id="a6da1-136">Öffnen Sie die Datei **./Views/Shared/_Layout. cshtml**, und ersetzen Sie den gesamten Inhalt durch den folgenden Code, um das globale Layout der App zu aktualisieren.</span><span class="sxs-lookup"><span data-stu-id="a6da1-136">Open the **./Views/Shared/_Layout.cshtml** file, and replace its entire contents with the following code to update the global layout of the app.</span></span>

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Shared/_Layout.cshtml" id="LayoutSnippet":::

1. <span data-ttu-id="a6da1-137">Öffnen **Sie ./wwwroot/css/site.css,** und fügen Sie den folgenden Code am Ende der Datei hinzu.</span><span class="sxs-lookup"><span data-stu-id="a6da1-137">Open **./wwwroot/css/site.css** and add the following code at the bottom of the file.</span></span>

    :::code language="css" source="../demo/GraphTutorial/wwwroot/css/site.css" id="CssSnippet":::

1. <span data-ttu-id="a6da1-138">Öffnen Sie **die Datei ./Views/Home/index.cshtml,** und ersetzen Sie den Inhalt durch Folgendes.</span><span class="sxs-lookup"><span data-stu-id="a6da1-138">Open the **./Views/Home/index.cshtml** file and replace its contents with the following.</span></span>

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Home/Index.cshtml" id="HomeIndexSnippet":::

1. <span data-ttu-id="a6da1-139">Erstellen Sie ein neues Verzeichnis im **Verzeichnis ./wwwroot** mit dem Namen **img**.</span><span class="sxs-lookup"><span data-stu-id="a6da1-139">Create a new directory in the **./wwwroot** directory named **img**.</span></span> <span data-ttu-id="a6da1-140">Fügen Sie eine Bilddatei Ihrer Wahl mit dem Namen **no-profile-photo.png** in diesem Verzeichnis hinzu.</span><span class="sxs-lookup"><span data-stu-id="a6da1-140">Add an image file of your choosing named **no-profile-photo.png** in this directory.</span></span> <span data-ttu-id="a6da1-141">Dieses Bild wird als Foto des Benutzers verwendet, wenn der Benutzer kein Foto in Microsoft Graph hat.</span><span class="sxs-lookup"><span data-stu-id="a6da1-141">This image will be used as the user's photo when the user has no photo in Microsoft Graph.</span></span>

    > [!TIP]
    > <span data-ttu-id="a6da1-142">Sie können das in diesen Screenshots verwendete Bild von [GitHub herunterladen.](https://github.com/microsoftgraph/msgraph-training-aspnet-core/blob/master/demo/GraphTutorial/wwwroot/img/no-profile-photo.png)</span><span class="sxs-lookup"><span data-stu-id="a6da1-142">You can download the image used in these screenshots from [GitHub](https://github.com/microsoftgraph/msgraph-training-aspnet-core/blob/master/demo/GraphTutorial/wwwroot/img/no-profile-photo.png).</span></span>

1. <span data-ttu-id="a6da1-143">Speichern Sie alle Änderungen, und starten Sie den Server neu ( `dotnet run` ).</span><span class="sxs-lookup"><span data-stu-id="a6da1-143">Save all of your changes and restart the server (`dotnet run`).</span></span> <span data-ttu-id="a6da1-144">Jetzt sollte die App ganz anders aussehen.</span><span class="sxs-lookup"><span data-stu-id="a6da1-144">Now, the app should look very different.</span></span>

    ![Screenshot der neu gestalteten Homepage](./images/create-app-01.png)
