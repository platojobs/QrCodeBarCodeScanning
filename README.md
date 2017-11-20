# QrCodeBarCodeScanning
ios原生的二维码和条形码扫描。

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

