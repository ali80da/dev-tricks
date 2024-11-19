# DevTricks
Tricks 4 Designing &amp; Developing All Kinds of Programs with C# Programming Language &amp; .NET Framework &amp; Dependencies

<br /><br />

## پیام خطایی که دریافت کرده‌اید، به این معنی است که NuGet تلاش می‌کند تا از یک منبع HTTP برای دانلود بسته‌ها استفاده کند، در حالی که برای افزایش امنیت، استفاده از پروتکل HTTPS الزامی شده است. برای رفع این مشکل، باید اتصال امن (HTTPS) را در فایل NuGet.Config خود فعال کنید.
## کانفیگ فایل NuGet.Config برای استفاده از مسیر های دیگر دانلود پکیج ها : رفع خطای زیر
( NuGet package restore failed. Please see Error List window for detailed warnings and errors.
Error occurred while restoring NuGet packages: The operation failed as details for project Application could not be loaded. )

### نکته : اگر از مسیری استفاده میکنید که از امن بودن آن مطمئن هستید حتما باید این attribute را استفاده کنید. allowInsecureConnections="true"
```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <!-- <add key="nuget.org" value="https://api.nuget.org/v3/index.json" protocolVersion="3" /> -->
    <!-- <add key="Microsoft Visual Studio Offline Packages" value="C:\Program Files (x86)\Microsoft SDKs\NuGetPackages\" /> -->
    <add key="PackFeed" value="http://nexus.morsa.local:8081/repository/nuget.org-proxy/index.json" allowInsecureConnections="true" />
  </packageSources>
  <disabledPackageSources>
    <add key="Microsoft Visual Studio Offline Packages" value="true" />
    <add key="nuget.org" value="true" />
  </disabledPackageSources>
  <packageRestore>
    <add key="enabled" value="True" />
    <add key="automatic" value="True" />
    <add key="PackFeed" value="True" />
  </packageRestore>
  <bindingRedirects>
    <add key="skip" value="False" />
  </bindingRedirects>
</configuration>
```
### برای پیدا کردن فایل NuGet.Config در ویندوز این مسیر رو امتحان کنید.
```
C:\Users\{UserName}\AppData\Roaming\NuGet
```
### برای مشاهده مستندات ماکروسافت از این لینک استفاده کنید.

