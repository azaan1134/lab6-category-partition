# Лаборатори №6: Тест тохиолдол боловсруулах — Category-Partition ба PICT

Программ хангамжийн чанарын баталгаа ба тест (F.CSA313)

**B232270052 С. Азжаргал**

---

## Хэсэг А: Category-Partition арга (гараар)

Тестлэх функц: `POST /transfer { "from": ID, "to": ID, "amount": NUM, "currency": CODE }`
Амжилт: `201 {"result":"OK"}` · Оролтын алдаа: `200 {"result":"ERROR_..."}`

**Тестийн орчны тохиргоо (бүх тестэд ижил):**
- `ACC-A` — идэвхтэй данс, валют MNT, үлдэгдэл 10000 MNT, өдрийн лимит 1000 MNT
- `ACC-B` — идэвхтэй данс (хүлээн авагч), валют MNT
- `ACC-F` — царцаагдсан данс
- `ACC-999` — байхгүй данс
- Ханш: 1 USD = 10 MNT (`Different` валют нь USD)

### Даалгавар A1 — Сонголт ба төлөөлөх утгууд

| Параметр | Сонголт | Төлөөлөх утга | Тайлбар |
| :--- | :--- | :--- | :--- |
| **from** | `Valid` | `ACC-A` | Хүчинтэй, идэвхтэй илгээгч данс |
| | `Frozen` | `ACC-F` | Царцаагдсан данс |
| | `Missing` | `ACC-999` | Системд байхгүй илгээгч данс |
| **to** | `Valid` | `ACC-B` | Хүчинтэй хүлээн авагч данс |
| | `Missing` | `ACC-999` | Байхгүй хүлээн авагч данс |
| | `SameAsFrom` | `ACC-A` | Илгээгчтэй ижил данс |
| **amount** | `Normal` | `50` | Хэвийн дүн (0 < дүн ≤ үлдэгдэл, лимитэд багтана) |
| | `Zero` | `0` | Тэг дүн (≤ 0) |
| | `OverBalance` | `15000` | Үлдэгдэл (10000)-ээс их |
| | `OverLimit` | `2000` | Үлдэгдэлд багтах боловч өдрийн лимит (1000)-ээс их |
| **currency** | `Same` | `MNT` | Дансны валюттай ижил |
| | `Different` | `USD` | Ханшийн хөрвүүлэлт хийгдэнэ (1 USD = 10 MNT) |
| **Далд орчин** | `Fresh` | өнөөдөр 0 MNT зарцуулсан | Өдрийн лимит хоосон |
| | `NearFull` | өнөөдөр 980 MNT зарцуулсан | Лимит бараг дүүрсэн (980 + 50 > 1000) |

Далд орчны сонголт байгаа: **өдрийн лимит хэр дүүрсэн байсан бэ** (`DailyLimitStatus`). Энэ нь хүсэлтийн бие дотор байхгүй боловч `ERROR_LIMIT`-ийн үр дүнд нөлөөлнө.

### Даалгавар A2 — Хязгаарлалтууд

