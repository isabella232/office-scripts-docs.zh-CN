---
title: Office 脚本和 VBA 宏之间的区别
description: Office 脚本和 Excel VBA 宏之间的行为和 API 差异。
ms.date: 03/23/2020
localization_priority: Normal
ms.openlocfilehash: 3a0f2c9a2ed7181a10e41d1f45b3af695877a680
ms.sourcegitcommit: d556aaefac80e55f53ac56b7f6ecbc657ebd426f
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 03/26/2020
ms.locfileid: "42978700"
---
# <a name="differences-between-office-scripts-and-vba-macros"></a>Office 脚本和 VBA 宏之间的区别

Office 脚本和 VBA 宏具有很多共同之处。 它们都允许用户通过易于使用的操作录制器自动执行解决方案，并允许编辑这些录制。 这两个框架都旨在让可能不会考虑自己的程序员在 Excel 中创建小型程序的人员。
基本区别在于，VBA 宏是为桌面解决方案开发的，而 Office 脚本是通过跨平台支持和安全性设计的，以指导原则为依据。 目前，仅在 web 上的 Excel 中支持 Office 脚本。

![显示不同 Office 扩展性解决方案的重点领域的四象限图。 Office 脚本和 VBA 宏都旨在帮助最终用户创建解决方案，但 Office 脚本是为 web 和协作（而 VBA 是针对桌面）而构建的。](../images/office-programmability-diagram.png)

本文介绍 VBA 宏（以及常规的 VBA）和 Office 脚本之间的主要差异。 由于 Office 脚本仅适用于 Excel，所以这里只讨论唯一的主机。

## <a name="platform-and-ecosystem"></a>平台和生态系统

VBA 设计用于桌面，而 Office 脚本是为 web 设计的。 VBA 可以与用户桌面进行交互。 这使其能够与类似的技术（如 COM 和 OLE）集成。 但是，VBA 无法方便地调用到 internet。

Office 脚本使用通用运行时或 JavaScript。 这将提供一致的行为和可访问性，而无需考虑用于运行脚本的计算机。 它们还可以调用其他 web 服务。

## <a name="security"></a>安全性

VBA 宏与 Excel 具有相同的安全净空。 这样，他们就可以拥有对桌面的完全访问权限。 Office 脚本仅具有对工作簿的访问权限，而不是承载工作簿的计算机。 此外，不能与脚本共享任何 JavaScript 身份验证令牌，因此脚本永远不能通过外部服务进行身份验证。

管理员具有三个 VBA 宏选项：允许租户上的所有宏、不允许租户上的宏，或仅允许带有签名证书的宏。 这种缺乏的粒度使得难以隔离单个损坏的主角。 目前，Office 脚本是针对租户的 "打开" 或 "关闭"。 不过，我们正在努力为管理员提供对各个脚本和脚本编写者的更多控制。

## <a name="coverage"></a>报道

目前，VBA 提供了更全面的 Excel 功能，尤其是在桌面客户端上提供的功能。 Office 脚本涵盖了 web 上的 Excel 的几乎所有方案。 此外，在 web 上 debut 新功能时，Office 脚本将同时为操作记录器和 JavaScript Api 支持这些功能。

## <a name="power-automate"></a>电源自动化

可以通过 "Power 自动化" 运行 Office 脚本。 您的工作簿可以通过计划或事件驱动的流进行更新，让您无需打开 Excel 即可自动执行工作流。 这意味着只要您的工作簿存储在 OneDrive 中（并可供电源自动访问），流就可以运行您的脚本，而不管您和您的组织使用的是 Excel 的桌面、Mac 还是 web 客户端。

VBA 没有与电源自动化的集成。 所有受支持的 VBA 方案都涉及用户参与宏的执行。

## <a name="see-also"></a>另请参阅

- [Excel 网页版中的 Office 脚本](../overview/excel.md)
- [Office 脚本与 Office 外接程序之间的差异](add-ins-differences.md)
- [Office 脚本疑难解答](../testing/troubleshooting.md)
- [Excel VBA 参考](/office/vba/api/overview/excel)