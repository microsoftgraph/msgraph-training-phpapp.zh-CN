<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="66e1b-101">在本练习中，将把 Microsoft Graph 合并到应用程序中。</span><span class="sxs-lookup"><span data-stu-id="66e1b-101">In this exercise you will incorporate the Microsoft Graph into the application.</span></span> <span data-ttu-id="66e1b-102">对于此应用程序，您将使用 [microsoft graph](https://github.com/microsoftgraph/msgraph-sdk-php) 库对 microsoft graph 进行调用。</span><span class="sxs-lookup"><span data-stu-id="66e1b-102">For this application, you will use the [microsoft-graph](https://github.com/microsoftgraph/msgraph-sdk-php) library to make calls to Microsoft Graph.</span></span>

## <a name="get-calendar-events-from-outlook"></a><span data-ttu-id="66e1b-103">从 Outlook 获取日历事件</span><span class="sxs-lookup"><span data-stu-id="66e1b-103">Get calendar events from Outlook</span></span>

1. <span data-ttu-id="66e1b-104">在 **/app** 目录中创建一个名为的新目录 `TimeZones` ，然后在名为的该目录中创建一个新文件， `TimeZones.php` 并添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="66e1b-104">Create a new directory in the **./app** directory named `TimeZones`, then create a new file in that directory named `TimeZones.php`, and add the following code.</span></span>

    :::code language="php" source="../demo/graph-tutorial/app/TimeZones/TimeZones.php":::

    <span data-ttu-id="66e1b-105">此类实现 Windows 时区名称到 IANA 时区标识符的简化映射。</span><span class="sxs-lookup"><span data-stu-id="66e1b-105">This class implements a simplistic mapping of Windows time zone names to IANA time zone identifiers.</span></span>

1. <span data-ttu-id="66e1b-106">在 " **/app/Http/Controllers** " 目录中创建一个名为的新文件 `CalendarController.php` ，并添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="66e1b-106">Create a new file in the **./app/Http/Controllers** directory named `CalendarController.php`, and add the following code.</span></span>

    ```php
    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use Illuminate\Http\Request;
    use Microsoft\Graph\Graph;
    use Microsoft\Graph\Model;
    use App\TokenStore\TokenCache;
    use App\TimeZones\TimeZones;

    class CalendarController extends Controller
    {
      public function calendar()
      {
        $viewData = $this->loadViewData();

        $graph = $this->getGraph();

        // Get user's timezone
        $timezone = TimeZones::getTzFromWindows($viewData['userTimeZone']);

        // Get start and end of week
        $startOfWeek = new \DateTimeImmutable('sunday -1 week', $timezone);
        $endOfWeek = new \DateTimeImmutable('sunday', $timezone);

        $queryParams = array(
          'startDateTime' => $startOfWeek->format(\DateTimeInterface::ISO8601),
          'endDateTime' => $endOfWeek->format(\DateTimeInterface::ISO8601),
          // Only request the properties used by the app
          '$select' => 'subject,organizer,start,end',
          // Sort them by start time
          '$orderby' => 'start/dateTime',
          // Limit results to 25
          '$top' => 25
        );

        // Append query parameters to the '/me/calendarView' url
        $getEventsUrl = '/me/calendarView?'.http_build_query($queryParams);

        $events = $graph->createRequest('GET', $getEventsUrl)
          // Add the user's timezone to the Prefer header
          ->addHeaders(array(
            'Prefer' => 'outlook.timezone="'.$viewData['userTimeZone'].'"'
          ))
          ->setReturnType(Model\Event::class)
          ->execute();

        return response()->json($events);
      }

      private function getGraph(): Graph
      {
        // Get the access token from the cache
        $tokenCache = new TokenCache();
        $accessToken = $tokenCache->getAccessToken();

        // Create a Graph client
        $graph = new Graph();
        $graph->setAccessToken($accessToken);
        return $graph;
      }
    }
    ```

    <span data-ttu-id="66e1b-107">考虑此代码执行的操作。</span><span class="sxs-lookup"><span data-stu-id="66e1b-107">Consider what this code is doing.</span></span>

    - <span data-ttu-id="66e1b-108">将调用的 URL 为 `/v1.0/me/calendarView` 。</span><span class="sxs-lookup"><span data-stu-id="66e1b-108">The URL that will be called is `/v1.0/me/calendarView`.</span></span>
    - <span data-ttu-id="66e1b-109">`startDateTime`和 `endDateTime` 参数定义视图的开始和结束。</span><span class="sxs-lookup"><span data-stu-id="66e1b-109">The `startDateTime` and `endDateTime` parameters define the start and end of the view.</span></span>
    - <span data-ttu-id="66e1b-110">`$select`参数将为每个事件返回的字段限制为仅显示视图实际使用的字段。</span><span class="sxs-lookup"><span data-stu-id="66e1b-110">The `$select` parameter limits the fields returned for each events to just those the view will actually use.</span></span>
    - <span data-ttu-id="66e1b-111">`$orderby`参数按其创建日期和时间对结果进行排序，最新项目最先开始。</span><span class="sxs-lookup"><span data-stu-id="66e1b-111">The `$orderby` parameter sorts the results by the date and time they were created, with the most recent item being first.</span></span>
    - <span data-ttu-id="66e1b-112">`$top`参数将结果限制为25个事件。</span><span class="sxs-lookup"><span data-stu-id="66e1b-112">The `$top` parameter limits the results to 25 events.</span></span>
    - <span data-ttu-id="66e1b-113">`Prefer: outlook.timezone=""`标头会导致响应中的开始和结束时间调整为用户的首选时区。</span><span class="sxs-lookup"><span data-stu-id="66e1b-113">The `Prefer: outlook.timezone=""` header causes the start and end times in the response to be adjusted to the user's preferred time zone.</span></span>

1. <span data-ttu-id="66e1b-114">更新 **/routes/web.php** 中的路由，以将路由添加到此新控制器。</span><span class="sxs-lookup"><span data-stu-id="66e1b-114">Update the routes in **./routes/web.php** to add a route to this new controller.</span></span>

    ```php
    Route::get('/calendar', 'CalendarController@calendar');
    ```

1. <span data-ttu-id="66e1b-115">登录并单击导航栏中的 " **日历** " 链接。</span><span class="sxs-lookup"><span data-stu-id="66e1b-115">Sign in and click the **Calendar** link in the nav bar.</span></span> <span data-ttu-id="66e1b-116">如果一切正常，应在用户的日历上看到一个 JSON 转储的事件。</span><span class="sxs-lookup"><span data-stu-id="66e1b-116">If everything works, you should see a JSON dump of events on the user's calendar.</span></span>

## <a name="display-the-results"></a><span data-ttu-id="66e1b-117">显示结果</span><span class="sxs-lookup"><span data-stu-id="66e1b-117">Display the results</span></span>

<span data-ttu-id="66e1b-118">现在，您可以添加一个视图，以对用户更友好的方式显示结果。</span><span class="sxs-lookup"><span data-stu-id="66e1b-118">Now you can add a view to display the results in a more user-friendly manner.</span></span>

1. <span data-ttu-id="66e1b-119">在 **/resources/views** 目录中创建一个名为的新文件 `calendar.blade.php` ，并添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="66e1b-119">Create a new file in the **./resources/views** directory named `calendar.blade.php` and add the following code.</span></span>

    :::code language="php" source="../demo/graph-tutorial/resources/views/calendar.blade.php" id="CalendarSnippet":::

    <span data-ttu-id="66e1b-120">这将遍历一组事件并为每个事件添加一个表行。</span><span class="sxs-lookup"><span data-stu-id="66e1b-120">That will loop through a collection of events and add a table row for each one.</span></span>

1. <span data-ttu-id="66e1b-121">`return response()->json($events);`从 `calendar` **/app/Http/Controllers/CalendarController.php**中删除操作的行，并将其替换为以下代码。</span><span class="sxs-lookup"><span data-stu-id="66e1b-121">Remove the `return response()->json($events);` line from the `calendar` action in **./app/Http/Controllers/CalendarController.php**, and replace it with the following code.</span></span>

    ```php
    $viewData['events'] = $events;
    return view('calendar', $viewData);
    ```

1. <span data-ttu-id="66e1b-122">刷新页面，应用现在应呈现一个事件表。</span><span class="sxs-lookup"><span data-stu-id="66e1b-122">Refresh the page and the app should now render a table of events.</span></span>

    ![事件表的屏幕截图](./images/add-msgraph-01.png)