| Утга | Тэмдэглэгээ | Үндэслэл |
| :--- | :--- | :--- |
| from = `Frozen` | `[ERROR]` | Тодорхойлолтоор царцаагдсан данснаас шилжүүлэх боломжгүй → `ERROR_FROZEN`. |
| from = `Missing` | `[ERROR]` | Илгээгч данс байх ёстой → `ERROR_NO_FROM`. |
| to = `Missing` | `[ERROR]` | Хүлээн авагч данс байх ёстой → `ERROR_NO_TO`. |
| to = `SameAsFrom` | `[ERROR]` | to нь from-оос өөр байх ёстой → `ERROR_SAME_ACCOUNT`. |
| amount = `Zero` | `[ERROR]` | amount > 0 байх ёстой → `ERROR_BAD_AMOUNT`. |
| amount = `OverBalance` | `[ERROR]` | Үлдэгдлээс хэтэрсэн → `ERROR_INSUFFICIENT`. |
| amount = `OverLimit` | `[ERROR]` | Өдрийн лимитээс хэтэрсэн → `ERROR_LIMIT`. |
| DailyLimitStatus = `NearFull` | `[SINGLE]` | Далд төлөв нэг удаа шалгахад хангалттай; хэвийн дүнтэй ч гэсэн `ERROR_LIMIT` гаргадаг онцгой тохиолдол. |
| amount = `Zero` / `OverBalance` / `OverLimit` | `[IF from ≠ Missing AND to ≠ Missing]` | Данс байхгүй үед дүнг шалгах хүртэл очдоггүй (давуу эрэмбээр from/to алдаа түрүүлж буцна), тиймээс эдгээр хослол утгагүй. |
| `Valid`, `Normal`, `Same`, `Different`, `Fresh` | (тэмдэглэгээгүй) | Хэвийн утга — бусад сонголттой бүрэн хосолно. |

**Бүрэн олонлог ба хязгаарлалтын тооцоо (N → M):**

- Бүрэн олонлог: `from`(3) × `to`(3) × `amount`(4) × `currency`(2) × `DailyLimitStatus`(2) = **N = 144**
- Хязгаарлалтын дараа: хэвийн утгуудын хослол 1 × 1 × 1 × 2 × 1 = **2** (happy path, валют `Same`/`Different`) + `[ERROR]` утга бүр нэг удаа = **7** + `[SINGLE]` = **1** → **M = 10**
- **Үр дүн: 144 → 10**

(PICT-ийн `IF` хязгаарлалтаар хэмжвэл: 72 хос (DailyLimitStatus-гүй) → 42. Амжилттай `Amount = Normal` үед 3 × 3 × 1 × 2 = 18, бусад 3 дүнд from ≠ Missing, to ≠ Missing байх 2 × 2 × 3 × 2 = 24; нийт 18 + 24 = 42.)

### Даалгавар A3 — Спецификацийн хүснэгт ба алдааны давуу эрэмбэ

**Давуу эрэмбэ:** `from` → `to` → `amount`. Дотроо: `ERROR_NO_FROM` → `ERROR_FROZEN` → `ERROR_NO_TO` → `ERROR_SAME_ACCOUNT` → `ERROR_BAD_AMOUNT` → `ERROR_INSUFFICIENT` → `ERROR_LIMIT`.

**Үндэслэл:** Хүсэлтийг хамгийн "суурь" баримтаас нь эхлэн шалгана — эхлээд илгээгч (байгаа эсэх, царцсан эсэх), дараа нь хүлээн авагч, хамгийн сүүлд дүн; дүнгийн шалгалт үлдэгдэл, лимитийг (мөн ханшийн хөрвүүлэлтийг) ашигладаг тул хоёр данс хүчинтэй болсны дараа л утгатай. Хэрэв нэг хүсэлтэд олон алдаа байвал энэ эрэмбээр эхний алдааны код буцна.

