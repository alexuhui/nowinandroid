# 模块化学习之旅

在这个学习之旅中，你将了解到模块化以及在 Now in Android 应用程序中使用的模块化策略。

## 概述

模块化是将单一模块代码库的概念分解为松耦合的、自包含的模块的实践。

### 模块化的好处

这带来了许多好处，包括：

**可扩展性** - 在紧密耦合的代码库中，单一更改可能引发一系列的修改。一个正确模块化的项目将采用“关注点分离”原则。这反过来赋予贡献者更多的自主权，同时也强制执行架构模式。

**并行工作** - 模块化有助于减少版本控制冲突，并使更大团队的开发人员能够更有效地并行工作。

**所有权** - 一个模块可以有一个专门的所有者，负责维护代码和测试，修复错误，并审查更改。

**封装** - 隔离的代码更容易阅读、理解、测试和维护。

**减少构建时间** - 利用Gradle的并行和增量构建可以减少构建时间。

**动态交付** - 模块化是[Play Feature Delivery](https://developer.android.com/guide/playcore/feature-delivery)的要求，允许应用程序的某些功能有条件地交付或按需下载。

**可重用性** - 正确的模块化为代码共享和从相同基础构建多个应用程序提供了机会，跨不同平台。

### 模块化陷阱

然而，模块化是一种可能被滥用的模式，模块化应用程序时需要注意一些问题：

**模块太多** - 每个模块都带来了一些开销，表现为构建配置复杂性的增加。这可能导致Gradle同步时间增加，并产生持续的维护成本。此外，与单一的单块模块相比，添加更多模块会增加项目的Gradle设置的复杂性。可以通过使用约定插件来缓解这一问题，将可重复使用和可组合的构建配置提取到类型安全的Kotlin代码中。在Now in Android应用程序中，这些约定插件可以在[`build-logic`文件夹](https://github.com/android/nowinandroid/tree/main/build-logic)中找到。

**模块不足** - 相反，如果模块太少、太大并且紧密耦合，最终会得到另一个单块。这意味着您会失去一些模块化的好处。如果您的模块庞大且没有单一、明确定义的目的，您应该考虑拆分它。

**太复杂** - 这里没有银弹。事实上，并不总是有意义对项目进行模块化。一个主导因素是代码库的大小和相对复杂性。如果您的项目不预计会增长到某个阈值以上，可扩展性和构建时间的提高就没有意义。

## 模块化策略

重要的是要注意，没有适用于所有项目的单一模块化策略。然而，有一些通用的指导原则可以遵循，以确保最大化其好处并最小化其缺点。

一个简单的模块是一个带有Gradle构建脚本的目录。通常，一个模块将包含一个或多个源集，以及可能的资源或资产集合。模块可以独立构建和测试。由于Gradle的灵活性，对于如何组织项目，几乎没有约束。一般来说，应该追求低耦合和高内聚。

* **低耦合** - 模块之间应尽可能独立，这样对一个模块的更改对其他模块的影响为零或最小。它们不应了解其他模块的内部工作原理。

* **高内聚** - 模块应包括作为系统的一部分的代码集合。它应该具有明确定义的责任，并在某些领域知识的边界内。例如，Now in Android中的[`core:network`模块](https://github.com/android/nowinandroid/tree/main/core/network)负责发出网络请求，处理来自远程数据源的响应，并向其他模块提供数据。

## Now in Android中的模块类型

![显示Now in Android中模块类型及其依赖关系的图表](images/modularization-graph.drawio.png "显示Now in Android中模块类型及其依赖关系的图表")

**顶级提示**：模块图（如上所示）在模块化计划中可用于可视化模块之间的依赖关系。

Now in Android应用程序包含以下类型的模块：

* `app`模块 - 包含绑定代码库其余部分的应用级别和脚手架类，例如`MainActivity`，`NiaApp`以及应用程序级控制的导航。通过`NiaNavHost`设置导航，通过`TopLevelDestination`设置底部导航栏。`app`模块依赖于所有`feature`模块和所需的`core`模块。

* `feature:` 模块 - 特定功能的模块，被限定为处理应用程序中的单一责

任。当需要时，这些模块可以被任何应用程序重复使用，包括测试或其他变种的应用程序，同时仍然保持分离和隔离。如果一个类仅由一个`feature`模块使用，它应该保留在该模块内。否则，应将其提取到适当的`core`模块中。`feature`模块不应依赖于其他`feature`模块。它们只依赖于它们需要的`core`模块。

* `core:` 模块 - 包含辅助代码和需要在应用程序的其他模块之间共享的特定依赖项的通用库模块。这些模块可以依赖于其他`core`模块，但不应该依赖于`feature`模块或`app`模块。

* 其他模块 - 例如`sync`，`benchmark`和`test`模块，以及`app-nia-catalog` - 一个用于快速显示我们设计系统的目录应用程序。


# 模块

使用上述模块化策略，Now in Android 应用程序具有以下模块：

| 名称 | 职责 | 关键类和示例 |
| --- | --- | --- |
| `app` | 将所有必要的内容组合在一起，使应用程序能够正确运行。包括 UI 脚手架和导航。 | `NiaApp, MainActivity`<br>通过 `NiaNavHost, NiaAppState, TopLevelDestination` 实现应用程序级控制的导航 |
| `feature:1, feature:2, ...` | 与特定功能或用户流程相关的功能。通常包含从其他模块读取数据的 UI 组件和 ViewModel。<br>示例包括：<br>- [`feature:topic`](https://github.com/android/nowinandroid/tree/main/feature/topic) 在 TopicScreen 上显示有关主题的信息。<br>- [`feature:foryou`](https://github.com/android/nowinandroid/tree/main/feature/foryou) 显示用户的新闻动态，并在首次运行时进行引导，在 For You 屏幕上。 | `TopicScreen`<br>`TopicViewModel` |
| `core:data` | 从多个来源获取应用程序数据，由不同功能共享。 | `TopicsRepository` |
| `core:designsystem` | 包括核心 UI 组件（其中许多是定制的 Material 3 组件）、应用主题和图标的设计系统。可以通过运行 `app-nia-catalog` 配置查看设计系统。 | `NiaIcons`、`NiaButton`、`NiaTheme` |
| `core:ui` | 由功能模块使用的复合 UI 组件和资源，如新闻动态。与 `designsystem` 模块不同，它依赖于数据层，因为它渲染模型，如新闻资源。 | `NewsFeed`、`NewsResourceCardExpanded` |
| `core:common` | 在模块之间共享的常见类。 | `NiaDispatchers`、`Result` |
| `core:network` | 进行网络请求并处理来自远程数据源的响应。 | `RetrofitNiaNetworkApi` |
| `core:testing` | 测试依赖关系、存储库和实用类。 | `NiaTestRunner`、`TestDispatcherRule` |
| `core:datastore` | 使用 DataStore 存储持久数据。 | `NiaPreferences`、`UserPreferencesSerializer` |
| `core:database` | 使用 Room 进行本地数据库存储。 | `NiaDatabase`、`DatabaseMigrations`、`Dao` 类 |
| `core:model` | 在整个应用程序中使用的模型类。 | `Topic`、`Episode`、`NewsResource` |


## Modularization in Now in Android

我们的模块化方法是在考虑“Now in Android”项目路线图、即将进行的工作和新功能的基础上定义的。此外，这一次我们的目标是找到过度模块化相对较小应用程序的正确平衡，同时利用这个机会展示一个适用于更大规模代码库、更接近实际生产环境的模块化模式。

这种方法已经与Android社区进行了讨论，并在考虑到他们的反馈的基础上发展演变。然而，对于模块化而言，并没有一种解决方案是所有其他方案都错误的。最终，有许多方法和途径来对应用程序进行模块化，很少有一种方法适用于所有目的、代码库和团队偏好。这就是为什么在预先规划并考虑所有目标、试图解决的问题、未来工作以及预测潜在障碍的情况下，定义适合您独特情况下的最佳结构是至关重要的步骤。开发人员可以通过头脑风暴会议来绘制模块和依赖关系的图表，以更好地可视化和计划。

我们的方法就是这样的一个例子 - 我们不希望它成为一个适用于所有情况的不可改变的结构，实际上，它可能会在未来发生演变和变化。这是一个我们发现适用于我们项目的通用指导方针，我们将其作为一个您可以进一步修改、扩展和建立在其基础上的示例提供。其中一种方法是增加代码库的粒度。粒度是指代码库由模块组成的程度。如果您的数据层很小，将其保留在一个单一模块中是可以接受的。但是一旦存储库和数据源的数量开始增长，考虑将它们拆分成单独的模块可能是值得的。

我们也一直对您的建设性反馈持开放态度 - 从社区中学习和交流想法是改进我们指导的关键要素之一。

