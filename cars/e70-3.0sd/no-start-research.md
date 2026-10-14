# E70 3.0sd: стартер крутить, не заводиться — дослідження форумів (10.10.2026)

## Стан на момент дослідження
- Стартер крутить (~155 об/хв з прикурюванням), диму немає, «не лапає».
- Тиск у рейці при прокручуванні 400–490 бар (заданий ~400–414) → паливо до рейки доходить.
- Помилок DDE, крім свічок розжарення, немає. Роз'єми перевірені власником.
- АКБ у глибокому розряді; використовувались пускові пристрої і зарядка в авто.
- Перед незапуском: заміна EGR, зняття і чистка впускного колектора (лише повітряна частина).

## Чи може імобілайзер блокувати лише впорскування?
**Так.** За описами на форумах (не офіційна документація BMW):
- CAS розпізнає ключ і дозволяє стартер → стартер крутить.
- DDE окремо перевіряє закодований обмін з CAS (EWS4). Якщо відповідь невірна / синхронізація збита → **DDE не дає впорскування** (а іноді й не керує ТНВД).
- Типова ознака: помилка «EWS manipulation» у DDE, **але вона з'являється не завжди**.
- **Тривалий низький заряд АКБ / глибокий розряд — описана причина втрати синхронізації EWS між модулями.**
- Рішення з форумів: синхронізація DDE↔CAS (EWS) у INPA / ISTA (Rheingold). Деякі описують: після синхронізації вийняти ключ, зачекати 20–30 с, або закрити/відкрити авто брелоком.

## Усі причини «тиск є, впорскування немає» з форумів
1. **Синхронізація EWS DDE↔CAS** після розряду АКБ / пускових пристроїв.
2. **Датчик розподільного вала** — при зніманні впускного колектора за процедурою BMW його від'єднують; не підключений / не на місці → немає синхронізації циліндрів.
3. **Датчик колінвала** — не на місці, перетиснутий провід (у нас оберти видно → менш імовірно).
4. **Заслінка / anti-shudder valve, що тримається закритою** → двигун не заводиться (у нас раніше 483D!). Перевірити, що відкрита після збирання.
5. **Контур тиску в рейці (регулятор + датчик)**: якщо DDE бачить неправдоподібний зворотний зв'язок, він не керує ТНВД/форсунками, **навіть без помилки**.
6. **Повітря в паливі** після робіт → прокачати, довше прокручувати.
7. **Тиск при прокручуванні < ~250 бар** → не запуститься (у нас 400–490, ок).
8. **Живлення/маса форсунок, кола керування в DDE** (пошкодження стрибком напруги) → перевірка контрольною лампою / осцилографом на роз'ємі форсунки при прокручуванні.
9. **Розряджена АКБ**: модулі «голодують», DDE не стартує коректно навіть при прокручуванні.
10. Реле клеми 50 у CAS (після прикурювання) — але тоді не крутить стартер, у нас крутить → малоймовірно.

## План (від простого)
1. Повністю зарядити ВЛАСНУ АКБ (або поставити справну) — прибрати фактор «голодних» модулів.
2. Датчик розподільного вала: роз'єм і посадка (знімався разом з колектором?).
3. Заслінка: відкрита після збирання.
4. FS_LESEN у D_MOTOR і D_CAS одразу після прокручування (шукати EWS / Nockenwelle / Startfreigabe / Klemme 15).
5. Якщо EWS → синхронізація DDE↔CAS у ISTA/INPA (робити лише з відповідним софтом і стабільним живленням).
6. Якщо не EWS → перевірка сигналу керування форсунками при прокручуванні (осцилограф / діодний пробник).

## Інструменти власника
- INPA + EDIABAS (+ Toolset32, Delphi DS150E). ISTA немає.
- План для EWS: спершу в INPA лише ЧИТАТИ статус EWS у CAS і DDE (скриншоти), вирішувати щодо синхронізації після аналізу. Перед будь-якою синхронізацією: АКБ повністю заряджена + зарядний пристрій ~13.5 В на весь час.

