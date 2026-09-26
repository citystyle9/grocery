# Grocery Planner Pro — Project Documentation

**دستاویز کی تاریخ:** 2026-09-23  
**مقصد:** اس فائل میں موجودہ Grocery Planner Pro کی ساخت، behavior، data format، اہم business rules، print workflows اور تبدیلی کرتے وقت احتیاطیں درج ہیں۔ کسی AI agent یا developer کو کام شروع کرنے سے پہلے اسے `index.html` کے ساتھ پڑھنا چاہیے۔

> یہ دستاویز موجودہ source code کے مطابق ہے۔ اسے مستقبل کے ارادوں یا کسی نامکمل redesign کی specification نہ سمجھا جائے۔ اگر کوڈ اور اس دستاویز میں فرق ہو تو موجودہ `index.html` اصل implementation کا source of truth ہے، اور اختلاف کو دستاویز میں بھی درج کیا جائے۔

## 1. پروجیکٹ کا خلاصہ

Grocery Planner Pro ایک browser-based، single-page گروسری لسٹ اور بجٹ پلانر ہے۔ صارف اشیاء، پیک کے سائز، مقدار، اکائی اور ریٹ درج کرتا ہے؛ منتخب اشیاء کی کل لاگت اور بجٹ میں باقی رقم ایپ خود نکالتی ہے۔ منتخب فہرست کو template کے طور پر محفوظ، JSON backup کے طور پر export/import، یا دو print formats میں پرنٹ کیا جا سکتا ہے۔

### بنیادی خصوصیات

- پہلے سے موجود grocery items کی فہرست اور نئی item شامل کرنا۔
- وزن، حجم اور عدد کی مقدار؛ item کی مقدار کے لیے متعلقہ چھوٹی یا بڑی اکائی منتخب کرنا۔
- مقدار یا اکائی کی inline single-item editing اور bulk editing۔
- کمپنی اور package size سمیت item کی شناخت؛ read mode اور print میں یہ تفصیل ایک ساتھ نظر آتی ہے، مگر editing fields اور stored properties الگ ہیں۔
- category، search، selected/unselected filters، drag-and-drop ترتیب، اور select all/deselect all۔
- کل بجٹ، منتخب اشیاء کی کل لاگت، باقی بجٹ، منتخب اشیاء کی تعداد، اور فی آئٹم amount۔
- فی آئٹم rate history، templates، JSON backup/restore، measurement guide، اور print/PDF۔

## 2. فائلیں اور runtime

| فائل | استعمال |
|---|---|
| `index.html` | پورا UI، CSS، default items، business logic، browser persistence اور print code۔ |
| `Grocery_Planner_v29_2026-09-23_17-23-48.json` | وہ reference backup جس سے موجودہ default catalog اور category data مرتب کیا گیا؛ یہ runtime import یا source code نہیں۔ |
| `PROJECT_DOCUMENTATION.md` | یہ technical/functional handover۔ |

اس workspace میں الگ `package.json`، bundler، source modules، backend یا test suite موجود نہیں۔ ایپ کا JavaScript `index.html` کے inline `<script>` میں ایک strict-mode IIFE کے اندر ہے۔ CSS بھی اسی فائل میں ہے۔ اسے browser میں کھولنا کافی ہے؛ development build step نہیں۔ مقامی preview فی الحال `http://127.0.0.1:8000/` پر استعمال ہوتا ہے۔

Inspection کے وقت یہ folder valid Git working tree کے طور پر resolve نہیں ہوا؛ edits سے پہلے دستی backup بنانا مفید ہے اور Git history/revert کو دستیاب فرض نہ کریں۔

### بیرونی dependency

- Google Fonts stylesheet سے `Noto Sans Arabic` (اردو/عربی UI) اور `Plus Jakarta Sans` (English/numeric presentation کے کچھ حصے) لوڈ کیے جاتے ہیں۔
- Font request دستیاب نہ ہو تو CSS fallback fonts ہیں؛ بنیادی application logic پھر بھی browser میں چلتی ہے۔
- کوئی API، server-side database، analytics SDK یا third-party JavaScript library اس source میں شامل نہیں۔

### Browser APIs

`localStorage`, native `<dialog>`, `FileReader`, `Blob`, `URL.createObjectURL`, `window.print()` اور browser download استعمال ہوتے ہیں۔ Print کے لیے A4 portrait اور `@page` margin boxes ہیں؛ page-footer behavior کو target Chromium/Edge browser میں verify کرنا چاہیے کیونکہ print CSS support browser/version کے لحاظ سے فرق کر سکتی ہے۔

## 3. ایپ کے حصے

1. **Header/tool bar:** view mode (کارڈ ویو / ٹیبل ویو)، templates، bulk edit، guide، activity log، backup export/import، print، reset، theme toggle۔
2. **Dashboard:** کل بجٹ، باقی بجٹ، کل لاگت، منتخب اشیاء۔ RTL ترتیب دائیں سے بائیں یہی ہے۔ موبائل پر خودکار 2×2 ریسپانسیو گرڈ۔
3. **Controls:** search، category filter، تمام/منتخب/غیر منتخب filter pills، active-template badge، نئی item شامل کرنے کا بٹن۔
4. **Items table / Cards view:** ڈیسک ٹاپ پر روایتی ٹیبل، جبکہ موبائل اسکرینز (<= 768px) پر خودکار طور پر جدید کارڈ ویو (Card View) جس میں ٹچ فرینڈلی سلیکشن، واضح باکسز (مقدار، ریٹ، کل رقم)، موبائل ٹچ ڈریگ و ڈراپ، اور فوری ایڈٹ شامل ہیں۔ صارف ٹول بار سے جب چاہے کارڈ اور ٹیبل ویو کے درمیان سوئچ کر سکتا ہے۔
5. **Sticky checkout bar:** منتخب اشیاء کی مکمل رقم اور select/deselect all actions (موبائل سیف ایریا پیڈنگ کے ساتھ)۔
6. **Dialogs:** templates، print mode، add item، quantity unit switch، measurement guide، price history، activity log۔
7. **Static print wrapper:** الگ print DOM؛ interactive table کو براہ راست print کرنے کے بجائے `buildDedicatedPrintDOM()` اسے تیار کرتا ہے۔

