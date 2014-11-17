---
layout: post
title: "iOS代码库"
date: 2014-11-03 23:53:21 +0800
comments: true
categories: 
---

## 说明
### 版本信息

 - 2014.07.30 V1.0 创建iOS代码库文档初版

###创建iOS代码库目的

 - 整理项目中常见控件和功能，提高项目间功能复用度
 - 便于iOS工程师之间的技术交流，增强公司内部技术分享沟通氛围
 - 便于初级iOS工程师尽快提升iOS技术开发能力

###iOS代码库简介
![代码库简介][1]

TTILibrary
- Category (常用扩展)
- Helper (常用工具类)
- Macro (公共宏和第三方参数宏)
- NetWork(网络相关操作)
- UI(常用控件)
iOSCodeProject (项目名称，可根据实际进行修改)
- Resource (项目中常用资源)
- Depend （常用依赖库）
- Models (对象数据)
- Views （项目中共用控件）
- Controlers （项目中的控制器层）

---

## UI控件类

### 列表下拉刷新和上拉加载更多
使用PullingRefreshTableView实现上下拉动和分页效果，基本使用：
```
    //创建列表
    self.pullTable = [[PullingRefreshTableView alloc] init];
    self.pullTable.frame = self.view.bounds;
    self.pullTable.delegate=self;
    self.pullTable.dataSource = self;
    self.pullTable.pullingDelegate = self;
    self.pullTable.autoresizingMask = UIViewAutoresizingFlexibleHeight|UIViewAutoresizingFlexibleBottomMargin;
    [self.view addSubview:self.pullTable];
   
    //设置委托
    #pragma mark - UIScrollView Delegate
-(void)scrollViewDidScroll:(UIScrollView *)scrollView
{
    [_pullTable tableViewDidScroll:scrollView];
}
- (void)scrollViewDidEndDragging:(UIScrollView *)scrollView willDecelerate:(BOOL)decelerate
{
    [_pullTable tableViewDidEndDragging:scrollView];
}

-(NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section
{
    return [_sourceArray count];
}

-(UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath{
    static NSString *CellIdentifier = @"cell";
    UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:CellIdentifier];
    if (cell==nil) {
        cell = [[UITableViewCell alloc] initWithStyle:UITableViewCellStyleDefault reuseIdentifier:CellIdentifier];
        cell.selectionStyle = UITableViewCellSelectionStyleNone;
    }

    cell.textLabel.text = _sourceArray[indexPath.row];
    
    return cell;
    
}

#pragma  mark - PullingRefreshTableView Delegate
-(void)pullingTableViewDidStartRefreshing:(PullingRefreshTableView *)tableView
{
    [self refreshAction];
}
-(void)pullingTableViewDidStartLoading:(PullingRefreshTableView *)tableView
{
    [self loadingAction];
}

//处理刷新事件
-(void)refreshAction
{
    [self.sourceArray insertObject:@"下拉刷新" atIndex:0];
    [self performSelector:@selector(stopRefresh) withObject:nil afterDelay:1.5f];
    
}
-(void)loadingAction
{
    [self.sourceArray addObject:@"上拉加载更多"];
    [self performSelector:@selector(stopRefresh) withObject:nil afterDelay:1.5f];
}

-(void)stopRefresh{
    [_pullTable tableViewDidFinishedLoading];
    [_pullTable reloadData];
}

```
    
### 图片缩放和图片浏览器
图片的缩放和图片浏览器采用MWPhotoBrowser来实现，使用方法参考工程

### 自定义索引栏/通讯录列表；
对于常见的通讯录，部分项目上希望自定义索引兰的文字颜色、字体和背景
 - 支持修改索引兰文字颜色、字体和大小
 - 支持点击索引兰，处于高亮状态时的背景颜色

```
    //创建索引兰
    TTIIndexBar *indexBar = [[TTIIndexBar alloc] initWithFrame:CGRectMake
                             (self.view.frame.size.width-35, 10.0, 28.0, self.view.frame.size.height-80)];
    indexBar.textColor = [UIColor greenColor];      //索引栏文字颜色
    indexBar.highlightedBackgroundColor = [UIColor redColor];       //高亮选中背景的颜色
    [indexBar setIndexes:[NSMutableArray arrayWithObjects:@"A",@"B",@"C",@"D",@"E",@"F",@"G", nil]];
    indexBar.delegate = self;
    [self.view addSubview:indexBar];
    
    //处理索引兰事件
    #pragma mark - TTIIndexBarDelegate methods
- (void)indexSelectionDidChange:(TTIIndexBar *)indexBar index:(NSInteger)index title:(NSString*)title{

    NSLog(@"%d %@",index,title);
}
    
```

