---
ms.openlocfilehash: c954903f38d48bcca4c534f4d0cfbe1605cbf7f6
ms.sourcegitcommit: 6341ad07cd5b03269e7fd20cd3212e48baee7c07
ms.translationtype: MT
ms.contentlocale: de-DE
ms.lasthandoff: 01/23/2021
ms.locfileid: "49942148"
---
<!-- markdownlint-disable MD002 MD041 -->

In diesem Abschnitt integrieren Sie Microsoft Graph in die Anwendung. Für diese Anwendung verwenden Sie die [Microsoft Graph-Clientbibliothek für .NET,](https://github.com/microsoftgraph/msgraph-sdk-dotnet) um Aufrufe an Microsoft Graph zu senden.

## <a name="get-calendar-events-from-outlook"></a>Abrufen von Kalenderereignissen von Outlook

Erstellen Sie zunächst einen neuen Controller für Kalenderansichten.

1. Fügen Sie eine  neue Datei CalendarController.cs im Verzeichnis **"./Controllers"** hinzu, und fügen Sie den folgenden Code hinzu.

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

1. Fügen Sie der Klasse die folgenden Funktionen `CalendarController` hinzu, um die Kalenderansicht des Benutzers zu erhalten.

    :::code language="csharp" source="../demo/GraphTutorial/Controllers/CalendarController.cs" id="GetCalendarViewSnippet":::

    Überlegen Sie, was der Code `GetUserWeekCalendar` macht.

    - Es verwendet die Zeitzone des Benutzers, um utc-Start- und Enddatum/-uhrzeitwerte für die Woche zu erhalten.
    - Er fragt die Kalenderansicht des Benutzers ab, um alle Ereignisse zu erhalten, die zwischen dem Start- und Enddatum/der Endzeit liegen. [](/graph/api/calendar-list-calendarview?view=graph-rest-1.0) Wenn Sie eine Kalenderansicht verwenden, anstatt [Ereignisse](/graph/api/user-list-events?view=graph-rest-1.0) auflisten zu müssen, werden wiederkehrende Ereignisse erweitert, und es werden alle Vorkommen im angegebenen Zeitfenster angezeigt.
    - Sie verwendet die Kopfzeile, um Ergebnisse in der Zeitzone des `Prefer: outlook.timezone` Benutzers zurück zu erhalten.
    - Es wird verwendet, um die Felder zu beschränken, die auf die von der App `Select` verwendeten zurückkommen.
    - Sie wird `OrderBy` verwendet, um die Ergebnisse chronologisch zu sortieren.
    - Es wird eine `PageIterator` ["to"-Seite durch die Ereignissammlung verwendet.](/graph/sdks/paging) Dadurch wird der Fall behandelt, dass der Benutzer mehr Ereignisse im Kalender hat als die angeforderte Seitengröße.

1. Fügen Sie der Klasse die folgende Funktion `CalendarController` hinzu, um eine temporäre Ansicht der zurückgegebenen Daten zu implementieren.

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

1. Starten Sie die App, melden Sie sich an, und klicken Sie auf den Link **Kalender** in der Navigationsleiste. Wenn alles funktioniert, sollte ein JSON-Abbild von Ereignissen im Kalender des Benutzers angezeigt werden.

## <a name="display-the-results"></a>Anzeigen der Ergebnisse

Jetzt können Sie eine Ansicht hinzufügen, um die Ergebnisse benutzerfreundlicher anzuzeigen.

### <a name="create-view-models"></a>Erstellen von Ansichtsmodellen

1. Erstellen Sie eine neue Datei **namens CalendarViewEvent.cs** im Verzeichnis **"./Models",** und fügen Sie den folgenden Code hinzu.

    :::code language="csharp" source="../demo/GraphTutorial/Models/CalendarViewEvent.cs" id="CalendarViewEventSnippet":::

1. Erstellen Sie eine neue Datei **namens DailyViewModel.cs** im Verzeichnis **"./Models",** und fügen Sie den folgenden Code hinzu.

    :::code language="csharp" source="../demo/GraphTutorial/Models/DailyViewModel.cs" id="DailyViewModelSnippet":::

1. Erstellen Sie eine neue Datei **namens CalendarViewModel.cs** im Verzeichnis **"./Models",** und fügen Sie den folgenden Code hinzu.

    :::code language="csharp" source="../demo/GraphTutorial/Models/CalendarViewModel.cs" id="CalendarViewModelSnippet":::

### <a name="create-views"></a>Erstellen von Ansichten

1. Erstellen Sie ein neues Verzeichnis mit **dem Namen "Kalender"** im Verzeichnis **"./Views".**

1. Erstellen Sie eine neue Datei namens **_DailyEventsPartial.cshtml** im Verzeichnis **./Views/Calendar,** und fügen Sie den folgenden Code hinzu.

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Calendar/_DailyEventsPartial.cshtml" id="DailyEventsPartialSnippet":::

1. Erstellen Sie eine neue Datei namens **"Index.cshtml"** im Verzeichnis **"./Views/Calendar",** und fügen Sie den folgenden Code hinzu.

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Calendar/Index.cshtml" id="CalendarIndexSnippet":::

### <a name="update-calendar-controller"></a>Aktualisieren des Kalendercontrollers

1. Öffnen **Sie ./Controllers/CalendarController.cs,** und ersetzen Sie die vorhandene `Index` Funktion durch Folgendes.

    :::code language="csharp" source="../demo/GraphTutorial/Controllers/CalendarController.cs" id="IndexSnippet":::

1. Starten Sie die App, melden Sie sich an, und klicken Sie auf den Link **Kalender**. Die App sollte nun eine Tabelle mit Ereignissen rendern.

    ![Ein Screenshot der Tabelle mit Ereignissen](./images/add-msgraph-01.png)