## 4. Application architecture اور اہم code areas

`index.html` میں یہ ذمہ داریاں الگ helper functions میں ہیں، مگر یہ سب ایک ہی IIFE اور فائل کا حصہ ہیں:

| Code area / identifiers | ذمہ داری |
|---|---|
| `STORAGE_KEY`, `TEMPLATES_KEY`, `DEFAULT_ITEMS`, `DEFAULT_CATEGORIES` | localStorage keys اور first-run/reset seed data؛ catalog میں 111 آئٹمز ہیں۔ |
| `UNIT_INFO`, `QUANTITY_TYPES`, `getQuantityType()` | اکائی سے group شناخت کرنا اور valid unit group بتانا۔ |
| `normalizeMeasurement()`, `convertMeasurement()` | value/unit کو normalize یا اسی group میں convert کرنا۔ |
| `normalizeItemUnits()`, `normalizeItemList()` | load/import کے وقت item shape اور unit values صاف کرنا۔ |
| `loadState()`, `saveState()`, `saveTemplates()` | browser persistence۔ |
| `UNIT_MULTIPLIERS`, `getDisplayRate()`, `calculateItemAmount()` | rate basis اور amount calculation۔ |
| `updateCategoryDropdowns()`, `renderCategoryManagerList()` | category filter، dedicated modal، drag-and-drop / buttons reordering اور add/rename/delete category UI۔ |
| `logRateChange()`, `showPriceHistoryModal()` | rate history لکھنا اور دکھانا۔ |
| `renderTemplatesList()`, `updateActiveTemplateUI()` | template CRUD، selection load/update اور badge۔ |
| `initViewMode()` | موبائل کارڈ ویو اور روایتی ٹیبل ویو کے درمیان موڈ کنٹرول اور لوکل اسٹوریج میں حالت محفوظ رکھنا۔ |
| `renderTable()`, `updateCalculations()` | interactive table/cards اور dashboard/checkout totals۔ |
| `attachTableDelegations()` | table/cards کے dynamically rendered input/change/click/drag اور mobile touch-drag actions۔ |
| `buildDedicatedPrintDOM()` | selected items، categories، subtotals اور print mode کے مطابق print DOM بنانا۔ |
| `initApp()` | state load، listeners، modal actions، theme/view mode init اور initial render۔ |

### Dynamic table کا event contract

Table بار بار `innerHTML` سے render ہوتا ہے۔ اس لیے row controls پر listeners الگ الگ لگانے کے بجائے `attachTableDelegations()`، `#tableBody` پر delegated `input`, `change`, `click`, `dragstart`, `dragover`, `drop` وغیرہ listeners لگاتا ہے۔ Row controls کے `.in-*` classes، `data-id` اور `data-action`، JavaScript کے ساتھ مشترک contract ہیں۔ ان میں تبدیلی کرنے سے پہلے render template اور delegated handlers دونوں کو trace کریں۔

## 5. Data model

### Local application state

`GROCERY_PLANNER_PRO_DATA_V29` میں state JSON کے طور پر محفوظ ہوتا ہے:

```json
{
  "items": [],
  "categories": [],
  "selected": {},
  "budget": 0,
  "nextItemId": 152
}
```

اہم بات: `selected` boolean map ہے جس کی keys item IDs کی strings بن جاتی ہیں۔ منتخب item کی کل رقم اور تعداد اسی map سے بنتی ہے۔ `nextItemId` اگلا numeric item ID ہے؛ موجودہ built-in catalog میں زیادہ سے زیادہ ID `151` ہے، اس لیے fresh state میں اگلا ID `152` ہے۔

### Item record

```json
{
  "id": 1,
  "name": "Example item",
  "category": "",
  "company": "",
  "sizeVal": "",
  "sizeUnit": "",
  "qty": 500,
  "qtyUnit": "گرام",
  "qtyType": "weight",
  "rate": 400,
  "priceHistory": []
}
```

- `name`, `company`, `sizeVal`, `sizeUnit`: شناخت کی الگ stored/editable fields۔ Read-only main table اور print ان کو ایک تفصیل میں جوڑتے ہیں؛ اس سے underlying fields merge نہیں ہوتے۔
- `category`: category کا متن؛ عام/بغیر category item کے لیے خالی string۔
- `qty`, `qtyUnit`: عدد اور اس کی واضح اکائی ایک جوڑا ہیں؛ عدد کو اکائی کے بغیر الگ نہیں سمجھنا چاہیے۔
- `qtyType`: `weight`, `volume`, یا `count`؛ load normalization اسے `qtyUnit` سے اخذ کر کے درست کرتا ہے۔
- `rate`: number؛ اس کا meaning item کی quantity group سے بندھا ہے۔
- `priceHistory`: تازہ ترین اندراج پہلے؛ زیادہ سے زیادہ 10 records رکھے جاتے ہیں۔ ایک history row میں `rate`, `unit`, `date`, `iso` ہوتے ہیں۔
- `id`: item کی مستقل numeric شناخت؛ یہ visible row number/order نہیں۔ Drag reorder یا delete سے موجودہ آئٹمز کے IDs تبدیل نہیں ہوتے۔
- `nextItemId`: صرف state-level allocator ہے، ہر item row کا field نہیں؛ نئی اور cloned items اسے استعمال کر کے اسے فوراً increment کرتی ہیں۔

### Built-in default catalog

`DEFAULT_ITEMS` میں `Grocery_Planner_v29_2026-09-23_17-23-48.json` کے 111 item records کی بنیادی catalog معلومات شامل ہیں۔ ہر default item میں نام، کمپنی، سائز اور سائز کی اکائی، category، quantity، `qtyType` اور `qtyUnit` seed ہوتے ہیں۔ Catalog میں 75 weight items (`گرام` یا `کلوگرام`) اور 36 count items (`عدد`) ہیں؛ اس backup snapshot میں quantity group `volume` والا کوئی item نہیں۔ Package size کی volume unit (مثلاً ملی لیٹر) الگ `sizeUnit` کے طور پر موجود ہو سکتی ہے۔

