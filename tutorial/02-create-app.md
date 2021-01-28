<!-- markdownlint-disable MD002 MD041 -->

首先创建一个新的 Laravel 项目。

1. 打开命令行界面 (CLI) ，导航到您具有创建文件权限的目录，然后运行以下命令以创建新的 PHP 应用。

    ```Shell
    laravel new graph-tutorial
    ```

1. 导航到 **图形教程** 目录并输入以下命令以启动本地 Web 服务器。

    ```Shell
    php artisan serve
    ```

1. 打开浏览器，并导航到 `http://localhost:8000`。 如果一切正常，你将看到默认的 Laravel 页面。 如果看不到该页面，请查看 [Laravel 文档](https://laravel.com/docs/8.x)。

## <a name="install-packages"></a>安装程序包

在继续之前，请安装一些稍后将使用的附加程序包：

- 用于处理登录和 OAuth 令牌流的[oauth2](https://github.com/thephpleague/oauth2-client)客户端。
- [用于调用](https://github.com/microsoftgraph/msgraph-sdk-php) Microsoft Graph 的 microsoft graph。

1. 在 CLI 中运行以下命令。

    ```Shell
    composer require league/oauth2-client microsoft/microsoft-graph
    ```

## <a name="design-the-app"></a>设计应用

1. 在 **./resources/views** 目录中创建一个名为的新文件 `layout.blade.php` ，并添加以下代码。

    :::code language="php" source="../demo/graph-tutorial/resources/views/layout.blade.php" id="LayoutSnippet":::

    此代码为 [简单样式设置添加 Bootstrap，](http://getbootstrap.com/) 为一些简单图标添加 ["字体](https://fontawesome.com/) 出色"。 它还定义具有导航栏的全局布局。

1. 在名为的目录中创建新目录，然后在名为 "的目录中 `./public` `css` `./public/css` 新建文件 `app.css` "。 添加以下代码。

    :::code language="css" source="../demo/graph-tutorial/public/css/app.css":::

1. 打开 `./resources/views/welcome.blade.php` 该文件，并将其内容替换为以下内容。

    :::code language="php" source="../demo/graph-tutorial/resources/views/welcome.blade.php" id="WelcomeSnippet":::

1. 通过将以下函数 `Controller` 添加到类来更新 **./app/Http/Controllers/Controller.php** 中的基类。

    :::code language="php" source="../demo/graph-tutorial/app/Http/Controllers/Controller.php" id="LoadViewDataSnippet":::

1. 在名为的目录中创建新文件 `./app/Http/Controllers` 并 `HomeController.php` 添加以下代码。

    :::code language="php" source="../demo/graph-tutorial/app/Http/Controllers/HomeController.php":::

1. 更新路由以 `./routes/web.php` 使用新控制器。 将此文件的全部内容替换为以下内容。

    ```php
    <?php

    use Illuminate\Support\Facades\Route;

    Route::get('/', 'HomeController@welcome');
    ```

1. 打开 **./app/Providers/RouteServiceProvider.php** 并取消对声明 `$namespace` 的注释。

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

1. 保存所有更改并重新启动服务器。 现在，应用看起来应该非常不同。

    ![重新设计主页的屏幕截图](./images/create-app-01.png)
