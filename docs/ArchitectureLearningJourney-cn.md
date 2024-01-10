# 架构学习之旅

在这个学习之旅中，你将了解Now in Android应用程序的架构：其层次结构、关键类以及它们之间的交互。

## 目标和要求

该应用程序架构的目标有：

- 尽可能地遵循[官方架构指南](https://developer.android.com/jetpack/guide)。
- 对开发人员易于理解，不涉及太多实验性的内容。
- 支持多名开发人员共同在同一代码库上工作。
- 便于在开发人员的机器上进行本地和仪表化测试，以及使用持续集成（CI）。
- 最小化构建时间。

## 架构概述

该应用程序架构有三个层次结构：[数据层](https://developer.android.com/jetpack/guide/data-layer)、[领域层](https://developer.android.com/jetpack/guide/domain-layer)和[UI层](https://developer.android.com/jetpack/guide/ui-layer)。


# Architecture Learning Journey

在这个学习之旅中，你将了解Now in Android应用程序的架构：其层次结构、关键类以及它们之间的交互。

## 目标和要求

该应用程序架构的目标有：

- 尽可能地遵循[官方架构指南](https://developer.android.com/jetpack/guide)。
- 对开发人员易于理解，不涉及太多实验性的内容。
- 支持多名开发人员共同在同一代码库上工作。
- 便于在开发人员的机器上进行本地和仪表化测试，以及使用持续集成（CI）。
- 最小化构建时间。

## 架构概述

该应用程序架构有三个层次结构：[数据层](https://developer.android.com/jetpack/guide/data-layer)、[领域层](https://developer.android.com/jetpack/guide/domain-layer)和[UI层](https://developer.android.com/jetpack/guide/ui-layer)。

<center>
<img src="images/architecture-1-overall.png" width="600px" alt="Diagram showing overall app architecture" />
</center>

该架构遵循具有[单向数据流](https://developer.android.com/jetpack/guide/ui-layer#udf)的响应式编程模型。通过将数据层放在底部，主要概念包括：

- 更高层对较低层的更改做出反应。
- 事件向下流动。
- 数据向上流动。

数据流使用流进行实现，使用[Kotlin Flows](https://developer.android.com/kotlin/flow)。

### 例子：在"For You"屏幕上显示新闻

当应用程序首次运行时，它将尝试从远程服务器加载新闻资源列表（选择`prod`构建风格时，`demo`构建将使用本地数据）。加载后，根据用户选择的兴趣将这些新闻资源展示给用户。

以下图表显示了发生的事件以及数据如何从相关对象流动以实现此目标。

![Diagram showing how news resources are displayed on the For You screen](images/architecture-2-example.png "Diagram showing how news resources are displayed on the For You screen")

以下是每个步骤中正在发生的情况。查找相关代码的最简单方法是将项目加载到Android Studio中，并在“代码”列中搜索文本（方便的快捷键：双击 <kbd>⇧ SHIFT</kbd>）。

<table>
  <tr>
   <td><strong>步骤</strong>
   </td>
   <td><strong>描述</strong>
   </td>
   <td><strong>代码 </strong>
   </td>
  </tr>
  <tr>
   <td>1
   </td>
   <td>在应用程序启动时，会将同步所有存储库的<code>[WorkManager](https://developer.android.com/topic/libraries/architecture/workmanager)</code>作业入队。
   </td>
   <td><code>Sync.initialize</code>
   </td>
  </tr>
  <tr>
   <td>2
   </td>
   <td><code>ForYouViewModel</code>调用<code>GetUserNewsResourcesUseCase</code>以获取具有其收藏/保存状态的新闻资源流。在用户和新闻存储库都发出项目之前，不会将任何项目发出到此流中。在等待期间，将feed状态设置为<code>Loading</code>。
   </td>
   <td>搜索<code>NewsFeedUiState.Loading</code>的用法
   </td>
  </tr>
  <tr>
   <td>3
   </td>
   <td>用户数据存储库从由Proto DataStore支持的本地数据源获取<code>UserData</code>对象流。
   </td>
   <td><code>NiaPreferencesDataSource.userData</code>
   </td>
  </tr>
  <tr>
   <td>4
   </td>
   <td>WorkManager执行同步作业，调用<code>OfflineFirstNewsRepository</code>开始与远程数据源同步数据。
   </td>
   <td><code>SyncWorker.doWork</code>
   </td>
  </tr>
  <tr>
   <td>5
   </td>
   <td><code>OfflineFirstNewsRepository</code>调用<code>RetrofitNiaNetwork</code>执行实际的API请求，使用<a href="https://square.github.io/retrofit/">Retrofit</a>。
   </td>
   <td><code>OfflineFirstNewsRepository.syncWith</code>
   </td>
  </tr>
  <tr>
   <td>6
   </td>
   <td><code>RetrofitNiaNetwork</code>调用远程服务器上的REST API。
   </td>
   <td><code>RetrofitNiaNetwork.getNewsResources</code>
   </td>
  </tr>
  <tr>
   <td>7
   </td>
   <td><code>RetrofitNiaNetwork</code>从远程服务器接收网络响应。
   </td>
   <td><code>RetrofitNiaNetwork.getNewsResources</code>
   </td>
  </tr>
  <tr>
   <td>8
   </td>
   <td><code>OfflineFirstNewsRepository</code>通过在本地<a href="https://developer.android.com/training/data-storage/room">Room数据库</a>中插入、更新或删除数据，将远程数据与<code>NewsResourceDao</code>同步。
   </td>
   <td><code>OfflineFirstNewsRepository.syncWith</code>
   </td>
  </tr>
  <tr>
   <td>9
   </td>
   <td>当<code>NewsResourceDao</code>中的数据发生更改时，它将被发出到新闻资源数据流中（这是一个<a href="https://developer.android.com/kotlin/flow">Flow</a>）。
   </td>
   <td><code>NewsResourceDao.getNewsResources</code>
   </td>
  </tr>
  <tr>
   <td>10
   </td>
   <td><code>OfflineFirstNewsRepository</code>充当此流的<a href="https://developer.android.com/kotlin/flow#modify">中间操作者</a>，将传入的<code>PopulatedNewsResource</code>（数据层内部的数据库模型）转换为由其他层使用的公共<code>NewsResource</code>模型。
   </td>
   <td><code>OfflineFirstNewsRepository.getNewsResources</code>
   </td>
  </tr>
  <tr>
   <td>11
   </td>
   <td><code>GetUserNewsResourcesUseCase</code>将新闻资源列表与用户数据结合起来，发出<code>UserNewsResource</code>列表。
   </td>
   <td><code>GetUserNewsResourcesUseCase.invoke</code>
   </td>
  </tr>
  <tr>
   <td>12
   </td>
   <td>当<code>ForYouViewModel</code>接收到可保存的新闻资源时，它将feed状态更新为<code>Success</code>。
   </td>
   <td>搜索<code>NewsFeedUiState.Success</code>的实例
   </td>
  </tr>
</table>



## 数据层

数据层作为应用程序数据和业务逻辑的离线优先源进行实现。它是应用程序中所有数据的真实来源。

![显示数据层架构的图表](images/architecture-3-data-layer.png "显示数据层架构的图表")

每个存储库都有其自己的模型。例如，`TopicsRepository` 有一个 `Topic` 模型，`NewsRepository` 有一个 `NewsResource` 模型。

存储库是其他层的公共 API，它们提供访问应用程序数据的**唯一**方式。存储库通常提供一种或多种读写数据的方法。

### 读取数据

数据以数据流的形式公开。这意味着存储库的每个客户端都必须准备好对数据更改做出反应。数据不是作为快照（例如 `getModel`）公开的，因为无法保证在使用时它仍然有效。

读取是从本地存储执行的，因此从 `Repository` 实例读取时不期望出现错误。但是，在尝试协调本地存储中的数据与远程源时可能会发生错误。有关错误协调的更多信息，请查看下面的数据同步部分。

_示例：读取主题列表_

可以通过订阅 `TopicsRepository::getTopics` 流来获取主题列表，该流会发出 `List<Topic>`。

每当主题列表更改时（例如，添加新主题时），更新后的 `List<Topic>` 会被发出到流中。

### 写入数据

要写入数据，存储库提供挂起函数。调用者负责确保其执行受到适当的范围约束。

_示例：关注主题_

只需调用 `UserDataRepository.toggleFollowedTopicId`，并传递用户希望关注的主题的 ID 以及 `followed=true` 表示应关注该主题（使用 `false` 取消关注主题）。

### 数据源

存储库可能依赖于一个或多个数据源。例如，`OfflineFirstTopicsRepository` 依赖于以下数据源：


<table>
  <tr>
   <td><strong>Name</strong>
   </td>
   <td><strong>Backed by</strong>
   </td>
   <td><strong>Purpose</strong>
   </td>
  </tr>
  <tr>
   <td>TopicsDao
   </td>
   <td><a href="https://developer.android.com/training/data-storage/room">Room/SQLite</a>
   </td>
   <td>与主题相关的持久关系数据
   </td>
  </tr>
  <tr>
   <td>NiaPreferencesDataSource
   </td>
   <td><a href="https://developer.android.com/topic/libraries/architecture/datastore">Proto DataStore</a>
   </td>
   <td>与用户首选项相关的持久非结构化数据，特别是用户感兴趣的主题。这在 .proto 文件中定义和建模，使用 protobuf 语法。
   </td>
  </tr>
  <tr>
   <td>NiaNetworkDataSource
   </td>
   <td>使用 Retrofit 访问的远程 API
   </td>
   <td>通过 REST API 端点以 JSON 形式提供主题数据。
   </td>
  </tr>
</table>

### 数据同步

存储库负责协调本地存储中的数据与远程源的数据。一旦从远程数据源获取数据，它将立即写入本地存储。更新后的数据从本地存储（Room）发出到相关的数据流中，并被任何正在监听的客户端接收。

这种方法确保了应用程序的读取和写入关注点是分离的，彼此不会干扰。

在数据同步期间发生错误的情况下，会采用指数回退策略。这由 `WorkManager` 通过 `SyncWorker` 实现，`SyncWorker` 实现了 `Synchronizer` 接口。

请参阅 `OfflineFirstNewsRepository.syncWith` 以查看数据同步的示例。

## 领域层

[领域层](https://developer.android.com/topic/architecture/domain-layer) 包含用例。这些是具有单个可调用方法 (`operator fun invoke`) 的类，包含业务逻辑。

这些用例用于简化和消除 ViewModel 中的重复逻辑。它们通常会从存储库中组合和转换数据。

例如，`GetUserNewsResourcesUseCase` 将 `NewsRepository` 中的 `NewsResource` 流（使用 `Flow` 实现）与 `UserDataRepository` 中的 `UserData` 对象流组合，以创建 `UserNewsResource` 流。此流由各种 ViewModel 用于显示屏幕上的新闻资源及其书签状态。

值得注意的是，Now in Android 中的领域层**目前不包含**任何用于事件处理的用例。事件由 UI 层直接调用存储库上的方法来处理。

## UI 层

[UI 层](https://developer.android.com/topic/architecture/ui-layer) 包括：



*   使用 [Jetpack Compose](https://developer.android.com/jetpack/compose) 构建的 UI 元素
*   [Android ViewModels](https://developer.android.com/topic/libraries/architecture/viewmodel)

ViewModel 从用例和存储库接收数据流，并将其转换为 UI 状态。UI 元素反映此状态，并提供用户与应用程序交互的方式。这些交互作用作为事件传递到 ViewModel，在那里对其进行处理。


![显示 UI 层架构的图表](images/architecture-4-ui-layer.png "显示 UI 层架构的图表")


### 对 UI 状态建模

UI 状态被建模为使用接口和不可变数据类的密封层次结构。状态对象只通过数据流的变换发出。这种方法确保：



*   UI 状态始终代表底层应用程序数据 - 应用程序数据是源头的真实性。
*   UI 元素处理所有可能的状态。

**示例：在“为您”屏幕上的新闻订阅**

“为您”屏幕上的新闻资源的订阅（列表）使用 `NewsFeedUiState` 进行建模。这是一个密封接口，它创建了两种可能状态的层次结构：



*   `Loading` 表示数据正在加载
*   `Success` 表示数据已成功加载。 Success 状态包含新闻资源列表。

`feedState` 被传递到 `ForYouScreen` 组合函数中，该函数处理这两种状态。


### 将流转换为 UI 状态

ViewModel 从一个或多个用例或存储库接收数据流，这些流是 [cold flows](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-flow/index.html)。这些流被 [组合](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/combine.html)在一起，或者简单地被 [映射](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/map.html) ，以产生一个单一的 UI 状态流。然后，这个单一的流通过 [stateIn](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/state-in.html) 转换为热流。将其转换为状态流使得 UI 元素能够从流中读取到最后已知的状态。

**示例：显示关注的主题**

`InterestsViewModel` 将 `uiState` 公开为 `StateFlow<InterestsUiState>`。通过获取 `GetFollowableTopicsUseCase` 提供的 `List<FollowableTopic>` 的 cold flow 来创建此 hot flow。每次发出新的列表时，它都会被转换为一个 `InterestsUiState.Interests` 状态，该状态暴露给 UI。


### 处理用户交互

用户操作通过常规方法调用从 UI 元素传递到 ViewModel。这些方法作为 lambda 表达式传递给 UI 元素。

**示例：关注主题**

`InterestsScreen` 接受一个名为 `followTopic` 的 lambda 表达式，该表达式由 `InterestsViewModel.followTopic` 提供。每次用户点击要关注的主题时，将调用此方法。然后，ViewModel 处理此操作，通知用户数据存储库。


## 进一步阅读

[应用程序架构指南](https://developer.android.com/topic/architecture)

[Jetpack Compose](https://developer.android.com/jetpack/compose)
