# Роли Apple и запрос владельцу

Проверено по документации Apple 26.10.2026. Для конкретной операции сверяй её раздел Required role: общие описания ролей не заменяют точные ограничения.

## Основные роли

| Роль | Для чего подходит | Граница |
|---|---|---|
| Account Holder | Владелец membership, договоры Apple и полный доступ | Один на membership; роль передаётся отдельно |
| Admin | Все приложения, пользователи и роли, Certificates/Identifiers/Profiles, ряд деклараций | Не заменяет владельца при подписании договоров; не ограничивается одним приложением |
| App Manager | Управление доступными приложениями, версиями, IAP, TestFlight и подача на App Review | Некоторые декларации и ключи требуют Admin/владельца |
| Developer | Разработка, загрузка билдов, часть TestFlight и development signing | Не роль полного выпуска приложения |
| Finance | Финансовые операции и отчёты в рамках выданных прав | Не замена роли выпуска |
| Marketing | Метаданные и маркетинговые материалы | Не полное управление релизом |
| Sales | Продажи и аналитика; отчёты при отдельном доступе | Не signing/подача бинарника |
| Customer Support | Просмотр и ответы на отзывы | Не управление выпуском |

Это краткая карта, не полная матрица. Источники: [Account Holder](https://developer.apple.com/help/glossary/account-holder/), [Admin](https://developer.apple.com/help/glossary/admin/), [App Manager](https://developer.apple.com/help/glossary/app-manager/), [Developer](https://developer.apple.com/help/glossary/developer/), [матрица разрешений](https://developer.apple.com/help/account/access/roles/).

## Дополнительные доступы

**Certificates, Identifiers & Profiles** — отдельный допуск для App Manager/Developer в организации. Он нужен для соответствующих операций с App IDs, capabilities, сертификатами, устройствами и профилями; точные операции всё равно зависят от роли. Это не переключатель на Admin. Допуск может расширять видимость ресурсов всех приложений команды.

**Create Apps / доступ к созданию app records** проверяй отдельно, если интерфейс его предоставляет. Право загрузить билд не означает право создать новую карточку.

**Access to Reports** расширяет доступ к отчётам и может влиять на видимость приложений. Не запрашивай его для обычного выпуска без необходимости.

Вход в браузере не авторизует Xcode автоматически. Участник организации работает под своим Apple Account с выбранной Team; отдельная личная membership обычно не нужна. Не начинай личное enrollment вместо выбора команды.

## Операции, которые часто требуют владельца

| Действие | Кто может | Что передать |
|---|---|---|
| Подписать Paid Apps Agreement | Account Holder | Business → Agreements, проверить Active и актуальные условия |
| Принять договорные обновления Apple Developer Program | Account Holder | developer.apple.com/account, конкретное уведомление |
| Создать IAP server key | Account Holder или Admin | Users and Access → Integrations → In-App Purchase; безопасно передать `.p8`, Key ID, Issuer ID; ключ скачивается один раз |
| Декларация regulated medical device | Account Holder или Admin | App → App Information → App Store Regulations & Permits → Declare Regulated Medical Device |
| Изменить роль пользователя | Account Holder или Admin | Users and Access → People → пользователь → Roles → Save |

Для платного приложения/IAP проверь также банковские и налоговые статусы. Не проси присылать реквизиты в чат; точный доступ к каждой финансовой операции уточняй отдельно.

Admin удобен для команды, самостоятельно выпускающей несколько приложений, если владелец согласен на широкий доступ ко всем приложениям и управление пользователями. Альтернатива — App Manager с нужными дополнительными допусками и точечные действия владельца. Не утверждай, что Admin сможет подписывать договоры за Account Holder.

## Медицинская декларация

По текущему правилу она нужна при распространении в EU/EEA, UK или US, если приложение имеет категорию Health & Fitness/Medical либо указало частую медицинскую/лечебную информацию в возрастной анкете. Доступность формы и юридический ответ — разные вопросы. Не выбирай «Нет» только потому, что приложение является справочником, ветеринарным или не связано с внешним оборудованием. Установи фактический статус с ответственным лицом; при неопределённости не придумывай ответ. Не снимай релевантную категорию или информацию ради обхода формы.

## Что написать владельцу

Кратко перечисли готовое, конкретные недоступные операции и два варианта: выполнить их самому либо предоставить подходящую роль, пояснив её широту. Отдельно оставь договоры Account Holder. Не включай секреты и не приписывай владельцу ещё не проверенные статусы.

Источники для операций: [договоры](https://developer.apple.com/help/app-store-connect/manage-agreements/sign-and-update-agreements), [IAP keys](https://developer.apple.com/help/app-store-connect/configure-in-app-purchase-settings/generate-keys-for-in-app-purchases/), [медицинская декларация](https://developer.apple.com/help/app-store-connect/manage-app-information/declare-regulated-medical-device-status), [роли пользователей](https://developer.apple.com/help/app-store-connect/manage-your-team/add-and-edit-users), [доступ к приложениям](https://developer.apple.com/help/app-store-connect/manage-your-team/edit-access-to-apps).
