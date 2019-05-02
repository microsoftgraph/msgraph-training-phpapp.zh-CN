<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="c98fb-101">打开命令行接口 (CLI), 导航到您有权创建文件的目录, 并运行以下命令以创建新的 PHP 应用程序。</span><span class="sxs-lookup"><span data-stu-id="c98fb-101">Open your command-line interface (CLI), navigate to a directory where you have rights to create files, and run the following command to create a new PHP app.</span></span>

```Shell
laravel new graph-tutorial
```

<span data-ttu-id="c98fb-102">Laravel 创建一个名`graph-tutorial`为和搭建基架 PHP 应用程序的新目录。</span><span class="sxs-lookup"><span data-stu-id="c98fb-102">Laravel creates a new directory called `graph-tutorial` and scaffolds a PHP app.</span></span> <span data-ttu-id="c98fb-103">导航到此新目录, 然后输入以下命令以启动本地 web 服务器。</span><span class="sxs-lookup"><span data-stu-id="c98fb-103">Navigate to this new directory and enter the following command to start a local web server.</span></span>

```Shell
php artisan serve
```

<span data-ttu-id="c98fb-104">打开浏览器，并导航到 `http://localhost:8000`。</span><span class="sxs-lookup"><span data-stu-id="c98fb-104">Open your browser and navigate to `http://localhost:8000`.</span></span> <span data-ttu-id="c98fb-105">如果一切正常, 您将看到一个默认的 Laravel 页面。</span><span class="sxs-lookup"><span data-stu-id="c98fb-105">If everything is working, you will see a default Laravel page.</span></span> <span data-ttu-id="c98fb-106">如果看不到该页面, 请查看[Laravel 文档](https://laravel.com/docs/5.6)。</span><span class="sxs-lookup"><span data-stu-id="c98fb-106">If you don't see that page, check the [Laravel docs](https://laravel.com/docs/5.6).</span></span>

<span data-ttu-id="c98fb-107">在继续操作之前, 请先安装其他一些库, 稍后将使用这些库:</span><span class="sxs-lookup"><span data-stu-id="c98fb-107">Before moving on, install some additional libraries that you will use later:</span></span>

- <span data-ttu-id="c98fb-108">[oauth2-](https://github.com/thephpleague/oauth2-client)处理登录和 OAuth 令牌流的客户端。</span><span class="sxs-lookup"><span data-stu-id="c98fb-108">[oauth2-client](https://github.com/thephpleague/oauth2-client) for handling sign-in and OAuth token flows.</span></span>
- <span data-ttu-id="c98fb-109">[Microsoft](https://github.com/microsoftgraph/msgraph-sdk-php) graph, 用于调用 microsoft graph。</span><span class="sxs-lookup"><span data-stu-id="c98fb-109">[microsoft-graph](https://github.com/microsoftgraph/msgraph-sdk-php) for making calls to Microsoft Graph.</span></span>

<span data-ttu-id="c98fb-110">在 CLI 中运行以下命令。</span><span class="sxs-lookup"><span data-stu-id="c98fb-110">Run the following command in your CLI.</span></span>

```Shell
composer require league/oauth2-client:dev-master microsoft/microsoft-graph
```

## <a name="design-the-app"></a><span data-ttu-id="c98fb-111">设计应用程序</span><span class="sxs-lookup"><span data-stu-id="c98fb-111">Design the app</span></span>

<span data-ttu-id="c98fb-112">首先, 创建应用的全局布局。</span><span class="sxs-lookup"><span data-stu-id="c98fb-112">Start by creating the global layout for the app.</span></span> <span data-ttu-id="c98fb-113">在名为`./resources/views` `layout.blade.php`的目录中创建一个新文件, 并添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="c98fb-113">Create a new file in the  `./resources/views` directory named `layout.blade.php` and add the following code.</span></span>

```php
<!DOCTYPE html>
<html>
  <head>
    <title>PHP Graph Tutorial</title>

    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.1.1/css/bootstrap.min.css"
        integrity="sha384-WskhaSGFgHYWDcbwN70/dfYBj47jz9qbsMId/iRN3ewGhXQFZCSftd1LZCfmhktB" crossorigin="anonymous">
    <link rel="stylesheet" href="https://use.fontawesome.com/releases/v5.1.0/css/all.css"
        integrity="sha384-lKuwvrZot6UHsBSfcMvOkWwlCMgc0TaWr+30HWe3a4ltaBwTZhyTEggF5tJv8tbt" crossorigin="anonymous">
    <link rel="stylesheet" href="{{ asset('/css/app.css') }}">
    <script src="https://code.jquery.com/jquery-3.3.1.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.14.3/umd/popper.min.js"
        integrity="sha384-ZMP7rVo3mIykV+2+9J3UJ46jBk0WLaUAdn689aCwoqbBJiSnjAK/l8WvCWPIPm49" crossorigin="anonymous"></script>
    <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.1.1/js/bootstrap.min.js"
        integrity="sha384-smHYKdLADwkXOn1EmN1qk/HfnUcbVRZyYmZ4qpPea6sjB/pTJ0euyQp0Mk8ck+5T" crossorigin="anonymous"></script>
  </head>

  <body>
    <nav class="navbar navbar-expand-md navbar-dark fixed-top bg-dark">
      <div class="container">
        <a href="/" class="navbar-brand">PHP Graph Tutorial</a>
        <button class="navbar-toggler" type="button" data-toggle="collapse" data-target="#navbarCollapse"
            aria-controls="navbarCollapse" aria-expanded="false" aria-label="Toggle navigation">
          <span class="navbar-toggler-icon"></span>
        </button>
        <div class="collapse navbar-collapse" id="navbarCollapse">
          <ul class="navbar-nav mr-auto">
            <li class="nav-item">
              <a href="/" class="nav-link {{$_SERVER['REQUEST_URI'] == '/' ? ' active' : ''}}">Home</a>
            </li>
            @if(isset($userName))
              <li class="nav-item" data-turbolinks="false">
                <a href="/calendar" class="nav-link{{$_SERVER['REQUEST_URI'] == '/calendar' ? ' active' : ''}}">Calendar</a>
              </li>
            @endif
          </ul>
          <ul class="navbar-nav justify-content-end">
            <li class="nav-item">
              <a class="nav-link" href="https://developer.microsoft.com/graph/docs/concepts/overview" target="_blank">
                <i class="fas fa-external-link-alt mr-1"></i>Docs
              </a>
            </li>
            @if(isset($userName))
              <li class="nav-item dropdown">
                <a class="nav-link dropdown-toggle" data-toggle="dropdown" href="#" role="button"
                  aria-haspopup="true" aria-expanded="false">
                  @if(isset($user_avatar))
                    <img src="{{ $user_avatar }}" class="rounded-circle align-self-center mr-2" style="width: 32px;">
                  @else
                    <i class="far fa-user-circle fa-lg rounded-circle align-self-center mr-2" style="width: 32px;"></i>
                  @endif
                </a>
                <div class="dropdown-menu dropdown-menu-right">
                  <h5 class="dropdown-item-text mb-0">{{ $userName }}</h5>
                  <p class="dropdown-item-text text-muted mb-0">{{ $userEmail }}</p>
                  <div class="dropdown-divider"></div>
                  <a href="/signout" class="dropdown-item">Sign Out</a>
                </div>
              </li>
            @else
              <li class="nav-item">
                <a href="/signin" class="nav-link">Sign In</a>
              </li>
            @endif
          </ul>
        </div>
      </div>
    </nav>
    <main role="main" class="container">
      @if(session('error'))
        <div class="alert alert-danger" role="alert">
          <p class="mb-3">{{ session('error') }}</p>
          @if(session('errorDetail'))
            <pre class="alert-pre border bg-light p-2"><code>{{ session('errorDetail') }}</code></pre>
          @endif
        </div>
      @endif

      @yield('content')
    </main>
  </body>
</html>
```

<span data-ttu-id="c98fb-114">此代码添加简单样式的[引导](http://getbootstrap.com/), 并添加一些简单图标的[字体](https://fontawesome.com/)。</span><span class="sxs-lookup"><span data-stu-id="c98fb-114">This code adds [Bootstrap](http://getbootstrap.com/) for simple styling, and [Font Awesome](https://fontawesome.com/) for some simple icons.</span></span> <span data-ttu-id="c98fb-115">它还定义具有导航栏的全局布局。</span><span class="sxs-lookup"><span data-stu-id="c98fb-115">It also defines a global layout with a nav bar.</span></span>

<span data-ttu-id="c98fb-116">现在打开`./public/css/app.css`并将其全部内容替换为以下内容。</span><span class="sxs-lookup"><span data-stu-id="c98fb-116">Now open `./public/css/app.css` and replace its entire contents with the following.</span></span>

```css
body {
  padding-top: 4.5rem;
}

.alert-pre {
  word-wrap: break-word;
  word-break: break-all;
  white-space: pre-wrap;
}
```

<span data-ttu-id="c98fb-117">现在更新默认页面。</span><span class="sxs-lookup"><span data-stu-id="c98fb-117">Now update the default page.</span></span> <span data-ttu-id="c98fb-118">打开`./resources/views/welcome.blade.php`文件, 并将其内容替换为以下内容。</span><span class="sxs-lookup"><span data-stu-id="c98fb-118">Open the `./resources/views/welcome.blade.php` file and replace its contents with the following.</span></span>

```php
@extends('layout')

@section('content')
<div class="jumbotron">
  <h1>PHP Graph Tutorial</h1>
  <p class="lead">This sample app shows how to use the Microsoft Graph API to access Outlook and OneDrive data from PHP</p>
  @if(isset($userName))
    <h4>Welcome {{ $userName }}!</h4>
    <p>Use the navigation bar at the top of the page to get started.</p>
  @else
    <a href="/signin" class="btn btn-primary btn-large">Click here to sign in</a>
  @endif
</div>
@endsection
```

<span data-ttu-id="c98fb-119">通过将以下`Controller`函数添加`./app/Http/Controllers/Controller.php`到类中, 在中更新基类。</span><span class="sxs-lookup"><span data-stu-id="c98fb-119">Update the base `Controller` class in `./app/Http/Controllers/Controller.php` by adding the following function to the class.</span></span>

```php
public function loadViewData()
{
  $viewData = [];

  // Check for flash errors
  if (session('error')) {
    $viewData['error'] = session('error');
    $viewData['errorDetail'] = session('errorDetail');
  }

  // Check for logged on user
  if (session('userName'))
  {
    $viewData['userName'] = session('userName');
    $viewData['userEmail'] = session('userEmail');
  }

  return $viewData;
}
```

<span data-ttu-id="c98fb-120">接下来, 为主页添加一个控制器。</span><span class="sxs-lookup"><span data-stu-id="c98fb-120">Next, add a controller for the home page.</span></span> <span data-ttu-id="c98fb-121">在名为`./app/Http/Controllers` `HomeController.php`的目录中创建一个新文件, 并添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="c98fb-121">Create a new file in the `./app/Http/Controllers` directory named `HomeController.php` and add the following code.</span></span>

```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use Illuminate\Http\Request;

class HomeController extends Controller
{
  public function welcome()
  {
    $viewData = $this->loadViewData();

    return view('welcome', $viewData);
  }
}
```

<span data-ttu-id="c98fb-122">最后, 更新中`./routes/web.php`的路由以使用新的控制器。</span><span class="sxs-lookup"><span data-stu-id="c98fb-122">Finally, update the route in `./routes/web.php` to use the new controller.</span></span> <span data-ttu-id="c98fb-123">将此文件的全部内容替换为以下内容。</span><span class="sxs-lookup"><span data-stu-id="c98fb-123">Replace the entire contents of this file with the following.</span></span>

```php
<?php

Route::get('/', 'HomeController@welcome');
```

<span data-ttu-id="c98fb-124">保存所有更改, 然后重新启动服务器。</span><span class="sxs-lookup"><span data-stu-id="c98fb-124">Save all of your changes and restart the server.</span></span> <span data-ttu-id="c98fb-125">现在, 应用程序看起来应非常不同。</span><span class="sxs-lookup"><span data-stu-id="c98fb-125">Now, the app should look very different.</span></span>

![重新设计的主页的屏幕截图](./images/create-app-01.png)