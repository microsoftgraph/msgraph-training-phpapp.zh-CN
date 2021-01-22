<!-- markdownlint-disable MD002 MD041 -->

在此部分中，您将添加在用户日历上创建事件的能力。

## <a name="create-new-event-form"></a>创建新的事件表单

1. 在 **./resources/views** 目录中创建一个名为的新文件 `newevent.blade.php` ，并添加以下代码。

    :::code language="php" source="../demo/graph-tutorial/resources/views/newevent.blade.php" id="NewEventFormSnippet":::

## <a name="add-controller-actions"></a>添加控制器操作

1. 打开 **./app/Http/Controllers/CalendarController.php** 并添加以下函数以呈现表单。

    :::code language="php" source="../demo/graph-tutorial/app/Http/Controllers/CalendarController.php" id="getNewEventFormSnippet":::

1. 添加以下函数以在用户提交时接收表单数据，并创建用户日历上的新事件。

    :::code language="php" source="../demo/graph-tutorial/app/Http/Controllers/CalendarController.php" id="createNewEventSnippet":::

    考虑此代码执行哪些功能。

    - 它将与会者字段输入转换为 Graph [与会者](https://docs.microsoft.com/graph/api/resources/attendee?view=graph-rest-1.0) 对象数组。
    - 它通过 [表单输入](https://docs.microsoft.com/graph/api/resources/event?view=graph-rest-1.0) 生成事件。
    - 它将 POST 发送到 `/me/events` 终结点，然后重定向回日历视图。

1. 保存所有更改并重新启动服务器。 使用 **"新建事件** "按钮导航到新事件窗体。

1. 填写表单上的值。 使用从当前周开始的开始日期。 选择“创建”。

    ![新事件表单的屏幕截图](images/create-event-01.png)

1. 当应用重定向到日历视图时，请验证新事件是否存在于结果中。