### GIF加载

 - 使用YLGIFImage加载GIF方便快速
 - 占用内存少

```
YLImageView* imageView = [[YLImageView alloc] initWithFrame:CGRectMake(0, 160, 320, 240)];
[self.view addSubview:imageView];
imageView.image = [YLGIFImage imageNamed:@"joy.gif"];
```
### 自定义选择器（时间、字符串）
使用系统的选取和toolbar自定义常用的时间和字符串选择器

 - 字符串选择器
 - 时间选择器

```
//选择字符串
TTIPickerView *pickerView  = [[TTIPickerView alloc] init];
pickerView.sourceArray = [NSMutableArray arrayWithArray:
 @[@"测试1",@"测试2",@"测试3",@"测试4",@"测试5",@"测试6"]];
[pickerView show:^(NSInteger row, NSString *titleOfRow) {
    NSLog(@"%d %@",row,titleOfRow);
}];
    
//选择时间
TTIDateView *dateView = [[TTIDateView alloc] init];
[dateView show:^(NSString *dateString) {
    NSLog(@"%@",dateString);
}];
    
```

### iOS7风格的Switch
项目中通常会设计为iOS7风格的switch，但是在iOS6上的效果和iOS7上是有一定差异的，该控制支持此功能。
```
SevenSwitch *swich = [[SevenSwitch alloc] initWithFrame: CGRectMake(260, 10, 50, 30)];
[self.view addSubview: swich];
```

### 富文本控件

![富文本效果][2]

 - 支持显示不同字体、不同颜色和大小的富文本控件
 - 支持hmtl标签形式
 - 支持图片居中显示

```
//创建富文本控件
    self.scrollView = [[UIScrollView alloc] initWithFrame:self.view.bounds];
	self.scrollView.autoresizingMask = UIViewAutoresizingFlexibleWidth | UIViewAutoresizingFlexibleHeight;
    
    self.coreTextView = [[FTCoreTextView alloc] initWithFrame:CGRectInset(bounds, 20.0f, 0)];
	self.coreTextView.autoresizingMask = UIViewAutoresizingFlexibleWidth |
    UIViewAutoresizingFlexibleHeight;
    
    //添加风格
    [self.coreTextView addStyles:[self coreTextStyle]];
    
    //设置文本
    self.coreTextView.text = [self textForView];
    self.coreTextView.delegate = self;
   
    [self.scrollView addSubview:self.coreTextView];
    [self.view addSubview:self.scrollView];
    
    //更多用法可参考:https://github.com/FuerteInternational/FTCoreText
    
    //可自定义设置多种样式
```

### 云标签
云标签效果图

```
NSArray *labelARy =[NSArray arrayWithObjects:@"吃惊威龙",@"摧残人僧",@"赏金杀手",@"疯狂原始人",@"神偷奶爸",@"致命黑兰",@"冥界警局",@"狂鲨之灾",@"北海巨妖",@"海扁王2",@"变形金刚3",@"史前一亿年",@"大片",nil];
   
 //创建云标签 
CloudView *cv=[[CloudView alloc] initWithFrame:CGRectMake(0, 0, 320, 320)];
[cv reloadData:labelARy];
[self.view addSubview:cv];
[cv setBackgroundColor: [UIColor blackColor]];
```

### 广告页Banners循环滚动效果

采用类似UItableview的委托形式，创建支持循环滚动的Banners效果控件
```
self.cycleScrollView = [[TTICycleScrollView alloc] initWithFrame:CGRectMake(0, 30, 320, 100)];
self.cycleScrollView.delegate = self;
self.cycleScrollView.datasource = self;
[self.view addSubview:self.cycleScrollView];

    
    for (int i = 0; i < 10; i++) {
        NSString *num = [NSString stringWithFormat:@"%d_index",i + 1];
        [self.dataSource addObject:num];
    }
    
    [self.cycleScrollView reloadData];
    
    //设置代理和数据源
    #pragma mark - TTICycleScrollViewDelegate methods
- (void)didClickPage:(TTICycleScrollView *)csView atIndex:(NSInteger)index{

    //点击事件
}

#pragma mark - TTICycleScrollViewDatasource methods
- (NSInteger)numberOfPages{

    return self.dataSource.count;
}

- (UIView *)pageAtIndex:(NSInteger)index{

//这里的视图可以根据实际情况自定义
    UILabel *tip = [[UILabel alloc] initWithFrame:CGRectMake(0, 0, 320, 100)];
    if (index%3 == 0) {
        tip.backgroundColor = [UIColor redColor];
    }else if (index%3 == 1){
        tip.backgroundColor = [UIColor grayColor];
    }else{
        tip.backgroundColor = [UIColor greenColor];
    }
    
    tip.text = [self.dataSource objectAtIndex:index];
    tip.textColor = [UIColor blackColor];
    tip.textAlignment = NSTextAlignmentCenter;
    
    return tip;
}

```

