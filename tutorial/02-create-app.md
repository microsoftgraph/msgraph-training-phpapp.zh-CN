<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="fe1d7-101">首先创建一个新的 Laravel 项目。</span><span class="sxs-lookup"><span data-stu-id="fe1d7-101">Begin by creating a new Laravel project.</span></span>

1. <span data-ttu-id="fe1d7-102">打开命令行界面 (CLI) ，导航到您有权在其中创建文件的目录，然后运行以下命令以创建新的 PHP 应用程序。</span><span class="sxs-lookup"><span data-stu-id="fe1d7-102">Open your command-line interface (CLI), navigate to a directory where you have rights to create files, and run the following command to create a new PHP app.</span></span>

    ```Shell
    laravel new graph-tutorial
    ```

1. <span data-ttu-id="fe1d7-103">导航到 **graph 教程** 目录并输入以下命令，以启动本地 web 服务器。</span><span class="sxs-lookup"><span data-stu-id="fe1d7-103">Navigate to the **graph-tutorial** directory and enter the following command to start a local web server.</span></span>

    ```Shell
    php artisan serve
    ```

1. <span data-ttu-id="fe1d7-104">打开浏览器，并导航到 `http://localhost:8000`。</span><span class="sxs-lookup"><span data-stu-id="fe1d7-104">Open your browser and navigate to `http://localhost:8000`.</span></span> <span data-ttu-id="fe1d7-105">如果一切正常，您将看到一个默认的 Laravel 页面。</span><span class="sxs-lookup"><span data-stu-id="fe1d7-105">If everything is working, you will see a default Laravel page.</span></span> <span data-ttu-id="fe1d7-106">如果看不到该页面，请查看 [Laravel 文档](https://laravel.com/docs/7.x)。</span><span class="sxs-lookup"><span data-stu-id="fe1d7-106">If you don't see that page, check the [Laravel docs](https://laravel.com/docs/7.x).</span></span>

## <a name="install-packages"></a><span data-ttu-id="fe1d7-107">安装程序包</span><span class="sxs-lookup"><span data-stu-id="fe1d7-107">Install packages</span></span>

<span data-ttu-id="fe1d7-108">在继续操作之前，请先安装将使用的其他一些程序包：</span><span class="sxs-lookup"><span data-stu-id="fe1d7-108">Before moving on, install some additional packages that you will use later:</span></span>

- <span data-ttu-id="fe1d7-109">[oauth2-](https://github.com/thephpleague/oauth2-client) 处理登录和 OAuth 令牌流的客户端。</span><span class="sxs-lookup"><span data-stu-id="fe1d7-109">[oauth2-client](https://github.com/thephpleague/oauth2-client) for handling sign-in and OAuth token flows.</span></span>
- <span data-ttu-id="fe1d7-110">[Microsoft](https://github.com/microsoftgraph/msgraph-sdk-php) graph，用于调用 microsoft graph。</span><span class="sxs-lookup"><span data-stu-id="fe1d7-110">[microsoft-graph](https://github.com/microsoftgraph/msgraph-sdk-php) for making calls to Microsoft Graph.</span></span>

1. <span data-ttu-id="fe1d7-111">运行以下命令以删除的现有版本 `guzzlehttp/guzzle` 。</span><span class="sxs-lookup"><span data-stu-id="fe1d7-111">Run the following command to remove the existing version of `guzzlehttp/guzzle`.</span></span> <span data-ttu-id="fe1d7-112">Laravel 安装的版本与 Microsoft Graph PHP SDK 所需的版本冲突。</span><span class="sxs-lookup"><span data-stu-id="fe1d7-112">The version installed by Laravel conflicts with the version required by the Microsoft Graph PHP SDK.</span></span>

    ```Shell
    composer remove guzzlehttp/guzzle
    ```

1. <span data-ttu-id="fe1d7-113">在 CLI 中运行以下命令。</span><span class="sxs-lookup"><span data-stu-id="fe1d7-113">Run the following command in your CLI.</span></span>

    ```Shell
    composer require league/oauth2-client microsoft/microsoft-graph
    ```

## <a name="design-the-app"></a><span data-ttu-id="fe1d7-114">设计应用程序</span><span class="sxs-lookup"><span data-stu-id="fe1d7-114">Design the app</span></span>

1. <span data-ttu-id="fe1d7-115">在 **/resources/views** 目录中创建一个名为的新文件 `layout.blade.php` ，并添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="fe1d7-115">Create a new file in the **./resources/views** directory named `layout.blade.php` and add the following code.</span></span>

    :::code language="php" source="../demo/graph-tutorial/resources/views/layout.blade.php" id="LayoutSnippet":::

    <span data-ttu-id="fe1d7-116">此代码添加简单样式的 [引导](http://getbootstrap.com/) ，并添加一些简单图标的 [字体](https://fontawesome.com/) 。</span><span class="sxs-lookup"><span data-stu-id="fe1d7-116">This code adds [Bootstrap](http://getbootstrap.com/) for simple styling, and [Font Awesome](https://fontawesome.com/) for some simple icons.</span></span> <span data-ttu-id="fe1d7-117">它还定义具有导航栏的全局布局。</span><span class="sxs-lookup"><span data-stu-id="fe1d7-117">It also defines a global layout with a nav bar.</span></span>

1. <span data-ttu-id="fe1d7-118">在名为的目录中创建一个新目录 `./public` `css` ，然后在名为的目录中创建一个新文件 `./public/css` `app.css` 。</span><span class="sxs-lookup"><span data-stu-id="fe1d7-118">Create a new directory in the `./public` directory named `css`, then create a new file in the `./public/css` directory named `app.css`.</span></span> <span data-ttu-id="fe1d7-119">添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="fe1d7-119">Add the following code.</span></span>

    :::code language="css" source="../demo/graph-tutorial/public/css/app.css":::

1. <span data-ttu-id="fe1d7-120">打开 `./resources/views/welcome.blade.php` 文件，并将其内容替换为以下内容。</span><span class="sxs-lookup"><span data-stu-id="fe1d7-120">Open the `./resources/views/welcome.blade.php` file and replace its contents with the following.</span></span>

    :::code language="php" source="../demo/graph-tutorial/resources/views/welcome.blade.php" id="WelcomeSnippet":::

1. <span data-ttu-id="fe1d7-121">`Controller`通过将以下函数添加到类来更新 **/app/Http/Controllers/Controller.php**中的基类。</span><span class="sxs-lookup"><span data-stu-id="fe1d7-121">Update the base `Controller` class in **./app/Http/Controllers/Controller.php** by adding the following function to the class.</span></span>

    :::code language="php" source="../demo/graph-tutorial/app/Http/Controllers/Controller.php" id="LoadViewDataSnippet":::

1. <span data-ttu-id="fe1d7-122">在名为的目录中创建一个新文件 `./app/Http/Controllers` `HomeController.php` ，并添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="fe1d7-122">Create a new file in the `./app/Http/Controllers` directory named `HomeController.php` and add the following code.</span></span>

    :::code language="php" source="../demo/graph-tutorial/app/Http/Controllers/HomeController.php":::

1. <span data-ttu-id="fe1d7-123">更新中的路由 `./routes/web.php` 以使用新的控制器。</span><span class="sxs-lookup"><span data-stu-id="fe1d7-123">Update the route in `./routes/web.php` to use the new controller.</span></span> <span data-ttu-id="fe1d7-124">将此文件的全部内容替换为以下内容。</span><span class="sxs-lookup"><span data-stu-id="fe1d7-124">Replace the entire contents of this file with the following.</span></span>

    ```php
    <?php

    use Illuminate\Support\Facades\Route;

    Route::get('/', 'HomeController@welcome');
    ```

1. <span data-ttu-id="fe1d7-125">保存所有更改，然后重新启动服务器。</span><span class="sxs-lookup"><span data-stu-id="fe1d7-125">Save all of your changes and restart the server.</span></span> <span data-ttu-id="fe1d7-126">现在，应用程序看起来应非常不同。</span><span class="sxs-lookup"><span data-stu-id="fe1d7-126">Now, the app should look very different.</span></span>

    ![重新设计的主页的屏幕截图](./images/create-app-01.png)
