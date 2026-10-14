# E70 3.0sd (M57TU2) — підсумок сесії діагностики

Стислий підсумок для продовження роботи локально або в новому чаті.
Деталі: `no-start-research.md` (головний журнал EWS), `scan-plan-01.md`, `dde-d62m57a0-jobs.md`.

## Авто
- BMW X5 E70 2008 3.0sd, M57TU2 TOP.
- DDE: **D62M57A0** (Bosch, BMW 8509036, адреса 0x12).
- CAS: варіант **CAS** (Siemens 9237046, HW C4, адреса 0x40), **EWS3 активний**.

## Що вирішено
- Чорний дим і слабкий набір обертів: у старого EGR відпала тарілка, її знайшли у впускному колекторі. EGR замінено (Pierburg 7.00450.10.0 / 11 71 7 804 351), колектор почищено.
- Свічки розжарення (4212–4262) — відома проблема, ігноруємо.

## Поточна проблема: не заводиться після ремонту
- Тиск у рейці є (400–490 бар), але **впорскування блокує імобілайзер EWS3**.
- Помилка DDE **4A63 «EWS Manipulation – falscher Wechselcode»**: DDE отримує плаваючий код замість стартового значення.
- Провід EWS цілий: DDE отримує код від CAS.

### Встановлено
| Дія | Результат |
|---|---|
| `D_MOTOR EWS_STARTWERT 0x01` (reset DDE) | 0x00 «DDE bereit» |
| `EWS_EMPFANG` (без аргументів, телеграма `21 06`) | 0x00 одразу після reset |
| `EWS_STARTWERT 0x00` (store) | відхиляється |
| Після вимкнення запалювання | стан «готовий» втрачається, 4A63 |
| reset → CAS abgleich | `EWS_EMPFANG` 0x02 (Wechselcode замість Startwert) |
| INPA CAS «Sync. Motor ECU» | Write start value i.O., **Reset CAS i.O.**, Prepare DME n.i.O. (INPA не знає D62M57A0) |
| Tool32 `D_CAS steuergeraete_reset` | `7F 11 22` (conditions not correct) |

Ключовий висновок: між записом стартового значення і підготовкою DDE потрібен **Reset CAS**.

## Наступний крок: комбінована процедура в ОДНОМУ циклі запалювання
Підключіть донора із заведеним двигуном. Напруга має стабільно триматися **≥13 В**.
1. INPA → CAS → F6 → F2: Write start value + Reset CAS, обидва мають бути i.O.
2. Закрийте INPA.
3. Toolset32 → D_MOTOR:
   - `EWS_STARTWERT 0x01`;
   - зачекайте 5–10 с;
   - `EWS_EMPFANG` (очікуємо 0x00);
   - `EWS_STARTWERT 0x00`.
4. Вимкніть запалювання, зачекайте 30 с, пробуйте завести.

Після запуску:
- FS_LESEN DDE і CAS;
- стерти 4A63 та старі помилки;
- зареєструвати акумулятор;
- перевірити струм спокою.

## Альтернатива: ISTA+ 4.53.40 (налаштування не завершене)
- EDIABAS 7.7.1 для ISTA: `C:\Program Files\BMW\ISPI\TRIC\ISTA\Ediabas`, зараз `Interface = ENET`. Для K+DCAN потрібен STD:OBD і драйвер OBD.
- Другий EDIABAS 7.3 з `Interface = STD:passthrud` — це режим J2534. Без такого адаптера його можна ігнорувати.
- Робочий `C:\EDIABAS\BIN\EDIABAS.INI` (STD:OBD) **НЕ змінювати**. Перед будь-якою зміною робіть бекап кожного INI.
- Потрібно:
  - список файлів `...\ISTA\Ediabas\BIN\` і `C:\EDIABAS\BIN\` (OBD32.DLL / OBD64.DLL / OBD.INI);
  - скриншот ISTA VCI Config.

## Обмеження безпеки
- Не публікувати ISN і механічний код ключа.
- Не запускати:
  - `steuern_*` (isn, ews4_sk, schluessel_gesperrt, kl15_abschaltung);
  - `set_key_*`, `*_schreiben`, `flash_*`, `aif_schreiben`, `istufe_schreiben_cas`, `aktualisieren_chip_card_daten`;
  - `*_loeschen`, окрім свідомого стирання помилок.
- Не повторювати reset DDE багато разів поспіль.
- Процедури EWS виконувати лише при стабільних ≥13 В.
