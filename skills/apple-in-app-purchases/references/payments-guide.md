# Товары, серверная проверка и Sandbox

Проверено 26.10.2026. Ориентируйся на текущие StoreKit/App Store Server API для своего стека; это не библиотека проверки JWS и не универсальная схема базы данных.

## Магазин и ключи

Проверь активность Paid Apps Agreement и применимые банковские/налоговые требования. Договор подписывает Account Holder. Создание In-App Purchase key требует Account Holder/Admin; `.p8` скачивается один раз. Храни его приватно; в клиент и репозиторий передавай только необходимую публичную конфигурацию, сам ключ — только серверу в разрешённом сценарии.

Товары:

| Тип | Особенность |
|---|---|
| Consumable | Расходуемая единица, расход/повторную покупку учитывать в выбранной модели |
| Non-consumable | Постоянная покупка; поддерживать восстановление |
| Auto-renewable subscription | Группа и уровни подписок, период, renewal/expiry/grace |
| Non-renewing subscription | Срок и восстановление требуют собственной модели по правилам Apple |

Не создавай lifelong access как auto-renewable subscription. Не включай Family Sharing без поддержки соответствующей модели entitlement. Сверь реальные IDs в магазине, клиенте и сервере, локализации, цены/базовый регион, страны, review notes/image и состояние товара. Не публикуй приблизительную цену как гарантированную: storefront/currency/налоги могут отличаться.

Для первого IAP следуй текущей процедуре совместной подачи с app version. Факт создания товара не означает принятие его на review.

Источники: [договоры](https://developer.apple.com/help/app-store-connect/manage-agreements/sign-and-update-agreements), [ключи](https://developer.apple.com/help/app-store-connect/configure-in-app-purchase-settings/generate-keys-for-in-app-purchases/), [типы IAP](https://developer.apple.com/help/app-store-connect/reference/in-app-purchases-and-subscriptions/in-app-purchase-types), [подача IAP](https://developer.apple.com/help/app-store-connect/manage-submissions-to-app-review/submit-an-in-app-purchase/).

## Серверная проверка

Используй официальный способ проверки: например App Store Server API и/или официальную библиотеку проверки signed data. Проверь bundle/app identity, environment, product и тип, transaction/original transaction, владельца и актуальный статус. Алгоритм выбора текущего состояния subscription зависит от используемого API; старый успешный transaction не доказывает действующую подписку.

При webhook сначала проверь signed payload документированным способом либо используй его только как недоверенную подсказку для последующего авторитетного запроса Apple. Простое base64-декодирование JWS не проверяет подпись. Если данные пришли непосредственно из аутентифицированного TLS-ответа официального Apple API, не смешивай этот источник с непроверенным JWS клиента/webhook.

Свяжи покупку с пользователем до выдачи доступа, предусмотрев restore, повторную доставку, параллельную обработку и удаление аккаунта. Не переносить original transaction другому пользователю без обоснованной продуктовой модели. Временная ошибка Apple не должна выдавать Premium и не должна безвозвратно терять оплаченную транзакцию.

Перезапуск, возврат из background и restore могут использовать разные пути синхронизации. Проверь обновление серверных данных и UI, а не только ответ checkout.

Источники: [App Store Server API](https://developer.apple.com/documentation/appstoreserverapi), [appAccountToken](https://developer.apple.com/documentation/storekit/transaction/appaccounttoken), [Server Notifications](https://developer.apple.com/documentation/appstoreservernotifications), [Server Library](https://developer.apple.com/documentation/appstoreserverapi/simplifying-your-implementation-by-using-the-app-store-server-library).

## Проверки по уровням

| Проверка | Что доказывает | Чего не доказывает |
|---|---|---|
| Unit/stub tests | Конкретную логику в заданных условиях | Приём настоящей транзакции Apple |
| StoreKit Testing в Xcode | Поведение клиента в локальных сценариях | Авторизацию реального server key и настоящую server цепочку |
| Product query | Доступность данных товаров | Завершённую покупку |
| Проба ключа к Apple | Авторизацию запроса в данной среде | Фактическую выдачу доступа |
| Реальный Sandbox/TestFlight checkout | Транзакцию тестовой среды | Сам по себе — entitlement и все остальные товары |
| Profile/catalogue после checkout | Видимый доступ пользователя | Renewal/refund/manual restore без отдельных проверок |

TestFlight-покупки проходят в Sandbox без реальных списаний. Для базовой TestFlight-покупки не обязательно создавать отдельный Sandbox tester; такой аккаунт нужен для соответствующего сценария и дополнительных Sandbox controls. Следуй официальному sign-in workflow. Не проси выходить из основного Apple Account устройства вместо нужного Media & Purchases/Sandbox входа.

При проблемах Simulator не подменяй реальную Apple транзакцию локальным config. Apple DTS описывает случай получения товаров при невозможности Sandbox-покупки в Simulator; для этого сценария используй физический device. Ограничение конкретной среды не является дефектом серверной выдачи доступа и не делает устройство формальным обязательным условием подачи.

Источники: [Sandbox](https://developer.apple.com/documentation/storekit/testing-in-app-purchases-with-sandbox), [создание tester](https://developer.apple.com/help/app-store-connect/test-in-app-purchases/create-a-sandbox-apple-account/), [ответ Apple DTS по Simulator](https://developer.apple.com/forums/thread/768966).

## 401 только в production

До первого публичного релиза production API может быть недоступен при рабочем Sandbox-ключе. Это наблюдение подтверждено [Apple Staff](https://developer.apple.com/forums/thread/806452), но не универсальное объяснение любого401.

Проверь JWT/ES256, срок/часы, Key ID/Issuer, bundle/Team, endpoint и результат отдельного Sandbox-запроса. Не публикуй JWT или ключ в диагностике. Используй отсутствие известной тестовой транзакции, корректные response codes и контроль с неверной подписью только как ограниченное свидетельство авторизации, не как замену checkout.

Не добавляй общий production→Sandbox fallback по любому сбою. Для подтверждённого prerelease-случая оцени ограниченный вариант с явным разрешением Sandbox, белым списком ошибок и неизменной повторной проверкой Apple/ownership. 5xx/network failures не являются разрешением выдать доступ. Не переноси код этого частного случая в другой проект вслепую.

## Минимальный журнал проверки

Для каждого товара зафиксируй checkout success/cancel/pending, server verify, профиль/замки, сохранение после relaunch и фактический путь restore. Проверяй также чужой app account, повторную доставку и применимые subscription renewal/expiry/refund.

После переустановки сервер может автоматически вернуть уже сохранённый entitlement. Это проверка сохранности доступа, а не доказательство manual StoreKit restore. Если UI не даёт войти в restore при активном Premium, запиши ограничение вместо выдуманного PASS.

Для независимых чистых прогонов выбирай тестовые аккаунты/состояния так, чтобы не нарушать ownership. Не удаляй production-покупки и не переноси цепочки между пользователями ради удобства теста. При отсутствии устройства отчёт должен явно говорить «real purchase/restore не выполнялись»; пользователь может выбрать подачу с этим пробелом.
