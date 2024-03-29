<div dir="rtl" align='justify'>

# آزمایش ۵ - ارتباط بین پردازه‌ای

## ۵.۱ مقدمه

در این جلسه از آزمایشگاه مکانیزم‌های مربوط به ارتباط و تبادل
پیام بین پردازه‌ها در سیستم عامل لینوکس را خواهیم آموخت.

### ۵.۱.۱ پیش‌نیازها

انتظار می‌رود که دانشجویان با موارد زیر از پیش آشنا باشند:

1. نحوه‌ی ایجاد پردازه‌ها در سیستم عامل لینوکس (مطالب جلسه ی
چهارم)
1. برنامه‌نویسی به زبان C++/C
1. دستورات پوسته‌ی لینوکس که در جلسات قبل فرا گرفته شده‌اند.

## ۵.۲ ارتباط بین پردازه‌ها

در جلسات قبل نحوه‌ی ایجاد پردازه‌های جدید را آموختیم. در
این جلسه سعی داریم روش‌های ارتباط میان این پردازه‌ها را
بررسی کنیم. مکانیزم‌های متعددی برای تبادل پیام بین
پردازه‌ها وجود دارد که در این جلسه دو روش
استفاده از Pipe و Signal را بررسی خواهیم کرد.
از جمله کاربردهای ارتباط بین پردازه‌ای می‌توان به همگام سازی
و انتقال اطلاعات اشاره کرد.

دستورات Pipe برای کاربران پوسته‌ی لینوکس آشنا هستند. برای مثال شما
می‌توانید برای مشاهده ی لیست پردازه‌هایی که در آن‌ها کلمه‌ی init
وجود دارد از دستور `ps aux | grep init`
 استفاده کنید. در
اینجا دو پردازه به کمک یک Pipe‌به هم متصل شده‌اند. نکته‌ای
که در اینجا مهم است آن است که این Pipe ایجاد شده تنها در
یک جهت (از پردازه ی اول به پردازه ی دوم) اطلاعات را جابه‌جا
می‌کند. به کمک فراخوانی‌های سیستمی می‌توان Pipe های دو سویه
و حلقوی نیز ایجاد کرد.

## ۵.۳ شرح آزمایش

### ۵.۳.۱ ایجاد یک Pipe یک‌سویه

* (آ) برای ایجاد Pipe‌ یک سویه در سیستم عامل لینوکس از
فراخوانی سیستمی `pipe` استفاده می‌شود. به کمک
دستور `man 2 pipe` خلاصه‌ای از نحوه‌ی کار آن را ملاحظه کنید.
* (ب) به کمک کد زیر یک Pipe‌ ایجاد کنید.

<div dir="ltr" >
        
```c
int fd[2];
int res = pipe(fd);
```
        
</div>
        
دستور `pipe` در اینجا دو File Descriptor ایجاد می کند
(آرایه ی fd). یکی از آن‌ها برای خواندن و دیگری برای نوشتن
مورد استفاده قرار خواهد گرفت. `fd[0]` برای خواندن
و `fd[1]` برای نوشتن خواهد بود.
* (ج) تا اینجا تنها یک پردازه داریم و می‌توان شمای
کلی fdهای ایجاد شده را در شکل ۵.۱ نشان داد:

![شمای کلی Pipe با یک پردازه](./image/5/5-1.png)

هر چیزی که بر روی `fd[1]` نوشته شود، قابل خواندن با `fd[0]`
خواهد بود. حال توجه کنید که در صورتی که عملیات fork انجام
گیرد، پردازه‌ی فرزند، File Descriptor های پدر را به ارث
خواهد برد. بنابراین، بعد از انجام شدن عملیات `fork`
و ایجاد پردازه ی فرزند، ساختار بالابه شکل ۵.۲ در خواهد آمد.

![شمای کلی Pipe با چند پردازه](./image/5/5-2.png)

مشکل مهمی که در اینجا با آن مواجه هستیم آن است که در صورتی
که هر دو پردازه بخواهند بر روی Pipe بنویسند و یا از آن
بخوانند، به دلیل اینکه تنها یک بافر مشترک داریم، رفتار سیستم
قابل پیش بینی نخواهد بود. در این حالت یک پردازه ممکن است
داده‌ای که خودش بر روی Pipe قرار داده است را بخواند!
بنابراین نیاز است که یک طرف تنها بر روی Pipe‌ بنویسد و یک
طرف تنها از آن بخواند. برای مثال فرض کنید پردازه‌ی فرزند
قصد خواندن از Pipe‌ و پردازه‌ی والد قصد نوشتن بر روی آن را
دارد. به کمک فراخوانی سیستمی `close` ، پردازه والد `fd[0]`
خود را می‌بندد (زیرا قصد خواندن ندارد) و پردازه‌ی فرزند
نیز `fd[1]` را خواهد بست. به این ترتیب یک
ارتباط Half-Duplex بین این دو پردازه ایجاد می‌شود.

