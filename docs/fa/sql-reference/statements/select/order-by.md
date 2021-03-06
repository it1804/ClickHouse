---
machine_translated: true
machine_translated_rev: 72537a2d527c63c07aa5d2361a8829f3895cf2bd
---

# ORDER BY {#select-order-by}

این `ORDER BY` بند شامل یک لیست از عبارات, که می تواند هر یک را با نسبت داده `DESC` (نزولی) یا `ASC` (صعودی) اصلاح که تعیین جهت مرتب سازی. اگر جهت مشخص نشده است, `ASC` فرض بر این است, بنابراین معمولا حذف. جهت مرتب سازی شامل یک عبارت واحد, نه به کل لیست. مثال: `ORDER BY Visits DESC, SearchPhrase`

ردیف که مقادیر یکسان برای لیست عبارات مرتب سازی خروجی در جهت دلخواه, که همچنین می تواند غیر قطعی شود (مختلف در هر زمان).
اگر منظور توسط بند حذف شده است, منظور از ردیف نیز تعریف نشده, و ممکن است غیر قطعی و همچنین.

## مرتب سازی مقادیر ویژه {#sorting-of-special-values}

دو روش برای `NaN` و `NULL` مرتب سازی سفارش:

-   به طور پیش فرض و یا با `NULLS LAST` تغییردهنده: ابتدا مقادیر, سپس `NaN` پس `NULL`.
-   با `NULLS FIRST` تغییردهنده: اول `NULL` پس `NaN`, سپس ارزش های دیگر.

### مثال {#example}

برای جدول

``` text
┌─x─┬────y─┐
│ 1 │ ᴺᵁᴸᴸ │
│ 2 │    2 │
│ 1 │  nan │
│ 2 │    2 │
│ 3 │    4 │
│ 5 │    6 │
│ 6 │  nan │
│ 7 │ ᴺᵁᴸᴸ │
│ 6 │    7 │
│ 8 │    9 │
└───┴──────┘
```

اجرای پرس و جو `SELECT * FROM t_null_nan ORDER BY y NULLS FIRST` برای دریافت:

``` text
┌─x─┬────y─┐
│ 1 │ ᴺᵁᴸᴸ │
│ 7 │ ᴺᵁᴸᴸ │
│ 1 │  nan │
│ 6 │  nan │
│ 2 │    2 │
│ 2 │    2 │
│ 3 │    4 │
│ 5 │    6 │
│ 6 │    7 │
│ 8 │    9 │
└───┴──────┘
```

هنگامی که اعداد ممیز شناور طبقه بندی شده اند, نان جدا از ارزش های دیگر هستند. صرف نظر از نظم مرتب سازی, نان در پایان. به عبارت دیگر برای مرتب سازی صعودی قرار می گیرند همانطور که بزرگتر از همه شماره های دیگر هستند در حالی که برای مرتب سازی کوچکتر از بقیه قرار می گیرند.

## پشتیبانی تلفیقی {#collation-support}

برای مرتب سازی بر اساس مقادیر رشته, شما می توانید میترا مشخص (مقایسه). مثال: `ORDER BY SearchPhrase COLLATE 'tr'` - برای مرتب سازی بر اساس کلمه کلیدی به ترتیب صعودی, با استفاده از الفبای ترکی, حساس به حروف, فرض کنید که رشته ها سخن گفتن-8 کد گذاری. تلفیق می تواند مشخص شود یا نه برای هر عبارت به منظور به طور مستقل. اگر مرکز کنترل و یا مرکز کنترل خارج رحمی مشخص شده است, تلفیقی بعد از مشخص. هنگام استفاده از برخورد, مرتب سازی است که همیشه غیر حساس به حروف.

ما فقط توصیه می کنیم با استفاده از تلفیق برای مرتب سازی نهایی تعداد کمی از ردیف, از مرتب سازی با تلفیقی کمتر موثر تر از مرتب سازی طبیعی با بایت است.

## پیاده سازی اطلاعات {#implementation-details}

رم کمتر استفاده می شود اگر به اندازه کافی کوچک [LIMIT](limit.md) علاوه بر مشخص شده است `ORDER BY`. در غیر این صورت, مقدار حافظه صرف متناسب با حجم داده ها برای مرتب سازی است. برای پردازش پرس و جو توزیع, اگر [GROUP BY](group-by.md) حذف شده است, مرتب سازی تا حدی بر روی سرور از راه دور انجام, و نتایج در سرور درخواست با هم ادغام شدند. این به این معنی است که برای مرتب سازی توزیع, حجم داده ها برای مرتب کردن می تواند بیشتر از مقدار حافظه بر روی یک سرور واحد.

در صورتی که رم به اندازه کافی وجود ندارد, ممکن است به انجام مرتب سازی در حافظه خارجی (ایجاد فایل های موقت بر روی یک دیسک). از تنظیمات استفاده کنید `max_bytes_before_external_sort` برای این منظور. اگر قرار است 0 (به طور پیش فرض), مرتب سازی خارجی غیر فعال است. اگر فعال باشد, زمانی که حجم داده ها برای مرتب کردن بر اساس تعداد مشخصی از بایت می رسد, اطلاعات جمع شده مرتب شده و ریخته را به یک فایل موقت. پس از همه داده ها خوانده شده است, تمام فایل های طبقه بندی شده اند با هم ادغام شدند و نتایج خروجی. فایل ها به نوشته `/var/lib/clickhouse/tmp/` دایرکتوری در پیکربندی (به طور پیش فرض, اما شما می توانید با استفاده از `tmp_path` پارامتر برای تغییر این تنظیم).

در حال اجرا یک پرس و جو ممکن است حافظه بیش از استفاده `max_bytes_before_external_sort`. به همین دلیل این تنظیم باید یک مقدار قابل توجهی کوچکتر از `max_memory_usage`. به عنوان مثال, اگر سرور شما 128 گیگابایت رم و شما نیاز به اجرای یک پرس و جو واحد, تنظیم `max_memory_usage` به 100 گیگابایت, و `max_bytes_before_external_sort` به 80 گیگابایت.

مرتب سازی خارجی کار می کند بسیار کمتر به طور موثر از مرتب سازی در رم.
