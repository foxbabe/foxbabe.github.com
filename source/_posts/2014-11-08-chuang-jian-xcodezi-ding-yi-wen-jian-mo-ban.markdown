---
layout: post
title: "创建Xcode自定义文件模板"
date: 2014-11-08 21:37:08 +0800
comments: true
categories: iOS开发专题
---
#前言
---
自从团队执行代码规范以来，对于直接继承系统控件编写的方式，如继承最常用的UIViewController控制器，系统会自动生成一些默认父类的方法，如viewDidLoad，didReceiveMemoryWarning等。而在实际项目中，对于一个控制器，我们希望自带的方法并不仅仅就这些，可能更多我们会按照编码风格在任何控制器里面都会按照一定的布局方式添加一些公共的方法，如数据初始化、界面初始化；或默认设置一些参数，如控制器标题、公共pragma等。之前最初的做法是采用 `code snippets` 代码片段的形式保存并设置相应参数来统一处理。但是同样的也会出现一些问题，如小伙伴没设置`code snippets`或设置的代码片段格式不正确，或开发过程中忘记设置代码片段，都还会造成一些差异。

但是如果有模板文件，创建时这些内容在模板文件里面都有了，上述的这些问题都可以很好的来避免了。

#Xcode自带文件模板介绍
---
使用Xcode继承UIView或UIViewController等时，系统会默认添加一些共用的方法，这些对于不同版本的Xcode创建自带的方法会不同。学习Xcode模板格式最简单的方式就是先学习Xcode自带的模板，但是不幸的是苹果目前并为提供模板的语法，不过我们可以体验下Xcode中自带的模板，并尝试自定义我们自己的模板。下面我们以Xcode6为例子，列举其自带的模板：

###Mac文件模板

    应用程序 ▸ Xcode.app ▸ Contents ▸ Developer ▸ Library ▸ Xcode ▸ Templates

###iOS文件模板

    应用程序 ▸ Xcode.app ▸ Contents ▸ Developer ▸ Platforms ▸ iPhoneOS.platform ▸ Developer ▸ Library ▸ Xcode ▸ Templates

我们可以按照相应的路径打开对应的文件，但是建议不要直接编辑这些文件，如果语法错误或编辑不合理，可能会造成Xcode奔溃。但是我们可以拷贝这些文件到对应的"user templates"文件夹下进行编辑，这个文件默认是没有的，需要我们自己创建，路径如下：

    用户 ▸ Fox ▸ 资源库 ▸ Developer ▸ Xcode ▸ Templates ▸ File Templates

任何自己创建或修改的文件模板都可以存放到这个目录中，每个目录最好根据对应文件模板的内容给出一个有意义的名称，例如你们公司的名称，例如下面的目录：