### 饼图、折线图、条形图

使用开源PNChart控件(https://github.com/kevinzhow/PNChart)和自定义的TTIGoalBar实现三种风格图标
### 自由截图视图
截图当前屏幕的图片
### 拼音视图
中文转换为拼音
### 自定义的控制器、导航控制器和标签栏控制器
继承系统的UIViewController、UINavagationContoller和UITabbarController，支持更多自定义操作，满足实际项目需求
### 瀑布流和分页加载
使用PSCollectionView和顶部、尾部加载控件，实现瀑布流和分页加载效果
### 自定义评分控件
支持评分和仅显示星级数功能
### 自定义滑动cell

 - 支持自定义滑动删除时的视图
 - 支持ios6上的滑动删除

### 自定义TextView和TextFIled

 - 支持TextFIled修改placeholder的文字颜色
 - 支持TextView添加placeholder功能和修改文字颜色

### 自定义PageControl

 - 支持自定义pagecontrol的颜色大小和间距

---

## 功能类
### 拍照/拍视频/录音
提供常见的拍照、拍适配和录音频的操作

    
```
//选择图片
    WEAKSELF
    [[TTIImagePickerManager shareInstance] imagePickerWithType:TTIImagePickerManagerBoth enableEditing:YES withDelegate:self comleteBlock:^(UIImage *resultImage, UIImage *orignImage) {
       
        weakSelf.selectImage.image = resultImage;
        
    } failedBlock:^(NSError *error) {
        
    }];
    
    //选择视频
        [[TTIVideoPickerManager shareInstance] videoPickerWithType:TTIVideoPickerManagerBoth withDelegate:self isConvertMP4:NO comleteBlock:^(NSString *resultPath, NSString *orignPath) {
        
        NSLog(@"视频地址:%@",resultPath);
        
    } failedBlock:^(NSError *error) {
        
    }];
    
    //选择音频
     [[TTIAudioPickerManager shareInstance] audioPickerWithType:TTIAudioPickerManagerBoth withDelegate:self isConvertMP3:YES comleteBlock:^(NSString *resultPath, NSString *orignPath) {
        
        NSLog(@"音频地址:%@",resultPath);
        
    } failedBloc:^(NSError *error) {
        
    }];
    
```

### 打电话、发短信、发邮件
常见工具操作
### 二维码扫描
使用ZBar进行二维码扫描功能
### ZIP文件的解压与压缩
对于数据包的形式，通常需要进行文件的压缩和解压
### 应用内语言切换
强制在程序内部进行语言切换

---

## 工具类
### 字符串处理

 - 去除前后空格
 - 字符串为空的判断
 - 字符串转各种数值类型
具体请见代码库中：NSString+Category.h扩展类

### 图片工具类
 - 倒影效果
 - 取特定位置
 - 转换成指定大小的图片
 - 等比缩放
 - 获取本地图片
具体请见代码库中：UIImage+Utils.h扩展类

### 日志工具类

 - 保存异常信息
 - 记录奔溃设备的信息
已弃用，使用友盟错误分析进行统计

### 日期工具类

 - 字符串转日期
 - 字符串日期的大小对比
 - 相差几天的对比
 - 判断是否在本周/上周/两周前
 - 与当前时间判断为”刚刚/几分钟前/几天前/几月前
 - ...
具体请见代码库中：NSDate+Utils.h扩展类

### 验证工具类

常用数据格式判断
 - 手机号
 - 邮箱
 - 身份证
 - 电话
 - 纯数字
 - 用户名和昵称
 - ...

### 字体工具类

 - 显示系统所有字体信息
 - 加载自定义字体
 - 程序字体统一处理
 - 加粗和斜体等字体操作

### 对话框工具类

 - UIAlterView支持block，避免多个弹出框时多个tag区分
 - 支持多种情况的block

### 加密处理 

 - MD5加密
 - AES加密解密
 - BASE64 编码
 - ...

### 视图工具类

 - 单独设置x/y/width/height
 - 直接获取各种位置信息
 - 直接记载nib文件
具体请见代码库中：UIView+Utils.h扩展类

### 动画工具类

 - 平移
 - 翻转
 - 摇晃
 - 渐隐渐现
 - ....
具体请见代码库中：UIView+AnimationUtils.h扩展类

---

## 动画类

### 磨盘效果控件

 - 吉利汽车，车型展示效果
 - qq游戏大厅，游戏项目转动效果

### 3D画廊

 - 印象城中首页活动切换效果
 - 印象城中楼层切换效果

### 360、720度全景

 - 360度图片展示
 - 720度全景图展示

### 导航控制器切换效果

 - 支持滑动返回分层效果

### 抽屉效果

 - 左右滑动抽屉效果

---

## 数据存储

### 使用UserDefaults存储用户信息 

 - 存储用户配置信息
 - 存储本地对象，需支持NSCopying

### 使用Plist存储信息

 - 本地文件储存（plist即xml文件）
 - KV（Key-Vaule）形式进行存储
 - 支持NSString/NSArray/NSDictionary迅速储存

### SQList数据操作存储，采用FMDB操作

 - 采用FMDB，非常方便操作sqlite数据库
 - 常见SQL操作语句，可参考（https://github.com/ccgus/fmdb）

### CoreData数据存储

---

## 网络操作
### 数据对象操作（自动映射）
使用开源控件JSONModel（https://github.com/icanzilb/JSONModel）完成，支持如下功能：

 - 对象自动映射
 - 多类型支持
 - 多重映射支持
 - 映射字段自定义支持

### Http网络接口操作

基于开源库AFNetworking（https://github.com/AFNetworking/AFNetworking）封装的请求类：
 - TTIHttpClient处理所有接口请求和操作
 - TTIRequest处理网络请求信息
 - TTIResponse处理网络返回数据信息

### TCP Socket通信操作

基于开源库CocoaAsyncSocket（https://github.com/robbiehanson/CocoaAsyncSocket）封装的请求类
 - TTISocketClient处理tcp连接、断开、重连、信息发送和粘包等处理
 - TTISocketRequest处理发送出去的帧信息
 - TTISocketResponse处理接受到的帧信息

### 网络状况判断

 - 判断是否有网、是否为2G/3G环境、是否为WIFI环境
 - 网络切换时的通知发送
 - 处理方式一：使用datatify处理（http://huytd.github.io/datatify/）
 - 处理方式二：使用AFNetworkReachabilityManager处理
 - 处理方式三：使用Reachability处理

---

## 第三方平台

### 分享

 - 新浪、腾讯微博、qq控件、qq好友、朋友圈、微信好友、邮件等使用share sdk 进行分享，可分包下载平台支持库
 - 文本、图片、连接和应用的分享
 - 第三方登录（微信平台需要申请）

### 支付

 - 支付宝
 - 银联
 - 建行
 - iOS应用内支付
 - ...

### 地图

 - 百度
 - 谷歌
 - 高德

### 统计

 - 使用友盟
 - 数据统计
 - 自定义事件统计
 - 错误分析

### 推送

 - 极光
 - 信鸽
 - 个推


推送中需注意的事项：

 - 角标问题
 - 程序在前台收到推送和在后台收到推送
 - 标签和别名
 - 注销推送

 
---

## 常用开源库

### AFNetworking网络
 - 优秀网络框架库
 
### SVProgressHUD和SVPullToRefresh
 - 提示语
 - 列表刷新加载
 
### YLGIFImage
 - GIF显示

### SDWebimage加载图片
 - 网络图片加载

### OpenUDID
 - 第三方设备唯一标识符

### JSONModel
 - 数据映射工具

### iCarousel常用转动效果
 - 各种炫酷视图切换效果

### FTCoreText富文本排版
 - 以html标签的形式加载富文本

### VVDoucumenter代码注释工具
 - 优秀代码注释工具

---

## TODO

 1. iOS代码库使用.a文件生成
 2. 完善iOS代码库内容
 3. 整理和清晰代码库提交流程和审核流程
 4. 使用iOS代码库，创建项目模板工程


  [1]: http://workplace.qiniudn.com/QQ20140730-1.png
  [2]: http://workplace.qiniudn.com/QQ20140730-3.png