| № | from | to | amount | currency | Далд орчин | Хүлээгдэх үр дүн (Oracle) | Тайлбар / Төрөл |
|---|---|---|---|---|---|---|---|
| 1 | Valid | Valid | Normal | Same | Fresh | `201 OK` | Happy path |
| 2 | Valid | Valid | Normal | Different | Fresh | `201 OK` | Валютын хөрвүүлэлттэй (50 USD = 500 MNT) |
| 3 | Missing | Valid | Normal | Same | Fresh | `200 ERROR_NO_FROM` | Илгээгч байхгүй [ERROR] |
| 4 | Frozen | Valid | Normal | Same | Fresh | `200 ERROR_FROZEN` | Царцаагдсан данс [ERROR] |
| 5 | Valid | Missing | Normal | Same | Fresh | `200 ERROR_NO_TO` | Хүлээн авагч байхгүй [ERROR] |
| 6 | Valid | SameAsFrom | Normal | Same | Fresh | `200 ERROR_SAME_ACCOUNT` | Ижил данс хооронд [ERROR] |
| 7 | Valid | Valid | Zero | Same | Fresh | `200 ERROR_BAD_AMOUNT` | Тэг дүн [ERROR] |
| 8 | Valid | Valid | OverBalance | Same | Fresh | `200 ERROR_INSUFFICIENT` | Үлдэгдлээс их [ERROR] |
| 9 | Valid | Valid | OverLimit | Same | Fresh | `200 ERROR_LIMIT` | Өдрийн лимитээс их [ERROR] |
| 10 | Valid | Valid | Normal | Same | NearFull | `200 ERROR_LIMIT` | Хэвийн дүн ч лимит дүүрсэн (980 + 50 > 1000) [SINGLE] |
| 11 | Valid | Valid | OverBalance | Different | Fresh | `200 ERROR_INSUFFICIENT` | 2000 USD = 20000 MNT > үлдэгдэл; хөрвүүлсний дараа шалгана [ERROR] |
| 12 | Missing | Missing | Normal | Same | Fresh | `200 ERROR_NO_FROM` | Давхар алдаа: from эхэлж шалгагдана |
| 13 | Frozen | Valid | Zero | Same | Fresh | `200 ERROR_FROZEN` | Давхар алдаа: from алдаа дүнгийн алдаанаас түрүүлнэ |
| 14 | Valid | SameAsFrom | Zero | Same | Fresh | `200 ERROR_SAME_ACCOUNT` | Давхар алдаа: to алдаа amount-аас түрүүлнэ |
| 15 | Valid | Valid | OverBalance | Same | NearFull | `200 ERROR_INSUFFICIENT` | Давхар алдаа: үлдэгдлийн алдаа лимитийн алдаанаас түрүүлнэ |

---

## Хэсэг Б: PICT хэрэгсэл

PICT-ийг `microsoft/pict` репозиторийн эх кодоос `cmake` ашиглан бүтээсэн (репогоос гадна `~/tools/pict` дотор). Нотолгоо: `results/pict-build.txt`. Бүх ажиллуулалтад `/r` ашиглаагүй (анхдагч тогтмол үр дүн).

### Даалгавар B1 — Browser жишээ (`browser.model`)

Загвар: `Content`(3), `Cookies`(3), `Popups`, `Addons`, `Attacks`, `Forgeries` (тус бүр 2).

| Горим | Мөрийн тоо (толгойгүй) | Файл |
|---|---|---|
| 2-way | **10** | `results/browser-2way.txt` |
| 3-way| **20** | `results/browser-3way.txt` |
| Бүрэн олонлог | 144 (3×3×2×2×2×2) | — |

**Greedy-гийн тайлбар:** Лекц дээр гараар 9 мөрөнд (хамгийн багадаа 3 × 3 = 9, учир нь `Content` × `Cookies`-ийн бүх 9 хос тус тусдаа мөр шаарддаг) багтаасан бол PICT 10 мөр гаргасан. PICT нь greedy алгоритм ашигладаг: алхам бүрт одоогоор хамрагдаагүй хосыг хамгийн их хамарсан мөрийг сонгодог тул дараагийн алхмуудад үүссэн давхардал нь заримдаа хамгийн цөөн мөрт (оновчтой) хүргэдэггүй. Харин хамгийн цөөн covering array олох нь NP-хүнд бодлого тул PICT хурд, ойролцоо оновчтой байдлыг сонгосон. 3-way нь 144 бүрэн олонлогийн хуваарьт ердөө 20 мөр буюу ойролцоогоор 14%-д багтсан.

### Даалгавар B2 — Шилжүүлгийн загваруудын харьцуулалт

**Model файлууд:**

`transfer-nc.model` (хязгаарлалтгүй):
```
From: Valid, Frozen, Missing
To: Valid, Missing, SameAsFrom
Amount: Zero, Normal, OverBalance, OverLimit
Currency: Same, Different
```

