# CyberSecs - Final Project

## پروژه ی Web Path Scanner

ابزار اسکنر برای اسکن کردن فایل های درون یک دایرکتوری استفاده می شود و این کمک را می کند که به محتویات درون پوشه ها پی ببریم
<br/>
این را هم باید در نظر داشت که باید لیستی از لینک هایی که میخواهیم را داشته باشیم تا وقتی که با main domain کانکت میشوند بتوانیم سایت هایی که در حالت عادی بدست نمی آيند را پیدا کنیم

اصول اصلی برنامه ی این پروژه به این صورت است که برای هر سایت دو فایل scanned و  queued ایجاد میکند و درون پوشه ای با نام پروژه قرار می دهد 
و آن پوشه را درون پوشه ی result قرار می دهد.

فایل های اصلی این پروژه شامل فایل های general.py - spider.py - link_finder.py - main.py  می شوند که هر کدام را به ترتیب توضیح می دهیم.

## فایل general.py:

### ساخت پوشه با نام پروژه در صورت عدم وجود پوشه از قبل:

```

def create_project_directory(directory):
    if not os.path.exists(f'results/{directory}'):
        print('Creating project', directory)
        os.makedirs(f'results/{directory}')

```

### ساخت فایل های queue و scanned

‍‍‌```
```
def create_data_files(project_name, base_url):
    queue = f'results/{project_name}/queue.txt'
    scanned = f'results/{project_name}/scanned.txt'
    # check if file exits
    if not os.path.isfile(queue):
        # do not create an empty queue as it avoids the program to start :)
        write_file(queue, base_url)
    if not os.path.isfile(scanned):
        # create an empty scanned file
        write_file(scanned, '')

```

### ساخت فایل جدید

```
def write_file(path, data):
    f = open(path, 'w')
    f.write(data)
    # close the files to avoid data leakage
    f.close()
```

### اضافه کردن  لینک ها به فایل موجود

```
def append_file(path, data):
    with open(path, 'a') as file:
        # add '\n' to the end of each line
        file.write(data + '\n')
```


### پاک کردن محتوای فایل


```
def delete_file_content(path):
    with open(path, 'w'):
        pass
```

### خواندن فایل و تبدیل هر خط به یک آیتم از ست

```
def file_to_set(file_name):
    results = set()
    with open(file_name, 'rt') as f:
        for line in f:
            results.add(line.replace('\n', ''))
    return results
```

### حرکت درون ست و هر آیتم یک خط جدید درون فایل است


```
def set_to_file(links, file):
    delete_file_content(file)
    for link in sorted(links):
        append_file(file, link)
```

### پیدا کردن دامنه ی اصلی


```
def get_domain_name(url):
    try:
        # Todo: check the stu.ac.ir instead of licotab.com
        results = get_sub_domain_name(url).split('.')
        if len(get_sub_domain_name(url).split('.')) == 3:
            return results[-2] + '.' + results[-1]
        elif len(get_sub_domain_name(url).split('.')) >= 3:
            return results[-3] + '.' + results[-2]
        return results[-2] + '.' + results[-1]
    except Exception as e:
        print(e)
```

### پیدا کردن زیر دامنه ها


```
def get_sub_domain_name(url):
    try:
        return urlparse(url).netloc  # (network location)
    except Exception as e:
        print(e)
        return ''
```
## فایل spider.py:


### کلاس اصلی spider
```
class Spider:
    project_name = ''
    base_url = ''
    # to check for the right domain for scanning (to avoid simple misatakes to redirecting to youtube or other social media sites)
    domain_name = ''
    queue_file = ''
    scanned_file = ''
    queue = set()
    scanned = set()

    def __init__(self, project_name, base_url, domain_name):
        # all spiders must see one queue and scanned list 
        Spider.project_name = project_name
        Spider.base_url = base_url
        Spider.domain_name = domain_name
        Spider.queue_file = f'results/{Spider.project_name}/queue.txt'
        Spider.scanned_file = f'results/{Spider.project_name}/scanned.txt'
        self.boot()
        # this happens once as the other spiders are gonna ignore it
        self.scan_page('First spider', Spider.base_url)
```

### تابع بوت برای همه ی اسپایدر ها اجرا میشه و منحصر به یک اسپایدر خاص نیست

```
    def boot():
        create_project_directory(Spider.project_name)
        create_data_files(Spider.project_name, Spider.base_url)
        # The very first time that it boots up (or a spider is created), it's gonna get a list of updated lists and
        # save it as a set for faster operations (istead of saving in files which takes more time)
        Spider.queue = file_to_set(Spider.queue_file)
        Spider.scanned = file_to_set(Spider.scanned_file)
