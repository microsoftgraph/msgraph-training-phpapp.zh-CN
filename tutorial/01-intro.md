<!-- markdownlint-disable MD002 MD041 -->

本教程向您介绍如何构建 PHP web 应用程序，该应用程序使用 Microsoft Graph API 检索用户的日历信息。

> [!TIP]
> 如果您只想下载已完成的教程，可以通过两种方式下载它。
>
> - 下载 [PHP 快速入门](https://developer.microsoft.com/graph/quick-start?platform=option-php) ，以分钟为单位获取工作代码。
> - 下载或克隆 [GitHub 存储库](https://github.com/microsoftgraph/msgraph-training-phpapp)。

## <a name="prerequisites"></a>先决条件

在开始本教程之前，您应在开发计算机上安装了 [PHP](http://php.net/downloads.php)、 [作曲](https://getcomposer.org/)者和 [Laravel](https://laravel.com/) 。

您还应具有一个个人的 Microsoft 帐户，其中包含 Outlook.com 上的邮箱，或 Microsoft 工作或学校帐户。 如果你没有 Microsoft 帐户，可以使用以下几种方法获取免费帐户：

- 你可以 [注册新的个人 Microsoft 帐户](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1)。
- 你可以 [注册 office 365 开发人员计划](https://developer.microsoft.com/office/dev-program) 以获取免费的 office 365 订阅。

> [!NOTE]
> 本教程使用 PHP 版本7.4.4、书写器版本1.10.10 和 Laravel installer 版本（3.2.0）编写。 本指南中的步骤可能适用于其他版本，但尚未经过测试。

## <a name="feedback"></a>反馈

请在 [GitHub 存储库](https://github.com/microsoftgraph/msgraph-training-phpapp)中提供有关本教程的任何反馈。
