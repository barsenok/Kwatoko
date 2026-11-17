# kwatoko — Android

Порт iOS-приложения [kwatoko](../README.md) на Android. Kotlin + Jetpack Compose, MVVM, Room, корутины/Flow.

Торговые роботы со стратегией «торговля по стакану» через [Tinkoff Invest API](https://github.com/Tinkoff/invest-api-java-sdk) (gRPC).

## Структура проекта

```
android/
├── build.gradle.kts            # root
├── settings.gradle.kts
├── gradle.properties
└── app/
    ├── build.gradle.kts        # зависимости: compose, room, grpc-okhttp, moshi, datastore
    └── src/main/
        ├── AndroidManifest.xml
        ├── java/ru/kwatoko/android/
        │   ├── KwatokoApplication.kt      # аналог AppDelegate/Core (DI-контейнер)
        │   ├── MainActivity.kt            # аналог SceneDelegate + TabBarController
        │   ├── core/                      # Sdk (аналог TinkoffInvestSDK), QuotationExt, DemoDataLoader
        │   ├── models/                    # Deal, Robot, Strategy, Profile, Instrument…
        │   ├── db/                        # Room: DAO + DatabaseService (аналог CoreData)
        │   ├── network/                   # NetworkService: gRPC-клиенты (Users, Orders, Sandbox…)
        │   ├── strategy/                  # ContestStrategy — перенос алгоритма 1-в-1
        │   └── ui/                        # robots / addrobot / orderbook / instruments /
        │                                  # strategies / history / deal / profile / theme
        └── res/
            ├── values/strings.xml         # переводы en (по умолчанию)
            ├── values-ru/strings.xml
            ├── values-ar/strings.xml
            └── raw/demo_data.json         # тестовые данные стакана (DemoData.json)
```

## Сборка

```bash
cd android
ANDROID_SDK_ROOT=/path/to/sdk ./gradlew assembleDebug
```

## Отличия от iOS-версии

| iOS | Android |
|-----|---------|
| Swift, VIPER (Ramba templates) | Kotlin, MVVM + StateFlow |
| CoreData (`DatabaseService.swift`) | Room |
| Combine + CombineGRPC (`AnyPublisher`) | Kotlin Flow / suspend функции |
| TinkoffInvestSDK (Swift gRPC) | `ru.tinkoff:fintech-invest-api` (gRPC-Java) |
| UIKit (UITableView/UICollectionView) | Jetpack Compose (LazyColumn) |
| SwiftGen (L10n) | Android resources (values/values-ru/values-ar) |
| Storyboard launch screen | Material Theme + splash |

Логика торгового алгоритма (`ContestStrategy`: состояния pending → preOpenOrderPosted →
openOrderPosted → openOrderExecuted → preCloseOrderPosted → closeOrderPosted, перезаказ
заявки при появлении более актуальной аномальной заявки с `reopenOffsetLimit = 4`,
стоп-лосс/тейк-профит) перенесена без изменений.

⚠️ Торговля на реальной бирже: используйте read-only токены, если не собираетесь торговать.
Здорово оценивайте риски при запуске робота для торговли на реальной бирже.
