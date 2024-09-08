# flutter_rustore_update

## [Документация RuStore](https://help.rustore.ru/rustore/for_developers/developer-documentation/sdk_updates/flutter)




- [flutter\_rustore\_update](#flutter_rustore_update)
  - [Документация RuStore](#документация-rustore)
    - [Общее](#общее)
    - [Пример пользовательского сценария](#пример-пользовательского-сценария)
    - [Подготовка требуемых параметров](#подготовка-требуемых-параметров)
    - [Настройка примера приложения](#настройка-примера-приложения)
    - [Условия корректной работы SDK](#условия-корректной-работы-sdk)
    - [Пример реализации](#пример-реализации)
  - [Подключение в проект](#подключение-в-проект)
  - [Проверка наличия обновлений](#проверка-наличия-обновлений)
  - [Скачивание обновления](#скачивание-обновления)
    - [Отложенное обновление](#отложенное-обновление)
    - [Принудительное обновление](#принудительное-обновление)
    - [Тихое обновление](#тихое-обновление)
  - [Установка обновления](#установка-обновления)
    - [Гибкое завершение обновления](#гибкое-завершение-обновления)
    - [Тихое завершение обновления](#тихое-завершение-обновления)
  - [Возможные ошибки](#возможные-ошибки)

### Общее

RuStore In-app updates SDK помогает поддерживать актуальную версию вашего приложения на устройстве пользователя.

Когда пользователи поддерживают приложение в актуальном состоянии, они могут опробовать новые функции, а также воспользоваться улучшениями производительности и исправлениями ошибок.

Вы можете использовать RuStore In-app updates SDK для отображения процесса обновления приложения, который обеспечивает фоновую загрузку и установку обновления с контролем состояния. Пользователь сможет использовать ваше приложение в момент загрузки обновления.

### Пример пользовательского сценария

<img src="https://gitflic.ru/project/rustore/flutter-rustore-update/blob/raw?file=flow.png" alt="Update flow" height="400px">

### Подготовка требуемых параметров

Для запуска примера, вам нужны следующие параметры:

1. `applicationId` - - из приложения, которое вы публиковали в консоль RuStore, находится в файле build.gradle вашего проекта

  ```
    android {
       defaultConfig {
       applicationId = "ru.rustore.sdk.billingexample"
       }
    }
   ```

2. `release.keystore` - подпись, которой было подписано приложение, опубликованное в консоль RuStore.

### Настройка примера приложения

1. Для проверки работы примера вам надо загрузить в консоль 2 версси приложения, с разным versionCode. И в тестировании указать меньший versionCode, нежели есть в консоли.

```
  defaultConfig {
    versionCode 1
  }
```

2. Замените `applicationId` в файле example/android/app/build.gradle, на applicationId apk-файла, который вы публиковали в консоль RuStore:

```
android {
  defaultConfig {
    applicationId = "ru.rustore.sdk.updateexample" // Зачастую в buildTypes приписывается .debug
  }
}
```

3. Замените подпись на подпись вашего приложения. Настройте параметры `key_alias`, `key_password`, `store_password`

```
android{
  signingConfigs {
        release {
            keyAlias keystoreProperties['keyAlias']
            keyPassword keystoreProperties['keyPassword']
            storeFile keystoreProperties['storeFile'] ? file(keystoreProperties['storeFile']) : null
            storePassword keystoreProperties['storePassword']
        }
    }
}
```

### Условия корректной работы SDK

Для работы RuStore In-app updates SDK необходимо соблюдение следующих условий:

- ОС Android версии 7.0 или выше.
- На устройстве пользователя должен быть установлен RuStore.
- Версия RuStoreApp на устройстве пользователя должна быть актуальной.  
- Приложению RuStore должна быть разрешена установка приложений.

### Пример реализации

Для того, чтобы узнать как правильно интегрировать пакет для работы с push-уведомлениями, рекомендуется ознакомиться с приложением-примером

[https://gitflic.ru/project/rustore/flutter-rustore-update](https://gitflic.ru/project/rustore/flutter-rustore-update)

## Подключение в проект

Для подключения пакета к проекту нужно выполнить команду

```sh
flutter pub add flutter_rustore_update
```

Эта команда добавит строчку в файл pubspec.yaml

```yml
dependencies:
    flutter_rustore_update: ^6.1.0
```

## Проверка наличия обновлений

Прежде чем запрашивать обновление, проверьте, доступно ли обновление для вашего приложения. Для проверки наличия обновлений вызовите метод info(). При вызове данного метода проверяются следующие условия:

- На устройстве пользователя должен быть установлен RuStore.
- Версия RuStoreApp на устройстве пользователя должна быть актуальной.  
- Пользователь и приложение не должны быть заблокированы в RuStore.  

В ответ на данный метод вы получите объект info, который будет содержать в себе информацию о необходимости обновления.

```dart
RustoreUpdateClient.info().then((info) {
    print(info);
}).catchError((err) {
    print(err);
});
```

Объект info содержит набор параметров, необходимых для определения доступности обновления:

- updateAvailability - доступность обновления:

- UPDATE_AILABILITY_NOT_AVAILABLE - обновление не нужно.
- UPDATE_AILABILITY_AVAILABLE - обновление требуется загрузить или обновление уже загружено на устройство пользователя.
- UPDATE_AILABILITY_IN_PROGRESS - обновление уже скачивается или установка уже запущена.
- UPDATE_AILABILITY_UNKNOWN - статус по умолчанию.

- installStatus - статус установки обновления, если пользователь уже устанавливает обновление в текущий момент времени:

- INSTALL_STATUS_DOWNLOADED - скачано.
- INSTALL_STATUS_DOWNLOADING - скачивается.
- INSTALL_STATUS_FAILED - ошибка.
- INSTALL_STATUS_INSTALLING - устанавливается.
- INSTALL_STATUS_PENDING - в ожидании.
- INSTALL_STATUS_UNKNOWN - по умолчанию.

Запуск скачивания обновления возможен только в том случае, если поле updateAvailability содержит значение UPDATE_AILABILITY_AVAILABLE.

Метод может вернуть ошибку, детально со списком ошибок можно ознакомиться в разделе ***Возможные ошибки**.

## Скачивание обновления

После подтверждения доступности обновления вы можете запросить у пользователя скачивание обновления, но перед этим необходимо запустить слушатель статуса скачивания обновления, используя метод listener()

```dart
RustoreUpdateClient.listener((value) {
  print("listener installStatus ${value.installStatus}");
  print("listener bytesDownloaded ${value.bytesDownloaded}");
  print("listener totalBytesToDownload ${value.totalBytesToDownload}");
  print("listener installErrorCode ${value.installErrorCode}");
 
  if (value.installStatus == INSTALL_STATUS_DOWNLOADED) {
    // тут можно вызывать метод complete()
  }
});
```

Объект state описывает текущий статус скачивания обновления. Объект содержит:

- installStatus - статус установки обновления, если пользователь уже устанавливает обновление в текущий момент времени:

- INSTALL_STATUS_DOWNLOADED - скачано.

- INSTALL_STATUS_DOWNLOADING - скачивается.
- INSTALL_STATUS_FAILED - ошибка.
- INSTALL_STATUS_INSTALLING - устанавливается.
- INSTALL_STATUS_PENDING - в ожидании.
- INSTALL_STATUS_UNKNOWN - по умолчанию.

- bytesDownloaded - количество загруженных байт.

- totalBytesToDownload - общее количество байт, которое необходимо скачать.
- installErrorCode - код ошибки во время скачивания. Детальнее с возможными ошибками можно ознакомиться в разделе **Возможные ошибки**.

### Отложенное обновление

Скачивание с UI от RuStore

Для запуска скачивания обновления приложения вызовите метод download().

```dart
RustoreUpdateClient.download().then((value) {
  print("download code ${value.code}");
}).catchError((err) {
  print("download err ${err}");
});
```

Если пользователь подтвердил скачивание обновления, то value.code = ACTIVITY_RESULT_OK, если отказался, то value.code = ACTIVITY_RESULT_CANCELED.

После вызова метода вы можете следить за статусом скачивания обновления в слушателе. Если в слушателе вы получили статус INSTALL_STATUS_DOWNLOADED, то вы можете вызвать метод установки обновления complete(). Рекомендуем уведомить пользователя о готовности установки обновления.

Метод может вернуть ошибку. Детальнее со списком ошибок можно ознакомиться в разделе **Возможные ошибки**.

### Принудительное обновление

Скачивание с UI от RuStore

Для запуска скачивания принудительного обновления приложения вызовите метод `immediate()`.

```js
RustoreUpdateClient.immediate().then((value) {
  print("silent code ${value.code}");
}).catchError((err) {
  print("immediate err ${err}");
});
```

`resultCode (Int)`:

- `ACTIVITY_RESULT_OK (-1)` — обновление выполнено, код может не быть получен, т. к. приложение в момент обновления завершается.
- `ACTIVITY_RESULT_CANCELED (0)` — флоу прервано пользователем, или произошла ошибка. Предполагается, что при получении этого кода следует завершить работу приложения.
- `ACTIVITY_RESULT_NOT_FOUND (2)` — RuStore не установлен, либо установлена версия, которая не поддерживает принудительное обновление (`RuStore versionCode` < `191`).

`throwable` — ошибка старта сценария обновления.

При успешном обновлении дальнейших действий не требуется.

### Тихое обновление

Скачивание без UI от RuStore

Для данного типа обновления рекомендуется реализовать свой интерфейс.

Для запуска скачивания тихого обновления приложения вызовите метод `silent()`.

```js
RustoreUpdateClient.silent().then((value) {
  print("silent code ${value.code}");
}).catchError((err) {
  print("silent err ${err}");
});
```

При вызове `then` с `code = ACTIVITY_RESULT_OK` будет зарегистрирована задача на скачивание обновления.

В данном сценарии может быть вызван только `then` с `ACTIVITY_RESULT_OK`, либо `catchError`.

После вызова метода вы можете следить за статусом скачивания обновления в слушателе.

После получения статуса `INSTALL_STATUS_DOWNLOADED` вы можете вызвать метод установки обновления. Рекомендуется уведомить пользователя о готовности обновления к установке.

## Установка обновления

### Гибкое завершение обновления

Обновление с UI от RuStore:

После завершения скачивания apk-файла обновления вы можете запустить установку обновления. Для запуска установки обновления вызовите метод completeUpdateFlexible().

```dart
RustoreUpdateClient.completeUpdateFlexible().catchError((err) {
  print("completeUpdateFlexible err ${err}");
});
```

1.Пользователю будет показан UI-диалог завершения обновления.

2.В случае успешного обновления приложение будет перезапущено.

Обновление происходит через нативный инструмент android. В случае успешного обновления приложение перезапустится.

### Тихое завершение обновления

Обновление без UI от RuStore:

После завершения скачивания apk-файла обновления вы можете запустить установку обновления. Для запуска установки обновления вызовите метод completeUpdateSilent().

```dart
RustoreUpdateClient.completeUpdateSilent().catchError((err) {
  print("completeUpdateSilent err ${err}");
});
```

1.UI-диалог завершения обновления не будет показан.

2.В случае успешного обновления приложение будет закрыто.

Обновление происходит через нативный инструмент android. В случае успешного обновления приложение будет закрыто.

На этапе обновления могут возникнуть ошибки. Детальнее с ними можно ознакомиться в разделе **Возможные ошибки**.

## Возможные ошибки

Если вы получили в ответ onFailure, то не рекомендуем самостоятельно отображать ошибку пользователю. Отображение ошибки может негативно повлиять на пользовательский опыт.

Список возможных ошибок:

- UPDATE_ERROR_DOWNLOAD - Ошибка при скачивании.
- UPDATE_ERROR_BLOCKED - Установка заблокированна системой.
- UPDATE_ERROR_INVALID_APK - Некорректный APK обновления.
- UPDATE_ERROR_CONFLICT - Конфликт с текущей версией приложения.
- UPDATE_ERROR_STORAGE - Недостаточно памяти на устройстве.
- UPDATE_ERROR_INCOMPATIBLE - Несовместимо с устройством.
- UPDATE_ERROR_APP_NOT_OWNED - Приложение не куплено.
- UPDATE_ERROR_INTERNAL_ERROR - Внутренняя ошибка.
- UPDATE_ERROR_ABORTED - Пользователь отказался от установки обновления.
- UPDATE_ERROR_APK_NOT_FOUND - apk для запуска установки не найден.
- UPDATE_ERROR_EXTERNAL_SOURCE_DENIED - Запуск обновления запрещён. Например, в первом методе вернулся ответ о том, что обновление недоступно, но пользователь вызывает второй метод.
