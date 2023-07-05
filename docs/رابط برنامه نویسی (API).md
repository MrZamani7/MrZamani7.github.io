---
layout: default
title: رابط برنامه نویسی (API)
nav_order: 19
---

# رابط برنامه نویسی (API)
## بررسی اجمالی
رابط برنامه نویسی زبیکس به شما این امکان را می دهد که به وسیله برنامه نویسی، پیکربندی زبیکس را دریافت و اصلاح کنید و دسترسی به داده های تاریخی را فراهم می کند. به طور گسترده برای موارد زیر استفاده می شود:

- ساخت برنامه کاربردی جدید برای کار با زبیکس؛
- یکپارچگی زبیکس با یک نرم افزار شخص ثالث؛
- اتوماسیون کارهای روتین.

رابط برنامه نویسی زبیکس بر پایه HTTP می باشد و به عنوان بخشی از رابط کاربری وب زبیکس منتشر شده است. از پروتکل JSON-RPC نسخه 2.0 استفاده می کند که این بدین معنی است:

- رابط برنامه نویسی از مجموعه متدهای (methods) مجزایی تشکیل شده است.
- درخواست ها و پاسخ های رد و بدل شده بین کلاینت و API با فرمت JSON کدگذاری شده اند.

## ساختار
رابط برنامه نویسی از متدهایی تشکیل شده است که در گروه هایی به صورت APIهایی مجزا دسته بندی شده اند. هر کدام از متدها کار مشخصی انجام می دهد. برای مثال host.create که به گروه host API تعلق دارد برای ایجاد هاست جدید استفاده می شود. سابقا این APIهای زبیکس گاهی اوقات به عنوان "کلاس" نامیده می شوند.

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
The following section will walk you through some examples of usage in a greater detail.

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
#### به وسیله هدر "Authorization"
تمامی درخواست ها یک احراز هویت و یا یک توکن نیاز دارند. شما می توانید اعتبارسنجی را از طریق هدر "Authorization" فراهم کنید:

```js
curl --request POST \
  --url 'https://example.com/zabbix/api_jsonrpc.php' \
  --header 'Authorization: Bearer 0424bd59b807674191e7d77572075f33'
```

#### به وسیله ویژگی "auth"
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

By Zabbix cookie
A "zbx_session" cookie is used to authorize an API request from Zabbix UI performed using JavaScript (from a module or a custom widget).

Retrieving hosts
Now you have a valid user authentication token that can be used to access the data in Zabbix. For example, you can use the host.get method to retrieve the IDs, host names and interfaces of all the configured hosts:

Request:

curl --request POST \
  --url 'https://example.com/zabbix/api_jsonrpc.php' \
  --header 'Authorization: Bearer ${AUTHORIZATION_TOKEN}' \
  --header 'Content-Type: application/json-rpc' \
  --data @data.json
data.json is a file that contains a JSON query. Instead of a file, you can pass the query in the --data argument.
data.json

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
The response object will contain the requested data about the hosts:

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
For performance reasons it is always recommended to list the object properties you want to retrieve. Thus, you will avoid retrieving everything.
Creating a new item
Now, create a new item on the host "Zabbix server" using the data you have obtained from the previous host.get request. This can be done using the item.create method:

curl --request POST \
  --url 'https://example.com/zabbix/api_jsonrpc.php' \
  --header 'Authorization: Bearer ${AUTHORIZATION_TOKEN}' \
  --header 'Content-Type: application/json-rpc' \
  --data '{"jsonrpc":"2.0","method":"item.create","params":{"name":"Free disk space on /home/joe/","key_":"vfs.fs.size[/home/joe/,free]","hostid":"10084","type":0,"value_type":3,"interfaceid":"1","delay":30},"id":3}'
A successful response will contain the ID of the newly created item, which can be used to reference the item in the following requests:

{
    "jsonrpc": "2.0",
    "result": {
        "itemids": [
            "24759"
        ]
    },
    "id": 3
}
The item.create method as well as other create methods can also accept arrays of objects and create multiple items with one API call.
Creating multiple triggers
Thus, if create methods accept arrays, you can add multiple triggers, for example, this one:

curl --request POST \
  --url 'https://example.com/zabbix/api_jsonrpc.php' \
  --header 'Authorization: Bearer ${AUTHORIZATION_TOKEN}' \
  --header 'Content-Type: application/json-rpc' \
  --data '{"jsonrpc":"2.0","method":"trigger.create","params":[{"description":"Processor load is too high on {HOST.NAME}","expression":"last(/Linux server/system.cpu.load[percpu,avg1])>5",},{"description":"Too many processes on {HOST.NAME}","expression":"avg(/Linux server/proc.num[],5m)>300",}],"id":4}'
The successful response will contain the IDs of the newly created triggers:

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
Updating an item
Enable an item by setting its status to "0":

curl --request POST \
  --url 'https://example.com/zabbix/api_jsonrpc.php' \
  --header 'Authorization: Bearer ${AUTHORIZATION_TOKEN}' \
  --header 'Content-Type: application/json-rpc' \
  --data '{"jsonrpc":"2.0","method":"item.update","params":{"itemid":"10092","status":0},"id":5}'
The successful response will contain the ID of the updated item:

{
    "jsonrpc": "2.0",
    "result": {
        "itemids": [
            "10092"
        ]
    },
    "id": 5
}
The item.update method as well as other update methods can also accept arrays of objects and update multiple items with one API call.
Updating multiple triggers
Enable multiple triggers by setting their status to "0":

curl --request POST \
  --url 'https://example.com/zabbix/api_jsonrpc.php' \
  --header 'Authorization: Bearer ${AUTHORIZATION_TOKEN}' \
  --header 'Content-Type: application/json-rpc' \
  --data '{"jsonrpc":"2.0","method":"trigger.update","params":[{"triggerid":"13938","status":0},{"triggerid":"13939","status":0}],"id":6}'
The successful response will contain the IDs of the updated triggers:

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
This is the preferred method of updating. Some API methods, such as the host.massupdate allow to write a simpler code. However, it is not recommended to use these methods as they will be removed in the future releases.
Error handling
Up to the present moment, everything you have tried has worked fine. But what would happen if you tried making an incorrect call to the API? Try to create another host by calling host.create but omitting the mandatory groups parameter:

curl --request POST \
  --url 'https://example.com/zabbix/api_jsonrpc.php' \
  --header 'Authorization: Bearer ${AUTHORIZATION_TOKEN}' \
  --header 'Content-Type: application/json-rpc' \
  --data '{"jsonrpc":"2.0","method":"host.create","params":{"host":"Linux server","interfaces":[{"type":1,"main":1,"useip":1,"ip":"192.168.3.1","dns":"","port":"10050"}]},"id":7}'
The response will then contain an error message:

{
    "jsonrpc": "2.0",
    "error": {
        "code": -32602,
        "message": "Invalid params.",
        "data": "No groups for host \"Linux server\"."
    },
    "id": 7
}
If an error has occurred, instead of the result property, the response object will contain the error property with the following data:

code - an error code;
message - a short error summary;
data - a more detailed error message.
Errors can occur in various cases, such as, using incorrect input values, a session timeout or trying to access non-existing objects. Your application should be able to gracefully handle these kinds of errors.

API versions
To simplify API versioning, since Zabbix 2.0.4, the version of the API matches the version of Zabbix itself. You can use the apiinfo.version method to find out the version of the API you are working with. This can be useful for adjusting your application to use version-specific features.

Zabbix guaranties feature backward compatibility inside a major version. When making backward incompatible changes between major releases, Zabbix usually leaves the old features as deprecated in the next release, and only removes them in the release after that. Occasionally, Zabbix may remove features between major releases without providing any backward compatibility. It is important that you never rely on any deprecated features and migrate to newer alternatives as soon as possible.

You can follow all the changes made to the API in the API changelog.
Further reading
Now, you have enough knowledge to start working with the Zabbix API, however, do not stop here. For further reading you are advised to have a look at the list of available APIs.