## Як імобілайзер блокує впорскування (пояснення)
- Ключ (транспондер) → кільцева антена → **CAS**: ключ свій → дозвіл на запалювання і стартер.
- CAS → **DDE** (по шині): закодований код дозволу. DDE порівнює з тим, що зберігає.
- Код вірний → впорскування дозволено. Невірний / немає → **DDE блокує впорскування**, хоча стартер крутить, а DDE живий (читає датчики, може тримати тиск у рейці).
- EWS3 (плаваючий код, змінюється на кожному пуску) — може «розсинхронізуватись», якщо одна сторона зберегла новий код, а інша ні (просідання напруги при прокручуванні, перезапуск DDE на ~6.5 В). EWS4 (запит-відповідь із секретним ключем) — стійкіший. Яка версія на цьому авто — не підтверджено (у IDENT DDE: ID_EWS_SS = 3).
- Ознаки EWS: стартер крутить, тиск є, імпульсів на форсунках НЕМАЄ, диму немає; помилка EWS у DDE/CAS — не завжди.

## Синхронізація CAS ↔ DDE (EWS) — процедура з джерел (неофіційно, перевіряти Job-Info)
Передумови: інші причини виключені (датчик розподільного вала, заслінка, АКБ); АКБ повністю заряджена + зарядний пристрій ~13.5 В на весь час; ключ у авто; спершу FS_LESEN у D_CAS і D_MOTOR.
- **Спосіб A (Tool32/Toolset32)**: SGBD блоку CAS (груповий `D_CAS`), job **`dme_startwert_abgleich`** (за diagnose-blog). Спершу прочитати Job-Info (опис), потім запуск, потім запалювання вимк., вийняти ключ, 20–30 с, запуск.
- **Спосіб B (INPA)**: двигун → Ansteuern (керування) → **DME<->EWS-Abgleich** → «EWS Startwert zurücksetzen», потім «EWS Startwert synchronisieren».
- ⚠️ Скидання стирає поточне значення; якщо синхронізація після нього не вдасться — авто не заведеться, поки не зробити правильно (ISTA/спеціаліст). Не робити, якщо двигун заводиться.
- ⚠️ Працює лише якщо ISN у CAS і DDE збігаються; помилка «невірне стартове значення» → можливо, різні I-level CAS/DDE → ISTA/спеціаліст.
- На форумі E70: код CAS **2F44 (EWS alignment)** + «крутить, не заводиться».

## D_CAS, 13.10.2026
- IDENT: варіант **CAS** (Siemens), BMW 9237046, HW C4, дата 23.06.2008, адреса 0x40, **ID_EWS_SS = 3** (як і в DDE → тип інтерфейсу EWS однаковий).
- FS_LESEN: **1 помилка — A0B2 Fehler_CAS_Versorgung** (живлення CAS), «momentan nicht vorhanden» → історична, від глибокого розряду АКБ.
- **Помилок EWS / синхронізації з DDE у CAS немає** → версія EWS слабшає (але не виключена: розсинхронізація пишеться не завжди).
- Далі: список job-ів CAS (шукати status/EWS/startwert/abgleich, Job-Info перед запуском), паралельно — маса двигуна / живлення DDE, зворотка форсунок, джгут, осцилограф.

### Job-и D_CAS (частина списку)
- Читання (безпечно): `status_aktuell_schluessel`, `status_dme_ringbuffer`, `status_ews4_sk`, `status_cas_diagnose`, `status_fertig_transp_werkst`, `status_elv`, `fs_lesen_detail`, `is_lesen`.
- Синхронізація: **`dme_startwert_abgleich`** (є!) — лише після аналізу Job-Info.
- НЕ запускати: `set_key_init`, `set_key_mode`, `*_schreiben`, `flash_*`, `steuergeraete_reset`, `aif_schreiben`, `aktualisieren_chip_card_daten`, `*_loeschen`.
- Ще статуси CAS (читання): `status_ews`, `status_schluessel_frei_gesperrt`, `status_klemmenverhinderer`, `status_kl15_abschaltung`, `status_fzg_zustand`, `status_wakeup_history`, `istufe_lesen_cas`.
- `status_isn` — секретний номер імобілайзера: не публікувати.
- НЕ запускати жоден `steuern_*` (особливо `steuern_isn`, `steuern_ews4_sk`, `steuern_schluessel_gesperrt`, `steuern_kl15_abschaltung`) та `istufe_schreiben_cas`.

### status_ews (D_CAS), 13.10.2026
| Параметр | Значення |
|---|---|
| EWS3_CAPABLE / EWS4_CAPABLE | 1 / 1 |
| **EWS3_ACTIVE** | **1** |
| **EWS4_ACTIVE** | **0** |
| EWS_EGS_ACTIVE | 1 (імобілайзер також на коробку EGS) |
| EWS4_SERVER_KEY_TXT | WAIT_FSC_UNLOCKED (EWS4 не налаштований — норма для EWS3) |

