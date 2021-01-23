---
ms.openlocfilehash: 5b1a776c28b6f9218c713dde68f45e571ebfd999
ms.sourcegitcommit: 6341ad07cd5b03269e7fd20cd3212e48baee7c07
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 01/23/2021
ms.locfileid: "49942155"
---
<!-- markdownlint-disable MD002 MD041 -->

Erstellen Sie zunächst eine ASP.NET Core-Web-App.

1. Öffnen Sie die Befehlszeilenschnittstelle (CLI) in einem Verzeichnis, in dem Sie das Projekt erstellen möchten. Führen Sie den folgenden Befehl aus.

    ```Shell
    dotnet new mvc -o GraphTutorial
    ```

1. Nachdem das Projekt erstellt wurde, überprüfen Sie, ob es funktioniert, indem Sie das aktuelle Verzeichnis in das Verzeichnis **"GraphTutorial"** ändern und den folgenden Befehl in Ihrer CLI ausführen.

    ```Shell
    dotnet run
    ```

1. Öffnen Sie Ihren Browser, und navigieren Sie zu `https://localhost:5001` . Wenn alles funktioniert, sollte eine Standardmäßige ASP.NET A0 angezeigt werden.

> [!IMPORTANT]
> Wenn Eine Warnung angezeigt wird, dass das Zertifikat für **localhost** nicht vertrauenswürdig ist, können Sie das Entwicklungszertifikat mithilfe der .NET Core CLI installieren und als vertrauenswürdig einstufe. Anweisungen zu bestimmten Betriebssystemen finden Sie unter "HTTPS [erzwingen" in ASP.NET Core.](/aspnet/core/security/enforcing-ssl?view=aspnetcore-5.0)

## <a name="add-nuget-packages"></a>Hinzufügen von NuGet-Paketen

Installieren Sie vor dem Wechsel einige zusätzliche NuGet-Pakete, die Sie später verwenden werden.

