# CyberSecs - Final Project

## پروژه ی Web Path Scanner

ابزار اسکنر برای اسکن کردن فایل های درون یک دایرکتوری استفاده می شود و این کمک را می کند که به محتویات درون پوشه ها پی ببریم

اصول اصلی برنامه ی این پروژه به این صورت است که برای هر سایت دو فایل scanned و  queued ایجاد میکند و درون پوشه ای با نام پروژه قرار می دهد 
و آن پوشه را درون پوشه ی result قرار می دهد.

فایل های اصلی این پروژه شامل فایل های general.py - spider.py - link_finder.py - main.py  می شوند که هر کدام را به ترتیب توضیح می دهیم.
<br />
فایل general.py:
```
def create_project_directory(directory):
    if not os.path.exists(f'results/{directory}'):
        print('Creating project', directory)
        os.makedirs(f'results/{directory}')
```
<br />
شامل توابعع کمکی مورد نیاز برای ایجاد پروژه - فایل های مختلف - پوشه ها - نوشتن در فایل - ایجاد ست ها و خواندن آن ها - پاک کردن محتوبات فایل - پیدا کردن نام دامنه ی فایل ها و ... است.
<br />
<br />
فایل spider.py:
<br />
در این فایل کلاس spider و توابع مربوط به آن قرار دارند. در واقع اسپایدر ها وارد یک سایت می شوند و تمام فایل های درون آن سایت را پیدا می کنند. نکته ی مهم این است که برنامه به صورت همروند انجام می شود و به اضافه هر فایل درون سایت اصلی ی ک اسپایدر ایجاد می شود و زمانی که اسکن اون فایل خاص تمام شود از بین میروند.
کلاس اسپایدر دارای توابعی مانند boot برای شروع اسکن سایت - add_links_to_queue برای اضافی کردن فایل ها به - update_files - آپدیت فایل های queue  و  scanned -
gather_links پیدا کردن لینک ها درون فایل html
<br />
link_finder.py:
<br />
که شامل منطق اسکن کردن پروژه است و با ایجاد thread ها به اسکن سایت میپردازد 
<br />
<br />
main.py:
کنار هم قرار دارنده ی تمام فایل های برنامه و منطق پروژه و ایجاد کننده ی thread ها است.
<br />
شامل توابع: create_workers: برای ساخت نخ به تعداد thread های مشخص شده - create_jobs: ارسال فایل به لینک فاندر و پیدا کردن لینک ها (از لیست صف می آید)-
work: برای انجام پیدا کردن فایل ها
scan: برای اسکن کردن فایل های موجود درون فایل snanned
<br />
<br />
منابع:

- https://github.com/maurosoria/dirsearch
- https://docs.python.org/3/library/urllib.request.html
- https://icc-aria.ir/courses/%D9%BE%D8%A7%DB%8C%D8%AA%D9%88%D9%86-%D9%BE%DB%8C%D8%B4%D8%B1%D9%81%D8%AA%D9%87/episode/%D8%A8%D8%B1%D9%86%D8%A7%D9%85%D9%87-%D9%86%D9%88%DB%8C%D8%B3%DB%8C-%DA%86%D9%86%D8%AF-%D9%86%D8%AE%DB%8C-multi-threading