Default items کے `rate` کو `0` اور `priceHistory` کو خالی array رکھا جاتا ہے؛ table میں `0` rate خالی دکھایا جاتا ہے۔ اس طرح backup کے موجودہ rates default catalog میں شامل نہیں ہوتے۔ Backup import کا الگ flow backup میں موجود rates کو restore کرتا ہے۔

`DEFAULT_CATEGORIES` میں backup کی پانچ categories شامل ہیں: `کریانہ`, `ڈرائی فروٹس`, `مصالحہ جات`, `سبزی و فروٹ`, `گوشت و انڈے`۔ Fresh browser/origin اور Reset دونوں میں یہ categories item seed کے ساتھ load ہوتی ہیں۔ Fresh install میں selected map خالی اور budget `0` رہتا ہے۔

Catalog کی 88 موجودہ ناموں والی items کے IDs پرانے defaults جیسے رکھے گئے تاکہ انہی items کو refer کرنے والے محفوظ template selection maps درست رہیں۔ Backup میں شامل باقی 23 items کو منفرد IDs `129–151` دیے گئے؛ پرانے defaults سے ہٹائے گئے 14 items کے IDs دوبارہ استعمال نہیں کیے گئے۔ پرانے custom templates اگر ان 14 ہٹائے گئے items کو select کرتے تھے تو ان selection keys کا اب کوئی item نہیں ہوگا۔

### Template record

`GROCERY_PLANNER_TEMPLATES_V10` میں templates الگ array کی صورت میں ہیں:

```json
{
  "name": "ماہانہ راشن",
  "selected": { "1": true, "2": false },
  "date": "localized display date"
}
```

Template item definitions کی snapshot نہیں؛ یہ صرف selection map اور label/date محفوظ کرتا ہے۔ Item rename/delete کے بعد templates خودکار طور پر migrate نہیں ہوتے۔ `activeTemplateName` runtime variable ہے؛ reload کے بعد active badge/print template name خود بخود بحال نہیں ہوتا، اگرچہ saved selection state موجود رہتا ہے۔

### Backup envelope

Export JSON میں `app`, `version`, `exportedAt`, `state`, `templates` شامل ہوتے ہیں۔ Import wrapper (`data.state`) یا direct state object دونوں کو قبول کرنے کی کوشش کرتا ہے۔ Export filename کی شکل `Grocery_Planner_v29_YYYY-MM-DD_HH-mm-ss.json` ہے؛ filename timestamp browser کے local clock سے ہے، جب کہ `exportedAt` ISO UTC timestamp ہے۔

## 6. اکائیاں اور quantity rules

### Supported quantity units

| قسم | اکائیاں | Base factor |
|---|---|---:|
| وزن | گرام، کلوگرام | 1 گرام، 1000 گرام فی کلوگرام |
| حجم | ملی لیٹر، لیٹر | 1 ملی لیٹر، 1000 ملی لیٹر فی لیٹر |
| عدد | عدد | 1 |

یہ set `UNITS`, `UNIT_INFO`, `QUANTITY_TYPES` اور `UNIT_MULTIPLIERS` میں موجود ہے۔ وزن/حجم ایک دوسرے میں تبدیل نہیں ہوتے۔ پرانی غیر معاون unit کے ساتھ import، normalization error دے سکتا ہے؛ پرانی unit silently guess/convert کرنے کا کوئی general legacy converter نہیں۔

### اصل اکائی بمقابلہ display normalization

مقدار میں **value اور منتخب unit دونوں واضح طور پر محفوظ** ہیں۔ `500` اور `گرام` کا مطلب 500 گرام ہے؛ `500` اور `کلوگرام` کا مطلب 500 کلوگرام۔ Unit button/modal user کو اسی physical group کے چھوٹے/بڑے unit میں conversion کرنے دیتا ہے اور اصل مقدار برقرار رکھتا ہے۔ مثال: 500 گرام → 0.5 کلوگرام؛ 0.5 کلوگرام → 500 گرام۔

`normalizeMeasurement(value, unit)` وزن/حجم کی canonical display unit منتخب کرتا ہے: base amount `1000` سے کم ہو تو چھوٹی unit (گرام/ملی لیٹر)، `1000` یا زیادہ ہو تو بڑی unit (کلوگرام/لیٹر)۔ یہ **ہر quantity input پر خودکار threshold behavior نہیں** ہے۔ عام quantity typing میں number موجودہ unit کے مطابق لیا جاتا ہے؛ نئی item کے `qtyUnit` selector اور موجودہ item کے unit button سے unit طے ہوتی ہے۔ `normalizeMeasurement()` خاص طور پر package size اور مخصوص unit-change/edit paths میں چلتا ہے۔ کسی AI کو یہ فرض کر کے code نہیں بدلنا چاہیے کہ 500 خود بخود grams یا 1200 خود بخود kilograms ہو جائیں گے۔

دوسری اہم باریکی: `getQuantityUnit()` موجودہ source میں مقدار کو round کر کے group کی چھوٹی unit لوٹاتا ہے؛ `normalizeMeasurement()` والا 1000 threshold اس function میں نہیں۔ ان functions کو آپس میں خلط ملط نہ کریں۔

### Rate اور amount formula

- Weight rate: فی کلوگرام۔ Grams amount کے لیے multiplier `0.001`، kilograms کے لیے `1`۔
- Volume rate: فی لیٹر۔ Milliliters multiplier `0.001`، liters کے لیے `1`۔
- Count rate: فی عدد؛ multiplier `1`۔
- Formula: `Math.round(quantity × UNIT_MULTIPLIERS[qtyUnit] × rate)`۔ Result nearest whole currency unit پر round ہوتا ہے۔

مثالیں: 500 گرام اور فی کلوگرام 400 کا ریٹ → `round(500 × 0.001 × 400) = 200`۔ 2 لیٹر اور فی لیٹر 300 → 600۔ 3 عدد اور فی عدد 50 → 150۔

`getDisplayRate()` rate کو convert نہیں کرتا؛ یہ stored rate دکھاتا ہے۔ Rate کی basis unit `getRateBasisLabel()` quantity group سے بنتی ہے۔