![自定义文件模板](http://dn-foxbabe.qbox.me/QQ20141116-1.png)

#创建自定义文件模板
---
关于如何自定义文件模板，上面介绍了对应的模板，接下来我们主要看下文件模板的组件和对应的语法。

###模板文件介绍
一个模板文件是一个以.xctemplate结尾的文件夹，里面包含一些文件，其实就是一个普通的文件夹，但是必须要以.xctemplate结尾才能被Xcode所识别为文件模板。如果前面看过默认的文件模板，你会发现每个文件夹下有下面几项内容：


| 文件        			| 描述  		| 
| -------------------- | -----:  	|
| TemplateInfo.plist   | 定义和配置文件模板 	               |   
| TemplateIcon.icns    | 创建文件模板的图标，默认大小为128&128 |
| ___FILEBASENAME___.h | 模板文件的头文件                    |
| ___FILEBASENAME___.m | 模板文件的实现文件                   |


接下来我们可以自己来创建自己的文件模板，在`File Templates` 中创建一个.xctemplate结尾的文件夹，并且在文件夹中键入上面的四个文件，可以先直接从Xcode自带的模板文件中拷贝过来再进行修改。

###理解 TemplateInfo.plist 文件
文件模板中最重要的就是TemplateInfo.plist，该文件配置了模板的所有信息。通过这个文件，可以定义输出一个文件还是多个文件，可以配置用户创建文件时Xcode上的样式。一般可以通过Xcode进行编辑plist文件里的内容，也可以通过文本编辑器，如Sublime text或 Byword等进行编辑。

文件分为两部分，第一部分：定义模板文件的基本配置信息；第二部分：定义模板文件的用户界面和输出变量。下面是模板文件基本配置信息的一些key值和对应的意思：


| Key        			| Value(s)  		| 
| -------------------- | -----: 	 	|
| AllowedTypes   | 定义允许的源代码类型，默认为支持objective-c和swift|   
| DefaultCompletionName    | 定义在保存对话框中的默认按钮，默认为类名 |
| Description | 模板的描述文字，在创建新文件的时候，在对话框中可以看到  |
| Kind | 未知，目前发现所有的文件模板该值均为Xcode.IDEKit.TextSubstitutionFileTemplateKind           |
| MainTemplateFile | 创建的主要的源代码或资源文件            |
| Platforms | 模板文件支持的平台                   |
| Summary | 模板文件简介                   |
| SortOrder | 在新建模板文件中该文件出现的顺序                   |
| BuildableType | 这个值可以被用来预填充目标的复选框中的“保存”对话框中的只，有效值包括：无与测试 |


理解上面的键值对，我们就可以创建一个模板文件了，有些非必要的键值我们可以不用填充

![自定义文件模板](http://dn-foxbabe.qbox.me/QQ20141116-2.png)

除了上述简单配置信息外，还需要创建相应的模板文件源文件。

###模板文件源文件

我们希望输出的模板文件是一个Objective-c类文件，配种中MainTemplateFile默认的值为___FILEBASENAME___.m.除了实现文件，我们还需要创建起对应的头文件。

___FILEBASENAME___.h

```
//
//  ___FILENAME___
//  ___PROJECTNAME___
//
//  Created by ___FULLUSERNAME___ on ___DATE___.
//  Copyright (c) ___YEAR___ ___ORGANIZATIONNAME___. All rights reserved.
//
 
@interface ___FILEBASENAMEASIDENTIFIER___ : NSObject
 
@end
```

___FILEBASENAME___.m

```
//
//  ___FILENAME___
//  ___PROJECTNAME___
//
//  Created by ___FULLUSERNAME___ on ___DATE___.
//  Copyright (c) ___YEAR___ ___ORGANIZATIONNAME___. All rights reserved.
//
 
 #import "___FILEBASENAME___.h"
 
@implementation ___FILEBASENAMEASIDENTIFIER___
 
@end

```


源模板有一些前缀和后缀三重下划线的值。这些是内置的，可以被用来提供在输出动态值的扩展宏。下面的表列出了已知的宏：


| 宏        			| 描述  		| 
| -------------------- | -----: 	|
| ___FILENAME___   | 文件名称|
| ___FILEBASENAMEASIDENTIFIER___   | 不带扩展名的文件名转换为有效的C风格的标识符|   
| ___PROJECTNAME___    | 项目名称|
| ___PROJECTNAMEASIDENTIFIER___ | 项目名称转换为有效的C风格的标识符  |
| ___USERNAME___ | 用户名，缩写   |
| ___FULLUSERNAME___ | 用户名，全称   |
| ___ORGANIZATIONNAME___ | 使用Xcode创建项目时，设定的组织名称 |
| ___DATE___ | 今天的日期 |
| ___TIME___ | 当前时间    |
| ___YEAR___ | 当前年份	 |


接来下我们可以使用我们自己新创建的模板文件创建信息文件了。

![自定义文件模板](http://dn-foxbabe.qbox.me/QQ20141116-3.png)

#总结
---
使用自定义的模板文件，对于公共代码部分可以比较好的统一，特别是对于多个人开始时，统一编码风格更为重要。

###参考
[Creating Custom Xcode 4 File Templates](http://www.bobmccune.com/2012/03/04/creating-custom-xcode-4-file-templates/)

[Creating Custom Xcode 4 Project Templates](http://meandmark.com/blog/2011/12/creating-custom-xcode-4-project-templates/)

