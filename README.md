<p align="center">
  <img src="https://github.com/user-attachments/assets/36d67d5c-6122-4134-bacd-67b0a5c78edb?raw=true" alt="CoordinatorX"/>
</p>

![Static Badge](https://img.shields.io/badge/SPM-ready-green)
![Static Badge](https://img.shields.io/badge/demo-In_progress-0000FF)
[![GitHub Release](https://img.shields.io/github/release/edon2005/coordinatorx.svg)](https://github.com/edon2005/coordinatorx/releases)
![GitHub License](https://img.shields.io/github/license/edon2005/coordinatorx)

## 👋 Getting started

As simple as only possible. 3 Steps are:

### 1️⃣ Create an enum for steps in a particular flow.

```swift
enum AppRoute: Route {
    case splash
    case onboarding
    case login
    case home
}
```

### 2️⃣ Create Coordinator class for handling steps.

```swift
final class AppCoordinator: ViewCoordinator {

    var initialRoute: AppRoute

    init(initialRoute: AppRoute) {
        self.initialRoute = initialRoute
    }

    @MainActor
    func prepareTransition(for route: AppRoute) -> ViewTransitionType {
        switch route {
        case .splash: .root
        case .onboarding: .fullScreen
        case .login: .sheet
        case .home: .multiple(.root, .dismiss)
        }
    }

    @MainActor
    @ViewBuilder
    func prepareView(for route: AppRoute, router: any Router<AppRoute>) -> some View {
        switch route {
        case .splash:
            let viewModel = TemplateViewModel(color: .red, nextStep: .onboarding, router: router)
            TemplateView(viewModel: viewModel)

        case .onboarding:
            let coordinator = OnboardingCoordinator(initialRoute: .screen1, parentRouter: router)
            OnboardingFlow(coordinator: coordinator)
        ....
        }
    }
}
```

### 3️⃣ Create Flow structure to make Coordinator working:

```swift
struct AppFlow: DefaultViewFlow {
    var coordinator: AppCoordinator
}
```

### 🏁 Using CoordinatorX from App

```swift
@main
struct CoordinatorX_ExampleApp: App {

    private let coordinator = AppCoordinator(initialRoute: .splash)

    var body: some Scene {
        WindowGroup {
            AppFlow(coordinator: coordinator)
        }
    }
}
```

## 🔔 Important details

### What is Coordinator protocol

`var initialRoute: Route` from which route a Flow should be started.

`func prepareTransition(for route: RouteType) -> TransitionType` notify Coordinator how to show a view for route.

`func prepareView(for route: RouteType, router: any Router<RouteType>) -> some View` prepare View to be showed.
 
There are 3 types of Coordinators prepared for your app:
- `ViewCoordinator`
- `RedirectionViewCoordinator`
- `NavigationCoordinator`

**‼️** Every `fullScreen`, `overlay`, `sheet` has own Context. That means that you can call from them appearing another part of `fullScreen`, `overlay`, `sheet`. And it will be handled by the same `Coordinator`

### ViewCoordinator
It is to present a single root `View` and the root `View` can be covered with `sheet`, `fullscreen`, `overlay` or replaced by another `View`.\
`ViewCoordinator` supports next `Transition` types:
```swift
    case dismiss
    case fullScreen
    case none
    case overlay
    case root
    case set
    case sheet
```

`Transition` actions:\
`dismiss` can be applied to dismiss `fullScreen`, `overlay`, `sheet`\
`fullScreen` is applied to cover root `View` with modal `View`\
`none` just do nothing\
`overlay` is applied to cover root `View` with overlay `View`\
`root` is applied to replace root `View` with another one. It was created to be used from `fullScreen`, `overlay`, `sheet`\
`set` should be applied to replace `View` which is presented as `fullScreen`, `overlay`, `sheet`\
`sheet` is applied to cover root `View` with sheet `View`

### RedirectionViewCoordinator
It is similar to `ViewCoordinator`, but can be understood as children coordinator. It has identical `Transition` types plus one additional:
```swift
    case parent(ParentRouteType)
```
Which is used to triger action on parent flow.

### NavigationCoordinator
It is to present Navigation Flow. It has identical `Transition` types as `ViewCoordinator` plus few additional:
```swift
    case pop
    case popToRoot
    case push
```
Supposing the meaning of these `Transition` is clean to everyone 🙄

## Last but not least 

Thanks for reading till the end! 🫡
