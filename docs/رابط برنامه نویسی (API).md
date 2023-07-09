---
layout: default
title: رابط برنامه نویسی (API)
nav_order: 19
---

# رابط برنامه نویسی (API)
## بررسی اجمالی
رابط برنامه نویسی زبیکس به شما این امکان را می دهد که به وسیله برنامه نویسی، پیکربندی زبیکس را دریافت و اصلاح کنید و نیز دسترسی به داده های تاریخی را فراهم می کند. API به طور گسترده برای موارد زیر استفاده می شود:

- ساخت برنامه کاربردی جدید برای کار با زبیکس؛
- یکپارچگی زبیکس با یک نرم افزار شخص ثالث؛
- اتوماسیون کارهای روتین.

رابط برنامه نویسی زبیکس بر پایه HTTP می باشد و به عنوان بخشی از رابط کاربری وب زبیکس منتشر شده است. از پروتکل JSON-RPC نسخه 2.0 استفاده می کند که این بدین معنی است:

- رابط برنامه نویسی از مجموعه متدهای (methods) مجزایی تشکیل شده است.
- درخواست ها و پاسخ های رد و بدل شده بین کلاینت و API با فرمت JSON کدگذاری شده اند.

## ساختار
رابط برنامه نویسی از متدهایی تشکیل شده است که در گروه هایی به صورت APIهایی مجزا دسته بندی شده اند. هر کدام از متدها کار مشخصی انجام می دهد. برای مثال host.create که به گروه host API تعلق دارد برای ایجاد هاست جدید استفاده می شود. سابقا این APIهای زبیکس گاهی اوقات به عنوان "کلاس" نامیده می شدند.

<dl><dt>
نکته: اکثر APIها دارای حداقل چهار متد هستند: get و create و update و delete به ترتیب برای دریافت، ایجاد، بروزرسانی و حذف اطلاعات استفاده می شوند؛ اما برخی از APIها ممکن است مجموعه کاملا متفاوتی از متدها را شامل شوند.
</dt></dl>

## انجام درخواست ها
به محض اینکه رابط کاربری زبیکس را راه اندازی کردید، می توانید با استفاده از HTTP requests رابط برنامه نویسی را صدا بزنید. برای این کار لازم است یک HTTP POST requests به فایل api_jsonrpc.php که در دایرکتوری رابط کاربری قرار گرفته است ارسال کنید. برای مثال اگر رابط کاربری در مسیر https://example.com/zabbix نصب شده است، یک HTTP request برای صدا زدن متد apiinfo.version شبیه مثال زیر است:

```js
curl --request POST \
  --url 'https://example.com/zabbix/api_jsonrpc.php' \
  --header 'Content-Type: application/json-rpc' \
  --data '{"jsonrpc":"2.0","method":"apiinfo.version","params":{},"id":1}'
```

لازم است که درخواست شامل یک هدر Content-Type با یکی از این مقادیر باشد:
- application/json-rpc
- application/json
- application/jsonrequest

یک درخواست شامل ویژگی های زیر است:

- jsonrpc - شماره نسخه پروتکل JSON-RPC که توسط API استفاده می شود (رابط برنامه نویسی زبیکس JSON-RPC نسخه 2.0 را اجرا می کند)؛
- method - متدی که فراخوانده می شود؛
- params - پارامترهایی که به متد ارسال می شود؛
- id - یک شناسه دلخواه برای درخواست.

اگر درخواست ارسال شده درست باشد، پاسخ ارسالی از طرف رابط برنامه نویسی باید شبیه زیر باشد:

```js
{
    "jsonrpc": "2.0",
    "result": "6.4.0",
    "id": 1
}
```

پاسخ به نوبه خود دارای ویژگی های زیر است:

- jsonrpc - نسخه پروتکل JSON-RPC؛
- result - داده های بازگردانده شده توسط متد؛
- id - شناسه درخواست مربوطه.

## نمونه هایی برای آشنایی با گردش کار

### احراز هویت
برای دسترسی به هر داده ای در زبیکس لازم است یکی از موارد زیر انجام شود:

- استفاده از توکن API موجود (ایجاد شده به وسیله رابط کاربری زبیکس یا با استفاده از Token API)؛
- استفاده از توکنی که به وسیله متد user.login به دست آمده باشد.

برای مثال جهت دستیابی به توکن ورود کاربر Admin، درخواست به فرم JSON شبیه زیر می باشد:

```js
curl --request POST \
  --url 'https://example.com/zabbix/api_jsonrpc.php' \
  --header 'Content-Type: application/json-rpc' \
  --data '{"jsonrpc":"2.0","method":"user.login","params":{"username":"Admin","password":"zabbix"},"id":1}'
```

در صورتی که اطلاعات اعتبارسنجی درست باشد پاسخ رابط برنامه نویسی شامل توکن احراز هویت کاربر خواهد بود:

```js
{
    "jsonrpc": "2.0",
    "result": "0424bd59b807674191e7d77572075f33",
    "id": 1
}
```

