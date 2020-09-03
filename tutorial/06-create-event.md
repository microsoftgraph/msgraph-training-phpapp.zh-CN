<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="4c685-101">在本节中，您将添加在用户日历上创建事件的功能。</span><span class="sxs-lookup"><span data-stu-id="4c685-101">In this section you will add the ability to create events on the user's calendar.</span></span>

## <a name="create-new-event-form"></a><span data-ttu-id="4c685-102">创建新的事件表单</span><span class="sxs-lookup"><span data-stu-id="4c685-102">Create new event form</span></span>

1. <span data-ttu-id="4c685-103">在 **/resources/views** 目录中创建一个名为的新文件 `newevent.blade.php` ，并添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="4c685-103">Create a new file in the **./resources/views** directory named `newevent.blade.php` and add the following code.</span></span>

    :::code language="php" source="../demo/graph-tutorial/resources/views/newevent.blade.php" id="NewEventFormSnippet":::

## <a name="add-controller-actions"></a><span data-ttu-id="4c685-104">添加控制器操作</span><span class="sxs-lookup"><span data-stu-id="4c685-104">Add controller actions</span></span>

1. <span data-ttu-id="4c685-105">打开 **/app/Http/Controllers/CalendarController.php** ，并添加以下函数以呈现表单。</span><span class="sxs-lookup"><span data-stu-id="4c685-105">Open **./app/Http/Controllers/CalendarController.php** and add the following function to render the form.</span></span>

    :::code language="php" source="../demo/graph-tutorial/app/Http/Controllers/CalendarController.php" id="getNewEventFormSnippet":::

1. <span data-ttu-id="4c685-106">添加以下函数以在用户提交时接收表单数据，并在用户的日历上创建一个新事件。</span><span class="sxs-lookup"><span data-stu-id="4c685-106">Add the following function to receive the form data when the user's submits, and create a new event on the user's calendar.</span></span>

    :::code language="php" source="../demo/graph-tutorial/app/Http/Controllers/CalendarController.php" id="createNewEventSnippet":::

    <span data-ttu-id="4c685-107">请考虑此代码执行的操作。</span><span class="sxs-lookup"><span data-stu-id="4c685-107">Consider what this code does.</span></span>

    - <span data-ttu-id="4c685-108">它将 "与会者" 字段输入转换为 Graph [与会者](https://docs.microsoft.com/graph/api/resources/attendee?view=graph-rest-1.0) 对象的数组。</span><span class="sxs-lookup"><span data-stu-id="4c685-108">It converts the attendees field input to an array of Graph [attendee](https://docs.microsoft.com/graph/api/resources/attendee?view=graph-rest-1.0) objects.</span></span>
    - <span data-ttu-id="4c685-109">它生成来自表单输入的 [事件](https://docs.microsoft.com/graph/api/resources/event?view=graph-rest-1.0) 。</span><span class="sxs-lookup"><span data-stu-id="4c685-109">It builds an [event](https://docs.microsoft.com/graph/api/resources/event?view=graph-rest-1.0) from the form input.</span></span>
    - <span data-ttu-id="4c685-110">它向终结点发送一篇文章 `/me/events` ，然后重定向回 "日历" 视图。</span><span class="sxs-lookup"><span data-stu-id="4c685-110">It sends a POST to the `/me/events` endpoint, then redirects back to the calendar view.</span></span>

1. <span data-ttu-id="4c685-111">更新 **/routes/web.php** 中的路由，以在控制器上为这些新函数添加路由。</span><span class="sxs-lookup"><span data-stu-id="4c685-111">Update the routes in **./routes/web.php** to add routes for these new functions on the controller.</span></span>

    ```php
    Route::get('/calendar/new', 'CalendarController@getNewEventForm');
    Route::post('/calendar/new', 'CalendarController@createNewEvent');
    ```

1. <span data-ttu-id="4c685-112">保存所有更改，然后重新启动服务器。</span><span class="sxs-lookup"><span data-stu-id="4c685-112">Save all of your changes and restart the server.</span></span> <span data-ttu-id="4c685-113">使用 " **新建事件** " 按钮导航到新的事件窗体。</span><span class="sxs-lookup"><span data-stu-id="4c685-113">Use the **New event** button to navigate to the new event form.</span></span>

1. <span data-ttu-id="4c685-114">填写表单上的值。</span><span class="sxs-lookup"><span data-stu-id="4c685-114">Fill in the values on the form.</span></span> <span data-ttu-id="4c685-115">使用从当前周开始的日期。</span><span class="sxs-lookup"><span data-stu-id="4c685-115">Use a start date from the current week.</span></span> <span data-ttu-id="4c685-116">选择“创建”\*\*\*\*。</span><span class="sxs-lookup"><span data-stu-id="4c685-116">Select **Create**.</span></span>

    ![新事件表单的屏幕截图](images/create-event-01.png)

1. <span data-ttu-id="4c685-118">当应用程序重定向到 "日历" 视图时，请验证结果中是否存在您的新事件。</span><span class="sxs-lookup"><span data-stu-id="4c685-118">When the app redirects to the calendar view, verify that your new event is present in the results.</span></span>