`transfer.model` (A2-ын хязгаарлалттай):
```
From: Valid, Frozen, Missing
To: Valid, Missing, SameAsFrom
Amount: Zero, Normal, OverBalance, OverLimit
Currency: Same, Different

IF [From] = "Missing" THEN [Amount] = "Normal";
IF [To] = "Missing" THEN [Amount] = "Normal";
```

`transfer-neg.model` (`~` сөрөг утгатай):
```
From: Valid, ~Frozen, ~Missing
To: Valid, ~Missing, ~SameAsFrom
Amount: Normal, ~Zero, ~OverBalance, ~OverLimit
Currency: Same, Different

IF [From] = "Missing" THEN [Amount] = "Normal";
IF [To] = "Missing" THEN [Amount] = "Normal";
```

**Харьцуулалт** (толгой мөрийг тооцоогүй; бүгд `results/*.txt` файлын мөрийн тоотой таарна):

| Загвар | Файл | Мөрийн тоо | Happy path мөр (Valid, Valid, Normal) | Нэг мөр дэх алдааны тоо (0 / 1 / 2 / 3) |
|---|---|---|---|---|
| `transfer-nc.model` | `results/transfer-nc.txt` | **13** | 1 мөр (`Valid Valid Normal Different`) | 1 / 1 / 8 / 3 |
| `transfer.model` | `results/transfer.txt` | **12** | **0 мөр** | 0 / 3 / 7 / 2 |
| `transfer-neg.model` | `results/transfer-neg.txt` | **16** | 2 мөр (`Same`, `Different`) | 2 / 14 / 0 / 0 |

**Дүгнэлт:**
- Энгийн утгатай (`nc`, хязгаарлалттай) загварууд нэг мөрөнд 2–3 алдаа хольдог. Хязгаарлалт нь `Missing`-тэй хамт `Zero/OverBalance/OverLimit` орохыг л хаасан тул 12 мөрөнд хэвээр хоёр, гурван алдаа үлдэж, ихэнх мөр юуг шалгаж байгаа нь тодорхойгүй (error masking: эхний алдаа бусдыг нууж, `ERROR_..` нь зөвхөн давуу эрэмбээр тодорхойлогдоно). Мөн `transfer.txt`-д happy path мөр огт гараагүй — хосын хамралт хангагдсан ч бүх зүйл хүчинтэй хэвийн хэрэг шалгагдаагүй.
- `~` ашигласан загвар нэг мөрөнд хамгийн ихдээ нэг алдаа оруулдаг (14 мөр яг 1 алдаатай) бөгөөд 2 happy path мөр гарсан. Мөр бүрийн oracle нь шууд тодорхой (нэг алдаа → нэг `ERROR_..`).
- `~` хэрэглэхэд A2-ын `IF` хязгаарлалтууд **практикт хэрэггүй болсон**: нэг мөрөнд нэгээс олон сөрөг утга байж болохгүй тул `From = ~Missing` үед `Amount` нь аль хэдийн сөрөг биш (Normal) байна. Тиймээс хязгаарлалт хэзээ ч идэвхжихгүй (загварт хэвээр үлдээсэн ч үр дүнд нөлөөлөхгүй). Харин `~`-гүй загварт хязгаарлалт шаардлагатай хэвээр. Хэрэв өөр давхар (маскинг) тохиолдлыг шалгах хэрэгтэй бол (ж: Frozen + Zero) тусад нь гараар нэмэх хэрэгтэй — `~` нь тэдгээрийг хасдаг.

**Бодит тест кейс ** Setup: дээрх орчин (`ACC-A` үлдэгдэл 10000 MNT, лимит 1000 MNT, өнөөдөр 0 зарцуулсан; `ACC-B`; `ACC-F` царцсан; `ACC-999` байхгүй; 1 USD = 10 MNT).

