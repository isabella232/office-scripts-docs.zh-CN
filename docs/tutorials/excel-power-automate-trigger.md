---
title: 将数据传递到自动运行的 Power Automate 流中的脚本
description: 有关在收到邮件时通过 Power Automate 在 Web 上运行 Office Scripts for Excel，并将流数据传递到脚本的教程。
ms.date: 07/24/2020
localization_priority: Priority
ms.openlocfilehash: f6842e27686909bad92138e6d2f9ac1892cac891
ms.sourcegitcommit: ce72354381561dc167ea0092efd915642a9161b3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/30/2020
ms.locfileid: "48319677"
---
# <a name="pass-data-to-scripts-in-an-automatically-run-power-automate-flow-preview"></a>将数据传递到自动运行的 Power Automate 流中的脚本（预览版）

本教程教你如何通过自动化的[ Power Automate ](https://flow.microsoft.com)工作流在 Web 上使用 Office Script for Excel。 每当你收到电子邮件时，脚本都会自动运行，并将电子邮件中的信息记录在 Excel 工作簿中。 能够将其他应用程序中的数据传递到 Office 脚本中，可以为你在自动化过程中提供极大的灵活性和自由度。

> [!TIP]
> 如果你不熟悉 Office 脚本，建议先查看[在 Excel 网页版中录制、编辑和创建 Office 脚本](excel-tutorial.md)教程。 如果你没有使用过 Power Automate，建议你从[从手动 Power Automate 流调用脚本](excel-power-automate-manual.md)开始。 [Office 脚本使用 TypeScript](../overview/code-editor-environment.md)，本教程面向在 JavaScript 或 TypeScript 方面具备初级到中级知识的人员。 如果你不熟悉 JavaScript，建议从 [Mozilla JavaScript 教程](https://developer.mozilla.org/docs/Web/JavaScript/Guide/Introduction)入手。

## <a name="prerequisites"></a>先决条件

[!INCLUDE [Tutorial prerequisites](../includes/power-automate-tutorial-prerequisites.md)]

## <a name="prepare-the-workbook"></a>准备工作簿

Power Automate 无法使用`Workbook.getActiveWorksheet`之类的[相对引用](../develop/power-automate-integration.md#avoid-using-relative-references)访问工作簿组件。 因此，我们需要一个具有一致名称的工作簿和工作表，以供 Power Automate 引用。

1. 创建名为 **MyWorkbook** 的新工作簿。

2. 转到 "**自动**" 选项卡，然后选择 "**代码编辑器**"。

3. 选择 "**New Script**"。

4. 将现有代码替换为以下脚本，然后按 "**运行**"。 这会将工作簿设置为一致的工作表、表和数据透视表名称。

    ```TypeScript
    function main(workbook: ExcelScript.Workbook) {
      // Add a new worksheet to store our email table
      let emailsSheet = workbook.addWorksheet("Emails");

      // Add data and create a table
      emailsSheet.getRange("A1:D1").setValues([
        ["Date", "Day of the week", "Email address", "Subject"]
      ]);
      let newTable = workbook.addTable(emailsSheet.getRange("A1:D2"), true);
      newTable.setName("EmailTable");

      // Add a new PivotTable to a new worksheet
      let pivotWorksheet = workbook.addWorksheet("Subjects");
      let newPivotTable = workbook.addPivotTable("Pivot", "EmailTable", pivotWorksheet.getRange("A3:C20"));

      // Setup the pivot hierarchies
      newPivotTable.addRowHierarchy(newPivotTable.getHierarchy("Day of the week"));
      newPivotTable.addRowHierarchy(newPivotTable.getHierarchy("Email address"));
      newPivotTable.addDataHierarchy(newPivotTable.getHierarchy("Subject"));
    }
    ```

## <a name="create-an-office-script"></a>创建 Office 脚本

我们来创建一个脚本来记录电子邮件中的信息。 我们想知道一周中的哪几天我们收到最多的邮件，以及有多少发件人发送邮件。 我们的工作簿中有一个表格，其中包含**日期**，**星期几**，**电子邮件地址**和**主题**列。 我们的工作表还具有一个数据透视表，该数据透视表在**星期**和**电子邮件地址**（这些是行层次结构）上进行透视。 唯一**主题**的计数是所显示的聚合信息（数据层次结构）。 更新电子邮件表后，我们的脚本将刷新该数据透视表。

1. 在 **"代码编辑器"** 中，选择 **"New Script"**。

2. 我们将在本指南后面创建流程发送有关收到的每封电子邮件的脚本信息。 脚本需要通过 `main` 函数中的参数接受该输入。 将默认脚本替换为以下脚本：

    ```TypeScript
    function main(
      workbook: ExcelScript.Workbook,
      from: string,
      dateReceived: string,
      subject: string) {

    }
    ```

3. 脚本需要访问工作簿的表和数据透视表。 将下面的代码添加到脚本主体中的起始 `{`后面：

    ```TypeScript
    // Get the email table.
    let emailWorksheet = workbook.getWorksheet("Emails");
    let table = emailWorksheet.getTable("EmailTable");
  
    // Get the PivotTable.
    let pivotTableWorksheet = workbook.getWorksheet("Subjects");
    let pivotTable = pivotTableWorksheet.getPivotTable("Pivot");
    ```

4. `dateReceived`参数的类型为`string`。 我们将其转换为 [`Date` 对象](../develop/javascript-objects.md#date)，以便我们可以轻松地获取一周中的一天。 之后，我们需要将当天的数字值映射到更易读的版本。 将以下代码添加到脚本的末尾（在结束 `}` 之前）：

    ```TypeScript
      // Parse the received date string to determine the day of the week.
      let emailDate = new Date(dateReceived);
      let dayName = emailDate.toLocaleDateString("en-US", { weekday: 'long' });
    ```

5. `subject` 字符串可能包含 "RE:" 回复标记。 从字符串中删除该对象，以便同一线程中的电子邮件具有相同的表格主题。 将以下代码添加到脚本的末尾（在结束 `}` 之前）：

    ```TypeScript
    // Remove the reply tag from the email subject to group emails on the same thread.
    let subjectText = subject.replace("Re: ", "");
    subjectText = subjectText.replace("RE: ", "");
    ```

6. 现在，电子邮件数据已经按照我们的喜好进行了格式化，让我们在电子邮件表中添加一行。 将以下代码添加到脚本的末尾（在结束 `}` 之前）：

    ```TypeScript
    // Add the parsed text to the table.
    table.addRow(-1, [dateReceived, dayName, from, subjectText]);
    ```

7. 最后，我们来确保刷新了数据透视表。 将以下代码添加到脚本的末尾（在结束 `}` 之前）：

    ```TypeScript
    // Refresh the PivotTable to include the new row.
    pivotTable.refresh();
    ```

8. 将脚本重命名为 "**录制电子邮件**"，然后按 "**保存脚本**"。

现在，你的脚本已准备就绪，可运行 Power Automate 工作流。 它应类似于以下脚本：

```TypeScript
function main(
  workbook: ExcelScript.Workbook,
  from: string,
  dateReceived: string,
  subject: string) {
  // Get the email table.
  let emailWorksheet = workbook.getWorksheet("Emails");
  let table = emailWorksheet.getTable("EmailTable");

  // Get the PivotTable.
  let pivotTableWorksheet = workbook.getWorksheet("Subjects");
  let pivotTable = pivotTableWorksheet.getPivotTable("Pivot");

  // Parse the received date string to determine the day of the week.
  let emailDate = new Date(dateReceived);
  let dayName = emailDate.toLocaleDateString("en-US", { weekday: 'long' });

  // Remove the reply tag from the email subject to group emails on the same thread.
  let subjectText = subject.replace("Re: ", "");
  subjectText = subjectText.replace("RE: ", "");

  // Add the parsed text to the table.
  table.addRow(-1, [dateReceived, dayName, from, subjectText]);

  // Refresh the PivotTable to include the new row.
  pivotTable.refresh();
}
```

## <a name="create-an-automated-workflow-with-power-automate"></a>使用 Power Automate 功能创建自动工作流

1. 登录 [Power Automate 网站](https://flow.microsoft.com)。

2. 在屏幕左侧显示的菜单中，按 "**创建**"。 这将带你进入创建新工作流的方式列表。

    ![Power Automate 中的“创建”按钮。](../images/power-automate-tutorial-1.png)

3. 在**从空白开始**部分中，选择**即时流**。 这将创建由事件（例如接收电子邮件）触发的工作流。

    ![Power Automate 中的 Automated 流程选项。](../images/power-automate-params-tutorial-1.png)

4. 在出现的对话框窗口中，在 "**流名称**" 文本框中输入流的名称。 然后从"**选择流的触发器**" 下的 "选项" 列表中选择 "**新电子邮件到达时**"。 可能需要使用搜索框搜索选项。 最后，按 **创建**。

    ![在 Power Automate 中“构建自动流程窗口”的一部分，其中显示“新邮件到达”选项。](../images/power-automate-params-tutorial-2.png)

    > [!NOTE]
    > 本教程使用 Outlook。 可改为使用你喜欢的电子邮件服务，但某些选项可能不同。

5. 按 **"新建步骤"**。

6. 选择 "**标准**" 选项卡，然后选择 "**Excel Online （企业）**"。

    ![Excel Online （商业版）的 Power Automate 选项。](../images/power-automate-tutorial-4.png)

7. 在 "**操作**"下，选择 **运行脚本（预览版）**。

    ![运行脚本（预览版）的 Power Automate 操作选项。](../images/power-automate-tutorial-5.png)

8. 接下来，选择要在流步骤中使用的工作簿、脚本和脚本输入参数。 对于本教程，你将使用在 OneDrive 中创建的工作簿，但可以在 OneDrive 或 SharePoint 网站中使用任何工作簿。 为 **运行脚本** 连接器指定以下设置：

    - **位置**：OneDrive for Business
    - **文档库**： OneDrive
    - **文件**： MyWorkbook.xlsx
    - **Script**： "记录电子邮件"
    - **from**：来自 *（Outlook 中的动态内容）*
    - **dateReceived**：收到时间 *（Outlook 中的动态内容）*
    - **subject**： "主题" *（Outlook 中的动态内容）*

    *请注意，仅当选择脚本后，才会显示脚本的参数。*

    ![运行脚本（预览版）的 Power Automate 操作选项的参数。](../images/power-automate-params-tutorial-3.png)

9. 按“**保存**”。

现已启用你的流程。 每次通过 Outlook 收到电子邮件时，它都会自动运行脚本。

## <a name="manage-the-script-in-power-automate"></a>在 Power Automate 功能中管理脚本

1. 在 Power Automate 主页面上，选择**我的流**。

    ![Power Automate 中的 "我的流" 按钮。](../images/power-automate-tutorial-7.png)

2. 选择你的流程。 可在此处查看 "运行历史记录"。 可刷新页面，或按 "刷新 **所有运行"** 按钮更新历史记录。 收到电子邮件后，流将立即触发。 通过发送自己的邮件来测试流。

当流被触发并成功运行脚本时，应该可以看到工作簿的表和数据透视表更新。

![流运行几次之后的电子邮件表。](../images/power-automate-params-tutorial-4.png)

![流之后的数据透视表已经运行了几次。](../images/power-automate-params-tutorial-5.png)

## <a name="next-steps"></a>后续步骤

访问[使用 Power Automate 运行 Office 脚本](../develop/power-automate-integration.md)，以了解有关将 Office Script 与 Power Automate 连接的更多信息。

你还可以查看[自动任务提醒示例场景](../resources/scenarios/task-reminders.md)，以了解如何将 Office 脚本和 Power Automate 与 Team Adaptive Cards 结合使用。