### روش های کسب مجوز
#### - به وسیله هدر "Authorization"
تمامی درخواست ها یک احراز هویت و یا یک توکن نیاز دارند. شما می توانید اعتبارسنجی را از طریق هدر "Authorization" فراهم کنید:

```js
curl --request POST \
  --url 'https://example.com/zabbix/api_jsonrpc.php' \
  --header 'Authorization: Bearer 0424bd59b807674191e7d77572075f33'
```

#### - به وسیله ویژگی "auth"
یک درخواست می تواند به وسیله ویژگی "auth" مجوز کسب کند.

<dl><dt>
توجه: ویژگی "auth" منسوخ شده است و در انتشارهای آینده حذف خواهد شد.
</dt></dl>

```js
curl --request POST \
  --url 'https://example.com/zabbix/api_jsonrpc.php' \
  --header 'Content-Type: application/json-rpc' \
  --data '{"jsonrpc":"2.0","method":"host.get","params":{"output":["hostid"]},"auth":"0424bd59b807674191e7d77572075f33","id":1}'
```

#### - به وسیله کوکی های زبیکس
یک کوکی (cookie) از "zbx_session" جهت مجوز دهی به درخواستِ رابط کاربری زبیکس که توسط جاوا اسکریپت پیاده سازی شده است مورد استفاده قرار می گیرد (از طریق یک ماژول یا یک ویجت سفارشی).

### دستیابی به هاست ها
حال با داشتن یک توکن معتبر می توان به داده های زبیکس دسترسی پیدا کرد. برای مثال با استفاده از متد host.get می توان IDها و نام هاست ها و اینترفیس های آن ها را دریافت کرد:

درخواست:

```js
curl --request POST \
  --url 'https://example.com/zabbix/api_jsonrpc.php' \
  --header 'Authorization: Bearer ${AUTHORIZATION_TOKEN}' \
  --header 'Content-Type: application/json-rpc' \
  --data @data.json
```
<dl><dt>
نکته: data.json فایلی شامل موارد درخواست به فرم JSON می باشد. در عوضِ استفاده از یک فایل، می توانید درخواست را به وسیله آرگومان data ارسال کنید.
</dt></dl>

محتویات فایل data.json:

```js
{
    "jsonrpc": "2.0",
    "method": "host.get",
    "params": {
        "output": [
            "hostid",
            "host"
        ],
        "selectInterfaces": [
            "interfaceid",
            "ip"
        ]
    },
    "id": 2
}
```

پاسخ شامل اطلاعات درخواستی از هاست ها خواهد بود:

```js
{
    "jsonrpc": "2.0",
    "result": [
        {
            "hostid": "10084",
            "host": "Zabbix server",
            "interfaces": [
                {
                    "interfaceid": "1",
                    "ip": "127.0.0.1"
                }
            ]
        }
    ],
    "id": 2
}
```

<dl><dt>
نکته: برای افزایش کارایی و بهبود عملکرد همیشه بهتر است که لیست ویژگی های مورد نیاز برای دریافت را وارد کنید و به این شکل از دریافت تمامی اطلاعات اجتناب نمایید.
</dt></dl>

### ایجاد یک item جدید
ایجاد یک آیتم جدید بر روی هاست "Zabbix server" با استفاده از اطلاعاتی که در درخواست قبلی بدست آوردید از طریق متد item.create امکان پذیر است:

```js
curl --request POST \
  --url 'https://example.com/zabbix/api_jsonrpc.php' \
  --header 'Authorization: Bearer ${AUTHORIZATION_TOKEN}' \
  --header 'Content-Type: application/json-rpc' \
  --data '{"jsonrpc":"2.0","method":"item.create","params":{"name":"Free disk space on /home/joe/","key_":"vfs.fs.size[/home/joe/,free]","hostid":"10084","type":0,"value_type":3,"interfaceid":"1","delay":30},"id":3}'
```

یک پاسخ موفقیت آمیز برای درخواست، شامل ID آیتم جدید خواهد بود که می تواند برای ارجاع از آن در درخواست پیش رو استفاده کرد:

```js
{
    "jsonrpc": "2.0",
    "result": {
        "itemids": [
            "24759"
        ]
    },
    "id": 3
}
```

<dl><dt>
نکته: متد item.create مانند سایر متد های create می تواند آرایه ای از اشیاء را دریافت و چندین آیتم را با یک بار فراخواندن API ایجاد کند.
</dt></dl>

### ایجاد چندین trigger
از آنجایی که متدهای create آرایه ها را نیز می پذیرند، مانند مثال زیر می توانید چندین trigger اضافه کنید:

```js
curl --request POST \
  --url 'https://example.com/zabbix/api_jsonrpc.php' \
  --header 'Authorization: Bearer ${AUTHORIZATION_TOKEN}' \
  --header 'Content-Type: application/json-rpc' \
  --data '{"jsonrpc":"2.0","method":"trigger.create","params":[{"description":"Processor load is too high on {HOST.NAME}","expression":"last(/Linux server/system.cpu.load[percpu,avg1])>5",},{"description":"Too many processes on {HOST.NAME}","expression":"avg(/Linux server/proc.num[],5m)>300",}],"id":4}'
```

