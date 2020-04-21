<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="f3edc-101">在本练习中，将把 Microsoft Graph 合并到应用程序中。</span><span class="sxs-lookup"><span data-stu-id="f3edc-101">In this exercise you will incorporate the Microsoft Graph into the application.</span></span> <span data-ttu-id="f3edc-102">对于此应用程序，您将使用[microsoft graph](https://github.com/microsoftgraph/msgraph-sdk-php)库对 microsoft graph 进行调用。</span><span class="sxs-lookup"><span data-stu-id="f3edc-102">For this application, you will use the [microsoft-graph](https://github.com/microsoftgraph/msgraph-sdk-php) library to make calls to Microsoft Graph.</span></span>

## <a name="get-calendar-events-from-outlook"></a><span data-ttu-id="f3edc-103">从 Outlook 获取日历事件</span><span class="sxs-lookup"><span data-stu-id="f3edc-103">Get calendar events from Outlook</span></span>

1. <span data-ttu-id="f3edc-104">在 " **/app/Http/Controllers** " 目录中创建一个名为`CalendarController.php`的新文件，并添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="f3edc-104">Create a new file in the **./app/Http/Controllers** directory named `CalendarController.php`, and add the following code.</span></span>

    ```php
    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use Illuminate\Http\Request;
    use Microsoft\Graph\Graph;
    use Microsoft\Graph\Model;
    use App\TokenStore\TokenCache;

    class CalendarController extends Controller
    {
      public function calendar()
      {
        $viewData = $this->loadViewData();

        // Get the access token from the cache
        $tokenCache = new TokenCache();
        $accessToken = $tokenCache->getAccessToken();

        // Create a Graph client
        $graph = new Graph();
        $graph->setAccessToken($accessToken);

        $queryParams = array(
          '$select' => 'subject,organizer,start,end',
          '$orderby' => 'createdDateTime DESC'
        );

        // Append query parameters to the '/me/events' url
        $getEventsUrl = '/me/events?'.http_build_query($queryParams);

        $events = $graph->createRequest('GET', $getEventsUrl)
          ->setReturnType(Model\Event::class)
          ->execute();

        return response()->json($events);
      }
    }
    ```

    <span data-ttu-id="f3edc-105">考虑此代码执行的操作。</span><span class="sxs-lookup"><span data-stu-id="f3edc-105">Consider what this code is doing.</span></span>

    - <span data-ttu-id="f3edc-106">将调用的 URL 为`/v1.0/me/events`。</span><span class="sxs-lookup"><span data-stu-id="f3edc-106">The URL that will be called is `/v1.0/me/events`.</span></span>
    - <span data-ttu-id="f3edc-107">`$select`参数将为每个事件返回的字段限制为仅显示视图实际使用的字段。</span><span class="sxs-lookup"><span data-stu-id="f3edc-107">The `$select` parameter limits the fields returned for each events to just those the view will actually use.</span></span>
    - <span data-ttu-id="f3edc-108">`$orderby`参数按其创建日期和时间对结果进行排序，最新项目最先开始。</span><span class="sxs-lookup"><span data-stu-id="f3edc-108">The `$orderby` parameter sorts the results by the date and time they were created, with the most recent item being first.</span></span>

1. <span data-ttu-id="f3edc-109">更新 **/routes/web.php**中的路由，以将路由添加到此新控制器。</span><span class="sxs-lookup"><span data-stu-id="f3edc-109">Update the routes in **./routes/web.php** to add a route to this new controller.</span></span>

    ```php
    Route::get('/calendar', 'CalendarController@calendar');
    ```

1. <span data-ttu-id="f3edc-110">登录并单击导航栏中的 "**日历**" 链接。</span><span class="sxs-lookup"><span data-stu-id="f3edc-110">Sign in and click the **Calendar** link in the nav bar.</span></span> <span data-ttu-id="f3edc-111">如果一切正常，应在用户的日历上看到一个 JSON 转储的事件。</span><span class="sxs-lookup"><span data-stu-id="f3edc-111">If everything works, you should see a JSON dump of events on the user's calendar.</span></span>

## <a name="display-the-results"></a><span data-ttu-id="f3edc-112">显示结果</span><span class="sxs-lookup"><span data-stu-id="f3edc-112">Display the results</span></span>

<span data-ttu-id="f3edc-113">现在，您可以添加一个视图，以对用户更友好的方式显示结果。</span><span class="sxs-lookup"><span data-stu-id="f3edc-113">Now you can add a view to display the results in a more user-friendly manner.</span></span>

1. <span data-ttu-id="f3edc-114">在 **/resources/views**目录中创建一个名为`calendar.blade.php`的新文件，并添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="f3edc-114">Create a new file in the **./resources/views** directory named `calendar.blade.php` and add the following code.</span></span>

    :::code language="php" source="../demo/graph-tutorial/resources/views/calendar.blade.php" id="CalendarSnippet":::

    <span data-ttu-id="f3edc-115">这将遍历一组事件并为每个事件添加一个表行。</span><span class="sxs-lookup"><span data-stu-id="f3edc-115">That will loop through a collection of events and add a table row for each one.</span></span>

1. <span data-ttu-id="f3edc-116">从/App/Http/Controllers/CalendarController.php `return response()->json($events);`中删除`calendar`操作的行 **./app/Http/Controllers/CalendarController.php**，并将其替换为以下代码。</span><span class="sxs-lookup"><span data-stu-id="f3edc-116">Remove the `return response()->json($events);` line from the `calendar` action in **./app/Http/Controllers/CalendarController.php**, and replace it with the following code.</span></span>

    ```php
    $viewData['events'] = $events;
    return view('calendar', $viewData);
    ```

1. <span data-ttu-id="f3edc-117">刷新页面，应用现在应呈现一个事件表。</span><span class="sxs-lookup"><span data-stu-id="f3edc-117">Refresh the page and the app should now render a table of events.</span></span>

    ![事件表的屏幕截图](./images/add-msgraph-01.png)
