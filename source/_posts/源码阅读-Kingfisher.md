---
title: 源码阅读 Kingfisher
date: 2018-10-09 20:33:02
tags: 源码阅读
---
[Kingfisher](https://github.com/onevcat/Kingfisher)是一个用于图片下载和缓存的轻量级、纯Swift库。通过[喵神](https://weibo.com/onevcat?profile_ftype=1&is_all=1#_rnd1539003449528)的介绍，可以得知`Kingfisher`有以下特点：

- 实现了图片的异步下载和缓存
- 基于`URLSession`的网络，提供基本图像处理器和过滤器。
- 内存和磁盘的多层缓存。
- 可取消下载和处理任务以提高性能。
- 独立的组件，根据需要单独使用下载器或缓存系统。
- 预览图像并在以后需要时从缓存中显示它们。
- 对`UIImageView`,` NSImage`和`UIButton`的扩展，可以直接从`URL`设置图像。
- 设置图像时内置过渡动画。
- 加载图像时可自定义占位符。
- 可扩展的图像处理和图像格式支持。

## 目录结构
在项目中，我们使用`CocoaPods `下载安装`Kingfisher `。
我们查看`Kingfisher`的目录结构，如下
```
Kingfisher
    AnimatedImageView.swift    //动画控件   
    Box.swift    //工具类
    CacheSerializer.swift    //序列化类，读写文件时Data和Image互转
    Filter.swift    //仅对CIImage有效
    FormatIndicatedCacheSerializer.swift    //PNG/JPEG/GIF和Data互转
    Image.swift    //图片格式转换
    ImageCache.swift    //图片缓存
    ImageDownloader.swift    //图片下载
    ImageModifier.swift    //图片修改
    ImagePrefetcher.swift    //图片下载管理类，对并发多个下载任务的处理
    ImageProcessor.swift    //数据处理类，将Data转为Image
    ImageTransition.swift    //动画效果
    ImageView+Kingfisher.swift    //扩展ImageView添加下载图片的方法
    Indicator.swift    //动画相关
    Kingfisher.h    //版本号
    Kingfisher.swift    //类，扩展ImageView添加属性kf
    KingfisherManager.swift    //管理类，封装图片下载和缓存的逻辑
    KingfisherOptionsInfo.swift    //枚举类
    Placeholder.swift    //默认图片管理类
    RequestModifier.swift    //协议，修改原始URLRequest参数
    Resource.swift    //协议，声明下载链接和缓存key
    String+MD5.swift    //MD5加密
    ThreadHelper.swift    //工具类
    UIButton+Kingfisher.swift    //扩展UIButton添加下载图片的方法
```
## 调用方法
很简单的一句话：
```
self.imageV.kf.setImage(with: imgUrl)
```
如果想在图片加载的过程中添加默认图片，可以添加`placeholder`方法，监听加载的过程`progressBlock`，图片加载完成后的回调`completionHandler`。
## 查看方法
查看`kf.setImage`方法，我们会跳到*ImageView+Kingfisher.swift*文件里。这里我们对方法做一个简单的介绍
```
@discardableResult
public func setImage(with resource: Resource?,
                     placeholder: Placeholder? = nil,
                     options: KingfisherOptionsInfo? = nil,
                     progressBlock: DownloadProgressBlock? = nil,
                     completionHandler: CompletionHandler? = nil) -> RetrieveImageTask{...}
```
我们可以看到，在这个方法里面，包括了ImageView的资源、默认图片、作者封装的枚举类、加载进度的回调以及完成结果。
`@discardableResult`方法是为了取消不使用返回值的警告。
在这个方法里面(方法太长就不列举出来，捡主要的说)，
- 首先判断参数合法性，当`resource `为`nil`时，展示默认图片。
- 设置读取策略和启动动画。比如常见的一个问题：当用户头像改变但图片`URL`没有改变时，怎么去处理用户头像。一般有两种方法，一种是在缓存用户头像时保存当前时间。另一种就是设置读取策略，`KingfisherOptionsInfo`是一个枚举，设置它为`forceRefresh `时，可以强制刷新。
- 从内存、文件或网络`URL`获取对应图片数据。
- 获取图片完成后，在主线程刷新界面。

我们查看一下这里面的参数：
### Resource
Resource是一个协议，我们查看源码可以看到：
```
public protocol Resource {
    var cacheKey: String { get }
    var downloadURL: URL { get }
}
```
`cacheKey`是图片保存的key值，当`cacheKey`为`nil`时，取`downloadURL.absoluteString`(有兴趣的可以去了解一下`absoluteString`和`path`的区别)。`downloadURL`不言而喻，就是图片的`URL`。
### Placeholder
```
public protocol Placeholder {
    func add(to imageView: ImageView)
    func remove(from imageView: ImageView)
}
```
`Placeholder`是一个协议，作者为它定义了`add`和`remove`方法，任何。默认实现了`Image`，如果想用`View`充当`Placeholder`，只要让`view`遵守协议即可
```
extension Placeholder where Self: View{}
```
### KingfisherOptionsInfo
`KingfisherOptionsInfo`是一个类型别名，点击查看
```
public typealias KingfisherOptionsInfo = [KingfisherOptionsInfoItem]
```
所以我们关注的应该是`KingfisherOptionsInfoItem`是什么东西？那么它是什么呢？
```
public enum KingfisherOptionsInfoItem {
    case targetCache(ImageCache)    //系统缓存位置。可以设置的属性
    case originalCache(ImageCache)    //系统缓存原始图像位置（只用于自己设置placeholder） 
    case downloader(ImageDownloader)    //获取更改session属性，设置请求
    case transition(ImageTransition)    //自定义动画
    case downloadPriority(Float)    //下载优先级（0-1）
    case forceRefresh    //每次请求忽略缓存，直接下载
    case fromMemoryCacheOrRefresh    //先取缓存再去文件，再去下载
    case forceTransition    //强制移动
    case cacheMemoryOnly     //只从缓存读取，不读取本机沙盒图片
    case onlyFromCache    //从缓存、沙盒读取，没有也不下载网络，显示placeholder
    case backgroundDecode    //设置后，显示前在后台线程解码
    case callbackDispatchQueue(DispatchQueue?)    //自定义回调队列，默认主线程
    case scaleFactor(CGFloat)    //自定义图片data -> Image缩放比例，不指定按屏幕2x\3x缩放
    case preloadAllAnimationData    //预先加载data成图片缓存
    case requestModifier(ImageDownloadRequestModifier)    //改变请求
    case processor(ImageProcessor)    //自定义Data转图片样式
    case cacheSerializer(CacheSerializer)    //自定义缓存Data 转图像样式
    case imageModifier(ImageModifier)    //修改图像
    case keepCurrentImageWhileLoading     //包含这个意味着placeHolder设置无效，没有直接用默认
    case onlyLoadFirstFrame    //如果返回结果是.gif图，只取第一帧显示
    case cacheOriginalImage    //同时缓存原始图片和下载后的图片
}
```
### DownloadProgressBlock
```
public typealias DownloadProgressBlock = ((_ receivedSize: Int64, _ totalSize: Int64) -> ())
```
`DownloadProgressBlock`里面有`receivedSize`和`totalSize`，可以根据这两个参数得知图片下载了多少和图片多大，也可以计算图片的下载进度。
### CompletionHandler
```
public typealias CompletionHandler = ((_ image: Image?, _ error: NSError?, _ cacheType: CacheType, _ imageURL: URL?) -> ())
```
`CompletionHandler`的回调里会有`image`、`error`、`cacheType`、`imageURL`四个参数。`image`、`error`、`imageURL`不做介绍，直接就能看出什么内容。
主要看一下`cacheType`:
```
public enum CacheType {
    case none, memory, disk
    public var cached: Bool {
        switch self {
        case .memory, .disk: return true
        case .none: return false
        }
    }
}
```
- `none`检索图片时，图片尚未缓存
- `memory`图片缓存在内存中
- `disk`图片缓存在磁盘中



## 检索图片
OK，让我们继续看这些代码。在`setImage`方法中，从内存、文件或网络`URL`获取对应图片数据是怎么实现的呢？这里，我们可以查看`KingfisherManager.shared.retrieveImage`方法。
```
@discardableResult
public func retrieveImage(with resource: Resource,
    options: KingfisherOptionsInfo?,
    progressBlock: DownloadProgressBlock?,
    completionHandler: CompletionHandler?) -> RetrieveImageTask
{
    let task = RetrieveImageTask()
    let options = currentDefaultOptions + (options ?? KingfisherEmptyOptionsInfo)
    if options.forceRefresh {
        _ = downloadAndCacheImage(
            with: resource.downloadURL,
            forKey: resource.cacheKey,
            retrieveImageTask: task,
            progressBlock: progressBlock,
            completionHandler: completionHandler,
            options: options)
    } else {    
        tryToRetrieveImageFromCache(
            forKey: resource.cacheKey,
            with: resource.downloadURL, 
            retrieveImageTask: task,
            progressBlock: progressBlock,
            completionHandler: completionHandler,
            options: options)
    }
    
    return task
}
```
可以看到，代码会通过`KingfisherOptionsInfo`进行判断是强制刷新，网络下载并执行缓存策略还是从内存或文件中获取对应的Image。
### 图片下载
```
@discardableResult
func downloadAndCacheImage(with url: URL,
                         forKey key: String,
                  retrieveImageTask: RetrieveImageTask,
                      progressBlock: DownloadProgressBlock?,
                  completionHandler: CompletionHandler?,
                            options: KingfisherOptionsInfo) -> RetrieveImageDownloadTask?{...}
```
在`downloadAndCacheImage`方法中，会`return  downloader.downloadImage`方法，下载的主要逻辑在这里实现，对应的文件是*ImageDownloader.swift*。
```
@discardableResult
open func downloadImage(with url: URL,
               retrieveImageTask: RetrieveImageTask? = nil,
                         options: KingfisherOptionsInfo? = nil,
                   progressBlock: ImageDownloaderProgressBlock? = nil,
               completionHandler: ImageDownloaderCompletionHandler? = nil) -> RetrieveImageDownloadTask?{...}
```
`ImageDownloader.swift`文件中，主要参数：
- `downloadTimeout` 超时时间，默认15秒
- `trustedHosts ` 信任的请求地址，和自己实现请求代理设置冲突，二选一
- `sessionConfiguration` session配置设置
- `requestsUsePipelining` 请求是否管道类型，是否按顺序下载，默认`false`
- `sessionHandler`单独设计出的一个`ImageDownloaderSessionHandler`，是为了解决之前出现的[内存泄漏](https://github.com/onevcat/Kingfisher/issues/235)
- `delegate` 下载代理
- `authenticationChallengeResponder` 信任请求代理，和trustedHosts冲突二选一
- `fetchLoads` 下载完成每个URL可能有多个处理方式，优先取这里的
- 此外还有三个`DispatchQueue`：`barrierQueue`、`processQueue`、`cancelQueue`

下载完成后，在`completionHandler`回调中处理图片，如果下载失败：
```
if let error = error, error.code == KingfisherError.notModified.rawValue {
    //从缓存中读取，不需保存，直接返回
    targetCache.retrieveImage(forKey: key, options: options, completionHandler: { (cacheImage, cacheType) -> () in
        completionHandler?(cacheImage, nil, cacheType, url)
    })
    return
}
```
下载成功：
```
if let image = image, let originalData = originalData {
    //存储图片
    targetCache.store(image,
                      original: originalData,
                      forKey: key,
                      processorIdentifier:options.processor.identifier,
                      cacheSerializer: options.cacheSerializer,
                      toDisk: !options.cacheMemoryOnly,
                      completionHandler: nil)
        if options.cacheOriginalImage && options.processor != DefaultImageProcessor.default {
            let originalCache = options.originalCache
            let defaultProcessor = DefaultImageProcessor.default
            if let originalImage = defaultProcessor.process(item: .data(originalData), options: options) {
                originalCache.store(originalImage,
                        original: originalData,
                        forKey: key,
                        processorIdentifier: defaultProcessor.identifier,
                        cacheSerializer: options.cacheSerializer,
                        toDisk: !options.cacheMemoryOnly,
                        completionHandler: nil)
            }
        }
}
```
#### 存储图片
我们先来看一下实现的代码：
```
open func store(_ image: Image,
                  original: Data? = nil,
                  forKey key: String,
                  processorIdentifier identifier: String = "",
                  cacheSerializer serializer: CacheSerializer = DefaultCacheSerializer.default,
                  toDisk: Bool = true,
                  completionHandler: (() -> Void)? = nil){...}
```
代码中`memoryCache `是：
```
fileprivate let memoryCache = NSCache<NSString, AnyObject>()
```
可见图片存储首先是缓存在`NSCache`中，如果想存储在磁盘中(`if toDisk`)，利用串行队列异步的进行存储原图。
#### 获取图片
```
@discardableResult
open func retrieveImage(forKey key: String,
                           options: KingfisherOptionsInfo?,
                 completionHandler: ((Image?, CacheType) -> ())?) -> RetrieveImageDiskTask?{...}
```
- 首先从内存中获取图片`if let image = self.retrieveImageInMemoryCache(forKey: key, options: options)`
- 如果没有，在根据条件判断是否从磁盘上获取`if let image = sSelf.retrieveImageInDiskCache(forKey: key, options: options)`

#### 删除图片
```
open func removeImage(forKey key: String,
                      processorIdentifier identifier: String = "",
                      fromDisk: Bool = true,
                      completionHandler: (() -> Void)? = nil){...}
```

```
@objc public func clearMemoryCache() {...}
```
```
open func clearDiskCache(completion handler: (()->())? = nil) {...}
```
```
open func cleanExpiredDiskCache(completion handler: (()->())? = nil) {...}
```
```
@objc public func backgroundCleanExpiredDiskCache() {...}
```
其中，一些方法是通过通知的方法来实现：
```
// 系统内存警告
NotificationCenter.default.addObserver(self, selector: #selector(clearMemoryCache), name: .UIApplicationDidReceiveMemoryWarning, object: nil)    
// 程序终止
NotificationCenter.default.addObserver(self, selector: #selector(cleanExpiredDiskCache), name: .UIApplicationWillTerminate, object: nil)
// 程序进入后台
NotificationCenter.default.addObserver(self, selector: #selector(backgroundCleanExpiredDiskCache), name: .UIApplicationDidEnterBackground, object: nil)
```
此外，还有一些属性要注意:
- `maxMemoryCost `最大缓存量，在收到内存警告时会被清空。
- `pathExtension`沙盒后续拼接文件夹名称
- `maxCachePeriodInSecond `默认清除一周前的图片
- `maxDiskCacheSize `沙盒最大存储量，为0，默认无限制

以上就是对Kingfisher的简单描述，它有很多方法值得我们去借鉴，比如`@discardableResult`、`where`、`typealias`、`if case let`、善于利用`guard`、扩展协议等等。