### Unit group بدلنے کی احتیاط

`convertMeasurement()` صرف ایک ہی group میں conversion کرتا ہے۔ مختلف group ملنے پر اصل value واپس آتی ہے۔ Bulk edit میں `qtyType` selector تبدیل کرتے وقت handler موجودہ unit کے factor سے base number بناتا اور نئے group کے unit میں رکھتا ہے؛ اس لیے weight → volume یا count → weight جیسے group changes نئی semantic interpretation ہیں، physical conversion نہیں۔ اس behavior کو تبدیل کرنے کے لیے product-owner کی واضح requirement درکار ہے۔

## 7. User workflows

### Item شامل کرنا

Add dialog میں نام لازمی؛ category، company، package size/unit، quantity group، quantity unit، quantity اور rate درج ہوتے ہیں۔ Quantity group weight/volume/count میں سے ہوتا ہے۔ Group بدلنے پر unit options بدلتے ہیں اور rate label فی کلوگرام/فی لیٹر/فی عدد میں بدلتا ہے۔ نئی item ID موجودہ زیادہ سے زیادہ ID + 1 ہے، نئی item list کے آغاز میں جاتی ہے، default طور پر selected ہوتی ہے۔ دی گئی ابتدائی rate مثبت ہو تو initial history entry بھی بنتی ہے۔

### Single-item اور bulk edit

- Single edit row میں اسی table کے اندر کھلتا ہے؛ نام/company/size fields الگ رہتے ہیں اور مقدار/rate براہ راست edit ہوتے ہیں۔ Save، cancel، clone اور delete کے row actions موجود ہیں۔
- Bulk edit toolbar سب rows کے editable fields ایک ساتھ دکھاتا ہے؛ item detail fields اب بھی name/company/size میں الگ، اور category الگ ہیں۔
- Rate history button ہر row میں rate کے قریب ہے۔
- `input` اور `change` دونوں handlers values محفوظ/update کرتے ہیں؛ DOM row دوبارہ render ہونے والے cases میں event delegation اہم ہے۔

### Search، category، select اور ترتیب

- Search item name اور company میں case-insensitive substring تلاش کرتا ہے؛ size/category اس search میں شامل نہیں۔
- Category filter all، uncategorized، یا مخصوص category دکھاتا ہے۔
- Filter pills تمام، selected، unselected rows دکھاتے ہیں۔ یہ presentation filters ہیں؛ dashboard total calculation تمام selected state items سے ہوتا ہے، صرف filtered rows سے نہیں۔
- Select all / deselect all پوری item list پر لاگو ہوتے ہیں، صرف filtered rows پر نہیں۔
- Drag-and-drop ترتیب `state.items` array کی ترتیب بدلتی ہے۔
- Category header میں visible rows کی count اور visible selected rows کی subtotal دکھتی ہے۔

### Categories

Category list `state.categories` میں محفوظ ہوتی ہے۔ Add، rename، delete UI موجود ہے۔ Item کی category update کرنے اور category list rename/delete کے لیے category UI handlers استعمال کریں؛ صرف category array بدلنے سے item records لازماً درست نہیں ہوں گے۔ Uncategorized items کا category value `''` ہے۔

### Budget dashboard

- `budget` user-editable input ہے اور state میں save ہوتا ہے۔
- Selected total تمام selected items کے `calculateItemAmount()` کا مجموعہ ہے۔
- Remaining budget = `budget - selectedTotal`؛ negative result overspend ہے اور `metric-over` style لگتی ہے۔ Numeric text LTR isolated ہے تاکہ minus sign درست جگہ ہو۔
- Dashboard میں selected item count بھی ہے؛ bottom checkout total selected amount کی duplicate summary ہے، budget input نہیں۔

### Rate history

Positive changed rate پر `logRateChange()` entry prepend کرتا ہے؛ زیادہ سے زیادہ 10 records رکھتا ہے۔ Entry میں rate basis text، localized date اور ISO time شامل ہے۔ **Implementation note:** یہ helper rate input event سے invoke ہوتا ہے؛ type=number edit میں intermediate values بھی history میں آ سکتی ہیں۔ Rate history semantics کو بدلنے سے پہلے اس behavior اور مطلوبہ UX پر فیصلہ کریں۔

### Templates

Template بنانے کے لیے موجودہ selection غیر خالی اور نام لازم ہے۔ موجودہ نام دوبارہ دینے پر overwrite confirmation آتی ہے۔ Load template selection replace کرتا ہے؛ update template موجودہ selection لکھتا ہے؛ delete template record ہٹاتا ہے۔ Template data state کے JSON export میں شامل ہوتی ہے، مگر localStorage میں الگ key پر رہتی ہے۔

## 8. Persistence، import/export اور reset

- App state browser `localStorage` میں `GROCERY_PLANNER_PRO_DATA_V29` کے تحت ہے۔
- Templates `GROCERY_PLANNER_TEMPLATES_V10` کے تحت ہیں۔
- خالی storage پر `DEFAULT_ITEMS` clone/normalize اور `DEFAULT_CATEGORIES` کے ساتھ load ہوتے ہیں، اور `nextItemId` کم از کم default IDs کے max سے ایک زیادہ ہوتا ہے۔ موجودہ browser/origin میں پہلے سے محفوظ state ہو تو code update خودکار طور پر اسے نئی default catalog سے replace نہیں کرتا؛ backup کو Import UI سے restore کرنا یا Reset استعمال کرنا پڑتا ہے۔
- پرانے saved state یا backup میں `nextItemId` نہ ہو تو load/import موجودہ آئٹمز کے زیادہ سے زیادہ ID + 1 سے counter بناتا ہے۔ نئے backup میں یہ counter `state` کے ساتھ export ہوتا ہے۔ Import پر موجودہ browser counter، imported counter اور imported item IDs + 1 میں سے سب سے بڑا محفوظ ہوتا ہے، تاکہ اسی browser میں پرانا backup بحال کرنے سے جاری counter پیچھے نہ جائے۔
- `loadState()` items normalize کرتا ہے، missing `category` اور `priceHistory` defaults لگاتا ہے، supported `qtyUnit` validate کرتا اور `qtyType` derives کرتا ہے۔ Size unit ہو تو size normalize ہوتا ہے۔
- Import پہلے confirm لیتا ہے، items normalize کرتا، state replace کرتا اور templates کو اس وقت replace کرتا ہے جب backup میں `templates` array ہو۔ Restore کے بعد active template name reset ہوتا ہے۔
- Reset `DEFAULT_ITEMS` اور `DEFAULT_CATEGORIES` دوبارہ load کرتا، selection خالی اور budget `0` کرتا؛ `nextItemId` کو default max اور پہلے سے محفوظ counter میں سے بڑا رکھتا ہے، اس لیے Reset سے ID sequence پیچھے نہیں جاتی۔ Saved templates الگ localStorage key میں ہیں اور Reset انہیں حذف نہیں کرتا۔
- Storage write/read failure console میں log ہوتا ہے؛ الگ cloud/server backup نہیں۔

