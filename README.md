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

# دستورات nginx (وب سرور)

آزمایش و توسعه با NGINX
از فایل‌های HTML خود در مسیر C:\nginx\html استفاده کنید.
از NGINX برای Reverse Proxy یا Load Balancer در پروژه‌های توسعه خود بهره ببرید.

اجرای NGINX
باز کردن Command Prompt یا PowerShell:
کلیدهای `Win + R` را فشار دهید، `cmd` یا `powershell` را تایپ کنید و Enter را بزنید.
به مسیر `nginx.exe` بروید:

```Shell
cd C:\nginx
```
با دستور زیر NGINX را اجرا کنید:
```Shell
start nginx
```
پس از اجرا، NGINX به طور پیش‌فرض روی پورت 80 اجرا می‌شود. برای بررسی، مرورگر خود را باز کنید و به آدرس زیر بروید:
```Shell
http://localhost
```
شما باید صفحه پیش‌فرض NGINX را مشاهده کنید.
فایل تنظیمات اصلی NGINX در مسیر ` C:\nginx\conf\nginx.conf` قرار دارد. برای ویرایش تنظیمات:

فایل را با یک ویرایشگر متن مانند Notepad یا VS Code باز کنید.
```Conf
worker_processes  1;

events {
    worker_connections  1024;
}

http {
    server {
        listen       80;
        server_name  localhost;

        location / {
            root   html;
            index  index.html index.htm;
        }
    }
}
```
تغییرات دلخواه خود را اعمال کنید (مانند تنظیم پورت یا مسیر فایل‌های استاتیک).

برای اعمال تغییرات، NGINX را ری‌استارت کنید:
```Shell
nginx -s reload
```

پورت 80 در دسترس نیست: اگر پورت 80 قبلاً توسط سرویس دیگری استفاده می‌شود (مثل IIS):

فایل `nginx.conf` را باز کنید و پورت دیگری (مثلاً 8080) را تنظیم کنید:
```Shell
listen 8080;
```
پس از تغییر، NGINX را ری‌استارت کنید:
```Shell
nginx -s reload
```
اگر NGINX کار نمی‌کند: برای بررسی خطاها، فایل لاگ‌ها را بررسی کنید:
```Shell
C:\nginx\logs\error.log
```
برای متوقف کردن کامل NGINX:
این دستور بلافاصله NGINX را متوقف می‌کند.
```Shell
nginx -s stop
```

و برای توقف آرام:
این دستور تمام درخواست‌های فعلی را تکمیل کرده و سپس NGINX را متوقف می‌کند.
```Shell
nginx -s quit
```

و یا حتی میتونید از این روش استفاده کنید:

## استفاده از Task Manager
باز کردن `Task Manager`:
کلیدهای `Ctrl + Shift + Esc` را فشار دهید یا روی نوار وظیفه راست‌کلیک کرده و Task Manager را باز کنید.
پیدا کردن فرآیند NGINX:
به تب `Processes` بروید و فرآیندهای `nginx.exe` را پیدا کنید.
پایان دادن به فرآیند:
روی فرآیند NGINX راست‌ کلیک کرده و گزینه End Task را انتخاب کنید.

<br /><hr /><br />

# استفاده از عکس در Web Api
### To send a photo to a .NET Web API, the most common approach is to use multipart/form-data. This allows you to send the photo along with any additional data. Below are the steps to send and receive a photo in your .NET Web API.

## Sending the Photo to the API
When making the request to the API, the client (frontend or another application) should use a `multipart/form-data` format.

Configure Your API to Handle Large Files (Optional)
To handle large photos, update the Kestrel server limits in your `appsettings.json`:

appsettings.json:
```json
{
  "Kestrel": {
    "Limits": {
      "MaxRequestBodySize": 52428800 // 50 MB
    }
  }
}
```

Enable `multipart/form-data` in Swagger
## Swagger does not natively infer multipart/form-data from [FromForm]. You need to use the Swagger File Input in the SwaggerGen configuration.

