---
title: 通过手动 Power Automate 流呼叫脚本
description: 有关通过手动触发器在 Power Automate 中使用 Office 脚本的教程。
ms.date: 07/24/2020
localization_priority: Priority
ms.openlocfilehash: f447e465bc0b09043d64752266bc9b6dbe5a5d89
ms.sourcegitcommit: ff7fde04ce5a66d8df06ed505951c8111e2e9833
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/11/2020
ms.locfileid: "46616752"
---
# <a name="call-scripts-from-a-manual-power-automate-flow-preview"></a>通过手动 Power Automate 流呼叫脚本（预览版）

本教程将指导你如何通过 [Power Automate](https://flow.microsoft.com) 在 web 上运行 Office Script for Excel。 你将创建一个脚本，以当前时间更新两个单元格的值。 然后，你可以将该脚本连接到手动触发的 Power Automate 流，以便每当按下 Power Automate 中的按钮时，脚本便会运行。 了解基本模式后，可展开流以包括其他应用程序，并自动执行更多日常工作流。

> [!TIP]
> 如果你不熟悉 Office 脚本，建议先查看[在 Excel 网页版中录制、编辑和创建 Office 脚本](excel-tutorial.md)教程。 [Office 脚本使用 TypeScript](../overview/code-editor-environment.md)，本教程面向在 JavaScript 或 TypeScript 方面具备初级到中级知识的人员。 如果你不熟悉 JavaScript，建议从 [Mozilla JavaScript 教程](https://developer.mozilla.org/docs/Web/JavaScript/Guide/Introduction)入手。

## <a name="prerequisites"></a>先决条件

[!INCLUDE [Tutorial prerequisites](../includes/power-automate-tutorial-prerequisites.md)]

## <a name="prepare-the-workbook"></a>准备工作簿

Power Automate 无法使用 `Workbook.getActiveWorksheet` 访问工作簿组件等相对引用。 因此，我们需要一个具有 Power Automate 可以引用的一致名称的工作簿和工作表。

1. 创建名为 **MyWorkbook** 的新工作簿。

2. 在 **MyWorkbook** 工作簿中，创建一个名为 **TutorialWorksheet** 的工作表。

## <a name="create-an-office-script"></a>创建 Office 脚本

1. 转到 "**自动**" 选项卡，然后选择 "**代码编辑器**"。

2. 选择 "**New Script**"。

3. 将默认脚本替换为以下脚本。 此脚本将当前日期和时间添加到 **TutorialWorksheet** 工作表的前两个单元格。

    ```TypeScript
    function main(workbook: ExcelScript.Workbook) {
      // Get the "TutorialWorksheet" worksheet from the workbook.
      let worksheet = workbook.getWorksheet("TutorialWorksheet");

      // Get the cells at A1 and B1.
      let dateRange = worksheet.getRange("A1");
      let timeRange = worksheet.getRange("B1");

      // Get the current date and time using the JavaScript Date object.
      let date = new Date(Date.now());

      // Add the date string to A1.
      dateRange.setValue(date.toLocaleDateString());

      // Add the time string to B1.
      timeRange.setValue(date.toLocaleTimeString());
    }
    ```

4. 将脚本重命名为 "**设置日期和时间**"。 按脚本名称进行更改。

5. 按 "**保存脚本**" 保存脚本。

## <a name="create-an-automated-workflow-with-power-automate"></a>使用 Power Automate 功能创建自动工作流

1. 登录 [Power Automate 网站](https://flow.microsoft.com)。

2. 在屏幕左侧显示的菜单中，按 "**创建**"。 这将带你进入创建新工作流的方式列表。

    ![Power Automate 中的“创建”按钮。](../images/power-automate-tutorial-1.png)

3. 在**从空白开始**部分中，选择**即时流**。 这将创建手动激活的工作流。

    ![用于创建新工作流的 "即时流" 选项。](../images/power-automate-tutorial-2.png)

4. 在出现的对话框窗口中，在 "**流名称**" 文本框中输入流的名称，从 "**选择如何触发流**" 下的选项列表中，选择 "**手动触发流** "，然后按 "**创建**"。

    ![用于创建新的即时流的 "手动触发器" 选项。](../images/power-automate-tutorial-3.png)

    请注意，手动触发流仅是许多类型流中的一种。 在下一个教程中，你将创建收到电子邮件时自动运行的流程。

5. 按 **"新建步骤"**。

6. 选择 "**标准**" 选项卡，然后选择 "**Excel Online （企业）**"。

    ![Excel Online （商业版）的 Power Automate 选项。](../images/power-automate-tutorial-4.png)

7. 在 "**操作**"下，选择 **运行脚本（预览版）**。

    ![运行脚本（预览版）的 Power Automate 操作选项。](../images/power-automate-tutorial-5.png)

8. 接下来，选择要在流步骤中使用的工作簿和脚本。 对于本教程，你将使用在 OneDrive 中创建的工作簿，但可以在 OneDrive 或 SharePoint 网站中使用任何工作簿。 为 **运行脚本** 连接器指定以下设置：

    - **位置**：OneDrive for Business
    - **文档库**： OneDrive
    - **文件**： MyWorkbook.xlsx
    - **脚本**：设置日期和时间

    ![以 Power Automate 功能运行脚本的连接器设置。](../images/power-automate-tutorial-6.png)

9. 按“**保存**”。

现在，你的流程可以通过 Power Automate 运行。 可使用流编辑器中的 "**测试**" 按钮对其进行测试，或按照其余教程步骤运行流集合中的流程。

## <a name="run-the-script-through-power-automate"></a>通过 Power Automate 运行脚本

1. 在 Power Automate 主页面上，选择**我的流**。

    ![Power Automate 中的 "我的流" 按钮。](../images/power-automate-tutorial-7.png)

2. 从 "**我的流**" 选项卡中显示的流列表中选择 **我的教程流**。这将显示之前创建的流程的详细信息。

3. 按 **"运行"**。

    ![Power Automate 中的“运行”按钮。](../images/power-automate-tutorial-8.png)

4. 将显示用于运行流的任务窗格。 如果系统要求 **登录** 到 Excel Online，请按 **"继续"** 操作。

5. 按 **"运行流程"**。 此时将运行流，该流将运行相关的 Office 脚本。

6. 按“**完成**”。 你应该看到**运行**部分进行了相应的更新。

7. 刷新页面，查看 Power Automate 的结果。 如果成功，请转到工作簿查看已更新的单元格。 如果失败，请验证流的设置并再次运行。

    ![Power Automate 输出显示成功流运行。](../images/power-automate-tutorial-9.png)

## <a name="next-steps"></a>后续步骤

完成[将数据传递到自动运行的 Power Automate 流中的脚本](excel-power-automate-trigger.md)教程。 它教你如何将数据从工作流服务传递到你的 Office 脚本，并在发生特定事件时运行 Power Automate 流。
