<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="45054-101">在此部分中，您将添加在用户日历上创建事件的能力。</span><span class="sxs-lookup"><span data-stu-id="45054-101">In this section you will add the ability to create events on the user's calendar.</span></span>

## <a name="create-new-event-form"></a><span data-ttu-id="45054-102">创建新的事件表单</span><span class="sxs-lookup"><span data-stu-id="45054-102">Create new event form</span></span>

1. <span data-ttu-id="45054-103">在 **./resources/views** 目录中创建一个名为的新文件 `newevent.blade.php` ，并添加以下代码。</span><span class="sxs-lookup"><span data-stu-id="45054-103">Create a new file in the **./resources/views** directory named `newevent.blade.php` and add the following code.</span></span>

    :::code language="php" source="../demo/graph-tutorial/resources/views/newevent.blade.php" id="NewEventFormSnippet":::

## <a name="add-controller-actions"></a><span data-ttu-id="45054-104">添加控制器操作</span><span class="sxs-lookup"><span data-stu-id="45054-104">Add controller actions</span></span>

1. <span data-ttu-id="45054-105">打开 **./app/Http/Controllers/CalendarController.php** 并添加以下函数以呈现表单。</span><span class="sxs-lookup"><span data-stu-id="45054-105">Open **./app/Http/Controllers/CalendarController.php** and add the following function to render the form.</span></span>

    :::code language="php" source="../demo/graph-tutorial/app/Http/Controllers/CalendarController.php" id="getNewEventFormSnippet":::

1. <span data-ttu-id="45054-106">添加以下函数以在用户提交时接收表单数据，并创建用户日历上的新事件。</span><span class="sxs-lookup"><span data-stu-id="45054-106">Add the following function to receive the form data when the user's submits, and create a new event on the user's calendar.</span></span>

    :::code language="php" source="../demo/graph-tutorial/app/Http/Controllers/CalendarController.php" id="createNewEventSnippet":::

    <span data-ttu-id="45054-107">考虑此代码执行哪些功能。</span><span class="sxs-lookup"><span data-stu-id="45054-107">Consider what this code does.</span></span>

    - <span data-ttu-id="45054-108">它将与会者字段输入转换为 Graph [与会者](https://docs.microsoft.com/graph/api/resources/attendee?view=graph-rest-1.0) 对象数组。</span><span class="sxs-lookup"><span data-stu-id="45054-108">It converts the attendees field input to an array of Graph [attendee](https://docs.microsoft.com/graph/api/resources/attendee?view=graph-rest-1.0) objects.</span></span>
    - <span data-ttu-id="45054-109">它通过 [表单输入](https://docs.microsoft.com/graph/api/resources/event?view=graph-rest-1.0) 生成事件。</span><span class="sxs-lookup"><span data-stu-id="45054-109">It builds an [event](https://docs.microsoft.com/graph/api/resources/event?view=graph-rest-1.0) from the form input.</span></span>
    - <span data-ttu-id="45054-110">它将 POST 发送到 `/me/events` 终结点，然后重定向回日历视图。</span><span class="sxs-lookup"><span data-stu-id="45054-110">It sends a POST to the `/me/events` endpoint, then redirects back to the calendar view.</span></span>

1. <span data-ttu-id="45054-111">保存所有更改并重新启动服务器。</span><span class="sxs-lookup"><span data-stu-id="45054-111">Save all of your changes and restart the server.</span></span> <span data-ttu-id="45054-112">使用 **"新建事件** "按钮导航到新事件窗体。</span><span class="sxs-lookup"><span data-stu-id="45054-112">Use the **New event** button to navigate to the new event form.</span></span>

1. <span data-ttu-id="45054-113">填写表单上的值。</span><span class="sxs-lookup"><span data-stu-id="45054-113">Fill in the values on the form.</span></span> <span data-ttu-id="45054-114">使用从当前周开始的开始日期。</span><span class="sxs-lookup"><span data-stu-id="45054-114">Use a start date from the current week.</span></span> <span data-ttu-id="45054-115">选择“创建”。</span><span class="sxs-lookup"><span data-stu-id="45054-115">Select **Create**.</span></span>

    ![新事件表单的屏幕截图](images/create-event-01.png)

1. <span data-ttu-id="45054-117">当应用重定向到日历视图时，请验证新事件是否存在于结果中。</span><span class="sxs-lookup"><span data-stu-id="45054-117">When the app redirects to the calendar view, verify that your new event is present in the results.</span></span>
