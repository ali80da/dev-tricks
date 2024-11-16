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
`
C:\Users\{UserName}\AppData\Roaming\NuGet
`
### برای مشاهده مستندات ماکروسافت از این لینک استفاده کنید.
`
https://learn.microsoft.com/en-us/nuget/consume-packages/configuring-nuget-behavior#how-settings-are-applied
`





