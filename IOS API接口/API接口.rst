============
API接口调用与说明
============

调用示例
==========

- 第一步：配置参数调用初始化与检测社区通知接口

.. code-block:: c

    __weak typeof(self)weakSelf = self;  
    WeGamersSDKParams *gcParam = [[WeGamersSDKParams alloc] init];
    gcParam.sdkId = @"WeGamers内嵌社区分配的社区SDK编号(不为空)";
    gcParam.gameAccountId = @"游戏账号(不为空)";
    gcParam.nickName = @"游戏玩家昵称(不为空)";
    gcParam.skinType = GameCommunityThemDefault;//五种换肤类型，这里选默认UI类型

    [GameCommunityEntry initGameCommunity:gcParam showCommunityRed:^(BOOL bShow) {
        [weakSelf showNotifyRed:bShow];//游戏自己处理显示红点的UI回调block
    }supportGameCommunity:^(BOOL bSupport) {
        if (bSupport) {//这里回调当前系统环境是否支持游戏社区功能
            //支持游戏社区，游戏入口可以开放
        }
        else{
            //不支持游戏社区，游戏入口可以进行隐藏
        }
    }];

.. code-block:: c

    //注册游戏社区弹出通知 
    AppDelegate *delegate = (AppDelegate *)[UIApplication sharedApplication].delegate; 
    [GameCommunityEntry checkGameCommunityNotice:delegate.window completionBlock:^(NSError * _Nullable error) {
        //检测社区通知是否成功
    }];

具体参数类型请见后面接口方法的详细说明

- 第二步：打开游戏社区界面

.. code-block:: c

    - (void)onGotoCommunity:(id)sender
    {
        GameCommunityEntryResult* result = [GameCommunityEntry openGameCommunityHomePageAndwillExitLive:^{
            NSLog(@"关闭游戏社区");
        }];

        if (result.error && result.error.code == -1)
        {
            //TODO: 提示打开失败
            //...
        } 
        else if (result.error && ((result.error.code == -2) || (result.error.code == -3)))
        {
            //TODO: 取出主页视图，在指定页面弹出UIViewController* liveHomePage = result.wgHomePageViewController;//... 
        }
    }

- 第三步：设置游戏内嵌社区支持横竖屏

在AppDelegate添加方法并实现

.. code-block:: c

    - (UIInterfaceOrientationMask)application:(UIApplication *)application supportedInterfaceOrientationsForWindow:(UIWindow *)window
    {//AppDelegate 的supportedInterfaceOrientationsForWindow回调接口中处理
        return [GameCommunityEntry appOrientationMask];//或者调用return UIInterfaceOrientationMaskAllButUpsideDown;调用UIInterfaceOrientationMaskAllButUpsideDown 游戏自己界面需要处理好自己的横竖屏状态
    }


接口说明
=========

包含头文件：#import <GameCommunitySDK/GameCommunitySDK.h>

- GameCommunityParam参数配置

具体看以下代码注释说明

.. code-block:: c

    typedef enum : NSInteger {
        GameCommunityThemDefault = 0,   //默认皮肤
        GameCommunityThemPurple,        //紫色皮肤
        GameCommunityThemDark,          //深色皮肤
        GameCommunityThemLM,            //LM皮肤
        GameCommunityThemCC,            //CC皮肤
    } GameCommunityThemType;            //五种换肤类型

    @interface WeGamersSDKParams : NSObject
    @property(nonatomic, copy) NSString* sdkId;                     //WeGamers内嵌社区分配的社区SDK编号ID
    @property(nonatomic, copy) NSString* gameAccountId;             //游戏账号ID （游戏自己定义的账号ID）
    @property(nonatomic, copy) NSString* nickName;                  //游戏玩家昵称
    @property(nonatomic, assign) GameCommunityThemType skinType;    // 换肤类型
    @end

- 初始化接口

.. code-block:: c

    /**
    游戏社区初始化接口
    @param param 参数请参考WeGamersSDKParams定义。
    @param showNotifyRedBlock 红点提醒回调 YES代表有新消息通知需要红点显示，NO表示清除红点显示
    @param supportBlock 返回当前系统环境是否支持游戏社区
    */
    + (void)initGameCommunity:(WeGamersSDKParams*)param showCommunityRed:(void (^)(BOOL bShow))showNotifyRedBlock supportGameCommunity:(void (^)(BOOL bSupport))supportBlock;

配置好参数，在游戏打开游戏社区界面之前调用此初始化接口。showNotifyRedBlock 用来通知游戏外部UI游戏社区有新的评论或者通知红点UI显示或者隐藏 的回调。

- 检测游戏社区通知

.. code-block:: c

   /**游戏内弹出社区通知(在需要弹出的时机调用改接口)
   @param window 应用程序主Window
   @param completionHandler 调用回调情况 error nil表示成功 否则表示失败
   */
   + (void)checkGameCommunityNotice:(UIWindow *)window completionBlock:(void (^)(NSError * _Nullable error))completionHandler;

游戏自在需要显示通知弹窗的地方调用这个接口，检测到有新的通知消息会弹窗显示通知消息加入在传入的window层级，点击弹窗进入对应的通知消息内嵌社区界面

- 打开游戏内嵌社区界面

.. code-block:: c

  /**打开游戏社区页面
  @return 打开的窗口结果对象：
  1）社区主页视图控制器对象；
  2）NSError对象。其中，错误码为
  -1，表示社区主页视图控制器对象创建失败；
  -2，表示未找开应用主窗口；
  -3，表示弹出窗口异常；
  */
  + (GameCommunityEntryResult *)openGameCommunityHomePageAndwillExitLive:(void (^)(void))blockWillExit;

  游戏点击社区按钮时候调用改接口，blockWillExit为关闭社区的回调block。

- 防止游戏战斗时候弹出通知弹窗打断游戏战斗画面接口

.. code-block:: c

  /**
    用于防止游戏战斗时候checkGameCommunityNotice 弹出通知弹窗打断游戏战斗画面，游戏厂商可以再进入游戏战斗界面的时候调用这个接口防止通知弹窗打断战斗
    重新调用checkGameCommunityNotice战斗状态会清除
 
    参数说明：bInComBat ：YES进入战斗状态，NO为解除战斗状态 再次调用checkGameCommunityNotice会自动置NO
 */
 + (void)setInComBat:(BOOL)bInComBat;


工程代码改变
=============

为了正确的横竖屏方向，请务必按下面的步骤调用相关的方法进行初始化和设置！

- 初始设置所支持的屏幕方向

.. code-block:: c

    - (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
    {
        //告诉SDK应用所支持的屏幕方向
        [GameCommunityEntry initAppOrientationMask:XXX];
    }

- 设置应用支持的屏幕方向：用于支持游戏内嵌社区横竖屏切换，游戏社区关闭后该值会修改为初始设置的屏幕方向不会修改游戏初始方向

.. code-block:: c

    - (UIInterfaceOrientationMask)application:(UIApplication *)application supportedInterfaceOrientationsForWindow:(UIWindow *)window
    {//AppDelegate 的supportedInterfaceOrientationsForWindow回调接口中处理
        return [GameCommunityEntry appOrientationMask];//或者调用return UIInterfaceOrientationMaskAllButUpsideDown;调用UIInterfaceOrientationMaskAllButUpsideDown 游戏自己界面需要处理好自己的横竖屏状态
    }
  





