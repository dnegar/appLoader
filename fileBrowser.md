**برنامه‌ی کاوشگر فایل**

- **نویسنده**: احمد کنی
- **تاریخ**: 4 اسفند ۱۴۰۳  
- **وضعیت**: ~~پیش‌نویس~~/در حال بررسی/~~تایید شده~~

---
## **خلاصه**

این برنامه به عنوان یک برنامه‌ی نوشته شده با pure javascript و با استفاده از قابلیت‌های service worker و git، این امکان را به کاربر می‌دهد تا با استفاده از این کاوشگر فایل، اسناد را ایجاد، ویرایش و یا حذف کند.
این برنامه مانند یک native file explorer کار می‌کند و بدون اینکه کاربر را با قابلیت‌های git مواجه کند، امکان استفاده از قابلیت‌های git را به او می‌دهد.
در پیاده‌سازی این برنامه، سعی می‌کنیم تمام APIهای موجود در استاندارد POSIX را ایجاد کنیم و به اصطلاح یک برنامه‌ی سازگار با استاندارد POSIX باشد.

---

## **سامانه‌ی فایل‌بندی KFS**

برای این برنامه نیاز به استفاده از چند سامانه وجود دارد. اولین و اساسی‌ترین سامانه، سامانه‌ی فایل‌بندی یا file system است. این سامانه باید طوری باشد که بتواند به تنهایی و بدون هیچ وابستگی‌ای به ما امکان کار کردن با فایل‌ها را در محیط مرورگر بدهد.
این سامانه یک سامانه مشابه node fs خواهد بود که به ما امکان کار کردن با فایل‌ها را در محیط‌های مختلف مانند حافظه‌ی داخلی، indexed db، cache، دیسک و سایر حافظه‌ها می‌دهد، همچنین استاندارد مورد استفاده در KFS استاندارد POSIX است.
از این جهت که ما با حافظه‌ها و محیط‌های مختلفی درگیر هستیم نیاز است تا یک fs مجازی مشابه چیزی که در لینوکس وجود دارد ایجاد شود تا این vfs به عنوان یک interface میان ما و fs های متفاوت وجود داشته باشد و عملاً دسترسی ما به خود fs backend ها را محدود و پیچیدگی کار با fs های مختلف را دور از دید ما حل می‌کند و به ما از طریق یک interface به نام vfs امکان کار با fs های مختلف را می‌دهد.
همچنین برای کار با fs های متفاوت و سرعت بخشیدن به ارتباط‌گیری با آنها و مدیریت آنها نیاز به ایجاد مفهوم mount کردن در این vfs داریم.

### **رابط VFS**

این رابط به صورت یک کلاس در کد جاوااسکریپت تعریف می‌شود که وظیفه دارد متدهای اصلی fs ها را تعریف کند و ارتباط میان آنها، نحوه‌ی کار آنها با یکدیگر و فرآیندهای اساسی را برای آنها تعریف کند.
سعی می‌کنیم در این VFS تمام توابع آمده در POSIX filesystem API را ایجاد کنیم و در صورتی که حتی امکان عملیات واقعی و در سطح محل ذخیره را نداریم این عملیات را پشتیبانی کنیم و این توابع را داشته باشیم.
این توابع به شرح زیر هستند که از شرح و تفصیل آنها در این سند پرهیز می‌کنیم. مستندات مربوط به محتوای این توابع در مستندات فنی خواهند آمد:

- [_fs_errno()](https://docs.silabs.com/micrium/5.11.0/micrium-file-system-api/03-file-system-posix-api#fs-errno)
- [fs_perror()](https://docs.silabs.com/micrium/5.11.0/micrium-file-system-api/03-file-system-posix-api#fs-perror)
- [fs_chdir()](https://docs.silabs.com/micrium/5.11.0/micrium-file-system-api/03-file-system-posix-api#fs-chdir)
- [fs_getcwd()](https://docs.silabs.com/micrium/5.11.0/micrium-file-system-api/03-file-system-posix-api#fs-getcwd)
- [fs_opendir()](https://docs.silabs.com/micrium/5.11.0/micrium-file-system-api/03-file-system-posix-api#fs-opendir)
- [fs_closedir()](https://docs.silabs.com/micrium/5.11.0/micrium-file-system-api/03-file-system-posix-api#fs-closedir)
- [fs_readdir_r()](https://docs.silabs.com/micrium/5.11.0/micrium-file-system-api/03-file-system-posix-api#fs-readdir-r)
- [fs_mkdir()](https://docs.silabs.com/micrium/5.11.0/micrium-file-system-api/03-file-system-posix-api#fs-mkdir)
- [fs_remove()](https://docs.silabs.com/micrium/5.11.0/micrium-file-system-api/03-file-system-posix-api#fs-remove)
- [fs_rename()](https://docs.silabs.com/micrium/5.11.0/micrium-file-system-api/03-file-system-posix-api#fs-rename)
- [fs_rmdir()](https://docs.silabs.com/micrium/5.11.0/micrium-file-system-api/03-file-system-posix-api#fs-rmdir)
- [fs_stat()](https://docs.silabs.com/micrium/5.11.0/micrium-file-system-api/03-file-system-posix-api#fs-stat)
- [fs_fopen()](https://docs.silabs.com/micrium/5.11.0/micrium-file-system-api/03-file-system-posix-api#fs-fopen)
- [fs_fclose()](https://docs.silabs.com/micrium/5.11.0/micrium-file-system-api/03-file-system-posix-api#fs-fclose)
- [fs_fread()](https://docs.silabs.com/micrium/5.11.0/micrium-file-system-api/03-file-system-posix-api#fs-fread)
- [fs_fwrite()](https://docs.silabs.com/micrium/5.11.0/micrium-file-system-api/03-file-system-posix-api#fs-fwrite)
- [fs_ftruncate()](https://docs.silabs.com/micrium/5.11.0/micrium-file-system-api/03-file-system-posix-api#fs-ftruncate)
- [fs_feof()](https://docs.silabs.com/micrium/5.11.0/micrium-file-system-api/03-file-system-posix-api#fs-feof)
- [fs_ferror()](https://docs.silabs.com/micrium/5.11.0/micrium-file-system-api/03-file-system-posix-api#fs-ferror)
- [fs_clearerr()](https://docs.silabs.com/micrium/5.11.0/micrium-file-system-api/03-file-system-posix-api#fs-clearerr)
- [fs_fgetpos()](https://docs.silabs.com/micrium/5.11.0/micrium-file-system-api/03-file-system-posix-api#fs-fgetpos)
- [fs_fsetpos()](https://docs.silabs.com/micrium/5.11.0/micrium-file-system-api/03-file-system-posix-api#fs-fsetpos)
- [fs_fseek()](https://docs.silabs.com/micrium/5.11.0/micrium-file-system-api/03-file-system-posix-api#fs-fseek)
- [fs_ftell()](https://docs.silabs.com/micrium/5.11.0/micrium-file-system-api/03-file-system-posix-api#fs-ftell)
- [fs_rewind()](https://docs.silabs.com/micrium/5.11.0/micrium-file-system-api/03-file-system-posix-api#fs-rewind)
- [fs_fileno()](https://docs.silabs.com/micrium/5.11.0/micrium-file-system-api/03-file-system-posix-api#fs-fileno)
- [fs_fstat()](https://docs.silabs.com/micrium/5.11.0/micrium-file-system-api/03-file-system-posix-api#fs-fstat)
- [fs_flockfile()](https://docs.silabs.com/micrium/5.11.0/micrium-file-system-api/03-file-system-posix-api#fs-flockfile)
- [fs_ftrylockfile()](https://docs.silabs.com/micrium/5.11.0/micrium-file-system-api/03-file-system-posix-api#fs-ftrylockfile)
- [fs_funlockfile()](https://docs.silabs.com/micrium/5.11.0/micrium-file-system-api/03-file-system-posix-api#fs-funlockfile)
- [fs_setbuf()](https://docs.silabs.com/micrium/5.11.0/micrium-file-system-api/03-file-system-posix-api#fs-setbuf)
- [fs_setvbuf()](https://docs.silabs.com/micrium/5.11.0/micrium-file-system-api/03-file-system-posix-api#fs-setvbuf)
- [fs_fflush()](https://docs.silabs.com/micrium/5.11.0/micrium-file-system-api/03-file-system-posix-api#fs-fflush)
- [fs_asctime_r()](https://docs.silabs.com/micrium/5.11.0/micrium-file-system-api/03-file-system-posix-api#fs-asctime-r)
- [fs_ctime_r()](https://docs.silabs.com/micrium/5.11.0/micrium-file-system-api/03-file-system-posix-api#fs-ctime-r)
- [fs_localtime_r()](https://docs.silabs.com/micrium/5.11.0/micrium-file-system-api/03-file-system-posix-api#fs-localtime-r)
- [fs_mktime()](https://docs.silabs.com/micrium/5.11.0/micrium-file-system-api/03-file-system-posix-api#fs-mktime)

کلاس VFS به فرمت زیر خواهد بود که در این کلاس توابع سطح بالاتر آورده می‌شوند مثل توابع mount, unmount, Format, Instantiate و توابع دیگر مانند read, write, open, close, ... در کلاس‌های خصوصی که برای ذخیره‌سازهای متفاوت داریم تعریف می‌شوند تا برای هر کلاس در context همان کلاس تعریف شوند.

```
class VFS { 
	constructor(defaultFSType = null) {
		this.fsType = defaultFSType || detectFS();
		this.mounts = {};
	} 
	
	createFSInstance(fsType, options) {...}
	mount(path, fsInstance) {...} 
	unmount(path) {...} 
	format(path) {...}
	resolveFS(path) {...}
```

در کلاس vfs ما با استفاده از کلاس createFSInstance یکی از فضاهای حافظه را که قبل‌تر شرح دادیم، انتخاب می‌کنیم، سپس با استفاده از آن تمام عملیات دیگر را می‌توانیم انجام دهیم. به این صورت که هر موجودیت fs یا FS Instance متدهای read, write, delete, list, rename و ... را داراست و در context مربوط به خودش تعریف و عملیات آن تعریف شده است. با انتخاب آن موجودیت امکان استفاده از آن متدها و ارتباط‌گیری با هر fs را پیدا می‌کنیم.
این کلاس‌ها همان جایی است که ما توابع اساسی POSIX filesystem API را پیاده‌سازی می‌کنیم، همچنین بعضی توابع دیگر مانند listFiles که در API ذکر شده به صورت مستقیم وجود ندارد برای سهولت استفاده تعریف می‌شوند.
به طور مثال کلاس مربوط به memory به این صورت تعریف می‌شود:

```
class MemoryFS {
  constructor() {
    this.store = {};
  }

  async writeFile(path, content) {
    this.store[path] = content;
  }

  async readFile(path) {
    return this.store[path] || null;
  }

	...
	
  async deleteFile(path) {
    delete this.store[path];
  }

  async listFiles() {
    return Object.keys(this.store);
  }
}
```

این کلاس و هر کلاس مربوط به fs ها متدهای اساسی را پیاده‌سازی می‌کنند و امکان استفاده از fs های متفاوت را به ما می‌دهند.

#### **مفهوم mount کردن**

برای این که دسترسی سریع‌تری به مسیرهای موجود در یک fs داشته باشیم و بتوانیم با یک فایل در آن کار کنیم، از این مفهوم استفاده می‌کنیم. همچنین برای خالی کردن حافظه‌ی داخلی از اطلاعات اضافی‌ای که فعلا با آن کاری نداریم و همین‌طور کم کردن پیچیدگی و جلوگیری از خطا از مفهوم unmount که مخالف mount است استفاده می‌کنیم.
برای پیاده‌سازی این مفهوم از یک جدول و دو تابع استفاده می‌کنیم:
- جدول mount پیاده‌سازی مرتبط با یک object جاوااسکریپتی است: 

```
this.mounts = {
  "/cache": LocalStorageFS,  // Handles all files under "/cache"
  "/tmp": MemoryFS           // Handles all files under "/tmp"
};
```

- این ساختمان داده به ما کمک می‌کند تا در صورتی که یک آدرس را داشتیم و نیاز به تشخیص fs استفاده شده در آن داشتیم بتوانیم اطلاعات مهمی را از جدول زیر بگیریم:‌

| Path              | Matches Mount Point | Filesystem Used        |
| ----------------- | ------------------- | ---------------------- |
| `/cache/test.txt` | `/cache`            | LocalStorageFS         |
| `/tmp/notes.txt`  | `/tmp`              | MemoryFS               |
| `/global.txt`     | `No match`          | Default FS (IndexedDB) |
- همچنین توابع mount و unmount مقادیر fs را به شیء mounts اضافه و یا از آن کم می‌کنند. همچنین با استفاده از این توابع می‌توانیم آدرس‌ها و پارامترهای موقت را حذف و اضافه کنیم.

#### **عملیات‌ در vfs**

پس از تعریف vfs و همچنین کلاس‌های حافظه، اکنون می‌توانیم عملیات CRUD را در vfs تعریف کنیم و همچنین از آنها استفاده کنیم.
ابتدا برای استفاده از fs ها و همچنین تعریف عملیات در آنها نیاز داریم تا به فایل اصلی .git دسترسی داشته باشیم.
این کار در هر محیط به یک صورت انجام می‌شود که در کلاس‌های حافظه آنها را تعریف کرده‌ایم و سپس می‌توانیم روی این فایل اصلی عملیات CRUD را انجام دهیم که در سرفصل بعدی مفصلاً تشریح خواهد شد.

#### **پیاده‌سازی fs های متفاوت**

با توجه به اینکه کتابخانه‌ی lightning fs یک سری امکانات متفاوت برای استفاده از محل‌های ذخیره‌سازی متفاوت در نظر گرفته و این کتابخانه نیازهای ما را برای انواع دیتابیس اغنا می‌کند این کتابخانه را استفاده می‌کنیم.
اما با اینکه خود lightning fs یک vfs درونی دارد، ما vfs ای را که قبلا درباره‌ی آن صحبت شد استفاده می‌کنیم.
بر این اساس می‌توانیم کلاسی که برای memoryFs تعریف کرده بودیم را به این شکل درآوریم:
‍```
```
class MyCustomBackend {

	constructor() {
		this.files = new Map();
		}
	
	async saveSuperblock(superblock) {
		return this.files.set("!root", superblock);
		}
	
	async loadSuperblock() {
		return this.files.get("!root");
		}
	
	async readFile(filepath, opts) {
		if (!this.files.has(filepath)) {
			throw Object.assign(new Error("File not found"), { code: "ENOENT" });
		}
		return this.files.get(filepath);
		}
	
	async writeFile(filepath, data, opts) {
		this.files.set(filepath, data);
		}
	
	async unlink(filepath) {
		this.files.delete(filepath);
		}
	
	async stat(filepath) {
		if (!this.files.has(filepath)) {
			throw Object.assign(new Error("File not found"), { code: "ENOENT" });
		}
		return { type: "file", size: this.files.get(filepath).length };
		}
	}

const fs = new LightningFS("fs", { backend: new MyCustomBackend() });
```


---

## **عملیات CRUD**

برای پیاده سازی عملیات CRUD در سیستم، آنها را به شکل زیر با ترکیب عملیات fs با عملیات git انجام می‌دهیم:
1. **ایجاد (*C*) :** در این عمل، کاربر می‌تواند دو نوع فایل و پوشه را ایجاد کند، همچنین این عملیات عمل‌های پایه‌ای برگرفته از isomorphic git هستند که بدون بازکردن مسیرها و پوشه‌ها می‌توان آن‌ها را استفاده کرد. 
		**(همچنین قابل ذکر است که برای احراز هویت کاربر و امضای دیجیتال و همچنین پی بردن به هویت شخص ایجادکننده‌ی فایل می‌توانیم از commit's author استفاده کنیم.)
	- برای ایجاد فایل از این سلسله عملیات استفاده می‌کنیم: 
		- Git.WriteFileOnDotGit(filePath)
		- Git.commit(commitMessage : File *A* Created by User *A* with IP Address *1.1.1.1*) 
	- برای ایجاد پوشه از این سلسله عملیات استفاده می‌کنیم:
		- Git.mkdirRecursively(dirPath)
		- Git.commit(commitMessage : directory */A* Created by User *A* with IP Address *1.1.1.1*)

2. **خواندن (*R*):** در این عمل، کاربر محتوای یک فایل یا پوشه را می‌خواند:
	- برای خواندن محتوای یک فایل از تک عمل Git.readFileFromDotGit استفاده می‌کنیم، این عمل با استفاده از توابع پایه‌ای isomorphic git مانند readBlob قابل پیاده‌سازی است و کارکرد آن به این صورت است که فایل را از آخرین commit انجام شده می‌خواند.
	- برای خواندن محتوای یک پوشه از عمل Git.listFilesFromDotGit استفاده می‌کنیم این عمل نیز با استفاده از دستورات پایه است و لیست فایل‌ها را بدن بازکردن و خواندن فایل‌ها و پوشه‌ها از طریق دسترسی به درخت‌های ایجاد شده توسط گیت لیست می‌کند.
	
3. **به‌روزرسانی (*U*):** دو مدل به روزرسانی وجود دارد که به ترتیب سلسله عملیات مربتط با هرکدام را نیز می‌گوییم، این عملیات عمل‌های پایه‌ای برگرفته از isomorphic git هستند که بدون بازکردن مسیرها و پوشه‌ها می‌توان آن‌ها را استفاده کرد:
	- به روزرسانی محتوای یک فایل: 
		- Git.WriteFileOnDotGit(filePath)
		- Git.commit(commitMessage : File *A* Updated by User *A* with IP Address *1.1.1.1*)
	- به روزرسانی نام یک فایل یا پوشه:
		- Git.rename(oldname, newname)
		- Git.commit(commitMessage : File *A* renamed by User *A* with IP Address *1.1.1.1*)

4. **حذف (*D*):** این عمل برای دو نوع فایل و پوشه قابل انجام است:
	- برای حذف یک فایل از این سلسله عملیات استفاده می‌کنیم:
		- Git.remove(filePath)
		- Git.commit(commitMessage : File *A* removed by User *A* with IP Address *1.1.1.1*)


	- برای حذف یک پوشه از این سلسله عملیات استفاده می‌کنیم:
		- Git.removeRecursively(filePath)
		- Git.commit(commitMessage : directory *A* removed by User *A* with IP Address *1.1.1.1*)

این سلسله عملیات ها توسط توابع مرتبط با آنها پیاده خواهند شد.

---
## **فراداده یا MetaData**

در استاندارد POSIX ما به‌ازای هر یک از فایل‌ها، مسیرها و پوشه‌ها و همچنین اطلاعات سطح بالاتر شامل superblockها یک فراداده یا metadata را نیز ذخیره می‌کنیم، همچنین لایه‌ی مدیریت دسترسی را نیز به صورت یک فراداده‌ی مجزا در نظر می‌گیریم، چرا که نمی‌خواهیم به ازای هر تغییر در دسترسی ctime متفاوتی برای هر فراداده ایجاد شود. تمام این فراداده‌ها به صورت Git note روی object متناسب یا مرجع مناسب ذخیره می‌شوند. که این فراداده شامل این اطلاعات است:

|     نوع فراداده      | محل ذخیره note            |    معادل object در گیت     |                                                      محتوای فراداده                                                       |
| :------------------: | ------------------------- | :------------------------: | :-----------------------------------------------------------------------------------------------------------------------: |
|   فراداده‌ی Inode    | Attached to file Blob OID | Git Blob SHA (`objects/`)  | File mode (permissions), owner (UID/GID), size, timestamps (atime, mtime, ctime), file type (regular, directory, symlink) |
|   فراداده‌ی Dentry   | Attached to file Tree OID | Tree Objects (`objects/`)  |                   Directory structure, file names, references to inodes (blobs), execution permissions                    |
| فراداده‌ی SuperBlock | refs/notes/repo           | Repository (`.git` itself) |         Filesystem version, repository UUID, root tree reference, user ACL rules, repository-wide configurations          |
| Access Control (ACL) | refs/notes/acl            |           ندارد            |              Read/write/execute permissions for users/groups, admin list, restrictions on modifying metadata              |
برای واضح شدن اطلاعات بالا دستوراتی را به عنوان مثال برای هرکدام می‌آوریم که در سیستم گیت قابل پیاده‌سازی و اجرا است: 

Inode Metadata: ```
```
git notes add -f -m '{"inode": 12345678, "mode": "100644", "uid": 1000, "gid": 1000, "acl": {"user": "ahmad", "permissions": "rwx"}}' (file blob OID)
```

Dentry Metadata: ```
```
git notes add -f -m "{"dentry_id": 56789, "name": "mydir", "parent_inode": 12345}" (directory tree OID)
```

SuperBlock Metadata: ```
```
git notes --ref=repo add HEAD -m '{ "fs_type": "memory", "owner": "admin_user", "created_at": "2024-02-22T12:00:00Z", "default_acl": "user1:rwx,user2:rw-", "users": ["user1", "user2", "user3"], "acl_policy": "strict",}'
```

### **گروه و کاربر و سطوح کاربری**

قبل از ورود به سطوح کاربری باید بگوییم که گروه‌های مختلف کاربری اختیاری است و صرفاً برای استفاده‌ از قابلیت‌های مختلف مثل قابلیت شبکه، مدیریت سازمانی ایجاد می‌شوند.

**کاربر:** به استفاده کننده از این برنامه یک کاربر می‌گوییم.
**ادمین:** به کاربری که هنگام تنظیم سامانه به عنوان مدیر شناخته شده باشد ادمین می‌گوییم.
فرا ادمین: 
**حلقه:** به گروهی از کاربران که یک یا چند ادمین داشته باشند، یک حلقه می‌گوییم، نقش ادمین صرفاً نگه داشتن یک سری فراداده در سطح حلقه‌ی خودش است، همچنین هر کاربر می‌تواند یک ادمین باشد و این تصمیم صرفاً توسط تنظیم‌کننده‌ی سامانه یا ادمین قبلی قابل اتخاذ است.

همانطور که در استاندارد POSIX کاربر (User) و گروه (Group) وجود دارد، ما نیز از این قابلیت پشتیبانی می‌کنیم. برای مدیریت این داده‌ی اساسی که در سطحی بالاتر از مخزن وجود دارد نیاز به داده‌ای فراتر از سطح مخزن (فراداده‌ی مدیریتی یا Super Admin MetaData) داریم که در دست یک فرا ادمین یا گروه فرا-ادمین‌های سطح سازمان باشد و با هر بار نیاز یک کاربر به دسترسی به این داده (که نباید بیش از دو بار، یک بار در ابتدای mount کردن یک مخزن و یک بار هنگام unmount کردن برای چک کردن اینکه آیا تغییری در فراداده‌ی مدیریتی رخ داده یا خیر) این داده را از کاربران فرا ادمین می‌گیرد.
این داده صرفاً در سه سطح تعریف می‌شود و خود درون یک یا چند فایل وجود دارد:
- User List (قابل تعریف در سطح ادمین)
- Group List
- Global ACL Policies (قابل تعریف در سطح ادمین)
ارتباط با کاربران فراادمین برای تمام افراد درون شبکه امکان‌پذیر است و محدودیتی در این مورد وجود ندارد، اما فرا ادمین می‌تواند اینکه پاسخ آن کاربر را بدهد یا خیر انتخاب کند.
**کاربر فرا ادمین می‌تواند یک کلاینت همیشه روشن (یا همان سرور روی کلاینت) باشد**.
به صورت کلی نقش‌های موجود در این برنامه به این شکل تعریف می‌شوند:

| نقش       | سطح دسترسی                            | وظیفه                                                                                                                                                                                                                                                                                              | ضرورت                                                                    |
| --------- | ------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------ |
| فرا ادمین | دسترسی سطح سازمان (فراداده‌ی مدیریتی) | با دسترسی به فراداده‌ی مدیریتی می‌تواند گروه‌ها، نقش افراد و در صورت نیاز دسترسی‌های درون مخزن را ایجاد، حذف یا ویرایش کند. کنترل دسترسی مخزن توسط این کاربر صرفاً موقعی است که بخواهیم یک مخزن را در سطح مدیریتی دریافت و از آن استفاده کنیم مگرنه مخزن‌ها به صورت غیرمتمرکز محلی مدیریت می‌شوند. | وجود این سطح ضروری نیست و فقط برای ایجاد لایه‌ی سطح سازمان مورد نیاز است |
| ادمین     |                                       |                                                                                                                                                                                                                                                                                                    |                                                                          |
| کاربران   |                                       |                                                                                                                                                                                                                                                                                                    |                                                                          |
در صورتی که استفاده کننده بخواهد از قابلیت شبکه استفاده کند، نیاز است تا در یکی از سطوح ادمین یا فرا ادمین (این تصمیم تماماً باید در اختیار استفاده کننده باشد) فراداده‌ی کاربران را قرار دهد تا ارتباط بین کاربران به صورت P2P بتواند شکل بگیرد. 
### **امنیت فراداده‌ی دسترسی**


---
## **مخزن و مخازن تودرتو در KFS**



---

## **سامانه‌ی فایل‌بندی در یک نگاه**

این سامانه از سه بخش تشکیل می‌شود، اولین بخش و بالاترین سطح همان kfs یا فایل سیستیمی است که اعمال اصلی را برای کاربر اجرا می‌کند مانند read, write, delete, create و به عنوان بالاترین لایه امکان کار با داده را بدون ایجاد پیچیدگی برای کاربر ایجاد می‌کند این لایه با استفاده از اعمال گیت و با استفاده از دسترسی‌‌هایی که لایه‌های پایین‌تر به آن می‌دهند امکان دسترسی به فایل .git را به این لایه می‌دهند.
قسمت پایین تر همان vfs است که امکان اعمال عمل‌های لایه‌ی kfs را با لایه‌های پایین‌تر ایجاد می‌کند و بدون این که پیچیدگی‌های کار با انواع مختلف پایگاه داده و محل‌های ذخیره‌سازی را به این لایه بکشاند آن ها را ملزم به استفاده از یک استاندارد می‌کند. برای این لایه کتابخانه‌ی lightning fs انتخاب ماست.
لایه‌ی پایین تر لایه‌ای است که برای تعریف کار با هر کدام از ساختمان‌های حافظه نیاز داریم و این لایه ارتباط اصلی را با بخش ذخیره برقرار می‌کند.

---

## **همگام‌سازی با ریموت**

برای همگام‌سازی داده‌ی محلی با داده‌ی ریموت چند گزینه در اختیار کاربر قرار می‌دهیم که می‌تواند از بین آنها یکی را برای استراتژی همگام‌سازی خود انتخاب کند:

1. همگام‌سازی دستی : به این صورت است که کاربر همگام‌سازی  را تنها هنگامی انجام می‌دهد که خودش بخواهد و همگام‌سازی خودکار انجام نمی‌شود.
2. همگام‌سازی زمان‌محور: در این حالت همگام‌سازی های بازه‌ای انجام می‌شود، به این صورت که کاربر تعیین می‌کند پس از بازه‌های زمانی مشخصی این همگام‌سازی‌ها انجام شود.
3. همگام‌سازی رخدادمحور: در این مدل کاربر تعیین می‌کند که همگام‌سازی  پس از رخداد خاصی انجام شود، مثلا هنگام خروج کاربر از برنامه، یا هنگام ورود.

این تنظیمات را باید در یک قسمت مشخص برای کاربر ایجاد کنیم و اجازه‌ی ویرایش آن را به کاربر بدهیم.
این تنظیمات در یک فایل خاص در پوشه‌ی .git برای کاربر ایجاد  و از آنجا خوانده می‌شود.