یک پاسخ موفقیت آمیز برای این درخواست، شامل IDهای تریگرهای جدید خواهد بود:

```js
{
    "jsonrpc": "2.0",
    "result": {
        "triggerids": [
            "17369",
            "17370"
        ]
    },
    "id": 4
}
```

### بروزرسانی یک آیتم

فعال کردن یک آیتم با تنظیم کردن وضعیت آن بر روی "0":

```js
curl --request POST \
  --url 'https://example.com/zabbix/api_jsonrpc.php' \
  --header 'Authorization: Bearer ${AUTHORIZATION_TOKEN}' \
  --header 'Content-Type: application/json-rpc' \
  --data '{"jsonrpc":"2.0","method":"item.update","params":{"itemid":"10092","status":0},"id":5}'
```

یک پاسخ موفقیت آمیز برای این درخواست، شامل ID آیتم بروزرسانی شده خواهد بود:

```js
{
    "jsonrpc": "2.0",
    "result": {
        "itemids": [
            "10092"
        ]
    },
    "id": 5
}
```

<dl><dt>
نکته: متد item.update مانند سایر متد های update نیز می تواند آرایه ای از اشیاء را دریافت و چندین آیتم را با یک بار فراخواندن API بروزرسانی کند.
</dt></dl>

### بروزرسانی چندین تریگر
فعال کردن یک تریگر با تنظیم کردن وضعیت آن بر روی "0":

```js
curl --request POST \
  --url 'https://example.com/zabbix/api_jsonrpc.php' \
  --header 'Authorization: Bearer ${AUTHORIZATION_TOKEN}' \
  --header 'Content-Type: application/json-rpc' \
  --data '{"jsonrpc":"2.0","method":"trigger.update","params":[{"triggerid":"13938","status":0},{"triggerid":"13939","status":0}],"id":6}'
```

یک پاسخ موفقیت آمیز برای این درخواست، شامل IDهای تریگرهای بروزرسانی شده خواهد بود:

```js
{
    "jsonrpc": "2.0",
    "result": {
        "triggerids": [
            "13938",
            "13939"
        ]
    },
    "id": 6
}
```

<dl><dt>
نکته: این روش برای به روز رسانی ترجیح داده می شود. برخی متدها مانند host.massupdate اجازه نوشتن کدهای ساده تری می دهند. اگرچه استفاده از آن متدها پیشنهاد نمی شود زیرا در انتشارهای بعدی حذف خواهند شد.
</dt></dl>

### رسیدگی به خطا
ایجاد یک هاست دیگر با فراخوانی متد host.create اما این بار با حذف پارامترهای اجباری گروه ها:

```js
curl --request POST \
  --url 'https://example.com/zabbix/api_jsonrpc.php' \
  --header 'Authorization: Bearer ${AUTHORIZATION_TOKEN}' \
  --header 'Content-Type: application/json-rpc' \
  --data '{"jsonrpc":"2.0","method":"host.create","params":{"host":"Linux server","interfaces":[{"type":1,"main":1,"useip":1,"ip":"192.168.3.1","dns":"","port":"10050"}]},"id":7}'
```

پاسخ شامل پیام خطا خواهد بود:

```js
{
    "jsonrpc": "2.0",
    "error": {
        "code": -32602,
        "message": "Invalid params.",
        "data": "No groups for host \"Linux server\"."
    },
    "id": 7
}
```

اگر یک خطا رخ دهد، به جای ویژگی result، در پاسخ ویژگی error به همراه اطلاعات زیر وجود خواهد داشت:

- code - کد خطا؛
- message - خلاصه خطا؛
- data - توضیحات بیشتر در مورد خطا.

خطاها تحت شرایط مختلفی مانند استفاده از مقادیر ورودی نادرست، پایان مدت زمان نشست یا تلاش برای دسترسی به موارد ناموجود اتفاق می افتند. برنامه شما باید بتواند به خوبی این نوع خطاها را مدیریت کند.

## نسخه های API
برای ساده سازی نامگذاری نسخه های API، از ربیکس نسخه 2.0.4، شماره نسخه API با شماره نسخه زبیکس یکسان تعیین شد. با استفاده از متد apiinfo.version می توانید نسخه API که با آن کار می کنید به دست آورید. این مورد می تواند در هنگام تنظیم کردن برنامه تان برای استفاده از قابلیت های مختص به هر نسخه از زبیکس مفید واقع شود.

زبیکس در نسخه های اصلی (major) سازگاری با قابلیت های نسخه ی پیشین را ضمانت می کند. در صورت اعمال تغییرات ناسازگار با نسخه پیشین در یک انتشار اصلی، زبیکس معمولا ویژگی های قدیمی را به صورت منسوخ شده (deprecated) در انتشار جدید ارائه می کند و آن ها را در یک انتشار پس از آن حذف می کند. برخی اوقات ممکن است زبیکس ویژگی هایی را بدون رعایت سازگاری در انتشار جدید حذف کند. بنابراین مهم است که هیچگاه به ویژگی های منسوخ شده تکیه نکنید و هر چه سریع تر به جایگزین های جدید مهاجرت کنید.
