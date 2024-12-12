#

@Keep
@Route(path = "/app/main")
ViewModel：ViewModel 类设计用于以关注生命周期的方式来存储和管理与界面相关的数据。这意味着它能在 UI 控制器（如 Activities 和 Fragments）的配置变更（如屏幕旋转）期间存活，确保数据不会因这些变化而丢失。ViewModel 帮助实现 Model-View-ViewModel (MVVM) 设计模式，它将数据处理和业务逻辑从 UI 控制器中分离出来