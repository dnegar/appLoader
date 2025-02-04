**برنامه‌ی کاوشگر فایل**

- **نویسنده**: احمد کنی
- **تاریخ**: 11 بهمن ۱۴۰۳  
- **وضعیت**: پیش‌نویس/~~در حال بررسی~~/~~تایید شده~~

---
## **خلاصه**

این برنامه به عنوان یک برنامه‌ی نوشته شده با pure javascript و با استفاده از قابلیت‌های service worker و git، این امکان را به کاربر می‌دهد تا با استفاده از این کاوشگر فایل، اسناد را ایجاد، ویرایش و یا حذف کند.
این برنامه مانند یک native file explorer کار می‌کند و بدون اینکه کاربر را با قابلیت‌های git مواجه کند، امکان استفاده از قابلیت‌های git را به او می‌دهد.

---

## **سامانه‌ی فایل‌بندی**

برای این برنامه نیاز به استفاده از چند سامانه وجود دارد. اولین و اساسی‌ترین سامانه، سامانه‌ی فایل‌بندی یا file system است. این سامانه باید طوری باشد که بتواند به تنهایی و بدون هیچ وابستگی‌ای در محیط مرورگر کار کند.
این سامانه یک سامانه مشابه node fs است که فایل‌ها را طبقه‌بندی و در پایگاه داده‌ی indexed db ذخیره کند.

---

## **عملیات CRUD**

برای پیاده سازی عملیات CRUD در سیستم، آنها را به شکل زیر با ترکیب عملیات fs با عملیات git انجام می‌دهیم:
1. ایجاد (*C*) : در این عمل، کاربر می‌تواند دو نوع فایل و پوشه را ایجاد کند. 
	- برای ایجاد فایل از این سلسله عملیات استفاده می‌کنیم: 
		- fs.WriteFile(filePath)
		- Git.addFile(filePath)
		- Git.commit(commitMessage : File *A* Created by User *A* with IP Address *1.1.1.1*) 
	- برای ایجاد پوشه از این سلسله عملیات استفاده می‌کنیم:
		- fs.mkdirRecursively(dirPath)
		- Git.addFile(dirPath)
		- Git.commit(commitMessage : directory */A* Created by User *A* with IP Address *1.1.1.1*)

2. خواندن (*R*): در این عمل، کاربر محتوای یک فایل یا پوشه را می‌خواند:
	- برای خواندن محتوای یک فایل از تک عمل fs.readfile استفاده می‌کنیم.
	- برای خواندن محتوای یک پوشه از عمل fs.listFiles استفاده می‌کنیم.
	
3. به‌روزرسانی (*U*): دو مدل به روزرسانی وجود دارد که به ترتیب سلسله عملیات مربتط با هرکدام را نیز می‌گوییم:
	- به روزرسانی محتوای یک فایل: 
		- fs.WriteFile(filePath)
		- Git.addFile(filePath)
		- Git.commit(commitMessage : File *A* Updated by User *A* with IP Address *1.1.1.1*)
	- به روزرسانی نام یک فایل یا پوشه:
		- fs.rename(oldname, newname)
		- Git.addFile(filePath)
		- Git.commit(commitMessage : File *A* renamed by User *A* with IP Address *1.1.1.1*)

4. حذف (*D*): این عمل برای دو نوع فایل و پوشه قابل انجام است:
	- برای حذف یک فایل از این سلسله عملیات استفاده می‌کنیم:
		- git.remove(filePath)
		- fs.removeRecursively(filePath)
		- git.commit(commitMessage : File *A* removed by User *A* with IP Address *1.1.1.1*)


	- برای حذف یک پوشه از این سلسله عملیات استفاده می‌کنیم:
		- fs.removeRecursively(filePath)
		- git.commit(commitMessage : directory *A* removed by User *A* with IP Address *1.1.1.1*)

این سلسله عملیات ها توسط توابع مرتبط با آنها پیاده خواهند شد.
