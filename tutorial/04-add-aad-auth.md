---
ms.openlocfilehash: a024fb533c552563da6c9179301e16a2e1d09d5f
ms.sourcegitcommit: 6341ad07cd5b03269e7fd20cd3212e48baee7c07
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 01/23/2021
ms.locfileid: "49942162"
---
<!-- markdownlint-disable MD002 MD041 -->

In dieser Übung erweitern Sie die Anwendung aus der vorherigen Übung, um die Authentifizierung mit Azure AD zu unterstützen. Dies ist notwendig, um das erforderliche OAuth-Zugriffstoken zum Aufruf der Microsoft Graph-API abzurufen. In diesem Schritt konfigurieren Sie die [Microsoft.Identity.Web-Bibliothek.](https://www.nuget.org/packages/Microsoft.Identity.Web/)

> [!IMPORTANT]
> Um das Speichern der Anwendungs-ID und des geheimen Schlüssels in der Quelle zu vermeiden, verwenden Sie [den .NET Secret Manager,](/aspnet/core/security/app-secrets) um diese Werte zu speichern. Der geheime Manager dient nur zu Entwicklungszwecken, Produktions-Apps sollten einen vertrauenswürdigen geheimen Manager zum Speichern von geheimen Schlüsseln verwenden.

1. Öffnen **Sie ./appsettings.js,** und ersetzen Sie den Inhalt durch Folgendes.

    :::code language="json" source="../demo/GraphTutorial/appsettings.json" highlight="2-6":::

1. Öffnen Sie Ihre CLI im Verzeichnis, in dem **sich "GraphTutorial.csproj"** befindet, und führen Sie die folgenden Befehle aus, und ersetzen Sie sie durch Ihre Anwendungs-ID aus dem Azure-Portal und mit Ihrem Anwendungsgeheimnis. `YOUR_APP_ID` `YOUR_APP_SECRET`

    ```Shell
    dotnet user-secrets init
    dotnet user-secrets set "AzureAd:ClientId" "YOUR_APP_ID"
    dotnet user-secrets set "AzureAd:ClientSecret" "YOUR_APP_SECRET"
    ```

## <a name="implement-sign-in"></a>Implementieren der Anmeldung

Beginnen Sie, indem Sie der Anwendung die Microsoft Identity Platform Services hinzufügen.

1. Erstellen Sie eine neue Datei **namens GraphConstants.cs** im **Verzeichnis ./Graph,** und fügen Sie den folgenden Code hinzu.

    :::code language="csharp" source="../demo/GraphTutorial/Graph/GraphConstants.cs" id="GraphConstantsSnippet":::

1. Öffnen Sie **die STARTUP.CS-Datei,** und fügen Sie die folgenden Anweisungen am Oberen Rand der Datei `using` hinzu.

    ```csharp
    using Microsoft.AspNetCore.Authentication.OpenIdConnect;
    using Microsoft.AspNetCore.Authorization;
    using Microsoft.AspNetCore.Mvc.Authorization;
    using Microsoft.Identity.Web;
    using Microsoft.Identity.Web.UI;
    using Microsoft.IdentityModel.Protocols.OpenIdConnect;
    using Microsoft.Graph;
    using System.Net;
    using System.Net.Http.Headers;
    ```

1. Ersetzen Sie die vorhandene `ConfigureServices`-Funktion durch Folgendes.

    ```csharp
    public void ConfigureServices(IServiceCollection services)
    {
        services
            // Use OpenId authentication
            .AddAuthentication(OpenIdConnectDefaults.AuthenticationScheme)
            // Specify this is a web app and needs auth code flow
            .AddMicrosoftIdentityWebApp(Configuration)
            // Add ability to call web API (Graph)
            // and get access tokens
            .EnableTokenAcquisitionToCallDownstreamApi(options => {
                Configuration.Bind("AzureAd", options);
            }, GraphConstants.Scopes)
            // Use in-memory token cache
            // See https://github.com/AzureAD/microsoft-identity-web/wiki/token-cache-serialization
            .AddInMemoryTokenCaches();

        // Require authentication
        services.AddControllersWithViews(options =>
        {
            var policy = new AuthorizationPolicyBuilder()
                .RequireAuthenticatedUser()
                .Build();
            options.Filters.Add(new AuthorizeFilter(policy));
        })
        // Add the Microsoft Identity UI pages for signin/out
        .AddMicrosoftIdentityUI();
    }
    ```

1. Fügen Sie `Configure` in der Funktion die folgende Zeile über der Zeile `app.UseAuthorization();` hinzu.

    ```csharp
    app.UseAuthentication();
    ```

1. Öffnen **Sie ./Controllers/HomeController.cs,** und ersetzen Sie den Inhalt durch Folgendes.

    ```csharp
    using GraphTutorial.Models;
    using Microsoft.AspNetCore.Authorization;
    using Microsoft.AspNetCore.Mvc;
    using Microsoft.Extensions.Logging;
    using Microsoft.Identity.Web;
    using System.Diagnostics;
    using System.Threading.Tasks;

    namespace GraphTutorial.Controllers
    {
        public class HomeController : Controller
        {
            ITokenAcquisition _tokenAcquisition;
            private readonly ILogger<HomeController> _logger;

            // Get the ITokenAcquisition interface via
            // dependency injection
            public HomeController(
                ITokenAcquisition tokenAcquisition,
                ILogger<HomeController> logger)
            {
                _tokenAcquisition = tokenAcquisition;
                _logger = logger;
            }

            public async Task<IActionResult> Index()
            {
                // TEMPORARY
                // Get the token and display it
                try
                {
                    string token = await _tokenAcquisition
                        .GetAccessTokenForUserAsync(GraphConstants.Scopes);
                    return View().WithInfo("Token acquired", token);
                }
                catch (MicrosoftIdentityWebChallengeUserException)
                {
                    return Challenge();
                }
            }

            public IActionResult Privacy()
            {
                return View();
            }

            [ResponseCache(Duration = 0, Location = ResponseCacheLocation.None, NoStore = true)]
            public IActionResult Error()
            {
                return View(new ErrorViewModel { RequestId = Activity.Current?.Id ?? HttpContext.TraceIdentifier });
            }

            [ResponseCache(Duration = 0, Location = ResponseCacheLocation.None, NoStore = true)]
            [AllowAnonymous]
            public IActionResult ErrorWithMessage(string message, string debug)
            {
                return View("Index").WithError(message, debug);
            }
        }
    }
    ```

1. Speichern Sie Ihre Änderungen und starten Sie das Projekt. Melden Sie sich mit Ihrem Microsoft-Konto an.

1. Überprüfen Sie die Zustimmungsaufforderung. Die Liste der Berechtigungen entspricht der Liste der Berechtigungsbereiche, die in **./Graph/GraphConstants.cs konfiguriert sind.**

    - **Behalten Sie den Zugriff auf Daten,** auf die Sie zugriffen haben: ( ) Diese Berechtigung wird von MSAL angefordert, um Aktualisierungstoken `offline_access` abzurufen.
    - **Melden Sie sich an, und lesen** Sie Ihr Profil: ( ) Mit dieser Berechtigung kann die App das Profil und Profilfoto des angemeldeten `User.Read` Benutzers erhalten.
    - **Lesen Der Postfacheinstellungen:** ( ) Mit dieser Berechtigung kann die App die Postfacheinstellungen des Benutzers lesen, einschließlich Zeitzone `MailboxSettings.Read` und Zeitzone.
    - **Haben Vollzugriff auf** Ihre Kalender: ( ) Diese Berechtigung ermöglicht der App, Ereignisse im Kalender des Benutzers zu lesen, neue Ereignisse hinzuzufügen und `Calendars.ReadWrite` vorhandene zu ändern.

    ![Screenshot der Microsoft Identity Platform-Zustimmungsaufforderung](./images/add-aad-auth-03.png)

    Weitere Informationen zur Zustimmung finden Sie unter [Understanding Azure AD application consent experiences](/azure/active-directory/develop/application-consent-experience).

1. Stimmen Sie den angeforderten Berechtigungen zu. Der Browser leitet zur App um, in der Sie das Token sehen.

### <a name="get-user-details"></a>Benutzerdetails abrufen

Sobald sich der Benutzer angemeldet hat, können Sie dessen Informationen über Microsoft Graph abrufen.

1. Öffnen **Sie ./Graph/GraphClaimsPrincipalExtensions.cs,** und ersetzen Sie den gesamten Inhalt durch Folgendes.

    :::code language="csharp" source="../demo/GraphTutorial/Graph/GraphClaimsPrincipalExtensions.cs" id="GraphClaimsExtensionsSnippet":::

1. Öffnen **Sie ./Startup.cs,** und ersetzen Sie die vorhandene `.AddMicrosoftIdentityWebApp(Configuration)` Zeile durch den folgenden Code.

    :::code language="csharp" source="../demo/GraphTutorial/Startup.cs" id="AddSignInSnippet":::

    Überlegen Sie, was dieser Code macht.

    - Es wird ein Ereignishandler für das Ereignis `OnTokenValidated` hinzugefügt.
        - Sie verwendet die `ITokenAcquisition` Schnittstelle, um ein Zugriffstoken abzurufen.
        - Es ruft Microsoft Graph auf, um das Profil und Foto des Benutzers zu erhalten.
        - Die Graphinformationen werden der Identität des Benutzers hinzufügt.

1. Fügen Sie den folgenden Funktionsaufruf nach dem `EnableTokenAcquisitionToCallDownstreamApi` Aufruf und vor dem Aufruf `AddInMemoryTokenCaches` hinzu.

    :::code language="csharp" source="../demo/GraphTutorial/Startup.cs" id="AddGraphClientSnippet":::

    Dadurch wird ein authentifizierter **GraphServiceClient** über eine Abhängigkeitsinjektion für Controller verfügbar.

1. Öffnen **Sie ./Controllers/HomeController.cs,** und ersetzen Sie `Index` die Funktion durch Folgendes.

    ```csharp
    public IActionResult Index()
    {
        return View();
    }
    ```

1. Entfernen Sie alle Verweise `ITokenAcquisition` auf in der **HomeController-Klasse.**

1. Speichern Sie Ihre Änderungen, starten Sie die App, und gehen Sie den Anmeldevorgang durch. Sie sollten wieder auf der Startseite landen, aber die Benutzeroberfläche sollte geändert werden, um anzugeben, dass Sie angemeldet sind.

    ![Screenshot der Startseite nach dem Anmelden](./images/add-aad-auth-01.png)

1. Klicken Sie in der oberen rechten Ecke auf den Avatar des Benutzers, um auf den Link **"Abmelden" zu** klicken. Wenn Sie auf **Abmelden** klicken, wird die Sitzung zurückgesetzt und Sie kehren zur Startseite zurück.

    ![Screenshot des Dropdown-Menüs mit dem Link „Abmelden“](./images/add-aad-auth-02.png)

> [!TIP]
> Wenn Ihr Benutzername nicht auf der Startseite angezeigt wird und die Dropdownliste "Avatar verwenden" nach diesen Änderungen keinen Namen und keine E-Mail enthält, melden Sie sich ab und wieder an.

## <a name="storing-and-refreshing-tokens"></a>Speichern und Aktualisieren von Token

An diesem Punkt verfügt Ihre Anwendung über ein Zugriffstoken, das im `Authorization` Header von API-Aufrufen gesendet wird. Dies ist das Token, durch das die App im Namen des Benutzers auf Microsoft Graph zugreifen kann.

Dieses Token ist jedoch nur kurzzeitig verfügbar. Das Token läuft eine Stunde nach seiner Entsprechung ab. An dieser Stelle kommt das Aktualisierungstoken ins Spiel. Anhand des Aktualisierungstoken ist die App in der Lage, ein neues Zugriffstoken anzufordern, ohne dass der Benutzer sich erneut anmelden muss.

Da die App die Microsoft.Identity.Web-Bibliothek verwendet, müssen Sie keine Tokenspeicher- oder Aktualisierungslogik implementieren.

Die App verwendet den Speichertokencache, der für Apps ausreicht, die beim Neustart der App keine Token beibehalten müssen. Produktionsanwendungen können stattdessen die Optionen für [den verteilten Cache](https://github.com/AzureAD/microsoft-identity-web/wiki/token-cache-serialization) in der Microsoft.Identity.Web-Bibliothek verwenden.

Die `GetAccessTokenForUserAsync` Methode behandelt den Ablauf und die Aktualisierung von Token für Sie. Zuerst wird das zwischengespeicherte Token überprüft, und wenn es nicht abgelaufen ist, wird es zurückgegeben. Wenn es abgelaufen ist, verwendet es das zwischengespeicherte Aktualisierungstoken, um ein neues Token abzurufen.

Der **GraphServiceClient,** den Controller über die Abhängigkeitsinjektion erhalten, wird mit einem Authentifizierungsanbieter vorkonfiguriert, der für `GetAccessTokenForUserAsync` Sie verwendet wird.
