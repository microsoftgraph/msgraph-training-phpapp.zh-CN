<!-- markdownlint-disable MD002 MD041 -->

在此练习中，你将 Microsoft Graph 合并到应用程序中。 对于此应用程序，你将使用 [microsoft-graph](https://github.com/microsoftgraph/msgraph-sdk-php) 库调用 Microsoft Graph。

## <a name="get-calendar-events-from-outlook"></a>从 Outlook 获取日历事件

1. 在名为 **./app** 的目录中创建新目录，然后在名为该目录中创建新文件， `TimeZones` `TimeZones.php` 并添加以下代码。

    :::code language="php" source="../demo/graph-tutorial/app/TimeZones/TimeZones.php":::

    此类实现 Windows 时区名称到 IANA 时区标识符的简单映射。

1. 在名为 **./app/Http/Controllers** 目录中创建新文件 `CalendarController.php` ，并添加以下代码。

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

        $viewData['dateRange'] = $startOfWeek->format('M j, Y').' - '.$endOfWeek->format('M j, Y');

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

    考虑此代码正在做什么。

    - 将调用的 URL 为 `/v1.0/me/calendarView` 。
    - 和 `startDateTime` `endDateTime` 参数定义视图的起始和结束。
    - 该 `$select` 参数将每个事件返回的字段限制为仅视图将实际使用的字段。
    - 此参数按创建结果的日期和时间对结果进行排序，其中 `$orderby` 最新项是第一个。
    - 此 `$top` 参数将结果限制为 25 个事件。
    - 标头使响应中的开始时间和结束时间 `Prefer: outlook.timezone=""` 调整为用户的首选时区。

1. 更新 **./routes/web.php** 中的路由，以将路由添加到此新控制器。

    ```php
    Route::get('/calendar', 'CalendarController@calendar');
    ```

1. 登录并单击导航 **栏中** 的"日历"链接。 如果一切正常，应在用户日历上看到事件的 JSON 转储。

## <a name="display-the-results"></a>显示结果

现在，您可以添加视图以更用户友好的方式显示结果。

1. 在 **./resources/views** 目录中创建一个名为的新文件 `calendar.blade.php` ，并添加以下代码。

    :::code language="php" source="../demo/graph-tutorial/resources/views/calendar.blade.php" id="CalendarSnippet":::

    这将循环访问事件集合，并针对每个事件添加一个表格行。

1. 更新 **./routes/web.php** 中的路由以添加其路由 `/calendar/new` 。 你将在下一节中实现这些函数，但现在需要定义路由，因为 **calendar.blade.php** 会引用它。

    ```php
    Route::get('/calendar/new', 'CalendarController@getNewEventForm');
    Route::post('/calendar/new', 'CalendarController@createNewEvent');
    ```

1. 从 `return response()->json($events);` `calendar` **./app/Http/Controllers/CalendarController.php** 中的操作中删除该行，并将其替换为以下代码。

    ```php
    $viewData['events'] = $events;
    return view('calendar', $viewData);
    ```

1. 刷新页面，应用现在应呈现事件表。

    ![事件表的屏幕截图](./images/add-msgraph-01.png)