**فعالیت‌ها**

* به کمک توضیحات بالا و استفاده از فراخوانی‌های
سیستمی `write` و `read`، جمله‌ی `Hello World!` را از سمت
پردازه‌ی پدر به پردازه ی فرزند منتقل کرده و در پردازه‌ی فرزند
آن را چاپ کنید.

* همان‌طور که در جلسات پیش آموختیم، به کمک دستورات
خانواده ی `exec`، بعد از انجام `fork` می‌توان یک برنامه،
مثلا `ls` را اجرا نمود. به کمک دستورات `dup/dup2` برنامه‌ای
بنویسید که پردازه‌ی والد دستور `ls` و پردازه فرزند
دستور `wc` را اجرا کند و خروجی پردازه ی والد (دستور `ls`)
به عنوان ورودی به پردازه‌ی فرزند داده شود. 
  * (راهنمایی: یک Pipe‌ ایجاد کنید به نحوی که خروجی پردازه ی والد
ورودی Pipe‌ باشد و خروجی Pipe به عنوان ورودی پردازه ی فرزند
باشد. اینکار با استفاده از دستورات `dup/dup2‍` ممکن است.)
  * (راهنمایی ۲: خروجی برنامه ی `ls` در `stdout` قرار می‌گیرد که
یک File Descriptor با شماره ی ۱ است. ورودی برنامه `wc`
از `stdin` است که یک File Descriptor‌ با شماره ی ۰ است.)

* بررسی کنید که چطور ارتباطات تمام دو طرفه بین پردازه‌ها
داشته باشیم.

### ۵.۳.۲ سیگنال‌ها

بعضی اوقات نیاز است که برنامه‌ها بتوانند با برخی از شرایط
غیر قابل پیش بینی مواجه شده، آن‌ها را کنترل کنند.
برای مثال:

* درخواست بستن برنامه توسط کاربر به وسیله ی Ctrl + C
* رخ دادن خطا در محاسبات Floating Point
* مرگ پردازه ی فرزند

این رخدادها توسط سیستم عامل لینوکس شناخته می‌شوند و 
سیستم عامل با ارسال یک **سیگنال**، پردازه را از وقوع
آن‌ها آگاه می‌سازد. برنامه نویس می‌تواند این سیگنال‌ها را
نادیده بگیرد، یا در عوض با نوشتن کد آن‌ها را مدیریت و
کنترل نماید.

* به کمک دستور `man 7 signal` لیستی از سیگنال‌های موجود
در سیستم عامل لینوکس را ملاحظه کنید. سیگنال‌های زیر را
توضیح دهید.

SIGINT, SIGHUP, SIGSTOP, SIGCONT, SIGKILL
        
- [Information about signals](https://en.wikipedia.org/wiki/Signal_(IPC))
- [Nore information about Two signals: SIGSTOP and SIGCONT](https://major.io/2009/06/15/two-great-signals-sigstop-and-sigcont/)

* یک سیگنال ساده، سیگنال Alarm (SIGALRM) است. به کمک 
دستور `man` در مورد آن توضیح کوتاهی ارائه دهید.

* کد زیر به کمک این سیگنال نوشته شده است. آن را اجرا کرده
و در مورد کارکرد آن توضیح دهید.

<div dir="ltr" >
        
```c
#include <stdio.h>
#include <unistd.h>
int main() {
        alarm (5);
        printf ("Looping forever . . . \n");
        while (1);
        printf("This line should never be executed\n");
        return 0;
}
```
        
</div>

* به طور پیش فرض پردازه بعد از دریافت یکی از سیگنال‌های
تعریف شده، کشته می‌شود. به کمک تابع `sigaction`
می‌توان این رفتار را تغییر داد و کد مورد نظر برنامه نویس
را اجرا کرد. همچنین یک تابع دیگر به نام `pause`
وجود دارد که پردازه را تا زمانی که یک سیگنال دریافت کند،
متوقف می‌سازد (واضح است که این توابع از فراخوانی‌های سیستمی برای انجام کار خود استفاده می‌کنند). به کمک این دو تابع، برنامه‌ی بالا را
به گونه‌ای ویرایش کنید که بعد از دریافت سیگنال SIGALRM از
توقف خارج شود و خط آخر را در خروجی چاپ کند.

* برنامه‌ای بنویسید که در صورتی که کاربر کلید‌های Ctrl + C
را فشار دهد، برای بار اول خارج نشود
و پیامی برای فشار دادن دوباره ی آن‌ها چاپ کند. در دفعه ی دومِ رخ
دادن سیگنال، برنامه به پایان برسد.

</div>