- [Microsoft.Identity.Web zum](https://www.nuget.org/packages/Microsoft.Identity.Web/) Anfordern und Verwalten von Zugriffstoken.
- [Microsoft.Identity.Web.MicrosoftGraph zum](https://www.nuget.org/packages/Microsoft.Identity.Web.MicrosoftGraph/) Hinzufügen des Microsoft Graph SDK per Abhängigkeitsinjektion.
- [Microsoft.Identity.Web.UI](https://www.nuget.org/packages/Microsoft.Identity.Web.UI/) für die Anmelde- und Abmeldebenutzeroberfläche.
- [TimeZoneConverter](https://github.com/mj1856/TimeZoneConverter) für die plattformübergreifende Behandlung von Zeitzonenbezeichnern.

1. Führen Sie die folgenden Befehle in Ihrer CLI aus, um die Abhängigkeiten zu installieren.

    ```Shell
    dotnet add package Microsoft.Identity.Web --version 1.5.1
    dotnet add package Microsoft.Identity.Web.MicrosoftGraph --version 1.5.1
    dotnet add package Microsoft.Identity.Web.UI --version 1.5.1
    dotnet add package TimeZoneConverter
    ```

## <a name="design-the-app"></a>Entwerfen der App

In diesem Abschnitt erstellen Sie die grundlegende Benutzeroberflächenstruktur der Anwendung.

### <a name="implement-alert-extension-methods"></a>Implementieren von Warnungserweiterungsmethoden

In diesem Abschnitt erstellen Sie Erweiterungsmethoden für den `IActionResult` Typ, der von Controlleransichten zurückgegeben wird. Diese Erweiterung ermöglicht das Übergeben temporärer Fehler- oder Erfolgsmeldungen an die Ansicht.

> [!TIP]
> Sie können einen beliebigen Texteditor verwenden, um die Quelldateien für dieses Lernprogramm zu bearbeiten. Allerdings bietet [Visual Studio Code](https://code.visualstudio.com/) zusätzliche Features, z. B. Debugging und IntelliSense.

1. Erstellen Sie ein neues Verzeichnis im **Verzeichnis GraphTutorial** mit dem Namen **Alerts**.

1. Erstellen Sie eine neue Datei **namens WithAlertResult.cs** im Verzeichnis **./Alerts,** und fügen Sie den folgenden Code hinzu.

    :::code language="csharp" source="../demo/GraphTutorial/Alerts/WithAlertResult.cs" id="WithAlertResultSnippet":::

1. Erstellen Sie eine neue Datei **namens AlertExtensions.cs** im **Verzeichnis ./Alerts,** und fügen Sie den folgenden Code hinzu.

    :::code language="csharp" source="../demo/GraphTutorial/Alerts/AlertExtensions.cs" id="AlertExtensionsSnippet":::

### <a name="implement-user-data-extension-methods"></a>Implementieren von Methoden für die Benutzerdatenerweiterung

In diesem Abschnitt erstellen Sie Erweiterungsmethoden für das von der `ClaimsPrincipal` Microsoft Identity Platform generierte Objekt. Dadurch können Sie die vorhandene Benutzeridentität mit Daten aus Microsoft Graph erweitern.

> [!NOTE]
> Dieser Code ist vorerst nur ein Platzhalter, den Sie in einem späteren Abschnitt vervollständigen werden.

1. Erstellen Sie ein neues Verzeichnis im **GraphTutorial-Verzeichnis** mit dem Namen **Graph**.

1. Erstellen Sie eine neue Datei namens **GraphClaimsPrincipalExtensions.cs,** und fügen Sie den folgenden Code hinzu.

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

### <a name="create-views"></a>Erstellen von Ansichten

In diesem Abschnitt implementieren Sie die Ansichten "Ausschnitt" für die Anwendung.

1. Fügen Sie eine neue Datei namens **_LoginPartial.cshtml** im Verzeichnis **./Views/Shared** hinzu, und fügen Sie den folgenden Code hinzu.

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Shared/_LoginPartial.cshtml" id="LoginPartialSnippet":::

1. Fügen Sie eine neue Datei namens **_AlertPartial.cshtml** im Verzeichnis **./Views/Shared** hinzu, und fügen Sie den folgenden Code hinzu.

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Shared/_AlertPartial.cshtml" id="AlertPartialSnippet":::

1. Öffnen Sie die Datei **./Views/Shared/_Layout. cshtml**, und ersetzen Sie den gesamten Inhalt durch den folgenden Code, um das globale Layout der App zu aktualisieren.

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Shared/_Layout.cshtml" id="LayoutSnippet":::

1. Öffnen **Sie ./wwwroot/css/site.css,** und fügen Sie den folgenden Code am Ende der Datei hinzu.

    :::code language="css" source="../demo/GraphTutorial/wwwroot/css/site.css" id="CssSnippet":::

1. Öffnen Sie **die Datei ./Views/Home/index.cshtml,** und ersetzen Sie den Inhalt durch Folgendes.

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Home/Index.cshtml" id="HomeIndexSnippet":::

1. Erstellen Sie ein neues Verzeichnis im **Verzeichnis ./wwwroot** mit dem Namen **img**. Fügen Sie eine Bilddatei Ihrer Wahl mit dem Namen **no-profile-photo.png** in diesem Verzeichnis hinzu. Dieses Bild wird als Foto des Benutzers verwendet, wenn der Benutzer kein Foto in Microsoft Graph hat.

    > [!TIP]
    > Sie können das in diesen Screenshots verwendete Bild von [GitHub herunterladen.](https://github.com/microsoftgraph/msgraph-training-aspnet-core/blob/master/demo/GraphTutorial/wwwroot/img/no-profile-photo.png)

1. Speichern Sie alle Änderungen, und starten Sie den Server neu ( `dotnet run` ). Jetzt sollte die App ganz anders aussehen.

    ![Screenshot der neu gestalteten Homepage](./images/create-app-01.png)