```cs
builder.Services.AddSwaggerGen(c =>
{
    c.SwaggerDoc("v1", new OpenApiInfo { Title = "File Upload API", Version = "v1" });

    // Enable support for file uploads in Swagger
    c.OperationFilter<AddFileUploadOperation>();
});

// Add custom operation filter for file uploads
public class AddFileUploadOperation : IOperationFilter
{
    public void Apply(OpenApiOperation operation, OperationFilterContext context)
    {
        if (operation.OperationId == "UploadFile")
        {
            operation.RequestBody = new OpenApiRequestBody
            {
                Content = new Dictionary<string, OpenApiMediaType>
                {
                    ["multipart/form-data"] = new OpenApiMediaType
                    {
                        Schema = new OpenApiSchema
                        {
                            Type = "object",
                            Properties = new Dictionary<string, OpenApiSchema>
                            {
                                ["file"] = new OpenApiSchema
                                {
                                    Type = "string",
                                    Format = "binary"
                                },
                                ["description"] = new OpenApiSchema
                                {
                                    Type = "string"
                                }
                            },
                            Required = new HashSet<string> { "file" }
                        }
                    }
                }
            };
        }
    }
}
```
## Annotate the API Method
Use Swagger annotations to specify that the method accepts a `multipart/form-data` request:

```cs
/// <summary>
/// Upload a file with additional description.
/// </summary>
/// <param name="request">The upload request containing the file and description.</param>
/// <returns>Response with upload details.</returns>
[HttpPost]
[Route("upload")]
[Consumes("multipart/form-data")] // Important for Swagger to recognize file uploads
[ProducesResponseType(StatusCodes.Status200OK)]
[ProducesResponseType(StatusCodes.Status400BadRequest)]
public async Task<IActionResult> UploadFile([FromForm] UploadRequest request)
{
    // Method body as shown earlier
}
```

<br /><hr /><br />

# نمونه از لغو عملیات: برای مثال در Controller

```cs
#region RegisterUserWithPhoneNumber

[CustomSwagger("این اکشن برای ثبت نام کاربر است.")]
[HttpPost("register-with-phone")]
[Consumes("application/json")]
public async Task<IActionResult> RegisterUserWithPhoneNumber(RegisterUserDTO model, CancellationToken cancellation)
{
    if (ModelState.IsValid)
    {
        try
        {
            var res = await authService.RegisterUserWithPhoneNumber(model, cancellation);

            switch (res)
            {
                case RegisterUserResult.userExist:
                    return JsonResponseStatus.SendStatus(JsonResponseStatusType.Warning, "امکان ثبت این شماره تلفن وجود ندارد.", null);
                case RegisterUserResult.Error:
                    return JsonResponseStatus.SendStatus(JsonResponseStatusType.Error, "کلمه عبور باید بیشتر از 5 کاراکتر و همچنین حداقل دارای یک کاراکتر یکتا، حرف کوچک و حرف بزرگ باشد.", null);
                case RegisterUserResult.Success:
                    return JsonResponseStatus.SendStatus(JsonResponseStatusType.Success, "ثبت نام با موفقیت انجام شد", res);
            }
        }
            // در این قسمت اتفاقی که قرار است در صورت لغو عملیات انجام شود صورت میگیرد.
        catch (OperationCanceledException)
        {
            // عملیات لغو شد
            return BadRequest(new { success = false, message = "The request was canceled by the client." });
        }
        catch (Exception)
        {

            throw;
        }
    }
    logger.LogError("Error Sending Data");
    return BadRequest("خطا در ارسال اطلاعات");

}

#endregion
```

<br /><hr /><br />

## مراحل اضافه کردن کاربر و دادن دسترسی sudo
از دستور `adduser` استفاده کنید تا یک کاربر جدید ایجاد کنید:
```bash
sudo adduser username
```

## جایگزین کنید `username` با نام کاربری که می‌خواهید ایجاد کنید.
این دستور اطلاعاتی مانند رمز عبور، نام کامل و سایر جزئیات را درخواست می‌کند. رمز عبور را وارد کنید و اطلاعات دیگر را در صورت نیاز تکمیل کنید.

## اضافه کردن کاربر به گروه `sudo`
برای اضافه کردن کاربر به گروه `sudo` (که به کاربر اجازه اجرای دستورات مدیریتی را می‌دهد)، از دستور زیر استفاده کنید:

```bash
sudo usermod -aG sudo username
```

دستور `-aG`: به معنای اضافه کردن کاربر به گروه (append to group) است.
جایگزین کنید `username` با نام کاربری ایجاد شده.

## بررسی عضویت کاربر در گروه `sudo`
برای اطمینان از اینکه کاربر به درستی به گروه `sudo` اضافه شده است، از دستور زیر استفاده کنید:
```bash
groups username
```
این دستور لیستی از گروه‌هایی که کاربر به آنها تعلق دارد را نمایش می‌دهد. باید `sudo` در لیست باشد.

