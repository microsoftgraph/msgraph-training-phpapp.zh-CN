<!-- markdownlint-disable MD002 MD041 -->

首先创建一个新的 Laravel 项目。

1. 打开命令行接口（CLI），导航到您有权创建文件的目录，并运行以下命令以创建新的 PHP 应用程序。

    ```Shell
    laravel new graph-tutorial
    ```

1. 导航到**graph 教程**目录并输入以下命令，以启动本地 web 服务器。

    ```Shell
    php artisan serve
    ```

1. 打开浏览器，并导航到 `http://localhost:8000`。 如果一切正常，您将看到一个默认的 Laravel 页面。 如果看不到该页面，请查看[Laravel 文档](https://laravel.com/docs/7.x)。

## <a name="install-packages"></a>安装程序包

在继续操作之前，请先安装将使用的其他一些程序包：

- [oauth2-](https://github.com/thephpleague/oauth2-client)处理登录和 OAuth 令牌流的客户端。
- [Microsoft](https://github.com/microsoftgraph/msgraph-sdk-php) graph，用于调用 microsoft graph。

1. 在 CLI 中运行以下命令。

    ```Shell
    composer require league/oauth2-client microsoft/microsoft-graph
    ```

## <a name="design-the-app"></a>设计应用程序

1. 在 **/resources/views**目录中创建一个名为`layout.blade.php`的新文件，并添加以下代码。

    :::code language="php" source="../demo/graph-tutorial/resources/views/layout.blade.php" id="LayoutSnippet":::

    此代码添加简单样式的[引导](http://getbootstrap.com/)，并添加一些简单图标的[字体](https://fontawesome.com/)。 它还定义具有导航栏的全局布局。

1. 在`./public`名为`css`的目录中创建一个新目录，然后在名为`./public/css` `app.css`的目录中创建一个新文件。 添加以下代码。

    :::code language="css" source="../demo/graph-tutorial/public/css/app.css":::

1. 打开`./resources/views/welcome.blade.php`文件，并将其内容替换为以下内容。

    :::code language="php" source="../demo/graph-tutorial/resources/views/welcome.blade.php" id="WelcomeSnippet":::

1. 通过将以下`Controller`函数添加到类来更新 **/app/Http/Controllers/Controller.php**中的基类。

    :::code language="php" source="../demo/graph-tutorial/app/Http/Controllers/Controller.php" id="LoadViewDataSnippet":::

1. 在名为`./app/Http/Controllers` `HomeController.php`的目录中创建一个新文件，并添加以下代码。

    :::code language="php" source="../demo/graph-tutorial/app/Http/Controllers/HomeController.php":::

1. 更新中`./routes/web.php`的路由以使用新的控制器。 将此文件的全部内容替换为以下内容。

    ```php
    <?php

    use Illuminate\Support\Facades\Route;

    Route::get('/', 'HomeController@welcome');
    ```

1. 保存所有更改，然后重新启动服务器。 现在，应用程序看起来应非常不同。

    ![重新设计的主页的屏幕截图](./images/create-app-01.png)