### Workspace backup snapshot

فائل `Grocery_Planner_v29_2026-09-23_17-23-48.json` میں 111 items، 5 categories، 0 selected items، budget `50000`، اور quantity units کے counts 47 گرام، 28 کلوگرام، 36 عدد تھے۔ یہ counts صرف اس backup snapshot کی حالت بیان کرتے ہیں۔ اسی snapshot سے built-in default catalog بنایا گیا ہے، مگر default seed میں rates شامل نہیں؛ ہر default item کا rate `0` (UI میں خالی) اور history خالی ہے۔ Volume quantity units code میں supported ہیں، اگرچہ اس backup میں volume quantity والے items نہیں تھے۔ Item list کی مکمل تفصیل `index.html` کے `DEFAULT_ITEMS` میں ہے۔

## 9. Print/PDF behavior

Print modal دو modes دیتا ہے؛ print صرف selected items لیتا ہے اور selection خالی ہو تو روک دیتا ہے۔

### Blank / Shopkeeper list

- مقدار اور item identity موجود رہتی ہے۔
- Rate اور amount cells خالی لکھی جانے والی جگہ کے طور پر render ہوتے ہیں۔
- اوپر selected item count رہتا ہے؛ کل لاگت اور category subtotals چھپتے ہیں۔

### Full cost list

- Rate اور per-item amount دکھتے ہیں۔
- Category subtotal اور کل لاگت دکھتی ہے۔
- کل منتخب اشیاء مقدار کے column کے اوپر، کل لاگت رقم کے column کے اوپر، page کے آغاز میں summary row میں ہے؛ کوئی grand total آخر میں نہیں۔

### Print table/layout contract

- پانچ columns: `#`, `نام کی تفصیل`, `مقدار`, `ریٹ`, `رقم`۔
- نام، company اور size non-empty ہوں تو اسی ترتیب میں `–` separator سے joined identity بناتے ہیں۔
- Current column widths: 5%, 50%, 14%, 13%, 18%؛ summary table اور print table میں یہی widths رکھیں تاکہ count quantity اور total amount کے اوپر align ہوں۔
- Category header row `colspan="5"` ہے؛ column count بدلنے پر colspan بھی لازماً بدلیں۔
- Table head repeat (`table-header-group`)، category/row page-break rules اور A4 page-margin footer CSS میں ہیں۔ Footer: بائیں page number؛ دائیں active template name (یا default template label)۔
- `buildDedicatedPrintDOM()` selected items کو categories میں regroup کرتا ہے؛ blank/full mode کے مطابق cells اور subtotals fill/hide کرتا ہے۔ `printType` values `blank` اور `full` UI contract ہیں۔

پرنٹ میں تبدیلی کے بعد دونوں modes، multi-page output، long Urdu item/company/size text، no-template case اور active-template footer لازماً target browser میں دستی طور پر دیکھیں۔ Browser print settings (scale, margins, headers/footers) CSS کے output کو بدل سکتی ہیں۔

## 10. UI/UX specification اور replica guide

یہ حصہ موجودہ HTML/CSS/interaction states کو implementation-level design specification میں بدلتا ہے۔ Exact visual match کے لیے `index.html` کی CSS حتمی reference ہے؛ workspace میں الگ Figma file یا approved screenshot موجود نہیں۔ Browser viewport، font-load، operating system اور print settings کے فرق سے rendering میں معمولی فرق ممکن ہے۔

### 10.1 Canvas اور direction

- Document: `<html lang="ur" dir="rtl">`; viewport meta device width اور initial scale `1.0`۔ Urdu reading order RTL ہے؛ انگریزی اور اعداد ضرورت کے مطابق LTR ہیں۔
- `body`: `--bg` background، `--font-ur`، `--text-main`، line-height `1.5`، minimum height `100vh`، flex column، antialiased۔ Global reset `box-sizing:border-box`, margin/padding `0` ہے۔
- Main/header/footer content کا max-width `1350px` ہے۔ Main horizontal padding `1.25rem` ہے۔ Header sticky top `0`, z-index `100`, translucent white `rgba(255,255,255,.95)`، 12px blur اور bottom border رکھتا ہے۔

### 10.2 Design tokens

| Token | Value | استعمال |
|---|---|---|
| `--bg` | `#f8fafc` | App canvas |
| `--surface` | `#ffffff` | Cards، modal، table container |
| `--surface-subtle` | `#f1f5f9` | Inputs، table header، subtle blocks |
| `--border` | `#e2e8f0` | عام borders/dividers |
| `--border-focus` | `#0d9488` | Focus outline/border |
| `--text-main` | `#0f172a` | Primary text |
| `--text-muted` | `#475569` | Secondary labels |
| `--text-subtle` | `#94a3b8` | Placeholder/muted details |
| `--primary` | `#0f766e` | Teal primary actions/totals |
| `--primary-hover` | `#115e59` | Hover |
| `--primary-soft` | `#ccfbf1` | Soft selected/hover backgrounds |
| `--success` / soft | `#16a34a` / `#dcfce7` | Under-budget/save feedback |
| `--danger` / soft | `#dc2626` / `#fee2e2` | Over-budget/delete feedback |
| `--warning` / soft | `#d97706` / `#fef3c7` | Rate-history affordance |
| Radius `sm/md/lg` | `6px / 10px / 14px` | Inputs/buttons/cards/dialog |
| Shadow `sm/md/lg` | `0 1px 3px 0 rgba(0,0,0,.05)` / `0 4px 6px -1px rgba(0,0,0,.07)` / `0 10px 15px -3px rgba(0,0,0,.08)` | Elevation |

