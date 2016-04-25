# iOS-memory-manager

1.iOS内存泄露的问题
总结了一下控制器被强引用不走dealloc的原因无非就是三中情况：

一.block块使用不当。因为block会对方法中的变量自动retain一次。请检查控制器中block代码。（我的就是因为这没有走dealloc）

二.NSTimer没有销毁。在viewWillDisappear之前需要把控制器用到的NSTimer销毁。

三.控制器中的代理属性一定要是弱引用，不要强引用。


2.__weak和__strong的使用
在进入block代码以前应该要写上__weak __typeof(self) weakSelf = self,这是因为如果不写，然后block里面又调用到了self，那么就会造成循环引用无法释放self，所以在调动以前必须对它weak化，但是在进入block调用self以前又需要写上__strong __typeof(weakSelf) strongSelf = weakSelf;原因就是前面在block外面将self弱引用化了，但是在我们调用self以前必须再次强持有一次，如果有多层block嵌套，那么每一层都要做这种的处理，也许你会问为什么要再次强持有，因为前面你设置为了弱引用，这个地方必须去强持有这个弱引用，否则访问不到self的。
(关于__weak和__strong的追加解释：__weak的作用是来告诉编译器在block里面不要强应用self的属性，__strong是用来对self强引用一次)


3.block方法常用声明：@property (copy) void(^MyBlock)(void); 如果超出当前作用域之后仍然继续使用block，那么最好使用copy关键字，拷贝到堆区，防止栈区变量销毁。


积累的代码
+ (BOOL)checkPhoneNumber:(NSString *)phoneNumber{  
    //判断电话号码  
    NSString * MOBILE = @"^1(3[0-9]|5[0-35-9]|8[025-9])\\d{8}$";  
    NSString * CM = @"^1(34[0-8]|(3[5-9]|5[017-9]|8[278])\\d)\\d{7}$";  
    NSString * CU = @"^1(3[0-2]|5[256]|8[56])\\d{8}$";  
    NSString * CT = @"^1((33|53|8[09])[0-9]|349)\\d{7}$";  
      
    NSPredicate *regextestmobile = [NSPredicate predicateWithFormat:@"SELF MATCHES %@", MOBILE];  
    NSPredicate *regextestcm = [NSPredicate predicateWithFormat:@"SELF MATCHES %@", CM];  
    NSPredicate *regextestcu = [NSPredicate predicateWithFormat:@"SELF MATCHES %@", CU];  
    NSPredicate *regextestct = [NSPredicate predicateWithFormat:@"SELF MATCHES %@", CT];  
    BOOL res1 = [regextestmobile evaluateWithObject:phoneNumber];  
    BOOL res2 = [regextestcm evaluateWithObject:phoneNumber];  
    BOOL res3 = [regextestcu evaluateWithObject:phoneNumber];  
    BOOL res4 = [regextestct evaluateWithObject:phoneNumber];  
      
    if (res1 || res2 || res3 || res4 )  
    {  
        return YES;  
    }  
    else  
    {  
        return NO;  
    }  
}  

将字符颜色码转换为UIcolor 
+ (UIColor *)colorWithHexString:(NSString *)stringToConvert  
{  
    NSString *cString = [[stringToConvert stringByTrimmingCharactersInSet:[NSCharacterSet whitespaceAndNewlineCharacterSet]] uppercaseString];  
      
    if ([cString length] < 6)  
        return DEFAULT_VOID_COLOR;  
    if ([cString hasPrefix:@"#"])  
        cString = [cString substringFromIndex:1];  
    if ([cString length] != 6)  
        return DEFAULT_VOID_COLOR;  
      
    NSRange range;  
    range.location = 0;  
    range.length = 2;  
    NSString *rString = [cString substringWithRange:range];  
      
    range.location = 2;  
    NSString *gString = [cString substringWithRange:range];  
      
    range.location = 4;  
    NSString *bString = [cString substringWithRange:range];  
      
      
    unsigned int r, g, b;  
    [[NSScanner scannerWithString:rString] scanHexInt:&r];  
    [[NSScanner scannerWithString:gString] scanHexInt:&g];  
    [[NSScanner scannerWithString:bString] scanHexInt:&b];  
      
    return [UIColor colorWithRed:((float) r / 255.0f)  
                           green:((float) g / 255.0f)  
                            blue:((float) b / 255.0f)  
                           alpha:1.0f];  
}  


