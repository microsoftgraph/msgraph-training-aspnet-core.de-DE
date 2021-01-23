---
ms.openlocfilehash: a024fb533c552563da6c9179301e16a2e1d09d5f
ms.sourcegitcommit: 6341ad07cd5b03269e7fd20cd3212e48baee7c07
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 01/23/2021
ms.locfileid: "49942162"
---
<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="a85d4-101">In dieser Übung erweitern Sie die Anwendung aus der vorherigen Übung, um die Authentifizierung mit Azure AD zu unterstützen.</span><span class="sxs-lookup"><span data-stu-id="a85d4-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="a85d4-102">Dies ist notwendig, um das erforderliche OAuth-Zugriffstoken zum Aufruf der Microsoft Graph-API abzurufen.</span><span class="sxs-lookup"><span data-stu-id="a85d4-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph API.</span></span> <span data-ttu-id="a85d4-103">In diesem Schritt konfigurieren Sie die [Microsoft.Identity.Web-Bibliothek.](https://www.nuget.org/packages/Microsoft.Identity.Web/)</span><span class="sxs-lookup"><span data-stu-id="a85d4-103">In this step you will configure the [Microsoft.Identity.Web](https://www.nuget.org/packages/Microsoft.Identity.Web/) library.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="a85d4-104">Um das Speichern der Anwendungs-ID und des geheimen Schlüssels in der Quelle zu vermeiden, verwenden Sie [den .NET Secret Manager,](/aspnet/core/security/app-secrets) um diese Werte zu speichern.</span><span class="sxs-lookup"><span data-stu-id="a85d4-104">To avoid storing the application ID and secret in source, you will use the [.NET Secret Manager](/aspnet/core/security/app-secrets) to store these values.</span></span> <span data-ttu-id="a85d4-105">Der geheime Manager dient nur zu Entwicklungszwecken, Produktions-Apps sollten einen vertrauenswürdigen geheimen Manager zum Speichern von geheimen Schlüsseln verwenden.</span><span class="sxs-lookup"><span data-stu-id="a85d4-105">The Secret Manager is for development purposes only, production apps should use a trusted secret manager for storing secrets.</span></span>

1. <span data-ttu-id="a85d4-106">Öffnen **Sie ./appsettings.js,** und ersetzen Sie den Inhalt durch Folgendes.</span><span class="sxs-lookup"><span data-stu-id="a85d4-106">Open **./appsettings.json** and replace its contents with the following.</span></span>

    :::code language="json" source="../demo/GraphTutorial/appsettings.json" highlight="2-6":::

1. <span data-ttu-id="a85d4-107">Öffnen Sie Ihre CLI im Verzeichnis, in dem **sich "GraphTutorial.csproj"** befindet, und führen Sie die folgenden Befehle aus, und ersetzen Sie sie durch Ihre Anwendungs-ID aus dem Azure-Portal und mit Ihrem Anwendungsgeheimnis. `YOUR_APP_ID` `YOUR_APP_SECRET`</span><span class="sxs-lookup"><span data-stu-id="a85d4-107">Open your CLI in the directory where **GraphTutorial.csproj** is located, and run the following commands, substituting `YOUR_APP_ID` with your application ID from the Azure portal, and `YOUR_APP_SECRET` with your application secret.</span></span>

    ```Shell
    dotnet user-secrets init
    dotnet user-secrets set "AzureAd:ClientId" "YOUR_APP_ID"
    dotnet user-secrets set "AzureAd:ClientSecret" "YOUR_APP_SECRET"
    ```

## <a name="implement-sign-in"></a><span data-ttu-id="a85d4-108">Implementieren der Anmeldung</span><span class="sxs-lookup"><span data-stu-id="a85d4-108">Implement sign-in</span></span>

<span data-ttu-id="a85d4-109">Beginnen Sie, indem Sie der Anwendung die Microsoft Identity Platform Services hinzufügen.</span><span class="sxs-lookup"><span data-stu-id="a85d4-109">Start by adding the Microsoft Identity platform services to the application.</span></span>

1. <span data-ttu-id="a85d4-110">Erstellen Sie eine neue Datei **namens GraphConstants.cs** im **Verzeichnis ./Graph,** und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="a85d4-110">Create a new file named **GraphConstants.cs** in the **./Graph** directory and add the following code.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Graph/GraphConstants.cs" id="GraphConstantsSnippet":::

1. <span data-ttu-id="a85d4-111">Öffnen Sie **die STARTUP.CS-Datei,** und fügen Sie die folgenden Anweisungen am Oberen Rand der Datei `using` hinzu.</span><span class="sxs-lookup"><span data-stu-id="a85d4-111">Open the **./Startup.cs** file and add the following `using` statements to the top of the file.</span></span>

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

1. <span data-ttu-id="a85d4-112">Ersetzen Sie die vorhandene `ConfigureServices`-Funktion durch Folgendes.</span><span class="sxs-lookup"><span data-stu-id="a85d4-112">Replace the existing `ConfigureServices` function with the following.</span></span>

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

1. <span data-ttu-id="a85d4-113">Fügen Sie `Configure` in der Funktion die folgende Zeile über der Zeile `app.UseAuthorization();` hinzu.</span><span class="sxs-lookup"><span data-stu-id="a85d4-113">In the `Configure` function, add the following line above the `app.UseAuthorization();` line.</span></span>

    ```csharp
    app.UseAuthentication();
    ```

1. <span data-ttu-id="a85d4-114">Öffnen **Sie ./Controllers/HomeController.cs,** und ersetzen Sie den Inhalt durch Folgendes.</span><span class="sxs-lookup"><span data-stu-id="a85d4-114">Open **./Controllers/HomeController.cs** and replace its contents with the following.</span></span>

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

1. <span data-ttu-id="a85d4-115">Speichern Sie Ihre Änderungen und starten Sie das Projekt.</span><span class="sxs-lookup"><span data-stu-id="a85d4-115">Save your changes and start the project.</span></span> <span data-ttu-id="a85d4-116">Melden Sie sich mit Ihrem Microsoft-Konto an.</span><span class="sxs-lookup"><span data-stu-id="a85d4-116">Login with your Microsoft account.</span></span>

1. <span data-ttu-id="a85d4-117">Überprüfen Sie die Zustimmungsaufforderung.</span><span class="sxs-lookup"><span data-stu-id="a85d4-117">Examine the consent prompt.</span></span> <span data-ttu-id="a85d4-118">Die Liste der Berechtigungen entspricht der Liste der Berechtigungsbereiche, die in **./Graph/GraphConstants.cs konfiguriert sind.**</span><span class="sxs-lookup"><span data-stu-id="a85d4-118">The list of permissions correspond to list of permissions scopes configured in **./Graph/GraphConstants.cs**.</span></span>

    - <span data-ttu-id="a85d4-119">**Behalten Sie den Zugriff auf Daten,** auf die Sie zugriffen haben: ( ) Diese Berechtigung wird von MSAL angefordert, um Aktualisierungstoken `offline_access` abzurufen.</span><span class="sxs-lookup"><span data-stu-id="a85d4-119">**Maintain access to data you have given it access to:** (`offline_access`) This permission is requested by MSAL in order to retrieve refresh tokens.</span></span>
    - <span data-ttu-id="a85d4-120">**Melden Sie sich an, und lesen** Sie Ihr Profil: ( ) Mit dieser Berechtigung kann die App das Profil und Profilfoto des angemeldeten `User.Read` Benutzers erhalten.</span><span class="sxs-lookup"><span data-stu-id="a85d4-120">**Sign you in and read your profile:** (`User.Read`) This permission allows the app to get the logged-in user's profile and profile photo.</span></span>
    - <span data-ttu-id="a85d4-121">**Lesen Der Postfacheinstellungen:** ( ) Mit dieser Berechtigung kann die App die Postfacheinstellungen des Benutzers lesen, einschließlich Zeitzone `MailboxSettings.Read` und Zeitzone.</span><span class="sxs-lookup"><span data-stu-id="a85d4-121">**Read your mailbox settings:** (`MailboxSettings.Read`) This permission allows the app to read the user's mailbox settings, including time zone and time format.</span></span>
    - <span data-ttu-id="a85d4-122">**Haben Vollzugriff auf** Ihre Kalender: ( ) Diese Berechtigung ermöglicht der App, Ereignisse im Kalender des Benutzers zu lesen, neue Ereignisse hinzuzufügen und `Calendars.ReadWrite` vorhandene zu ändern.</span><span class="sxs-lookup"><span data-stu-id="a85d4-122">**Have full access to your calendars:** (`Calendars.ReadWrite`) This permission allows the app to read events on the user's calendar, add new events, and modify existing ones.</span></span>

    ![Screenshot der Microsoft Identity Platform-Zustimmungsaufforderung](./images/add-aad-auth-03.png)

    <span data-ttu-id="a85d4-124">Weitere Informationen zur Zustimmung finden Sie unter [Understanding Azure AD application consent experiences](/azure/active-directory/develop/application-consent-experience).</span><span class="sxs-lookup"><span data-stu-id="a85d4-124">For more information regarding consent, see [Understanding Azure AD application consent experiences](/azure/active-directory/develop/application-consent-experience).</span></span>

1. <span data-ttu-id="a85d4-125">Stimmen Sie den angeforderten Berechtigungen zu.</span><span class="sxs-lookup"><span data-stu-id="a85d4-125">Consent to the requested permissions.</span></span> <span data-ttu-id="a85d4-126">Der Browser leitet zur App um, in der Sie das Token sehen.</span><span class="sxs-lookup"><span data-stu-id="a85d4-126">The browser redirects to the app, showing the token.</span></span>

### <a name="get-user-details"></a><span data-ttu-id="a85d4-127">Benutzerdetails abrufen</span><span class="sxs-lookup"><span data-stu-id="a85d4-127">Get user details</span></span>

<span data-ttu-id="a85d4-128">Sobald sich der Benutzer angemeldet hat, können Sie dessen Informationen über Microsoft Graph abrufen.</span><span class="sxs-lookup"><span data-stu-id="a85d4-128">Once the user is logged in, you can get their information from Microsoft Graph.</span></span>

1. <span data-ttu-id="a85d4-129">Öffnen **Sie ./Graph/GraphClaimsPrincipalExtensions.cs,** und ersetzen Sie den gesamten Inhalt durch Folgendes.</span><span class="sxs-lookup"><span data-stu-id="a85d4-129">Open **./Graph/GraphClaimsPrincipalExtensions.cs** and replace its entire contents with the following.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Graph/GraphClaimsPrincipalExtensions.cs" id="GraphClaimsExtensionsSnippet":::

1. <span data-ttu-id="a85d4-130">Öffnen **Sie ./Startup.cs,** und ersetzen Sie die vorhandene `.AddMicrosoftIdentityWebApp(Configuration)` Zeile durch den folgenden Code.</span><span class="sxs-lookup"><span data-stu-id="a85d4-130">Open **./Startup.cs** and replace the existing `.AddMicrosoftIdentityWebApp(Configuration)` line with the following code.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Startup.cs" id="AddSignInSnippet":::

    <span data-ttu-id="a85d4-131">Überlegen Sie, was dieser Code macht.</span><span class="sxs-lookup"><span data-stu-id="a85d4-131">Consider what this code does.</span></span>

    - <span data-ttu-id="a85d4-132">Es wird ein Ereignishandler für das Ereignis `OnTokenValidated` hinzugefügt.</span><span class="sxs-lookup"><span data-stu-id="a85d4-132">It adds an event handler for the `OnTokenValidated` event.</span></span>
        - <span data-ttu-id="a85d4-133">Sie verwendet die `ITokenAcquisition` Schnittstelle, um ein Zugriffstoken abzurufen.</span><span class="sxs-lookup"><span data-stu-id="a85d4-133">It uses the `ITokenAcquisition` interface to get an access token.</span></span>
        - <span data-ttu-id="a85d4-134">Es ruft Microsoft Graph auf, um das Profil und Foto des Benutzers zu erhalten.</span><span class="sxs-lookup"><span data-stu-id="a85d4-134">It calls Microsoft Graph to get the user's profile and photo.</span></span>
        - <span data-ttu-id="a85d4-135">Die Graphinformationen werden der Identität des Benutzers hinzufügt.</span><span class="sxs-lookup"><span data-stu-id="a85d4-135">It adds the Graph information to the user's identity.</span></span>

1. <span data-ttu-id="a85d4-136">Fügen Sie den folgenden Funktionsaufruf nach dem `EnableTokenAcquisitionToCallDownstreamApi` Aufruf und vor dem Aufruf `AddInMemoryTokenCaches` hinzu.</span><span class="sxs-lookup"><span data-stu-id="a85d4-136">Add the following function call after the `EnableTokenAcquisitionToCallDownstreamApi` call and before the `AddInMemoryTokenCaches` call.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Startup.cs" id="AddGraphClientSnippet":::

    <span data-ttu-id="a85d4-137">Dadurch wird ein authentifizierter **GraphServiceClient** über eine Abhängigkeitsinjektion für Controller verfügbar.</span><span class="sxs-lookup"><span data-stu-id="a85d4-137">This will make an authenticated **GraphServiceClient** available to controllers via dependency injection.</span></span>

1. <span data-ttu-id="a85d4-138">Öffnen **Sie ./Controllers/HomeController.cs,** und ersetzen Sie `Index` die Funktion durch Folgendes.</span><span class="sxs-lookup"><span data-stu-id="a85d4-138">Open **./Controllers/HomeController.cs** and replace the `Index` function with the following.</span></span>

    ```csharp
    public IActionResult Index()
    {
        return View();
    }
    ```

1. <span data-ttu-id="a85d4-139">Entfernen Sie alle Verweise `ITokenAcquisition` auf in der **HomeController-Klasse.**</span><span class="sxs-lookup"><span data-stu-id="a85d4-139">Remove all references to `ITokenAcquisition` in the **HomeController** class.</span></span>

1. <span data-ttu-id="a85d4-140">Speichern Sie Ihre Änderungen, starten Sie die App, und gehen Sie den Anmeldevorgang durch.</span><span class="sxs-lookup"><span data-stu-id="a85d4-140">Save your changes, start the app, and go through the sign-in process.</span></span> <span data-ttu-id="a85d4-141">Sie sollten wieder auf der Startseite landen, aber die Benutzeroberfläche sollte geändert werden, um anzugeben, dass Sie angemeldet sind.</span><span class="sxs-lookup"><span data-stu-id="a85d4-141">You should end up back on the home page, but the UI should change to indicate that you are signed-in.</span></span>

    ![Screenshot der Startseite nach dem Anmelden](./images/add-aad-auth-01.png)

1. <span data-ttu-id="a85d4-143">Klicken Sie in der oberen rechten Ecke auf den Avatar des Benutzers, um auf den Link **"Abmelden" zu** klicken.</span><span class="sxs-lookup"><span data-stu-id="a85d4-143">Click the user avatar in the top right corner to access the **Sign Out** link.</span></span> <span data-ttu-id="a85d4-144">Wenn Sie auf **Abmelden** klicken, wird die Sitzung zurückgesetzt und Sie kehren zur Startseite zurück.</span><span class="sxs-lookup"><span data-stu-id="a85d4-144">Clicking **Sign Out** resets the session and returns you to the home page.</span></span>

    ![Screenshot des Dropdown-Menüs mit dem Link „Abmelden“](./images/add-aad-auth-02.png)

> [!TIP]
> <span data-ttu-id="a85d4-146">Wenn Ihr Benutzername nicht auf der Startseite angezeigt wird und die Dropdownliste "Avatar verwenden" nach diesen Änderungen keinen Namen und keine E-Mail enthält, melden Sie sich ab und wieder an.</span><span class="sxs-lookup"><span data-stu-id="a85d4-146">If you do not see your user name on the home page and the use avatar dropdown is missing name and email after making these changes, sign out and sign back in.</span></span>

## <a name="storing-and-refreshing-tokens"></a><span data-ttu-id="a85d4-147">Speichern und Aktualisieren von Token</span><span class="sxs-lookup"><span data-stu-id="a85d4-147">Storing and refreshing tokens</span></span>

<span data-ttu-id="a85d4-148">An diesem Punkt verfügt Ihre Anwendung über ein Zugriffstoken, das im `Authorization` Header von API-Aufrufen gesendet wird.</span><span class="sxs-lookup"><span data-stu-id="a85d4-148">At this point your application has an access token, which is sent in the `Authorization` header of API calls.</span></span> <span data-ttu-id="a85d4-149">Dies ist das Token, durch das die App im Namen des Benutzers auf Microsoft Graph zugreifen kann.</span><span class="sxs-lookup"><span data-stu-id="a85d4-149">This is the token that allows the app to access Microsoft Graph on the user's behalf.</span></span>

<span data-ttu-id="a85d4-150">Dieses Token ist jedoch nur kurzzeitig verfügbar.</span><span class="sxs-lookup"><span data-stu-id="a85d4-150">However, this token is short-lived.</span></span> <span data-ttu-id="a85d4-151">Das Token läuft eine Stunde nach seiner Entsprechung ab.</span><span class="sxs-lookup"><span data-stu-id="a85d4-151">The token expires an hour after it is issued.</span></span> <span data-ttu-id="a85d4-152">An dieser Stelle kommt das Aktualisierungstoken ins Spiel.</span><span class="sxs-lookup"><span data-stu-id="a85d4-152">This is where the refresh token becomes useful.</span></span> <span data-ttu-id="a85d4-153">Anhand des Aktualisierungstoken ist die App in der Lage, ein neues Zugriffstoken anzufordern, ohne dass der Benutzer sich erneut anmelden muss.</span><span class="sxs-lookup"><span data-stu-id="a85d4-153">The refresh token allows the app to request a new access token without requiring the user to sign in again.</span></span>

<span data-ttu-id="a85d4-154">Da die App die Microsoft.Identity.Web-Bibliothek verwendet, müssen Sie keine Tokenspeicher- oder Aktualisierungslogik implementieren.</span><span class="sxs-lookup"><span data-stu-id="a85d4-154">Because the app is using the Microsoft.Identity.Web library, you do not have to implement any token storage or refresh logic.</span></span>

<span data-ttu-id="a85d4-155">Die App verwendet den Speichertokencache, der für Apps ausreicht, die beim Neustart der App keine Token beibehalten müssen.</span><span class="sxs-lookup"><span data-stu-id="a85d4-155">The app uses the in-memory token cache, which is sufficient for apps that do not need to persist tokens when the app restarts.</span></span> <span data-ttu-id="a85d4-156">Produktionsanwendungen können stattdessen die Optionen für [den verteilten Cache](https://github.com/AzureAD/microsoft-identity-web/wiki/token-cache-serialization) in der Microsoft.Identity.Web-Bibliothek verwenden.</span><span class="sxs-lookup"><span data-stu-id="a85d4-156">Production apps may instead use the [distributed cache options](https://github.com/AzureAD/microsoft-identity-web/wiki/token-cache-serialization) in the Microsoft.Identity.Web library.</span></span>

<span data-ttu-id="a85d4-157">Die `GetAccessTokenForUserAsync` Methode behandelt den Ablauf und die Aktualisierung von Token für Sie.</span><span class="sxs-lookup"><span data-stu-id="a85d4-157">The `GetAccessTokenForUserAsync` method handles token expiration and refresh for you.</span></span> <span data-ttu-id="a85d4-158">Zuerst wird das zwischengespeicherte Token überprüft, und wenn es nicht abgelaufen ist, wird es zurückgegeben.</span><span class="sxs-lookup"><span data-stu-id="a85d4-158">It first checks the cached token, and if it is not expired, it returns it.</span></span> <span data-ttu-id="a85d4-159">Wenn es abgelaufen ist, verwendet es das zwischengespeicherte Aktualisierungstoken, um ein neues Token abzurufen.</span><span class="sxs-lookup"><span data-stu-id="a85d4-159">If it is expired, it uses the cached refresh token to obtain a new one.</span></span>

<span data-ttu-id="a85d4-160">Der **GraphServiceClient,** den Controller über die Abhängigkeitsinjektion erhalten, wird mit einem Authentifizierungsanbieter vorkonfiguriert, der für `GetAccessTokenForUserAsync` Sie verwendet wird.</span><span class="sxs-lookup"><span data-stu-id="a85d4-160">The **GraphServiceClient** that controllers get via dependency injection will be pre-configured with an authentication provider that uses `GetAccessTokenForUserAsync` for you.</span></span>
