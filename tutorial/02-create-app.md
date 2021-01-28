<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="5af74-101">首先创建一个新的 Laravel 项目。</span><span class="sxs-lookup"><span data-stu-id="5af74-101">Begin by creating a new Laravel project.</span></span>

1. <span data-ttu-id="5af74-102">打开命令行界面 (CLI) ，导航到您具有创建文件权限的目录，然后运行以下命令以创建新的 PHP 应用。</span><span class="sxs-lookup"><span data-stu-id="5af74-102">Open your command-line interface (CLI), navigate to a directory where you have rights to create files, and run the following command to create a new PHP app.</span></span>

    ```Shell
    laravel new graph-tutorial
    ```

1. <span data-ttu-id="5af74-103">导航到 **图形教程** 目录并输入以下命令以启动本地 Web 服务器。</span><span class="sxs-lookup"><span data-stu-id="5af74-103">Navigate to the **graph-tutorial** directory and enter the following command to start a local web server.</span></span>

    ```Shell
    php artisan serve
    ```

1. <span data-ttu-id="5af74-104">打开浏览器，并导航到 `http://localhost:8000`。</span><span class="sxs-lookup"><span data-stu-id="5af74-104">Open your browser and navigate to `http://localhost:8000`.</span></span> <span data-ttu-id="5af74-105">如果一切正常，你将看到默认的 Laravel 页面。</span><span class="sxs-lookup"><span data-stu-id="5af74-105">If everything is working, you will see a default Laravel page.</span></span> <span data-ttu-id="5af74-106">如果看不到该页面，请查看 [Laravel 文档](https://laravel.com/docs/8.x)。</span><span class="sxs-lookup"><span data-stu-id="5af74-106">If you don't see that page, check the [Laravel docs](https://laravel.com/docs/8.x).</span></span>

## <a name="install-packages"></a><span data-ttu-id="5af74-107">安装程序包</span><span class="sxs-lookup"><span data-stu-id="5af74-107">Install packages</span></span>

<span data-ttu-id="5af74-108">在继续之前，请安装一些稍后将使用的附加程序包：</span><span class="sxs-lookup"><span data-stu-id="5af74-108">Before moving on, install some additional packages that you will use later:</span></span>

- <span data-ttu-id="5af74-109">用于处理登录和 OAuth 令牌流的[oauth2](https://github.com/thephpleague/oauth2-client)客户端。</span><span class="sxs-lookup"><span data-stu-id="5af74-109">[oauth2-client](https://github.com/thephpleague/oauth2-client) for handling sign-in and OAuth token flows.</span></span>
- <span data-ttu-id="5af74-110">[用于调用](https://github.com/microsoftgraph/msgraph-sdk-php) Microsoft Graph 的 microsoft graph。</span><span class="sxs-lookup"><span data-stu-id="5af74-110">[microsoft-graph](https://github.com/microsoftgraph/msgraph-sdk-php) for making calls to Microsoft Graph.</span></span>

1. <span data-ttu-id="5af74-111">在 CLI 中运行以下命令。</span><span class="sxs-lookup"><span data-stu-id="5af74-111">Run the following command in your CLI.</span></span>

    ```Shell
    composer require league/oauth2-client microsoft/microsoft-graph
    ```

## <a name="design-the-app"></a><span data-ttu-id="5af74-112">设计应用</span><span class="sxs-lookup"><span data-stu-id="5af74-112">Design the app</span></span>

1. <span data-ttu-id="5af74-113">在 **./resources/views** 目录中创建一个名为的新文件 `layout.blade.php` ，并添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="5af74-113">Create a new file in the **./resources/views** directory named `layout.blade.php` and add the following code.</span></span>

    :::code language="php" source="../demo/graph-tutorial/resources/views/layout.blade.php" id="LayoutSnippet":::

    <span data-ttu-id="5af74-114">此代码为 [简单样式设置添加 Bootstrap，](http://getbootstrap.com/) 为一些简单图标添加 ["字体](https://fontawesome.com/) 出色"。</span><span class="sxs-lookup"><span data-stu-id="5af74-114">This code adds [Bootstrap](http://getbootstrap.com/) for simple styling, and [Font Awesome](https://fontawesome.com/) for some simple icons.</span></span> <span data-ttu-id="5af74-115">它还定义具有导航栏的全局布局。</span><span class="sxs-lookup"><span data-stu-id="5af74-115">It also defines a global layout with a nav bar.</span></span>

1. <span data-ttu-id="5af74-116">在名为的目录中创建新目录，然后在名为 "的目录中 `./public` `css` `./public/css` 新建文件 `app.css` "。</span><span class="sxs-lookup"><span data-stu-id="5af74-116">Create a new directory in the `./public` directory named `css`, then create a new file in the `./public/css` directory named `app.css`.</span></span> <span data-ttu-id="5af74-117">添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="5af74-117">Add the following code.</span></span>

    :::code language="css" source="../demo/graph-tutorial/public/css/app.css":::

1. <span data-ttu-id="5af74-118">打开 `./resources/views/welcome.blade.php` 该文件，并将其内容替换为以下内容。</span><span class="sxs-lookup"><span data-stu-id="5af74-118">Open the `./resources/views/welcome.blade.php` file and replace its contents with the following.</span></span>

    :::code language="php" source="../demo/graph-tutorial/resources/views/welcome.blade.php" id="WelcomeSnippet":::

1. <span data-ttu-id="5af74-119">通过将以下函数 `Controller` 添加到类来更新 **./app/Http/Controllers/Controller.php** 中的基类。</span><span class="sxs-lookup"><span data-stu-id="5af74-119">Update the base `Controller` class in **./app/Http/Controllers/Controller.php** by adding the following function to the class.</span></span>

    :::code language="php" source="../demo/graph-tutorial/app/Http/Controllers/Controller.php" id="LoadViewDataSnippet":::

1. <span data-ttu-id="5af74-120">在名为的目录中创建新文件 `./app/Http/Controllers` 并 `HomeController.php` 添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="5af74-120">Create a new file in the `./app/Http/Controllers` directory named `HomeController.php` and add the following code.</span></span>

    :::code language="php" source="../demo/graph-tutorial/app/Http/Controllers/HomeController.php":::

1. <span data-ttu-id="5af74-121">更新路由以 `./routes/web.php` 使用新控制器。</span><span class="sxs-lookup"><span data-stu-id="5af74-121">Update the route in `./routes/web.php` to use the new controller.</span></span> <span data-ttu-id="5af74-122">将此文件的全部内容替换为以下内容。</span><span class="sxs-lookup"><span data-stu-id="5af74-122">Replace the entire contents of this file with the following.</span></span>

    ```php
    <?php

    use Illuminate\Support\Facades\Route;

    Route::get('/', 'HomeController@welcome');
    ```

1. <span data-ttu-id="5af74-123">打开 **./app/Providers/RouteServiceProvider.php** 并取消对声明 `$namespace` 的注释。</span><span class="sxs-lookup"><span data-stu-id="5af74-123">Open **./app/Providers/RouteServiceProvider.php** and uncomment the `$namespace` declaration.</span></span>

    ```php
    /**
     * This namespace is applied to your controller routes.
     *
     * In addition, it is set as the URL generator's root namespace.
     *
     * @var string
     */
    protected $namespace = 'App\Http\Controllers';
    ```

1. <span data-ttu-id="5af74-124">保存所有更改并重新启动服务器。</span><span class="sxs-lookup"><span data-stu-id="5af74-124">Save all of your changes and restart the server.</span></span> <span data-ttu-id="5af74-125">现在，应用看起来应该非常不同。</span><span class="sxs-lookup"><span data-stu-id="5af74-125">Now, the app should look very different.</span></span>

    ![重新设计主页的屏幕截图](./images/create-app-01.png)