**Висновок**: на авто працює **EWS3 — плаваючий код** (змінюється на кожному пуску), саме той тип, що може розсинхронізуватись при просіданні напруги під час прокручування. Версія EWS посилюється. Синхронізація EWS3: CAS `dme_startwert_abgleich` або INPA DDE «EWS Startwert zurücksetzen / synchronisieren» (в DDE є job `ews_startwert`). Перед запуском — Job-Info обох.

### Крок 1 перед синхронізацією (13.10.2026 ~11:30) — стан збережено
- CAS FS_LESEN: A0B2 (живлення CAS, історична) + **НОВА: A126 Unplausibilitaet_ACK_KL15_Off — «Signal oder Wert unterhalb Schwelle», АКТИВНА** (неправдоподібне підтвердження вимкнення клеми 15, сигнал нижче порогу).
- status_ews: без змін (EWS3 активний).
- istufe_lesen_cas: сирі байти `0B 15 E1 3C CB 2C D4 34` (текст не декодовано).
- DDE: ті самі помилки (живлення/MAF — «bisher nicht aufgetreten», свічки), ews_empfang = 5.
- Далі: fs_lesen_detail 0xA126 у CAS; Job-Info `dme_startwert_abgleich`.
- **Job-Info `dme_startwert_abgleich` (CAS)**: «Kopiert bei der EWS-Stufe 3 die I… und startet dann die Initialisierung der DME»; Vorbedingungen: keine; KWP2000 $31 StartRoutineByLocalIdentifier $20; без аргументів; результат лише JOB_STATUS. → Це саме синхронізація EWS3 CAS→DDE. Текст обрізано праворуч — дочитати.
- fs_lesen_detail 0xA126: F_HFK = 3, 292 600 км, умов не записано → імовірно наслідок низької напруги.
- **status_ubatt (DDE, запалювання, без прокручування) = 10.03 В** ❌ — критично низько. **Синхронізацію НЕ робити.** Спершу АКБ: зарядити/замінити (після глибокого розряду ймовірно мертва), ціль ≥12.6 В спокою, 13–14 В з зарядним. Потім спробувати завести БЕЗ синхронізації; синхронізація — лише якщо все одно не заводиться.
- status_ubatt повторно: **12.04 В** (запалювання). Краще, але для синхронізації замало (ціль ≥12.8–13.5 В). План: (1) спроба запуску з прикурюванням від ПРАЦЮЮЧОГО донора без синхронізації; (2) якщо ні — синхронізація з донором на 1500 об/хв як стабільним джерелом (status_ubatt ≥13 В перед запуском job).

### ✅ Синхронізація виконана (13.10.2026)
- Перед цим status_ubatt = 13.33 В (донор працює).
- `dme_startwert_abgleich` (D_CAS): **JOB_STATUS = OKAY**, відповідь `71 20 44` (позитивна відповідь на StartRoutine $20).
- Далі: запалювання вимк., ключ вийняти, 30 с, запуск; після — ews_empfang, FS_LESEN DDE і CAS.
- Після синхронізації + перезапуску (12:13–12:15): DDE помилки без змін; **ews_empfang усе ще 5** («Programmierung bzw. Ruecksetzen im Fahrzyklus noch nicht ausgefuehrt»); CAS: **A126 досі активна при 13.3 В** (не лише від просідання напруги).
- Гіпотеза: DDE очікує **скидання стартового значення на своєму боці в тому ж циклі запалювання** (INPA: «EWS Startwert zurücksetzen» → «synchronisieren»), лише потім копіювання з CAS спрацює. Потрібен Job-Info `ews_startwert` (D_MOTOR). Паралельно: A126 — перевірити реле/живлення клеми 15.
- 4BC2 (MAF) знову «momentan vorhanden» — перевірити, що роз'єм MAF підключений.
- Job-Info `ews_startwert` (D_MOTOR): коментар «EWS-Empfangsstatus auslesen» (схоже на копію), KWP2000 $21; **аргумент PARAMETER (int) «Parameter zur Initialisierung»** — значення невідомі; результати EWS_STATUS / STAT_EWS_WERT «Rueckgabestatus bei der Startwertinitialisierung». → Це ініціалізація стартового значення в DDE, але без відомих значень параметра НЕ запускати. Шукати таблицю значень у Tabellen-Info або робити через INPA (скрипт знає параметри).
- INPA 5.0.6 (Auswahl E70 V0.08) → Engine → «DDE 6 for M57TOP/M57TU2»: **«Requested control unit D60M57D0, D62M57B0 not found. Found D62M57A0. Program will be stopped»** → скрипт INPA не підтримує варіант D62M57A0. Варіанти: новіші скрипти INPA; таблиці SGBD у Toolset32 (значення параметра ews_startwert); ISTA.
- Форум (bmw-syndikat): ews_startwert запускали в Tool32 з аргументом, після чого синхронізація в INPA допомогла; типовий статус «no free start value». Точних значень параметра не отримано (сторінки недоступні).

