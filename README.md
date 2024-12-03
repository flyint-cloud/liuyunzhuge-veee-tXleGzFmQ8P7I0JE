
#### 前言


在`Core 9.0`版本中新增的内容不算多,除了内置[OpenAPI](https://github.com "OpenAPI") 外 应该就属`MapStaticAssets`中间件最有价值了,最初MapStaticAssets主要是为了解决`Blazor`静态资源加载缓慢而生的,当然只要是`wwwroot`下的任何静态资产都是可以使用TA平替`UseStaticFiles`的,因此在了解了TA的优势后 建议升级到9\.0的小伙伴都使用TA平替UseStaticFiles!


#### 既有缺陷


通常,在我们既有的NETCore项目中,我们都是使用`UseStaticFiles`中间件来提供静态资产,不过TA存在以下的一些缺陷:


1. 缺乏静态资源的传输压缩 (当然,可以搭配压缩中间件,或者容器压缩(如IIS动态压缩))
2. 使用ETag进行低效缓存(依赖于文件修改时间戳,因此内容不变时间戳变更将会导致重新加载)
3. 缺乏指纹识别(浏览器可能会缓存和重复使用旧版本的资产，从而导致应用更新后出现不一致,影响用户体验)


#### 解决问题


`MapStaticAssets`旨在解决上述UseStaticFiles存在的一些缺陷:


1. 为应用中的所有资产生成时间压缩：


* 在开发期间 `gzip`，在发布期间 gzip \+ `brotli`
* 所有资产都经过压缩，目标是将资产大小降到最低。


2. 基于内容的 ETags：每个资源的 Etags 都是内容的 `SHA-256` 哈希的 Base64 编码字符串。 这可确保浏览器仅在文件内容发生更改时重新下载文件。
3. 指纹识别资源,通过资源唯一标识，可以防止浏览器重复使用旧版本。当应用程序更新时，指纹会发生变化，从而确保客户端始终收到最新的资产。


在`MapStaticAssets`内部的请求管道中TA做了下面这些事:


1. 设置 ETag 和 Last\-Modified 标头。
2. 设置缓存标头。
3. 使用 Caching Middleware。
4. 如果可能，提供压缩的静态资产。


#### 性能提升


下表显示了默认的 Razor Pages 模板中 CSS 和 JS 文件的原始大小和压缩大小：




| 文件 | 原始 | 压缩 | %缩减 |
| --- | --- | --- | --- |
| bootstrap.min.css | 163 | 17\.5 | 89\.26% |
| jquery.js | 89\.6 | 28 | 68\.75% |
| bootstrap.min.js | 78\.5 | 20 | 74\.52% |
| 总计 | 331\.1 | 65\.5 | 80\.20% |


在使用`Blazor`开发业务系统时将节省大量传输宽带,**极大**的提升加载速度


#### 不可替部分


当然`UseStaticFiles`仍然有TA不可替代的部分,比如虚拟文件提供者(如,嵌入的资产,其他磁盘路径资源,或网络资源等)


比如资源是嵌入到程序集的情况下你仍然必须使用:



```
var embeddedFileProvider = new EmbeddedFileProvider(typeof(ISetting).Assembly, "Biwen.Settings");
app.UseStaticFiles(new StaticFileOptions
{
    FileProvider = embeddedFileProvider,
    OnPrepareResponse = ctx =>
    {
        ctx.Context.Response.Headers.Append("Cache-Control", "public,max-age=3600");
    }
});

```

#### 结论


强烈建议在可替换`UseStaticFiles()`的情况下使用`MapStaticAssets()`


 本博客参考[slower加速器](https://chundaotian.com)。转载请注明出处！
