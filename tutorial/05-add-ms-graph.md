<!-- markdownlint-disable MD002 MD041 -->

在本练习中, 将把 Microsoft Graph 合并到应用程序中。 对于此应用程序, 您将使用[microsoft graph](https://github.com/microsoftgraph/msgraph-sdk-php)库对 microsoft graph 进行调用。

## <a name="get-calendar-events-from-outlook"></a>从 Outlook 获取日历事件

让我们先为日历视图添加一个控制器。 在名为`./app/Http/Controllers` `CalendarController.php`的文件夹中创建一个新文件, 并添加以下代码。

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

考虑此代码执行的操作。

- 将调用的 URL 为`/v1.0/me/events`。
- `$select`参数将为每个事件返回的字段限制为仅显示视图实际使用的字段。
- `$orderby`参数按其创建日期和时间对结果进行排序, 最新项目最先开始。

更新中`./routes/web.php`的路由以将路由添加到此新控制器

```php
Route::get('/calendar', 'CalendarController@calendar');
```

现在, 您可以对此进行测试。 登录并单击导航栏中的 "**日历**" 链接。 如果一切正常, 应在用户的日历上看到一个 JSON 转储的事件。

## <a name="display-the-results"></a>显示结果

现在, 您可以添加一个视图, 以对用户更友好的方式显示结果。 在名为`./resources/views` `calendar.blade.php`的目录中创建一个新文件, 并添加以下代码。

```php
@extends('layout')

@section('content')
<h1>Calendar</h1>
<table class="table">
  <thead>
    <tr>
      <th scope="col">Organizer</th>
      <th scope="col">Subject</th>
      <th scope="col">Start</th>
      <th scope="col">End</th>
    </tr>
  </thead>
  <tbody>
    @isset($events)
      @foreach($events as $event)
        <tr>
          <td>{{ $event->getOrganizer()->getEmailAddress()->getName() }}</td>
          <td>{{ $event->getSubject() }}</td>
          <td>{{ \Carbon\Carbon::parse($event->getStart()->getDateTime())->format('n/j/y g:i A') }}</td>
          <td>{{ \Carbon\Carbon::parse($event->getEnd()->getDateTime())->format('n/j/y g:i A') }}</td>
        </tr>
      @endforeach
    @endif
  </tbody>
</table>
@endsection
```

这将遍历一组事件并为每个事件添加一个表行。 从中`return response()->json($events);`的`calendar`操作中`./app/Http/Controllers/CalendarController.php`删除该行, 并将其替换为以下代码。

```php
$viewData['events'] = $events;
return view('calendar', $viewData);
```

刷新页面, 应用现在应呈现一个事件表。

![事件表的屏幕截图](./images/add-msgraph-01.png)