### ⭐ ews_startwert (D_MOTOR, порожній аргумент), 13.10.2026
- Запит `31 83 00` (StartRoutineByLocalIdentifier 0x83, параметр 0), відповідь `71 83 01`.
- **EWS_STATUS = «kein freier Startwert mit Freigabe vorhanden»**, STAT_EWS_WERT = 1.
- **Висновок: DDE підтверджує, що НЕ має дійсного стартового значення з дозволом → імобілайзер EWS3 блокує запуск/впорскування. Причину незапуску знайдено.**
- Синхронізація з CAS (dme_startwert_abgleich OKAY) не допомогла → потрібне скидання/ініціалізація стартового значення в DDE (інший параметр ews_startwert) у тому ж циклі, потім CAS sync. Значення параметра — шукати в Tabellen-Info D62M57A0 (таблиця з текстом «kein freier Startwert…»).

### Таблиці SGBD D62M57A0
**EWSSTART** (результат `ews_startwert`):
| Код | Текст |
|---|---|
| 0x00 | DDE bereit, Startwert zu empfangen (DDE готовий прийняти стартове значення) |
| **0x01** | **kein freier Startwert mit Freigabe vorhanden** ← зараз |
| 0x02 | noch kein Startwert gespeichert |
| 0x03 | Startwert nicht plausibel |

**EWSEMPFANGSSTATUS** (результат `ews_empfang`):
| Код | Текст |
|---|---|
| 0x00 | Startwertprogrammierung bzw. -ruecksetzen war erfolgreich ✅ |
| 0x01 | falscher Startwert beim Ruecksetzen (EWS u. DDE passen nicht) |
| 0x02 | Telegramminhalt war kein Startwert (event. Wechselcode) |
| 0x03 | Schnittstellenfehler DWA: Frame o. Parity oder kein Signal |
| 0x04 | Prozess laeuft |
| **0x05** | **Programmierung bzw. Ruecksetzen im Fahrzyklus noch nicht ausgefuehrt** ← після кожного перезапуску |
| 0x06 | gleiche Zufallszahl wie bei vorherigem Ruecksetzen |
| 0x07 | noch kein Startwert programmiert |
| 0x10 | Startwert nicht korrekt in Flash programmiert |
| 0x11 | Wechselcode nicht korrekt in EEPROM-Spiegel programmiert |
| 0x21 | 2-aus-3-Startwertablage im Flash nicht in Ordnung |

**Висновки**: це таблиці СТАТУСІВ, не параметрів. Статус 0x05 скидається на кожен цикл запалювання → після першої синхронізації ми читали ews_empfang ПІСЛЯ перезапуску, тож результат синхронізації не побачили. Потрібно: в ОДНОМУ циклі — ews_startwert → dme_startwert_abgleich (CAS) → одразу ews_empfang (результат синхронізації) → ews_startwert.

### Послідовність в одному циклі (13.10.2026, 13.88 В)
1. ews_startwert → 0x01 «kein freier Startwert mit Freigabe vorhanden»
2. CAS dme_startwert_abgleich → OKAY (71 20 44)
3. **ews_empfang одразу → 0x05 «… noch nicht ausgefuehrt»** — DDE не отримав / не обробив програмування від CAS.
Версії: (а) DDE не в стані «готовий прийняти» (0x00) — потрібне розблокування/скидання з боку DDE, параметр невідомий; (б) лінія EWS між CAS і DDE (окремий провід EWS / «DWA-інтерфейс») — обрив/пошкодження, у т.ч. біля колектора; (в) потрібен ISTA.
4. Через 10–15 с: ews_empfang → **0x05** знову; ews_startwert → **0x01** знову.

