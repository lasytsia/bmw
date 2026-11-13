# Android-магнітола 8259 для BMW E60/E61/E63/E90 (CCC)

## Покупка
- AliExpress, грудень 2021, 1569,39 zł, відправка з Польщі.
- Лот: «8 Core 128GB Android 11 Car Radio … BMW 5/3 E60 E61 E62 E63 E90 E91 CIC CCC», варіант «8 Core 128 ROM CCC».

## Версії (Про пристрій)
| Поле | Значення | Що це |
|---|---|---|
| HMI | `BMW.G5.D.Q.F.01.2021.11.29.15.00` | оболонка/лаунчер (`com.ts.MainUI`), збірка 29.11.2021 |
| MCU | `TsBMW.211019` | мікроконтролер BMW-інтеграції, збірка 19.10.2021 |
| MEDIA | `HC_v2.0.0_210506_1600(v1.84)` | медіа/системна частина |
| CAN | `YX-BMW-GD-2 V210(L077.B0DD.40VC01.1001,001)` | CAN-протокол (файл `can_app.bin`) |
| BTV | `BT.21.11.25.1655(02:11:20:76:55:20)` | Bluetooth-модуль, MAC у дужках |

Після оновлення CAN показувало `YX-BMW-GD-2 V210(L077.9000.C00000.0001,001)`: версія та сама (V210), у дужках інші параметри.

## Платформа
Майже такий самий блок (той самий MEDIA `HC_v2.0.0_210506_1600(v1.84)`, HMI `BMW.G5.D.Q.F…`, MCU `TsBMW`) розібрано тут:
https://github.com/PanduruIonut/bmw-e90-android-headunit-825x
- `ro.product.model = 825X_Pro`, чип AutoChips **AC8257/8259** (8 ядер).
- Насправді **Android 9** (SDK 28, збірка PPR1), хоч і показує «11».

## Заводські коди (вводяться натисканням на рядок HMI)
- `8878`: повне заводське меню (CAN, Radio, Display, камера…).
- `8888`: коротке меню.
- **Не вводити:** `9090`, `9191` (видаляють ID), `0`, `0128`, `4`, `5`, `180`, `1701` (оновлення MCU/CAN, скидання, калібрування).

## Оновлення CAN (інструкція продавця)
1. Відформатована флешка або SD.
2. Розпакувати архів і покласти `can_app.bin` у корінь.
3. «Інформація про авто» → довге натискання на функцію → «Protocol upgrade» → підтвердити. Магнітола перезапуститься.

Порядок, якщо оновлювати все: система → CAN → MCU.

## Прошивки: що є
- Офіційного публічного архіву для `TsBMW` / `YX-BMW-GD-2` немає. Файли дає продавець або виробник.
- Інструкція продавця згадує `YX-BMW-2 V201`, а в магнітолі вже стоїть **`YX-BMW-GD-2 V210`**. Це новіша версія іншого варіанту (GD), тож V201 НЕ ставити.
- https://github.com/chinesebmwheadunits/mcu — це прошивки для інших платформ (ZXW/KSW, HW7/HW8), **не для TsBMW**.
- Обговорення та файли від користувачів на XDA (неперевірені):
  - https://xdaforums.com/t/firmware-mcu-tsbmw-230627-w-android-13-for-bmw-e60-ccc-unit.4775514/
  - https://xdaforums.com/t/i-need-firmware-for-android-10-bmw-e60-ccc-unit.4375605/
  - https://xdaforums.com/t/the-chinese-unit-8259-needs-a-firmware-with-android-auto-support.4480791/
  - https://xdaforums.com/t/help-825x_pro-bmw-1920x720-black-screen-after-boot-logo-need-bootlogo-bin-firmware.4783167/

## Ризики
- Прошивка під іншу роздільність або авто може дати чорний екран або прибрати оригінальне меню CCC.
- Невідповідний MCU або CAN може зламати iDrive-джойстик, кнопки керма та задній хід.
- Перед змінами сфотографувати всі екрани заводського меню.

## Чи є прошивка, яку можна доопрацювати
Публічного повного образу саме під цю збірку (`HMI BMW.G5…2021.11.29` + `TsBMW.211019` + `HC_v2.0.0_210506`) не знайдено.

Схожі пакети є на XDA (сам я їх не відкривав, бачив лише в пошуку):
- Tmotec 825X_Pro (ALPS/Topway, той самий `HC_v2.0.0_210506`): https://xdaforums.com/t/bmw-f10-f11-cic-tmotec-825x_pro-firmware-software-mcu-update-alps.4697315/
- Junsun V1 Pro, у темі згадуються `TsBMW.211019` і `TsBMW.220511`: https://xdaforums.com/t/junsun-v1-pro-mtk8259-mtk8257-firmware.4283365/page-127
- AC8257, Android 11 + бездротовий Android Auto: https://xdaforums.com/t/ac8257-base-unit-new-firmware-mcu-update-with-android-11-android-auto-wireless.4467385/

**Найкраща основа: власний дамп із магнітоли.** Аналогічний блок має збірку `eng`/`test-keys` з SELinux у режимі permissive, тобто:
- `adb root` має працювати без жодного злому;
- системні APK можна переписати й підписати публічними ключами AOSP.

### Крок 1. Перевірка через ADB по Wi‑Fi
1. Налаштування → Про пристрій → 7 разів натиснути «Номер збірки» → Для розробників → USB debugging.
2. Вимкнути бездротові CarPlay/AutoLink, бо інакше магнітола не підключається до Wi‑Fi. Під'єднати магнітолу й ноутбук до однієї мережі (підійде точка доступу з телефона).
3. На ноутбуці:
```
adb connect <IP>:5555
adb root
adb shell getprop ro.build.type
adb shell getprop ro.build.version.sdk
adb shell getprop ro.build.tags
adb shell ls -l /dev/block/by-name/
adb shell cat /proc/partitions
```

### Крок 2. Дамп розділів (лише читання)
```
adb shell 'for p in boot recovery system vendor lk lk2 tee1 dtbo vbmeta preloader; do [ -e /dev/block/by-name/$p ] && dd if=/dev/block/by-name/$p of=/sdcard/$p.img bs=4M; done'
adb pull /sdcard/*.img .
```
Плюс окремо скопіювати `metazone`/config-розділ, а також `/system/app` і `/system/priv-app` (там `com.ts.MainUI`).

### Що реально можна змінити
- Можна: лаунчер та UI (`com.ts.MainUI`), `build.prop`, root через Magisk (пропатчений `boot.img`), вбудовані застосунки, налаштування камери.
- Практично ні: MCU (`TsBMW`) і CAN (`can_app.bin`). Це закриті бінарники без вихідних кодів; лише заміна на інші готові версії.

### Аварійне відновлення
SP Flash Tool з readback eMMC, як описано для AC8259 тут: https://github.com/initialChris/junsun-v1pro-recovery
Scatter-файл звідти чужий, тож без повного власного бекапу нічого не форматувати.