### تست دسترسی sudo
برای بررسی اینکه کاربر می‌تواند از دستورات `sudo` استفاده کند:
```bash
su - username
```

یک دستور `sudo` را اجرا کنید، مانند:
```bash
sudo apt update
```

### مثال کامل:
اگر می‌خواهید یک کاربر به نام `alida` ایجاد کنید و به آن دسترسی `sudo` بدهید:
```bash
sudo adduser alida
sudo usermod -aG sudo alida
groups alida
```

<br /><hr /><br />


## برای ویرایش hostname در لینوکس اوبونتو
می‌توانید از روش‌های زیر استفاده کنید. این تغییر بر نام میزبان سیستم تأثیر می‌گذارد، که در هنگام استفاده از شبکه یا نمایش در ترمینال مهم است.

### برای مشاهده نام میزبان فعلی، از دستور زیر استفاده کنید:
```bash
hostname
```

### تغییر نام میزبان موقت (بدون ری‌استارت)
برای تغییر نام میزبان موقتاً (تا زمان ری‌استارت بعدی سیستم)، از دستور زیر استفاده کنید:
```bash
sudo hostnamectl set-hostname new-hostname
```
#### این دستور، نام میزبان را در فایل‌های پیکربندی مربوطه به‌روز می‌کند.
#### نیازی به تنظیم دستی فایل‌های دیگر نیست.





<br /><hr /><br />

## تعدادی از دستورات لینوکس (ubuntu)
برای نمایش اطلاعات IP
```bash
ip a
```

برای چک کردن وضعیت فایروال
```bash
sudo ufw status
```

برای نصب پکیج
```bash
sudo apt-get install PackageName
```
##### لینوکس به حروف کوچیک و بزرگ حساسه




<br /><hr /><br />

## استفاده از دستور ساده `ip a` برای دریافت اطلاعات IP Address و ... در لینوکس (ubuntu)
```bash
ip a
```

<br /><hr /><br />

## برای استفاده از Proxy و ویرایش تنظیمات شبکه ای در لینوکس (ubuntu)
```bash
nano /etc/netplan/00-installer-config.yaml
```

<br /><hr /><br />

# ساختار تنظیمات فایل 00-installer-config.yaml
```yaml
# This is the network config written by 'subiquity'
network:
  ethernets:
    eth0:
      dhcp4: false
      addresses:
        - 172.17.61.9/20
    eth1:
      dhcp4: true
  version: 2
  renderer: networkd

```

<br /><hr /><br />

## دستور در حالت اجرا ماندن کانتیر داکر (حتی بعد از خاموشن شدن سیستم و روشن کردن)
```bash
sudo docker run -d --restart always --name container-name -p 8080:8080 image-name
```

<br /><hr /><br />

## مسیر تنظیمات فایل "resolv" برای استفاده از DNS
```bash
nano /etc/resolv.conf
```

<br /><hr /><br />

## نمونه محتوا فایل "resolv"
```bash
# Generated by NetworkManager

nameserver 127.0.0.53
options edns0 trust-ad
search domain.com

nameserver 172.30.0.61
nameserver 172.30.0.51
nameserver 172.30.0.62

# NOTE: the libc resolver may not support more than 3 nameservers.
# The nameservers listed below may not be recognized.
nameserver 172.30.0.53
```
<br /><hr /><br />

## مراحل خروجی APK در Blazor
```bash
keytool -genkey -v -keystore {appName}.keystore -alias {keyName} -keyalg RSA -keysize 2048 -validity 10000
```

### اضافه کردن به فایل XML
```xml
<TargetFramework  Condition="'$(Configuration)' == 'Release'">net9.0-android</TargetFramework>
<AndroidKeyStore>True</AndroidKeyStore>
<AndroidSigningKeyStore>{appName}.keystore</AndroidSigningKeyStore>
<AndroidSigningKeyAlias>{keyName}</AndroidSigningKeyAlias>
<AndroidSigningKeyPass>{P@ssw0rd}</AndroidSigningKeyPass>
<AndroidSigningStorePass>{P@ssw0rd}</AndroidSigningStorePass>
```
### و در مرحله آخر اجرا دستور زیر در cmd
```bash
 dotnet publish -f net9.0-android -c Release
```