**Підсумок програмної частини**: DDE не отримує / не приймає програмування від CAS. Далі програмно не ризикуємо (параметр розблокування DDE невідомий).
**Наступне**: (1) WDS: схема EWS CAS → DDE (провід EWS, піни, колір) → продзвонка, КЗ на масу/плюс; (2) осцилограф на проводі EWS біля DDE при ввімкненні запалювання (мають бути пакети даних); (3) огляд роз'ємів DDE і CAS; (4) якщо лінія ціла → ISTA (AOS або сервіс): тест-план EWS / синхронізація.
Підозра на пошкодження стрибком напруги (пускові пристрої): CAS має A0B2 (живлення) і A126 (KL15 ACK) — можливо, пошкоджено вихідний каскад EWS у CAS або вхід у DDE.

### Чому стартове значення не скидається (пояснення)
- Скидання в DDE відбувається ЛИШЕ коли DDE отримує від CAS телеграму скидання з правильними секретними даними → тоді «Freigabe» → новий стартовий запис.
- 0x05 = DDE за цикл НЕ отримав жодної телеграми програмування/скидання. Якби отримав з невірними даними — було б 0x01 («EWS u. DDE passen nicht»), з перешкодами — 0x03.
- Отже, телеграма не доходить або CAS відправляє її не одразу (можливо, лише на наступному KL15/старті).
- Тест: CAS dme_startwert_abgleich → запалювання вимк./ввімк. → 10 с → ews_empfang → прокрутити 3 с (ключ не вимикати) → ews_empfang. Якщо все одно 0x05 → телеграма фізично не доходить (провід EWS, вихід CAS, вхід DDE) → WDS + продзвонка / осцилограф / ISTA.
- Тест «CAS sync → запалювання вимк./ввімк. → ews_empfang → прокручування → ews_empfang»: **обидва 0x05**.
- **Висновок: телеграми EWS від CAS фізично не доходять до DDE** (ні при синхронізації, ні при старті). Це вже не «розсинхронізація», а обрив зв'язку CAS→DDE по EWS: провід/роз'єм, вихідний каскад CAS або вхід DDE (можливо, пошкоджені стрибком напруги).
- Далі: WDS (схема EWS E70: пін CAS ↔ пін DDE, колір) → продзвонка й перевірка на КЗ; осцилограф на лінії біля DDE при ввімкненні запалювання / старті; якщо лінія ціла і сигналу немає — CAS; якщо сигнал є — DDE. Фінально — ISTA / фахівець.
- Пошук пінів EWS CAS↔DDE для E70 в інтернеті: точних даних не знайдено → потрібна схема з WDS/ISTA на ПК власника. Корисні форумні згадки: CAS-коди 2E3C («EWS manipulation» після розряду АКБ, E70 CAS3+), 2E3D («Interface EWS-DDE»). У нашому CAS їх немає.

### ISTA — план
- Доступ: BMW AOS (aos.bmwgroup.com, ліцензія ISTA на години/добу для незалежних) або сервіс з ISTA. Офіційно потрібен інтерфейс BMW ICOM; K+DCAN з ISTA — лише неофіційно.
- У ISTA: ідентифікація по VIN → короткий тест → помилки CAS/DDE → тест-план EWS; сервісні функції (Vehicle access / CAS / EWS: синхронізація DME/DDE ↔ CAS) — назви пунктів перевірити на місці; схеми — вкладка схем електрообладнання (пошук CAS / EWS / DDE).
- Попросити сервіс: перевірити зв'язок EWS CAS→DDE і виконати синхронізацію; показати їм нотатки (ews_startwert 0x01, ews_empfang 0x05 при CAS-синхронізації OKAY).

### INPA → E70 → Body → Car Access System (працює; періодично IFH-0010)
- Read initialised indicator: CAS ECU = **Locked**, TRSP SW = **Activated** → норма для робочого авто (CAS ініціалізований і заблокований, ПЗ транспондера активне).
- Status RC signal: усі 00 (брелок, не стосується).
- Далі: CAN Signale → F1 Engine status, F2 Ignition/Key; F7 CAS/EWS version; F4 actual key.