iOS常用宏定义
// 1.判断是否为iOS7  
#define iOS7 ([[UIDevice currentDevice].systemVersion doubleValue] >= 7.0)  
  
// 2.获得RGB颜色  
#define RGBA(r, g, b, a)                    [UIColor colorWithRed:r/255.0f green:g/255.0f blue:b/255.0f alpha:a]  
#define RGB(r, g, b)                        RGBA(r, g, b, 1.0f)  
  
#define navigationBarColor RGB(33, 192, 174)  
#define separaterColor RGB(200, 199, 204)  
  
  
// 3.是否为4inch  
#define fourInch ([UIScreen mainScreen].bounds.size.height == 568)  
  
// 4.屏幕大小尺寸  
#define screen_width [UIScreen mainScreen].bounds.size.width  
#define screen_height [UIScreen mainScreen].bounds.size.height  
  
//重新设定view的Y值  
#define setFrameY(view, newY) view.frame = CGRectMake(view.frame.origin.x, newY, view.frame.size.width, view.frame.size.height)  
#define setFrameX(view, newX) view.frame = CGRectMake(newX, view.frame.origin.y, view.frame.size.width, view.frame.size.height)  
#define setFrameH(view, newH) view.frame = CGRectMake(view.frame.origin.x, view.frame.origin.y, view.frame.size.width, newH)  
  
  
//取view的坐标及长宽  
#define W(view)    view.frame.size.width  
#define H(view)    view.frame.size.height  
#define X(view)    view.frame.origin.x  
#define Y(view)    view.frame.origin.y  
  
//5.常用对象  
#define APPDELEGATE ((AppDelegate *)[UIApplication sharedApplication].delegate)  
  
  
//6.  
#define IOS_VERSION [[[UIDevice currentDevice] systemVersion] floatValue]  

在viewDidLoad函数中加上此句代码：
self.tableView.tableFooterView = [[UIView alloc] initWithFrame:CGRectZero];
即可去除UITableView底部多余行及分割线

plist各种key值含义
UIRequiresPersistentWiFi 在程序中弹出wifi选择的key（系统设置中需要将wifi提示打开）  
UIAppFonts 内嵌字体（http://www.minroad.com/?p=412 有详细介绍）  
UIApplicationExitsOnSuspend 程序是否在后台运行，自己在进入后台的时候exit(0)是很傻的办法  
UIBackgroundModes 后台运行时的服务，具体看iOS4的后台介绍  
UIDeviceFamily array类型（1为iPhone和iPod touch设备，2为iPad)  
UIFileSharingEnabled 开启itunes共享document文件夹  
UILaunchImageFile 相当于Default.png（更名而已）  
UIPrerenderedIcon icon上是否有高光  
UIRequiredDeviceCapabilities 设备需要的功能（具体点击这里查看）  
UIStatusBarHidden 状态栏隐藏（和程序内的区别是在于显示Default.png已经生效）  
UIStatusBarStyle 状态栏类型  
UIViewEdgeAntialiasing 是否开启抗锯齿  
CFBundleDisplayName app显示名  
CFBundleIconFile、CFBundleIconFiles 图标  
CFBundleName 与CFBundleDisplayName的区别在于这个是短名，16字符之内  
CFBundleVersion 版本  
CFBundleURLTypes 自定义url，用于利用url弹回程序  
CFBundleLocalizations 本地资源的本地化语言，用于itunes页面左下角显示本地话语种  
CFBundleDevelopmentRegion 也是本地化相关，如果用户所在地没有相应的语言资源，则用这个key的value来作为默认  