Font imports: Noto Sans Arabic weights `400–700` and Plus Jakarta Sans weights `500–800`. `--font-ur` is Noto Sans Arabic with system fallbacks; `--font-en` is Plus Jakarta Sans with system fallbacks. Urdu inputs/selects/buttons inherit Urdu font; numeric table inputs and price totals opt into English font. Dashboard number values are `direction:ltr` + `unicode-bidi:isolate` so minus signs stay on the numeric left side.

### 10.3 Header and tool bar

- White translucent sticky strip, max-width content aligned with app body, horizontal flex with brand at one side and actions at the other.
- Brand icon: `40×40px`, gradient `--primary → #14b8a6`, radius `10px`, white trolley SVG, soft teal shadow. Brand title `1.15rem`, weight `700`, line-height `1.2`.
- Icon buttons: `38×38px`, white fill, 1px border, `6px` radius; 18px outline SVG. Hover uses soft teal, teal text/border, and `translateY(-1px)`.
- Labeled bulk/template buttons use `0.45rem 0.85rem` padding, `0.82rem` type, teal outline; active bulk state becomes teal fill/white text.
- Toolbar actions in order/role: templates, bulk-edit toggle, guide, export, import, print, reset. Import’s file input is visually hidden and opened through its button.

### 10.4 Dashboard cards

- Desktop: one card panel with 4 equal grid tracks, `0.85rem` gap, `1rem 1.25rem` padding, subtle border/radius/shadow. At viewport width `≤760px`, grid switches to 2 columns; DOM/RTL order is **کل بجٹ → باقی بجٹ → کل لاگت → منتخب اشیاء** from right to left.
- Individual card: subtle surface, 1px border, `10px` radius, centered flex column, minimum height `72px`, padding `0.6rem 0.8rem`.
- Values `1.25rem/700`; labels `0.72rem/600`, muted. Total is teal. Remaining turns green (`metric-ok`) when `≥0`, red (`metric-over`) when `<0`. Budget is a transparent centered number input that gains a focus border/glow and white fill.
- The separate bottom checkout bar duplicates the selected total for quick access; it is fixed at viewport bottom and is not one of the four cards.

### 10.5 Search, filters and primary action

- Controls section is flex, wrapping, with `0.75rem` gaps and space-between distribution.
- Search input is flexible, minimum width `200px`, white surface, `0.9rem` type, `0.55rem 0.85rem 0.55rem 2.2rem` padding, `10px` radius; search SVG is absolutely positioned. Focus uses teal border and 3px soft glow.
- Category select is white with border, `0.45rem 0.75rem` padding and `10px` radius.
- Three filter pills: pill radius `9999px`; inactive white/muted; hover teal; active teal fill and white text. Pills wrap as needed.
- Active template badge is hidden by default; when active it becomes inline-flex with pale green surface, green border/text, name and close action.
- Add item button is primary teal with white text and plus SVG; hover darkens, raises slightly and adds soft shadow.

### 10.6 Main table: geometry, hierarchy and states

- Outer container: white, `1px` border, `14px` radius, subtle shadow, horizontal overflow. Table: fixed layout, minimum width `1040px`, `0.85rem` body type. On narrow screens the table scrolls horizontally rather than converting rows into cards.
- Header row: subtle slate fill, bottom `1.5px` border, `0.65rem 0.5rem` padding, `0.75rem/700` muted type. Identity is start-aligned; other column headers centered.
- Eight physical columns: drag (`32px`), checkbox (`36px`), row number (`36px`), item identity (`40%`), quantity (`15%`), rate (`12%`), amount (`12%`), actions (`75px`). Percentages are CSS hints in a fixed table with fixed control widths; do not recalculate them as an exact 100% grid without checking actual rendering.
- The main identity column title is `نام کی تفصیل (نام، کمپنی اور سائز)`. In read mode name, company and size are joined in that order by a compact en dash `–`; name is bold/main color, other details smaller/muted, wrapping allowed. Empty company or size is omitted. In edit mode these remain three separate fields (name, company, size value/unit) plus a category select.
- Three internal vertical dividers are on quantity, rate and amount cells; they separate identity→quantity, quantity→rate, rate→amount in RTL. Drag, check, index and actions do not receive extra dividers. Horizontal row rule remains.
- Category row spans all 8 columns; pale slate fill, thicker top/bottom separators, teal category title, selected/total badge and subtotal at the opposite edge.
- Row visual states: normal white; hover `#fcfdfe`; selected pale teal `#f0fdfa` and hover `#e6fffa`, teal custom checkbox; editing pale yellow `#fefce8`; dragging opacity `.35` and blue tint; drag target teal top border.
- Custom checkbox is `20×20px`, 2px slate border, `6px` radius. Check SVG appears/scales in only when selected.
- Quantity numeric input is fixed `65px`, centered; unit trigger is pale teal pill-like button (`0.8rem/700`) and keeps unit group. Hover darkens. The amount is bold teal, `0.95rem`, right-aligned. Rate uses `0.92rem/700` English numerals; small history button is adjacent. Row action buttons are `28×28px`, with semantic green save, teal copy, red delete and amber history states.
- Single edit is inline in the row; bulk edit opens fields for all items while preserving separate inputs. This screen does not use a separate single-item edit modal.

### 10.7 Sticky checkout bar

Fixed bottom bar spans viewport, translucent white with `14px` backdrop blur, top border, soft upward shadow, `0.8rem 1.25rem` padding and z-index `100`. Inner content max-width `1350px`, flex space-between, wraps. Total label is small/muted; total is `1.35rem/800` teal. Secondary select/deselect buttons sit together with `0.5rem` gap.

### 10.8 Modal/dialog system