[لینک دستورات کانفیگ NuGet وبسایت ماکروسافت](https://learn.microsoft.com/en-us/nuget/consume-packages/configuring-nuget-behavior#how-settings-are-applied)

<br /><hr /><br />

## استفاده از httpPatch در WebAPI با استفاده از پکیج زیر در دات نت 8
```
Microsoft.AspNetCore.Mvc.NewtonsoftJson
```
### اول از همه باید این کد در فایل Program.cs اضافه کنیم:
```csharp
builder.Services.AddControllers().AddNewtonsoftJson();
```
و در ورودی Controller به این شکل اطلاعات دریافت می کنیم:
```csharp
[HttpPatch("{userId}")]
public IActionResult PatchData(string userId, JsonPatchDocument<UpdateUserDataMo> model, CancellationToken cancellation)
{
  var user = await GetUserByIdAsync(userId, cancellation);
  // some codes
  model.ApplyTo(user, ModelState);
  return Ok($"کاربر با آیدی: {userId} بروز رسانی شد.");
}
```

### و برای استفاده در سرویس به این شکل عمل میکنیم:
```csharp
public async Task<UpdateUserResult> PatchUserAync(string userId, JsonPatchDocument<UpdateUserDataMo> model, CancellationToken cancellation)
{
    try
    {
        var user = await GetUserByIdAsync(userId, cancellation);
        if (user is null) return UpdateUserResult.NotFound;
        if (user.IsRemoved || !user.PhoneNumberConfirmed) return UpdateUserResult.NotAccess;

        //UpdateUserDataMo data = new()
        //{

        //    FirstNameFa = user.FirstNameFa?.SanitizeText(),
        //    LastNameFa = user.LastNameFa?.SanitizeText()
        //};

        //model.ApplyTo(data);

        user.LastUpdateTime = DateTime.Now;

        await userRepository.UpdateUserAsync(user.Id, cancellation);
        return UpdateUserResult.Success;
    }
    catch (Exception ex)
    {
        throw new Exception($"کاربر با آیدی: {userId} بروز رسانی نشد.", ex);
        //return UpdateUserResult.Error;
    }
}
```

<br /><hr /><br />

## اجازه دسترسی به گیت در Terminal برای اضافه کردن کدها و... مخصوصا برای .vs
### و برای خطا هایی مثل: fatal: adding files failed
`
error: open(".vs/ProjectOrSolutionName/FileContentIndex/2b3c3599-****-****-****-285080df9374.vsidx"): Permission denied
`
<br /> یــــــــــــــــا <br />
`
error: unable to index file '.vs/ProjectOrSolutionName/FileContentIndex/2b3c3599-****-****-****-285080df9374.vsidx'
`
### از این دستور استفاده کنید...
```bash
icacls .vs/ProjectOrSolutionName/FileContentIndex/77641f5e-****-****-****-e97400316658.vsidx /grant "YourUserName":F
```
### و برای بدست آوردن UserName در ویندوز از این دستور استفاده کنید...
```bash
whoami
```
#### و به بقیه مسیر خود ادامه دهید.

<br /><hr /><br />

## اصول نسخه بندی: برای مثال درک ساختار 10.19.01
### نسخه بندی در توسعه نرم افزار، یک سیستم سازماندهی بسیار مهم است که به ما کمک می‌کند تا تغییرات ایجاد شده در طول زمان را پیگیری کرده و نرم افزارهایمان را بهتر مدیریت کنیم.
<br />

#### به طور معمول، یک نسخه نرم افزاری به سه بخش اصلی تقسیم می‌شود:

| توضیحات | ----- |
| -------------: | ------------------------------------------------------------------------------------------------------: |
| اولین عدد (10) نشان‌دهنده تغییرات اساسی در نرم افزار است. این تغییرات ممکن است شامل تغییر در معماری، APIها، یا ویژگی‌های اصلی باشد که ممکن است با نسخه‌های قبلی ناسازگار باشد. | Major (اصلی) |
| عدد دوم (19) نشان‌دهنده افزودن ویژگی‌های جدید یا تغییرات قابل توجهی است که با نسخه‌های قبلی سازگار هستند. | Minor (فرعی) |
| عدد سوم (01) نشان‌دهنده رفع باگ‌ها و مشکلات جزئی در نرم افزار است. این تغییرات معمولاً با نسخه‌های قبلی سازگار هستند. | Patch (پچ) |

<br />

### چرا از این ساختار استفاده می‌شود؟
#### روشنایی و سادگی: این ساختار به توسعه‌دهندگان و کاربران این امکان را می‌دهد تا به سرعت متوجه شوند که یک نسخه جدید چه تغییراتی داشته است.
#### مدیریت وابستگی‌ها: با استفاده از این ساختار، می‌توان به راحتی وابستگی‌های بین نرم افزارهای مختلف را مدیریت کرد.
#### سازگاری: با دانستن اینکه کدام تغییرات اساسی و کدام تغییرات جزئی هستند، می‌توان به راحتی مشخص کرد که یک نسخه جدید با نسخه‌های قبلی سازگار است یا خیر.

<br />

## مثال‌های دیگر از نسخه بندی:

`
1.0.0: اولین نسخه پایدار یک نرم افزار.
`
2.3.5: نسخه دوم نرم افزار که سه ویژگی جدید به آن اضافه شده و پنج باگ در آن رفع شده است.
#### 4.0.0-beta: نسخه چهارم بتا (پیش‌نمایش) یک نرم افزار که تغییرات اساسی در آن ایجاد شده است.
## نکات مهم در نسخه بندی:

ثبات: `سعی کنید یک سیستم نسخه بندی ثابت را برای پروژه خود انتخاب کنید و به آن پایبند باشید.`
معنی‌دار بودن اعداد: `هر عدد باید معنای مشخصی داشته باشد و تغییرات را به درستی نشان دهد.`
استفاده از ابزارهای خودکار: ` برای مدیریت نسخه‌ها و ایجاد تگ‌های مناسب، از ابزارهای خودکار مانند Git استفاده کنید.`

## در مورد نسخه 10.19.01 به طور خاص:

با توجه به اعداد استفاده شده در این نسخه، می‌توانیم نتیجه بگیریم که:

نرم افزار تغییرات اساسی زیادی را از زمان نسخه 1.0.0 تجربه کرده است.
در نسخه 19، ویژگی‌های جدید قابل توجهی به نرم افزار اضافه شده است.
در نسخه 01، برخی از باگ‌ها و مشکلات جزئی نرم افزار رفع شده است.

<br />

موضوعات مرتبطی که ممکن است برای شما مفید باشد:

Semantic Versioning: ` یک استاندارد رسمی برای نسخه بندی نرم افزار است.`
Git Tagging: `نحوه ایجاد تگ برای نسخه‌های مختلف در Git.`
Version Control Systems:`سیستم‌های کنترل نسخه مانند Git و SVN.`


<br /><hr /><br />




