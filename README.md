# RedPandaAndroid для Pixel 9a (tegu)

Ветка `red_panda_16` сохранена для совместимости с существующим форком, но теперь
основана на **GrapheneOS 2026091900 / Android 17**. Это версия стабильного канала
`tegu` на момент проверки 27 сентября 2026 года. Подпись тега upstream проверена.

## Состав

`default.xml` закрепляет исходники релиза по commit SHA. Отличия от upstream:

- `build/make`: включение OpenEUICC в продукт;
- `packages/apps/Updater`: сервер RedPanda и сравнение времени сборки, поддерживающее номера `TIMON_Z…`;
- OpenEUICC и его готовые зависимости, также закреплённые по SHA;
- `sync-s="true"` для OpenEUICC: загрузка обязательных подмодулей LPAC и cJSON.

Updater сохраняет существующий адрес `http://91.238.248.89:6066/` и разрешение
HTTP из прежнего форка. Доступность и содержимое сервера при переносе не проверялись.

`config.yml` — конфигурация генератора upstream. Она сама по себе не содержит
перечисленные изменения RedPanda. При следующем обновлении нужно взять манифест
нового стабильного тега, перенести изменения двух форков и повторно добавить
закреплённые проекты OpenEUICC. Простая генерация через `adevtool generate-manifest`
заменит манифест и уберёт эти изменения.

## Синхронизация и сборка

Перед синхронизацией коммиты `platform_build` и `platform_packages_apps_Updater`,
указанные в `default.xml`, должны быть опубликованы в соответствующих репозиториях
BearIvan. Для приватных репозиториев нужен доступ Git по HTTPS.

В отдельном каталоге исходников, после установки зависимостей из
[официальной инструкции](https://grapheneos.org/build):

```bash
repo init -u https://github.com/BearIvan/RedPandaAndroid.git -b red_panda_16
repo sync -c -j8
source build/envsetup.sh
yarn --cwd vendor/adevtool/ install
adevtool generate-all -d tegu
lunch tegu-cur-user
export OFFICIAL_BUILD=true
export BUILD_NUMBER=TIMON_Z$(date -u +%Y%m%d)00
export BUILD_DATETIME=$(date -u +%s)
m -j16 target-files-package otatools-package
```

`OFFICIAL_BUILD=true` включает Updater с адресом RedPanda из этого форка.
После успешной сборки используйте **существующие ключи подписи** для `tegu`,
сохранённые отдельно, и выполните:

```bash
script/finalize.sh
script/generate-release.sh tegu "$(cat out/build_number.txt)"
```

До подписания необходимо проверить полный комплект ключей и номер установленной
на телефоне сборки. Ключи и пароли не должны попадать в Git. Образ ОС ещё не собран;
обновление манифеста и перенос изменений не подтверждают успешную компиляцию или
возможность установки на конкретную предыдущую сборку без сброса данных.
