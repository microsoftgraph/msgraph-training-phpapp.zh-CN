<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="d2edd-101">本教程向您介绍如何构建 PHP web 应用程序, 该应用程序使用 Microsoft Graph API 检索用户的日历信息。</span><span class="sxs-lookup"><span data-stu-id="d2edd-101">This tutorial teaches you how to build a PHP web app that uses the Microsoft Graph API to retrieve calendar information for a user.</span></span>

> [!TIP]
> <span data-ttu-id="d2edd-102">如果您只想下载已完成的教程, 可以通过两种方式下载它。</span><span class="sxs-lookup"><span data-stu-id="d2edd-102">If you prefer to just download the completed tutorial, you can download it in two ways.</span></span>
>
> - <span data-ttu-id="d2edd-103">下载[PHP 快速入门](https://developer.microsoft.com/graph/quick-start?platform=option-php), 以分钟为单位获取工作代码。</span><span class="sxs-lookup"><span data-stu-id="d2edd-103">Download the [PHP quick start](https://developer.microsoft.com/graph/quick-start?platform=option-php) to get working code in minutes.</span></span>
> - <span data-ttu-id="d2edd-104">下载或克隆[GitHub 存储库](https://github.com/microsoftgraph/msgraph-training-phpapp)。</span><span class="sxs-lookup"><span data-stu-id="d2edd-104">Download or clone the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-phpapp).</span></span>

## <a name="prerequisites"></a><span data-ttu-id="d2edd-105">先决条件</span><span class="sxs-lookup"><span data-stu-id="d2edd-105">Prerequisites</span></span>

<span data-ttu-id="d2edd-106">在开始本教程之前, 您应在开发计算机上安装了[PHP](http://php.net/downloads.php)、[作曲](https://getcomposer.org/)者和[Laravel](https://laravel.com/) 。</span><span class="sxs-lookup"><span data-stu-id="d2edd-106">Before you start this tutorial, you should have [PHP](http://php.net/downloads.php), [Composer](https://getcomposer.org/), and [Laravel](https://laravel.com/) installed on your development machine.</span></span>

> [!NOTE]
> <span data-ttu-id="d2edd-107">本教程是使用 PHP 7.2 版编写的。</span><span class="sxs-lookup"><span data-stu-id="d2edd-107">This tutorial was written with PHP version 7.2.</span></span> <span data-ttu-id="d2edd-108">本指南中的步骤可能适用于其他版本, 但尚未经过测试。</span><span class="sxs-lookup"><span data-stu-id="d2edd-108">The steps in this guide may work with other versions, but that has not been tested.</span></span>

## <a name="feedback"></a><span data-ttu-id="d2edd-109">反馈</span><span class="sxs-lookup"><span data-stu-id="d2edd-109">Feedback</span></span>

<span data-ttu-id="d2edd-110">请在[GitHub 存储库](https://github.com/microsoftgraph/msgraph-training-phpapp)中提供有关本教程的任何反馈。</span><span class="sxs-lookup"><span data-stu-id="d2edd-110">Please provide any feedback on this tutorial in the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-phpapp).</span></span>