### بنا به نسخه های مختلف دات نت - عدد نسخه دات نت تغییر میکنه
## و مسیر قرار گیری فایل APK در bin/Release/net9.0-android
<br /><hr /><br />

## استفاده از UseResponseCompression
```cs
// Configure the HTTP request pipeline.
app.UseWhen(context => !context.Request.Path.StartsWithSegments("/_framework"), appBuilder =>
{
    appBuilder.UseResponseCompression();
});
```

<br /><hr /><br />

یکی از راه حل های خطای زیر برای اتصال به دیتابیش در SQL Server Management
```
TITLE: Connect to Server
------------------------------

Cannot connect to ..

------------------------------
ADDITIONAL INFORMATION:

A network-related or instance-specific error occurred while establishing a connection to SQL Server. The server was not found or was not accessible. Verify that the instance name is correct and that SQL Server is configured to allow remote connections. (provider: Named Pipes Provider, error: 40 - Could not open a connection to SQL Server) (Microsoft SQL Server, Error: 2)

For help, click: https://docs.microsoft.com/sql/relational-databases/errors-events/mssqlserver-2-database-engine-error

------------------------------

The system cannot find the file specified

------------------------------
BUTTONS:

OK
------------------------------
```

اجرا دستور زیر در `win+r` یا تایپ کلمه `run` در search ویندوز:

```bash
mmc
```

Save a link to SQL Server Configuration Manager for another computer
On the Start menu, select Run.

<ul>
  <ol>In the Open box, type mmc -a (type mmc /32 -a on a 64-bit computer) to open the Microsoft Management Console in author mode.</ol>
  <ol>On the File menu, select Add/Remove Snap-in.</ol>

  <ol>In the Add/Remove Snap-in window, select Add.</ol>

  <ol>In the Add Standalone Snap-in window, select Computer Management and then select Add.</ol>
  <ol>In the Computer Management window, select Another computer, type the name of the remote computer you wish to manage, and then select Finish.</ol>
  <ol>In the Add Standalone Snap-in window, select Close.</ol>
  <ol>In the Add/Remove Snap-in window, select OK.</ol>
  <ol>Expand Computer Management (<computer name>), and Services and Applications.</ol>
  <ol>Right-click SQL Server Configuration Manager, and then select New Window from here.</ol>
  <ol>On the Window menu, select Console Root, to switch back to the first window, and delete the window.</ol>
  <ol>On the File menu, select Save As, and save the file in the desired folder, with an appropriate name with the .msc file extension. Close the Microsoft Management Console.</ol>
  <ol>To open SQL Server Configuration Manager on the target computer, double-click the file. If desired, save a link to the file on the desktop or in the Start menu.</ol>

</ul>

با این کار به صورت Local به دیتابیس دسترسی دارید...


---
نمونه محتوا فایل برنامه Inno
```code
[Setup]
AppName=YourAppName
AppVersion=1.0.0
DefaultDirName={localappdata}\Programs\YourAppName
DefaultGroupName=Your App Name
OutputBaseFilename=YourAppNameNasb
Compression=lzma2
SolidCompression=yes
OutputDir="D:\Username\YourAppNameNasb\Nasb"

[Files]
Source: "D:\Username\YourAppNameSetup\Release\*"; DestDir: "{app}"; Flags: ignoreversion recursesubdirs

[Icons]
Name: "{group}\Task Man"; Filename: "{app}\YourAppName.exe"
Name: "{commondesktop}\YourApp Name"; Filename: "{app}\YourAppName.exe"; Tasks: desktopicon

[Tasks]
Name: "desktopicon"; Description: "Create a &desktop icon"; GroupDescription: "Additional icons:"
Name: "autostart"; Description: "Start Task Man automatically with Windows"; GroupDescription: "Startup options:"

[Registry]
; Adds app to Windows startup if autostart is selected
Root: HKCU; Subkey: "Software\Microsoft\Windows\CurrentVersion\Run"; \
    ValueType: string; ValueName: "YourAppName"; \
    ValueData: """{app}\YourAppName.exe"""; Flags: uninsdeletevalue; Tasks: autostart

```

و توضیحاتی درباره برنامه Inno


---

## خواندن لاگ زنده با grep، یک Pattern (الگوی خاص)

