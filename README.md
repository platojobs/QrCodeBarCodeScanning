# QrCodeBarCodeScanning
摘要：
在IOS中实现二维码和条形码扫描，我们所知的有，两大开源组件 ZBar 与 ZXing. 这两大组件我们都有用过，这里总结下各自的缺点：
ZBar：在扫描的灵敏度上，和内存的使用上相对于ZXing上都是较优的，但是对于 “圆角二维码” 的扫描确很困难。
ZXing ：是 Google Code上的一个开源的条形码扫描库，是用java设计的，连Google Glass 都在使用的。但有人为了追求更高效率以及可移植性，出现了c++ port. Github上的Objectivc-C port，其实就是用OC代码封装了一下而已，而且已经停止维护。这样效率非常低，在instrument下面可以看到CPU和内存疯涨，在内存小的机器上很容易崩溃
AVFoundation：(苹果官方)无论在扫描灵敏度和性能上来说都是最优的，所以毫无疑问我们应该切换到AVFoundation，需要兼容IOS6或之前的版本可以用zbar或zxing代替；但是用到这个需要注意两点
1>图片很小的时候
AVCaptureSession 可以设置 sessionPreset 属性,来决定输出质量
2>另一个提升扫描速度和性能的就是设置解析的范围，在zbar和zxing中就是scanCrop, AVFoundation中设置 AVCaptureMetadataOutput 的 rectOfInterest 属性来配置解析范围。

那么今天就来搞下ios原生的二维码和条形码扫描。

|Author|cuishengxi|
|---|---
|E-mail|1300000608@qq.com

[新浪博客](http://blog.sina.com.cn/cuishengxisvip)
============================

效果图
======
![](https://github.com/ShengxiCui/QrCodeBarCodeScanning/blob/master/456.jpg?raw=true)

主要代码：
```objective-c

AVCaptureDevice * device = [AVCaptureDevice defaultDeviceWithMediaType:AVMediaTypeVideo];
//创建输入流
AVCaptureDeviceInput * input = [AVCaptureDeviceInput deviceInputWithDevice:device error:nil];
//创建输出流
AVCaptureMetadataOutput * output = [[AVCaptureMetadataOutput alloc]init];
//设置代理 在主线程里刷新
[output setMetadataObjectsDelegate:self queue:dispatch_get_main_queue()];

//初始化链接对象
session = [[AVCaptureSession alloc]init];
//高质量采集率
[session setSessionPreset:AVCaptureSessionPresetHigh];

[session addInput:input];
[session addOutput:output];
//设置扫码支持的编码格式(如下设置条形码和二维码兼容)
output.metadataObjectTypes=@[AVMetadataObjectTypeQRCode,AVMetadataObjectTypeEAN13Code, AVMetadataObjectTypeEAN8Code, AVMetadataObjectTypeCode128Code];

AVCaptureVideoPreviewLayer * layer = [AVCaptureVideoPreviewLayer layerWithSession:session];
layer.videoGravity=AVLayerVideoGravityResizeAspectFill;
layer.frame = CGRectMake(IPHONE5_SCALAE(100), IPHONE5_SCALAE(80), VIEW_WIDTH- 2 * IPHONE5_SCALAE(100), IPHONE5_SCALAE(440));
[AVCapView.layer insertSublayer:layer atIndex:0];
//开始捕获
[session startRunning];

```

```objective-c
#pragma mark==实现扫描成功后，想做的事情，就在这个代理方法里面实现就行了
-(void)captureOutput:(AVCaptureOutput *)captureOutput didOutputMetadataObjects:(NSArray *)metadataObjects fromConnection:(AVCaptureConnection *)connection{
if (metadataObjects.count>0) {
//[session stopRunning];
AVMetadataMachineReadableCodeObject * metadataObject = [metadataObjects objectAtIndex : 0 ];
//输出扫描字符串
NSLog(@"%@",metadataObject.stringValue);
if ([metadataObject.stringValue hasPrefix:@"http:"]) {
[webView loadRequest:[NSURLRequest requestWithURL:[NSURL URLWithString:metadataObject.stringValue]]];
}
[session stopRunning];
[self stopTimer];
[AVCapView removeFromSuperview];
//其它业务代码
}
}

```

###菜鸟一枚。不喜勿喷