### ⭐ Процедура з INPA-скрипта DDE6M57.IPO (екран «CAS-/EWS3-DDE start value synchronisation»), перевірено рядками файлу
Меню: F1 «CAS-Startwert uebertragen» (store), F2 «CAS-Startwert zuruecksetzen» (reset). Послідовність викликів:
| Крок | SGBD | Job | Аргумент |
|---|---|---|---|
| 1 | CAS | IDENT | — |
| 2 | CAS | DME_STARTWERT_ABGLEICH | — |
| 3 | DDE | EWS_STARTWERT | **0x00 = store**, **0x01 = reset** |
| 4 | DDE | (результат EWS_STATUS / STAT_EWS_WERT) | — |
| 5 | DDE | EWS_EMPFANG | **0x01** |
| 6 | DDE | (результат EWS_EMPFANGSSTATUS) | — |
Те саме в D50M57.IPO (DDE5). Ми робили лише варіант «store» (EWS_STARTWERT порожній = 0x00) → 0x01 «немає вільного стартового значення з дозволом». Варіант **reset (0x01)** ще не пробували; EWS_EMPFANG запускали без аргументу (а INPA — з 0x01).

### Форуми щодо «kein freier Startwert mit Freigabe» / скидання стартового значення
- motor-talk (E60 M47, DDE 6.0, INPA): ця сама помилка при синхронізації CAS↔DDE, авто не заводилось.
- bmw-syndikat: повторна спроба синхронізації (INPA) вдалась; найчастіша причина втрати — **низька напруга**; INPA іноді не може — тоді Tool32.
- e60-forum.de: синхронізація працює лише при достатній напрузі (~14 В).
- bimmerfest (E60 530d, «4A63 EWS Manipulation»): виправлено в INPA через скидання стартового значення CAS + reset; люди купують новий CAS, коли треба лише програмний reset.
- xoutpost E53: на деяких версіях INPA ресинхронізація некоректна → DIS/ISTA.
- Висновок: процедура «reset» — стандартне рішення; ключові умови — напруга ~13.5–14 В, можлива потреба повторити.

### ✅✅ Процедура «reset» УСПІШНА (13.39 В)
1. D_CAS IDENT → OKAY
2. D_CAS DME_STARTWERT_ABGLEICH → OKAY (71 20 44)
3. D_MOTOR EWS_STARTWERT **0x01** → запит `31 83 01`, відповідь `71 83 00`: **0x00 «DDE bereit, Startwert zu empfangen»**
4. D_MOTOR EWS_EMPFANG **0x01** → **0x00 «Startwertprogrammierung bzw. -ruecksetzen war erfolgreich»**
→ Синхронізацію EWS CAS↔DDE відновлено. Провід EWS цілий (теорію з обривом відкинуто). Далі: запалювання вимк., ключ вийняти, 30 с, запуск.