- Native `<dialog class="app-modal">`; centered, width `90%`, max-width `520px`, white surface, `14px` radius, large shadow. Backdrop is dark translucent navy with blur.
- Header: `1rem 1.25rem` padding, bottom border, title `1.05rem/700`, close icon at the opposite edge. Body: `1.25rem` padding, `0.88rem` muted text, max-height `75vh`, internal vertical scroll. Footer: top border, `0.85rem 1.25rem` padding, right/end-aligned actions and `0.5rem` gap.
- Add form uses paired fields in two equal columns (`form-row`, `0.75rem` gap); fields are full-width, subtle fill, border, `6px` radius, Urdu font, `0.55rem 0.75rem` padding. Focus switches to white/teal/glow. Category plus opens an embedded category panel; panel itself is hidden until `.show`, white, border/radius/shadow, category list max-height `120px`.
- Quantity-unit dialog explains conversion, offers only same-type choices, then Cancel/Apply. Guide dialog is scrollable and uses small tables for weight/volume equations plus count explanation. History dialog shows an empty-state message or vertical timeline with teal nodes, date then rate.
- Print choice uses two radio cards: blank shopkeeper list (default selected) and full cost list. Cards have `2px` border, `10px` radius, `0.85rem 1rem` padding; hover border teal/subtle fill; radio accent teal.
- Template rows use subtle fill/border, `6px` radius, name/metadata on one side, Load/Update/Delete controls on the other; empty-template state is centered muted instructional text.

### 10.9 Print/PDF visual contract

- `@page`: A4 portrait, margins `1.2cm 1cm 1.5cm 1cm`; left footer prints page count, right footer prints active template name. Interactive header/main/checkout/dialogs hide in print; static `#printWrapper` becomes visible.
- Masthead: title `1.25rem/800` deep teal, date opposite, teal 2px bottom rule. Metadata row shows template chip and mode (blank/full).
- Summary and table share 5 RTL columns and widths `5% / 50% / 14% / 13% / 18%`: row number, item detail, quantity, rate, amount. Summary count sits in the quantity track; total sits in the amount track. This alignment is an explicit contract; update both colgroups together.
- Summary chips pale teal with thin teal border. Table has `9pt` type, fixed layout, light slate grid `#cbd5e1`, header pale teal `#eaf3f2`, deep teal text. Category band pale slate, subtotal only in full mode. Table head repeats per page; rows/categories avoid page split where possible.
- Item identity prints joined `name – company – size`; blanks omitted. Blank mode leaves rate and amount cells writable and hides cost totals/subtotals. Full mode prints rate, row amount, category subtotal and top total. Neither mode places a grand total at document end.

### 10.10 Interaction states and UX feedback

| Trigger | Visible response |
|---|---|
| Focus a search/form/table input | Teal border and soft teal focus glow; form/table field background becomes white. |
| Hover an icon/primary/filter/unit control | Teal emphasis; primary buttons darken/raise, row action colors are semantic. |
| Select item | Checkbox fills teal; row background pale teal; dashboard/checkout totals update. |
| Toggle bulk edit | Button label/icon changes to completion state; all rows rerender with independent fields. |
| Click item unit label | Conversion dialog opens; same-group units only; Apply preserves quantity magnitude. |
| Enter positive changed rate | Item rate and amount recalculate; history entry is prepended. |
| Set budget below selected total | Remaining value becomes negative and card turns red. |
| Choose category/search/filter | Visible table and category summaries rerender; global selected dashboard remains based on all selected items. |
| Export/import/reset/delete/overwrite template | Relevant confirmation/alert appears before destructive replacement/deletion or after completion. |
| Print with no selected items | Print is blocked with an alert; otherwise format dialog opens. |

Empty states: template manager explains how to create a template; category manager says none exist; price history explains when empty. The current table renderer does not show a dedicated “no matching items” row for an empty search/filter result.

### 10.11 Responsive and accessibility notes

- Only explicit breakpoint is `max-width:760px`, which changes dashboard from 4 to 2 columns. Controls use flex wrapping; checkout content wraps; table remains fixed/min-width and horizontally scrollable. Add form `form-row` remains two columns in the CSS shown; do not assume it stacks automatically on mobile.
- Dialogs use native accessible `<dialog>`; icon buttons include Urdu `title` text; table selection uses `role="checkbox"` and updates `aria-checked`; category/print controls have labels/aria text. Unit button exposes `:focus-visible` teal outline.
- A faithful recreation should preserve RTL with explicit LTR isolation for numeric strings rather than relying on browser bidi heuristics.

### 10.12 UI/UX replica acceptance criteria

Treat the recreation as functionally/visually aligned only when these match: Urdu RTL reading order; the same four dashboard metrics and responsive order; sticky toolbar/checkout behavior; eight-column scrollable main table with grouped identity/read mode and separate editing fields; all row states and semantic colors; every modal’s purpose and empty/focus states; the two print modes; five print columns and summary alignment; repeated print heading and page/template footer. Use the token table and dimensions above as baseline, and compare at the same viewport and browser with the same fonts loaded.

## 11. Data integrity، escaping اور error paths

- User text کو `innerHTML` میں ڈالنے سے پہلے `escapeHTML()` استعمال کریں؛ بہتر ممکنہ تبدیلی `textContent`/DOM creation ہے، مگر موجودہ render templates میں escaping ایک security/integrity invariant ہے۔
- Item ID uniqueness ضروری ہے؛ new/clone flows محفوظ `nextItemId` استعمال کرتے ہیں، اسے delete کے بعد کم نہ کریں۔ Backup میں duplicate IDs ہوں تو selection اور row actions ambiguous ہو سکتے ہیں۔ پرانے state میں counter آنے سے پہلے حذف شدہ آخری ID معلوم نہیں ہو سکتی؛ migration صرف محفوظ آئٹمز کے max ID سے counter شروع کر سکتی ہے۔
- `normalizeItemUnits()` unknown quantity unit پر throw کرتا ہے؛ invalid backup import catch block میں alert دکھاتا ہے۔ Import کو silently old unit guess نہیں کرنی چاہیے۔
- Quantity `qty` اور `qtyUnit` ایک ہی record میں ساتھ update کریں؛ صرف unit label بدلنے سے rate calculation غلط ہو سکتی ہے۔ Unit conversion کے لیے `convertMeasurement()` استعمال کریں۔
- Rate, qty, selection، category، budget وغیرہ کی state update کے بعد `saveState()` call ہونا چاہیے۔ Template writes `saveTemplates()` سے الگ ہیں۔
- Reset یا restore میں budget input DOM value اور `state.budget` دونوں کو sync رکھیں۔
- Print changes میں summary cells، table colgroup، header cells، generated row cells اور category `colspan` کو ایک ہی وقت میں update کریں۔

