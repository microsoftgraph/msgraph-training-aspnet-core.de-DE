---
ms.openlocfilehash: c954903f38d48bcca4c534f4d0cfbe1605cbf7f6
ms.sourcegitcommit: 6341ad07cd5b03269e7fd20cd3212e48baee7c07
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 01/23/2021
ms.locfileid: "49942148"
---
<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="7f243-101">In diesem Abschnitt integrieren Sie Microsoft Graph in die Anwendung.</span><span class="sxs-lookup"><span data-stu-id="7f243-101">In this section you will incorporate Microsoft Graph into the application.</span></span> <span data-ttu-id="7f243-102">Für diese Anwendung verwenden Sie die [Microsoft Graph-Clientbibliothek für .NET,](https://github.com/microsoftgraph/msgraph-sdk-dotnet) um Aufrufe an Microsoft Graph zu senden.</span><span class="sxs-lookup"><span data-stu-id="7f243-102">For this application, you will use the [Microsoft Graph Client Library for .NET](https://github.com/microsoftgraph/msgraph-sdk-dotnet) to make calls to Microsoft Graph.</span></span>

## <a name="get-calendar-events-from-outlook"></a><span data-ttu-id="7f243-103">Abrufen von Kalenderereignissen von Outlook</span><span class="sxs-lookup"><span data-stu-id="7f243-103">Get calendar events from Outlook</span></span>

<span data-ttu-id="7f243-104">Erstellen Sie zunächst einen neuen Controller für Kalenderansichten.</span><span class="sxs-lookup"><span data-stu-id="7f243-104">Start by creating a new controller for calendar views.</span></span>

1. <span data-ttu-id="7f243-105">Fügen Sie eine  neue Datei CalendarController.cs im Verzeichnis **"./Controllers"** hinzu, und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="7f243-105">Add a new file named **CalendarController.cs** in the **./Controllers** directory and add the following code.</span></span>

    ```csharp
    using GraphTutorial.Models;
    using Microsoft.AspNetCore.Mvc;
    using Microsoft.Extensions.Logging;
    using Microsoft.Identity.Web;
    using Microsoft.Graph;
    using System;
    using System.Collections.Generic;
    using System.Threading.Tasks;
    using TimeZoneConverter;

    namespace GraphTutorial.Controllers
    {
        public class CalendarController : Controller
        {
            private readonly GraphServiceClient _graphClient;
            private readonly ILogger<HomeController> _logger;

            public CalendarController(
                GraphServiceClient graphClient,
                ILogger<HomeController> logger)
            {
                _graphClient = graphClient;
                _logger = logger;
            }
        }
    }
    ```

1. <span data-ttu-id="7f243-106">Fügen Sie der Klasse die folgenden Funktionen `CalendarController` hinzu, um die Kalenderansicht des Benutzers zu erhalten.</span><span class="sxs-lookup"><span data-stu-id="7f243-106">Add the following functions to the `CalendarController` class to get the user's calendar view.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Controllers/CalendarController.cs" id="GetCalendarViewSnippet":::

    <span data-ttu-id="7f243-107">Überlegen Sie, was der Code `GetUserWeekCalendar` macht.</span><span class="sxs-lookup"><span data-stu-id="7f243-107">Consider what the code in `GetUserWeekCalendar` does.</span></span>

    - <span data-ttu-id="7f243-108">Es verwendet die Zeitzone des Benutzers, um utc-Start- und Enddatum/-uhrzeitwerte für die Woche zu erhalten.</span><span class="sxs-lookup"><span data-stu-id="7f243-108">It uses the user's time zone to get UTC start and end date/time values for the week.</span></span>
    - <span data-ttu-id="7f243-109">Er fragt die Kalenderansicht des Benutzers ab, um alle Ereignisse zu erhalten, die zwischen dem Start- und Enddatum/der Endzeit liegen. [](/graph/api/calendar-list-calendarview?view=graph-rest-1.0)</span><span class="sxs-lookup"><span data-stu-id="7f243-109">It queries the user's [calendar view](/graph/api/calendar-list-calendarview?view=graph-rest-1.0) to get all events that fall between the start and end date/times.</span></span> <span data-ttu-id="7f243-110">Wenn Sie eine Kalenderansicht verwenden, anstatt [Ereignisse](/graph/api/user-list-events?view=graph-rest-1.0) auflisten zu müssen, werden wiederkehrende Ereignisse erweitert, und es werden alle Vorkommen im angegebenen Zeitfenster angezeigt.</span><span class="sxs-lookup"><span data-stu-id="7f243-110">Using a calendar view instead of [listing events](/graph/api/user-list-events?view=graph-rest-1.0) expands recurring events, returning any occurrences that happen in the specified time window.</span></span>
    - <span data-ttu-id="7f243-111">Sie verwendet die Kopfzeile, um Ergebnisse in der Zeitzone des `Prefer: outlook.timezone` Benutzers zurück zu erhalten.</span><span class="sxs-lookup"><span data-stu-id="7f243-111">It uses the `Prefer: outlook.timezone` header to get results back in the user's timezone.</span></span>
    - <span data-ttu-id="7f243-112">Es wird verwendet, um die Felder zu beschränken, die auf die von der App `Select` verwendeten zurückkommen.</span><span class="sxs-lookup"><span data-stu-id="7f243-112">It uses `Select` to limit the fields that come back to just those used by the app.</span></span>
    - <span data-ttu-id="7f243-113">Sie wird `OrderBy` verwendet, um die Ergebnisse chronologisch zu sortieren.</span><span class="sxs-lookup"><span data-stu-id="7f243-113">It uses `OrderBy` to sort the results chronologically.</span></span>
    - <span data-ttu-id="7f243-114">Es wird eine `PageIterator` ["to"-Seite durch die Ereignissammlung verwendet.](/graph/sdks/paging)</span><span class="sxs-lookup"><span data-stu-id="7f243-114">It uses a `PageIterator` to [page through the events collection](/graph/sdks/paging).</span></span> <span data-ttu-id="7f243-115">Dadurch wird der Fall behandelt, dass der Benutzer mehr Ereignisse im Kalender hat als die angeforderte Seitengröße.</span><span class="sxs-lookup"><span data-stu-id="7f243-115">This handles the case where the user has more events on their calendar than the requested page size.</span></span>

1. <span data-ttu-id="7f243-116">Fügen Sie der Klasse die folgende Funktion `CalendarController` hinzu, um eine temporäre Ansicht der zurückgegebenen Daten zu implementieren.</span><span class="sxs-lookup"><span data-stu-id="7f243-116">Add the following function to the `CalendarController` class to implement a temporary view of the returned data.</span></span>

    ```csharp
    // Minimum permission scope needed for this view
    [AuthorizeForScopes(Scopes = new[] { "Calendars.Read" })]
    public async Task<IActionResult> Index()
    {
        try
        {
            var userTimeZone = TZConvert.GetTimeZoneInfo(
                User.GetUserGraphTimeZone());
            var startOfWeek = CalendarController.GetUtcStartOfWeekInTimeZone(
                DateTime.Today, userTimeZone);

            var events = await GetUserWeekCalendar(startOfWeek);

            // Return a JSON dump of events
            return new ContentResult {
                Content = _graphClient.HttpProvider.Serializer.SerializeObject(events),
                ContentType = "application/json"
            };
        }
        catch (ServiceException ex)
        {
            if (ex.InnerException is MicrosoftIdentityWebChallengeUserException)
            {
                throw;
            }

            return new ContentResult {
                Content = $"Error getting calendar view: {ex.Message}",
                ContentType = "text/plain"
            };
        }
    }
    ```

1. <span data-ttu-id="7f243-117">Starten Sie die App, melden Sie sich an, und klicken Sie auf den Link **Kalender** in der Navigationsleiste.</span><span class="sxs-lookup"><span data-stu-id="7f243-117">Start the app, sign in, and click the **Calendar** link in the nav bar.</span></span> <span data-ttu-id="7f243-118">Wenn alles funktioniert, sollte ein JSON-Abbild von Ereignissen im Kalender des Benutzers angezeigt werden.</span><span class="sxs-lookup"><span data-stu-id="7f243-118">If everything works, you should see a JSON dump of events on the user's calendar.</span></span>

## <a name="display-the-results"></a><span data-ttu-id="7f243-119">Anzeigen der Ergebnisse</span><span class="sxs-lookup"><span data-stu-id="7f243-119">Display the results</span></span>

<span data-ttu-id="7f243-120">Jetzt können Sie eine Ansicht hinzufügen, um die Ergebnisse benutzerfreundlicher anzuzeigen.</span><span class="sxs-lookup"><span data-stu-id="7f243-120">Now you can add a view to display the results in a more user-friendly manner.</span></span>

### <a name="create-view-models"></a><span data-ttu-id="7f243-121">Erstellen von Ansichtsmodellen</span><span class="sxs-lookup"><span data-stu-id="7f243-121">Create view models</span></span>

1. <span data-ttu-id="7f243-122">Erstellen Sie eine neue Datei **namens CalendarViewEvent.cs** im Verzeichnis **"./Models",** und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="7f243-122">Create a new file named **CalendarViewEvent.cs** in the **./Models** directory and add the following code.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Models/CalendarViewEvent.cs" id="CalendarViewEventSnippet":::

1. <span data-ttu-id="7f243-123">Erstellen Sie eine neue Datei **namens DailyViewModel.cs** im Verzeichnis **"./Models",** und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="7f243-123">Create a new file named **DailyViewModel.cs** in the **./Models** directory and add the following code.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Models/DailyViewModel.cs" id="DailyViewModelSnippet":::

1. <span data-ttu-id="7f243-124">Erstellen Sie eine neue Datei **namens CalendarViewModel.cs** im Verzeichnis **"./Models",** und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="7f243-124">Create a new file named **CalendarViewModel.cs** in the **./Models** directory and add the following code.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Models/CalendarViewModel.cs" id="CalendarViewModelSnippet":::

### <a name="create-views"></a><span data-ttu-id="7f243-125">Erstellen von Ansichten</span><span class="sxs-lookup"><span data-stu-id="7f243-125">Create views</span></span>

1. <span data-ttu-id="7f243-126">Erstellen Sie ein neues Verzeichnis mit **dem Namen "Kalender"** im Verzeichnis **"./Views".**</span><span class="sxs-lookup"><span data-stu-id="7f243-126">Create a new directory named **Calendar** in the **./Views** directory.</span></span>

1. <span data-ttu-id="7f243-127">Erstellen Sie eine neue Datei namens **_DailyEventsPartial.cshtml** im Verzeichnis **./Views/Calendar,** und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="7f243-127">Create a new file named **_DailyEventsPartial.cshtml** in the **./Views/Calendar** directory and add the following code.</span></span>

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Calendar/_DailyEventsPartial.cshtml" id="DailyEventsPartialSnippet":::

1. <span data-ttu-id="7f243-128">Erstellen Sie eine neue Datei namens **"Index.cshtml"** im Verzeichnis **"./Views/Calendar",** und fügen Sie den folgenden Code hinzu.</span><span class="sxs-lookup"><span data-stu-id="7f243-128">Create a new file named **Index.cshtml** in the **./Views/Calendar** directory and add the following code.</span></span>

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Calendar/Index.cshtml" id="CalendarIndexSnippet":::

### <a name="update-calendar-controller"></a><span data-ttu-id="7f243-129">Aktualisieren des Kalendercontrollers</span><span class="sxs-lookup"><span data-stu-id="7f243-129">Update calendar controller</span></span>

1. <span data-ttu-id="7f243-130">Öffnen **Sie ./Controllers/CalendarController.cs,** und ersetzen Sie die vorhandene `Index` Funktion durch Folgendes.</span><span class="sxs-lookup"><span data-stu-id="7f243-130">Open **./Controllers/CalendarController.cs** and replace the existing `Index` function with the following.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Controllers/CalendarController.cs" id="IndexSnippet":::

1. <span data-ttu-id="7f243-131">Starten Sie die App, melden Sie sich an, und klicken Sie auf den Link **Kalender**.</span><span class="sxs-lookup"><span data-stu-id="7f243-131">Start the app, sign in, and click the **Calendar** link.</span></span> <span data-ttu-id="7f243-132">Die App sollte nun eine Tabelle mit Ereignissen rendern.</span><span class="sxs-lookup"><span data-stu-id="7f243-132">The app should now render a table of events.</span></span>

    ![Ein Screenshot der Tabelle mit Ereignissen](./images/add-msgraph-01.png)