| № | PICT мөр | Setup | Оролт (`POST /transfer` body) | Oracle |
|---|---|---|---|---|
| TC1 | `transfer-neg` мөр 1: `Valid Valid Normal Same` (happy path) | `ACC-A` идэвхтэй, 10000 MNT | `{"from":"ACC-A","to":"ACC-B","amount":50,"currency":"MNT"}` | `201`, `{"result":"OK"}` |
| TC2 | `transfer-neg` мөр 3: `Valid Valid ~OverBalance Different` | `ACC-A` 10000 MNT | `{"from":"ACC-A","to":"ACC-B","amount":2000,"currency":"USD"}` (= 20000 MNT) | `200`, `{"result":"ERROR_INSUFFICIENT"}` |
| TC3 | `transfer-neg` мөр 8: `~Missing Valid Normal Same` | `ACC-999` байхгүй | `{"from":"ACC-999","to":"ACC-B","amount":50,"currency":"MNT"}` | `200`, `{"result":"ERROR_NO_FROM"}` |
| TC4 | `transfer-neg` мөр 10: `Valid ~SameAsFrom Normal Different` | `ACC-A` идэвхтэй | `{"from":"ACC-A","to":"ACC-A","amount":50,"currency":"USD"}` | `200`, `{"result":"ERROR_SAME_ACCOUNT"}` |
| TC5 | `transfer` мөр 1: `Frozen SameAsFrom Zero Same` (3 алдаатай) | `ACC-F` царцсан | `{"from":"ACC-F","to":"ACC-F","amount":0,"currency":"MNT"}` | `200`, `{"result":"ERROR_FROZEN"}` — давуу эрэмбэ: from → to → amount, тиймээс from-ийн алдаа буцна |

### Даалгавар B3 — Дүгнэлт

Гараар хийсэн category-partition (A3) нь алдааны семантикийг илүү сайн барьдаг: `[ERROR]`, `[SINGLE]`, `[IF]` тэмдэглэгээ болон давуу эрэмбэ нь мөр бүрийн хүлээгдэх үр дүнг (oracle) тодорхой болгож, `NearFull` шиг далд орчны онцгой тохиолдлыг олж илрүүлсэн. Гэхдээ хүнд тулгуурласан тул хосуудыг гүйцэт хамарсан эсэх нь баталгаагүй, бүх сонголт хэн нэгний анхаарлаас гарч магадгүй. PICT нь хосын (pairwise) хамралтыг математикийн хувьд баталгаажуулж, 144 → 10–16 хүртэл цөөн мөр өгдөг боловч алдааны утга учрыг өөрөө ойлгодоггүй: энгийн утгатай үед нэг мөрөнд олон алдаа хольж (error masking), happy path мөр гаргаж өгөхгүй байж болно (`transfer.txt`). `~` сөрөг утга нь энэ дутагдлыг засаж, category-partition-ий `[ERROR]`-той дүйцсэн ашигтай арга боловч оролтын хоорондох давхар алдааг шалгахаа больдог. Тиймээс хамгийн сайн нь хоёрыг хослуулах: PICT-ээр хосын хамралтыг, гараар алдааны семантик ба давхар нөхцөлийг. Хамгийн их бодол шаардсан алхам нь A2 (утга бүрийн ERROR/SINGLE/IF ангилал ба давуу эрэмбийн үндэслэл) ба `~` нь A2-ын хязгаарлалтыг хэрэггүй болгосныг ойлгох явдал байлаа. PICT-ийн мөрийн тоо greedy-гээс болж гараар тооцсоноос ялгаатай гарсан нь урьдчилан таамаглах боломжгүй гэдгийг харуулсан.

---

## Файлууд

`README.md`, `browser.model`, `transfer-nc.model`, `transfer.model`, `transfer-neg.model`, `results/` (`pict-build.txt`, `browser-2way.txt`, `browser-3way.txt`, `transfer-nc.txt`, `transfer.txt`, `transfer-neg.txt`)