### Після reset — не завівся (14.10.2026)
- ews_startwert (0x00) → знову 0x01 «kein freier Startwert mit Freigabe»; ews_empfang → 0x05 (новий цикл).
- **НОВА помилка DDE: 4A63 «EWS Manipulation — falscher Wechselcode»** (неправильний плаваючий код при спробі запуску).
- Висновок: зв'язок CAS→DDE ПРАЦЮЄ (DDE отримав код, але неправильний). Reset лише перевів DDE у стан «готовий прийняти» (0x00), але стартове значення НЕ було перенесене → при запуску плаваючий код не збігся.
- Наступне: в ОДНОМУ циклі — спершу «reset» (F2: CAS abgleich + EWS_STARTWERT 0x01 + EWS_EMPFANG 0x01), одразу потім «store/übertragen» (F1: CAS abgleich + EWS_STARTWERT 0x00 + EWS_EMPFANG 0x01), перевірити статуси, потім ключ 30 с і запуск.
- Повтор (один цикл): EWS_STARTWERT 0x01 → 0x00 «DDE bereit»; EWS_EMPFANG 0x01 → 0x00 «erfolgreich»; EWS_STARTWERT **0x00 (store) → 0x01 «kein freier Startwert mit Freigabe»** (store відхилено навіть після reset); EWS_EMPFANG → 0x00.
- Гіпотеза: «store» через діагностику не працює; DDE в стані «готовий» має отримати стартове значення від CAS при наступному KL15/старті. При минулій спробі DDE отримав плаваючий код замість стартового значення → 4A63.
- Наступний безпечний тест: зап. вимк. 30 с → зап. ввімк. БЕЗ прокручування 10 с → ews_empfang 0x01 + ews_startwert 0x00 (що DDE отримав при KL15) → потім прокручування → ті самі job-и + FS_LESEN. Не робити reset багато разів підряд.
- Новий цикл (KL15 без прокручування і після прокручування): ews_empfang 0x05, ews_startwert(0x00) 0x01; 4A63 без змін. → Стан «готовий» після reset НЕ зберігається між циклами; CAS при KL15 стартове значення не передав.
- Спробувані порядки: (CAS abgleich → DDE reset) ✔ reset, але передачі немає; (+ store 0x00) відхилено.
- НЕ спробуваний порядок: **DDE reset (0x01) → одразу CAS dme_startwert_abgleich** (CAS передає, коли DDE уже «слухає») → ews_empfang / ews_startwert(0x00). Наступна спроба — саме так; якщо ні — ISTA.
- Порядок **DDE reset (0x01) → CAS dme_startwert_abgleich**: ews_empfang = **0x02 «Telegramminhalt war kein Startwert (event. Wechselcode)»**; ews_startwert(0x00) = 0x01.
- **Висновок**: DDE «слухає» і ОТРИМУЄ телеграму від CAS (лінія працює), але CAS надсилає плаваючий код замість стартового значення. Проблема на боці CAS: він не переходить у режим передачі стартового значення DDE. → Шукати в INPA CAS (F6 Activate) функцію CAS-сторони для EWS/DME (лише фото, не натискати) або ISTA. Подальші reset DDE не повторювати.
- INPA CAS → F6 Activate: F1 Activate iodigital, **F2 Sync. Motor ECU**, F3 RSE remote, **F4 Reset ECU**.
- Форум (bimmerfest, E60 530d, 4A63): допомогло «CAS start-value reset, then a reset» → відповідає F2 → F4 (перезапуск CAS після синхронізації).
- План: варіант A — F2 → F4 → зап. вимк. 30 с → запуск; варіант B (якщо A не спрацює) — Toolset32 DDE EWS_STARTWERT 0x01 → INPA CAS F2 → F4 → перевірка ews_empfang.

### ⭐ INPA CAS → Activate → F2 «Sync. Motor ECU» = повна офіційна послідовність
| Крок INPA | Результат |
|---|---|
| Write start value (CAS) | i.O. |
| **Reset CAS** | i.O. |
| Prepare DME | **n.i.O.** — INPA зупинився: варіант DDE D62M57A0 відсутній у списку скрипта CAS |
| Receiving status | Not tested |
Потім F4 Reset ECU (CAS) → done.
**Висновок**: ми пропускали крок **Reset CAS** між записом стартового значення і підготовкою DDE. Правильний порядок (в одному циклі): CAS `dme_startwert_abgleich` → CAS `steuergeraete_reset` → DDE `ews_startwert 0x01` (Prepare DME) → DDE `ews_empfang` (Receiving status). Кроки DDE робити в Toolset32, бо INPA не підтримує D62M57A0.
- Toolset32 D_CAS steuergeraete_reset → ERROR_ECU_CONDITIONS_NOT_CORRECT (7F 11 22) — CAS не приймає прямий reset; INPA робить «Reset CAS» успішно іншим способом.
- **Знайдено ISTA+ 4.53.40** на ПК власника → переходимо до ISTA: ідентифікація → короткий тест → помилки 4A63 (DDE) / CAS → тест-план EWS, або сервісні функції (синхронізація DME/DDE ↔ CAS).

### Конфігурація EDIABAS на ПК власника
- EDIABAS 7.3.0 (C:\EDIABAS): Interface = STD:OBD — працює з INPA/Toolset32, НЕ чіпати.
- Ще один EDIABAS 7.3.0 ini: Interface = STD:passthrud (J2534), RemoteHost 192.168.68.40 — інша інсталяція (уточнити шлях).
- EDIABAS 7.7.1 (ISTA, C:\Program Files\BMW\ISPI\TRIC\ISTA\Ediabas): Interface = ENET; у самому файлі зазначено, що драйвер STD:OBD більше не входить до пакета → ISTA з K+DCAN «з коробки» не працюватиме без додаткового налаштування.
- Рекомендація: спершу НЕ змінювати конфігурації; зробити комбіновану спробу INPA CAS F2 (Write start value + Reset CAS) → Toolset32 DDE EWS_STARTWERT 0x01 → EWS_EMPFANG.