## 12. موجودہ حدود اور آئندہ agent کے لیے اہم فرق

1. یہ full-stack/server app نہیں؛ persistence browser profile/origin سے بندھی ہے۔ Local preview URL بدلنے سے localStorage namespace بھی بدل سکتا ہے۔
2. Backup file app میں خودکار import نہیں ہوتی۔
3. 1000 threshold normalization کو ordinary quantity typing پر لاگو سمجھنا غلط ہوگا؛ code quantity value کو selected unit کے ساتھ store کرتا ہے۔
4. `sizeVal/sizeUnit` package identity ہیں؛ `qty/qtyUnit` خریداری کی مقدار ہیں۔ دونوں fields کو کبھی ایک نہ کریں۔
5. Print output الگ DOM generator سے آتا ہے، main screen table سے نہیں۔
6. `activeTemplateName` state/backup میں persistent نہیں؛ template selection map persistent ہے۔
7. Category filtering screen presentation بدلتی ہے؛ global selected total ہر selected item سے calculate ہوتا ہے۔
8. Rates integer rounded amount تک جا سکتے ہیں؛ fractional price/currency policy موجودہ code میں نہیں۔
9. Project میں automated test suite/build pipeline موجود نہیں؛ JS syntax check یا manual verification ہی current local workflow ہے۔

## 13. آئندہ تبدیلی کے لیے AI/developer protocol

اس پروجیکٹ کے owner کی مستقل ہدایت کے مطابق code بدلنے سے پہلے:

1. متعلقہ HTML، CSS، event handler، state shape اور backup/print paths پڑھیں؛ صرف visible UI سے اندازہ نہ لگائیں۔
2. اردو میں مختصر scope/summary، متاثر ہونے والے flows اور proposed behavior لکھیں اور owner کی explicit approval لیں۔
3. approval کے بعد کم سے کم متعلقہ code بدلیں؛ اپنی طرف سے unrelated redesign، data rewrite یا old backup modification نہ کریں۔
4. state schema/units/rate formula، item IDs اور backup compatibility کو محفوظ رکھیں یا migration واضح بنائیں۔
5. Inline script syntax، static IDs/references، affected render variants، import/export اور print modes validate کریں۔ UI/print تبدیلی کے لیے browser preview، edit mode اور PDF/print result بھی دیکھیں؛ اگر visual check ممکن نہ ہو تو اسے صاف بتائیں۔
6. آخر میں changed file، behavior، validation اور کوئی باقی limitation بتائیں۔

### تبدیلی کی quick impact map

| تبدیلی | لازمی code paths کی review |
|---|---|
| Unit/quantity | `UNIT_INFO`, `QUANTITY_TYPES`, normalization/conversion, add/edit handlers, amount formula, guide, backup import۔ |
| Rate/amount | `UNIT_MULTIPLIERS`, `getRateBasisLabel`, `getDisplayRate`, `calculateItemAmount`, dashboard/category/checkout/print totals, price history۔ |
| Item columns/identity | table header + category `colspan`, `renderTable()` read/edit templates, responsive CSS, search, print table generator۔ |
| Budget cards | state default/load/save/import/reset, budget input listener, `updateCalculations()`, card order/styles۔ |
| Templates/selection | template schema, active runtime variable, selected map, storage key, backup behavior, print footer۔ |
| Print | print modal values, `buildDedicatedPrintDOM()`, summary/table colgroups, cell count, `colspan`, print CSS, page footer support۔ |
| Backup | payload metadata, filename timestamp, import wrapper/direct-state support, normalization, templates and budget restoration۔ |

## 14. Suggested manual smoke-check checklist

یہ checklist regression جانچ کے لیے ہے، خودکار test suite ہونے کا دعویٰ نہیں:

- پہلی بار app کھولنے پر default items، categories، budget `0` اور calculations load ہوں۔ Reload کے بعد state برقرار ہو۔
- 500g × Rs 400/kg = Rs 200؛ 0.5L × Rs 300/L = Rs 150؛ 3 عدد × Rs 50/عدد = Rs 150۔
- Gram↔kilogram اور ml↔liter switch میں physical amount برقرار رہے؛ count میں multiplier 1 رہے۔
- 500g کو 0.5kg میں تبدیل کرنا اور واپس 500g کرنا، unit label اور amount دونوں کے لحاظ سے درست ہو۔
- Budget، selected total اور negative remaining value؛ minus sign value کے بائیں طرف دکھے۔
- Search/category/selected filters؛ selection totals filtered-out selected items سمیت درست رہیں۔
- Single edit، bulk edit، clone، delete، drag reorder اور price history۔
- Delete کے بعد نئی/clone item پہلے سے محفوظ `nextItemId` لے؛ آخری ID delete ہونے پر بھی وہ دوبارہ استعمال نہ ہو۔ Reset اور اسی browser میں پرانا backup import کرنے سے counter کم نہ ہو۔
- Template create/load/update/delete اور app reload کے بعد active badge behavior۔
- JSON backup filename میں local date/time، export/import میں items/categories/selection/budget/templates، invalid unit کی واضح error۔
- Blank print میں rate/amount spaces اور total hidden؛ full print میں rates/subtotals/top total۔ Multi-page A4 میں repeated table heading، footer، long Urdu identity اور summary alignment۔

---

**دستاویز کی بنیاد:** workspace میں موجود `index.html` اور `Grocery_Planner_v29_2026-09-23_17-23-48.json` backup کا source-level مطالعہ، 2026-09-23۔ مکمل default catalog `index.html` میں ہے؛ اس دستاویز میں catalog کی ساخت اور aggregate counts درج ہیں، تمام 111 product rows نقل نہیں کیے گئے۔