```

### اگر سایت اسکن نشده باشد و یعنی درون فایل اسکن شده ها نباشد، آن فایل را اسکن میکند

```
    def scan_page(thread_name, page_url):
        # if it is not craweled yet
        # print('--------------', Spider.scanned)
        if page_url not in Spider.scanned:
            print(thread_name, 'now scanning', page_url)
            print('Queue', str(len(Spider.queue)), '| Scanned', str(len(Spider.scanned)))
            Spider.add_links_to_queue(Spider.gather_links(page_url))
            # We update the sets for fast operations
            Spider.queue.remove(page_url)
            Spider.scanned.add(page_url)
            # Now we update the files
            Spider.update_files()
```

### اضافه کردن لینک ها به صف در صورت عدم وجود در اسکن شده ها 

```
    def add_links_to_queue(links):
        for url in links:
            if url in Spider.queue:
                continue
            if url in Spider.scanned:
                continue
            if Spider.domain_name not in url:
                continue
            Spider.queue.add(url)
```

### آپدیت فایل ها با اعمال تغییرات درون ست ها به فایل ها

```
    def update_files():
        set_to_file(Spider.queue, Spider.queue_file)
        set_to_file(Spider.scanned, Spider.scanned_file)
```
### جمع آوری لینک ها با دقت به امکان وجود ارور 

```
    def gather_links(page_url):
        html_string = ''
        # to handle weird server errors :)
        try:
            response = urlopen(page_url)
            # check if it is a natural html file
            if 'text/html' in response.getheader('Content-type'):
                # reads the raw response
                html_bytes = response.read()
                # convert bytes to actual strings
                html_string = html_bytes.decode('utf-8')
            scanner = LinkFinder(Spider.base_url, page_url)
            scanner.feed(html_string)

        except Exception as e:
            print(e)
            # we're gonna return an empty set
            return set()
        
        return scanner.page_links()
```

## link_finder.py:

### کلاس اصلی که از html parser ها ارث بری میکند

```
class LinkFinder(HTMLParser):
    def __init__(self, base_url, page_url):
        super().__init__()
        self.base_url = base_url
        self.page_url = page_url
        self.links = set()  # to store urls in here

    def error(self, message: str):
        pass

```
### تابع هندل کردن رشته ها

```
    def handle_starttag(self, tag: str, attrs):
        if tag == 'a':
            for (attribute, value) in attrs:
                if attribute == 'href':
                    # if it is a full url it's ok, otherwise we need to convert the relative url to full url
                    # the url join understands the difference
                    url = parse.urljoin(self.base_url, value)
                    self.links.add(url)
        else:
            pass
```
### برگرداندن لینک های درون صفحه

```
    def page_links(self):
        return self.links
```

## فایل main.py:

### ساخت نخ به تعداد مشخص شده 
```
def create_workers():
    for _ in range(NUMBER_OF_THREADS):
        t = threading.Thread(target=work)
        # Making the thread a daemon thread to make sure that it terminates when the main thread terminates
        t.daemon = True
        t.start()
```
### اجرای کار بعدی توی صف
```
def work():
    while True:
        url = queue.get()
        Spider.scan_page(thread_name=threading.current_thread().name, page_url=url)
        queue.task_done()
```
### هر آیتم درون صف یک job است
```
def create_jobs():
    for link in file_to_set(QUEUE_FILE):
        queue.put(link)
    # to avoid messing with each other
    queue.join()
    scan()
```
### چک کن که آیتمی توی لیست هست یا نه
```
def scan():
    queued_links = file_to_set(QUEUE_FILE)
    if len(queued_links) > 0:
        print(str(len(queued_links)), 'links in the queue')
        create_jobs()
```
### اضافی کردن لیستی از مواردی که میخواهیم سرپ بشن علاوه بر مواردی که به صورت اتوماتیک سرچ میشن
```
def append_list_to_queue(main_domain, random_links):
    if len(random_links) != 0:
        for link in random_links:
            append_file(QUEUE_FILE, f'{main_domain}{link}/')
```

## منابع:

- https://github.com/maurosoria/dirsearch
- https://docs.python.org/3/library/urllib.request.html
- https://icc-aria.ir/courses/%D9%BE%D8%A7%DB%8C%D8%AA%D9%88%D9%86-%D9%BE%DB%8C%D8%B4%D8%B1%D9%81%D8%AA%D9%87/episode/%D8%A8%D8%B1%D9%86%D8%A7%D9%85%D9%87-%D9%86%D9%88%DB%8C%D8%B3%DB%8C-%DA%86%D9%86%D8%AF-%D9%86%D8%AE%DB%8C-multi-threading