1. چند عبارت با grep -E (Extended regex):
```bash
tail -f /var/log/syslog | grep -E "ERROR|WARNING|CRITICAL"
```
`فقط لاگ‌هایی را نمایش می‌دهد که یکی از این کلمات را دارند: ERROR یا WARNING یا CRITICAL`

2. استفاده از grep -i برای غیرحساس بودن به حروف بزرگ/کوچک:
```bash
tail -f /var/log/syslog | grep -iE "error|fail|timeout"
```

3. برجسته کردن نتایج با رنگ (برای خوانایی بهتر):
```bash
tail -f /var/log/syslog | grep --color=always -E "ERROR|FAIL|CRITICAL"
```


---
### زمانی که از جایی داریم ریپو مگیریم که پسوند .git نداره و یا فایل ها کامل دریافت نمیشن
:: مطمئن شو ریموت درست ست شده
```bash
git remote set-url origin route-address
```
```bash
git fetch origin
```
:: برنچ api را مستقیماً از روی origin/api بساز (با tracking خودکار)
```bash
git checkout -b api origin/api
```

:: Pull امن (فقط fast-forward)
```bash
git pull --ff-only
```


---




# 📘 دستورات مهم grep

## 1. جستجوی ساده یک کلمه
```bash
grep "error" logfile.txt
```
🔹 تمام خط‌هایی که شامل واژه `error` هست رو در فایل نشون میده.

---

## 2. حساس نبودن به حروف بزرگ/کوچک (`i-`)
```bash
grep -i "error" logfile.txt
```
🔹 هم `Error`، هم `ERROR` و هم `error` رو پیدا میکنه.

---

## 3. نمایش شماره خط (`n-`)
```bash
grep -n "error" logfile.txt
```
🔹 شماره خط کنار نتیجه چاپ میشه.

---

## 4. شمارش تعداد نتایج (`c-`)
```bash
grep -c "error" logfile.txt
```
🔹 فقط تعداد خط‌هایی که شامل `error` هستن نمایش داده میشن.

---

## 5. فقط نمایش بخش Match (`-o`)
```bash
grep -o "error" logfile.txt
```
🔹 به جای کل خط، فقط خود واژه‌ی `error` رو نمایش میده.  

---

## 6. جستجوی الگوی Regex (`-E`)
```bash
grep -E "error|warning" logfile.txt
```
🔹 تمام خط‌هایی که شامل `error` یا `warning` هستند نمایش داده میشه.

---

## 7. معکوس (خط‌هایی که شامل الگو **نیستن**) (`v-`)
```bash
grep -v "error" logfile.txt
```
🔹 همه‌ی خط‌هایی که **کلمه‌ی error ندارن** نمایش داده میشه.

---

## 8. شمارش فقط تعداد Matchها
```bash
grep -o "error" logfile.txt | wc -l
```

---

## 9. نمایش تعداد خط‌های قبل/بعد از Match
- قبل از Match  
```bash
grep -B 2 "error" logfile.txt
```
- بعد از Match  
```bash
grep -A 2 "error" logfile.txt
```
- قبل و بعد  
```bash
grep -C 2 "error" logfile.txt
```

---

## 10. جستجو در چند فایل
```bash
grep "error" /var/log/*.log
```

---

## 11. نمایش نام فایل فقط (`l-`)
```bash
grep -l "error" /var/log/*.log
```

---

## 12. نمایش اسم فایل همراه نتیجه (`H-`)
```bash
grep -H "error" logfile.txt
```

---

## 13. Regex برای IP (مثال کاربردی)
```bash
grep -oE '([0-9]{1,3}\.){3}[0-9]{1,3}' logfile.txt
```
🔹 تمام IP های موجود در فایل استخراج میشه.

---

## 14. استفاده زنده (با tail -f)
```bash
tail -f logfile.txt | grep "error"
```

---

# 🎯 خلاصه پرکاربردترین گزینه‌ها

| گزینه | توضیح |
|-------|-------|
| `-i`  | حساس نبودن به بزرگی/کوچکی حروف |
| `-n`  | نمایش شماره خط |
| `-c`  | شمارش تعداد خط‌های Match شده |
| `-o`  | فقط نمایش بخش Match شده |
| `-v`  | نمایش خطوطی که شامل الگو نیستند |
| `-E`  | فعال‌سازی Regex پیشرفته |
| `-A n` | n خط بعد از Match |
| `-B n` | n خط قبل از Match |
| `-C n` | n خط قبل و بعد |











