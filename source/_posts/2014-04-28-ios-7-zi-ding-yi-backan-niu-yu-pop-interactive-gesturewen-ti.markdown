---
layout: post
title: "iOS 7 自定义Back按钮 与 Pop interactive gesture 问题"
date: 2014-04-28 20:40:05 +0800
comments: true
categories: iOS
---


###1、自定义Back按钮


iOS中很多时候我们都会自定义返回按钮，也是一件easy的事，类似如下：
	
	// 自定义返回按钮
	 - (void)showNavBackButton
	 {
	     UIButton *backButton = [UIButton buttonWithType:UIButtonTypeCustom];
	     [backButton addTarget:self action:@selector(backButtonAction:)
	          forControlEvents:UIControlEventTouchUpInside];
	     [backButton setBackgroundImage:[UIImage imageNamed:@"00_back_button"]
	                           forState:UIControlStateNormal];
	     [backButton setTitle:kLoc(@"Back") forState:UIControlStateNormal];
	     [backButton setTitleEdgeInsets:UIEdgeInsetsMake(0, 5, 1, 0)];
	     backButton.titleLabel.font = kMediumFont(12);
	     backButton.frame = CGRectMake(0, 0, 51, 31);
	     self.navigationItem.leftBarButtonItem
	     = [[UIBarButtonItem alloc] initWithCustomView:backButton];
	}

但是，这样在iOS7下Pop interactive gesture就不好使了。

这里 [Here](http://stuartkhall.com/posts/ios-7-development-tips-tricks-hacks) 有一个解决方法。

但是，测试发现在栈中推入一个controller后，快速向左平滑，将会引起崩溃。

查看崩溃日志，发现如下信息：

**nested pop animation can result in corrupted navigation bar**


###2、解决Pop interactive gesture问题


优化的解决方案是简单的让NavigationController自己成为响应的接受者，最好用一个UINavigationController的子类。

1)在过渡的时候禁用interactivePopGestureRecognizer

2)当新的视图控制器加载完成后再启用，建议使用UINavigationController的子类操作

	// 自定义NavigationController
	@interface DCBaseNavigationController ()
	<
	UINavigationControllerDelegate,
	UIGestureRecognizerDelegate
	>
	@end
	
	@implementation DCBaseNavigationController
	
	- (void)dealloc
	{
	    self.interactivePopGestureRecognizer.delegate = nil;
	    self.delegate = nil;
	    
	    [super dealloc];
	}
	
	#pragma mark - View lifecycle
	
	- (void)viewDidLoad
	{
	    [super viewDidLoad];
	    // Do any additional setup after loading the view.
	    
	    if ([self respondsToSelector:@selector(interactivePopGestureRecognizer)]) {
	        self.interactivePopGestureRecognizer.delegate = self;
	        self.delegate = self;
	    }
	}
	
	- (void)didReceiveMemoryWarning
	{
	    [super didReceiveMemoryWarning];
	    // Dispose of any resources that can be recreated.
	}
	
	#pragma mark - Override
	
	- (void)pushViewController:(UIViewController *)viewController animated:(BOOL)animated
	{
	    // Hijack the push method to disable the gesture
	    if ([self respondsToSelector:@selector(interactivePopGestureRecognizer)]) {
	        self.interactivePopGestureRecognizer.enabled = NO;
	    }
	    
	    [super pushViewController:viewController animated:animated];
	}
	
	#pragma mark - UINavigationControllerDelegate
	
	- (void)navigationController:(UINavigationController *)navigationController
	       didShowViewController:(UIViewController *)viewController
	                    animated:(BOOL)animate
	{
	    if ([navigationController respondsToSelector:@selector(interactivePopGestureRecognizer)]) {
	        navigationController.interactivePopGestureRecognizer.enabled = YES;
	    }
	}
	
	@end


###3、Pop interactive gesture冲突，造成页面假死问题


我遇到的情况是，Push/Pop页面时，没有立即得到想要的效果，页面没有显出出来，NavigationController的didShowViewController:回调方法也没有调用。

页面布局情况是这样的：视图A，有一个Pan手势；视图B是TabBarController，其ViewControllers都是NavigationController。视图B是视图A的子视图。

后来找到原因是：navigationController的interactive pop手势与视图A的pan手势冲突。

具体原因是：rootViewController加载时，调用了didShowViewController:，设置interactivePopGestureRecognizer可用，其实我们并不需要在root的时候也触发这个手势。所以稍加优化如下：

	// 优化
	- (void)navigationController:(UINavigationController *)navigationController
	       didShowViewController:(UIViewController *)viewController
	                    animated:(BOOL)animate
	{
	    if ([navigationController respondsToSelector:@selector(interactivePopGestureRecognizer)]) {
	        //if ([[navigationController.viewControllers firstObject] isEqual:viewController]) {
	        if ([navigationController.viewControllers count] == 1) {
	            // Disable the interactive pop gesture in the rootViewController of navigationController
	            navigationController.interactivePopGestureRecognizer.enabled = NO;
	        } else {
	            // Enable the interactive pop gesture
	            navigationController.interactivePopGestureRecognizer.enabled = YES;
	        }
	    }
	}

当前显示的是root时，设置interactivePopGestureRecognizer不可用，非root时设置interactivePopGestureRecognizer可用。

参考文章：[http://keighl.com/post/ios7-interactive-pop-gesture-custom-back-button/](http://keighl.com/post/ios7-interactive-pop-gesture-custom-back-button/)
