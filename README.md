<p align="center">
<a href="https://github.com/yulingtianxia/BlockHook">
<img src="Assets/logo.png" alt="BlockHook" />
</a>
</p>

[![CI Status](http://img.shields.io/travis/yulingtianxia/BlockHook.svg?style=flat)](https://travis-ci.org/yulingtianxia/BlockHook)
[![Version](https://img.shields.io/cocoapods/v/BlockHook.svg?style=flat)](http://cocoapods.org/pods/BlockHook)
[![Carthage compatible](https://img.shields.io/badge/Carthage-compatible-4BC51D.svg?style=flat)](https://github.com/Carthage/Carthage)
[![License](https://img.shields.io/cocoapods/l/BlockHook.svg?style=flat)](http://cocoapods.org/pods/BlockHook)
[![Platform](https://img.shields.io/cocoapods/p/BlockHook.svg?style=flat)](http://cocoapods.org/pods/BlockHook)

# BlockHook

Hook Objective-C blocks with libffi. It's a powerful AOP tool for blocks. BlockHook can run your code before/instead/after invoking a block. BlockHook can even notify you when a block dealloc. You can trace the whole lifecycle of a block using BlockHook!

## 📚 Article

- [Hook Objective-C Block with Libffi](http://yulingtianxia.com/blog/2018/02/28/Hook-Objective-C-Block-with-Libffi/)

## 🌟 Features

- [x] Easy to use.
- [x] Keep your code clear.
- [x] Reserve the whole arguments.
- [x] Support 4 hook modes: Before, Instead, After and Dead.
- [x] Use tokens to change hook mode dynamically.
- [x] Modify return value.
- [x] Self-managed tokens.

## 🔮 Example

BlockHook needs libffi, which is a submodule in this project. You should use `--recursive` when clone this sample, or you can use these commands get the submodule.

```
cd libffi
git submodule init
git submodule update
```

If targets in Xcode fails to compile, you need do these in libffi folder:

- run `./autogen.sh`
- run `./configure`
- run `python generate-darwin-source-and-headers.py`

The sample project "BlockHookSample" just only support iOS platform. You must build libffi for every architecture you need.

## 🐒 How to use

You can hook a block using 4 modes (before/instead/after/dead). This method returns a `BHToken` instance for more control. You can `remove` a `BHToken`, or set custom return value to its `retValue` property.

```
- (BHToken *)block_hookWithMode:(BlockHookMode)mode
                     usingBlock:(id)block
```

BlockHook is easy to use. Its APIs take example by Aspects. Here is a full set of usage of BlockHook.

```
NSObject *z = NSObject.new;
int (^block)(int, int) = ^(int x, int y) {
   int result = x + y;
   NSLog(@"I'm here! result: %d, z is a NSObject: %p", result, z);
   return result;
};
    
    
BHToken *tokenInstead = [block block_hookWithMode:BlockHookModeInstead usingBlock:^(BHToken *token, int x, int y){
   // change the block imp and result
   *(int *)(token.retValue) = x * y;
   NSLog(@"hook instead");
}];

BHToken *tokenAfter = [block block_hookWithMode:BlockHookModeAfter usingBlock:^(BHToken *token, int x, int y){
   // print args and result
   NSLog(@"hook after block! x:%d y:%d ret:%d", x, y, *(int *)(token.retValue));
}];

BHToken *tokenBefore = [block block_hookWithMode:BlockHookModeBefore usingBlock:^(id token){
   // BHToken has to be the first arg.
   NSLog(@"hook before block! token:%@", token);
}];
    
BHToken *tokenDead = [block block_hookWithMode:BlockHookModeDead usingBlock:^(id token){
   // BHToken is the only arg.
   NSLog(@"block dead! token:%@", token);
}];
    
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
   NSLog(@"hooked block");
   int ret = block(3, 5);
   NSLog(@"hooked result:%d", ret);
   // remove all tokens when you don't need.
   // reversed order of hook.
   [tokenBefore remove];
   [tokenAfter remove];
   [tokenInstead remove];
   NSLog(@"original block");
   ret = block(3, 5);
   NSLog(@"original result:%d", ret);
//        [tokenDead remove];
});
```

Here is the log:

```
hooked block
hook before block! token:<BHToken: 0x1d00e0f80>
hook instead
hook after block! x:3 y:5 ret:15
hooked result:15
original block
I'm here! result: 8, z is a NSObject: 0x1c0011ec0
original result:8
block dead! token:<BHToken: 0x1c40e8480>
```

## 📲 Installation

### CocoaPods

[CocoaPods](http://cocoapods.org) is a dependency manager for Cocoa projects. You can install it with the following command:

```bash
$ gem install cocoapods
```

To integrate BlockHook into your Xcode project using CocoaPods, specify it in your `Podfile`:


```
source 'https://github.com/CocoaPods/Specs.git'
platform :ios, '11.0'
use_frameworks!
target 'MyApp' do
	pod 'BlockHook'
end
```

You need replace "MyApp" with your project's name.

Then, run the following command:

```bash
$ pod install
```

### Carthage

[Carthage](https://github.com/Carthage/Carthage) is a decentralized dependency manager that builds your dependencies and provides you with binary frameworks.

You can install Carthage with [Homebrew](http://brew.sh/) using the following command:

```bash
$ brew update
$ brew install carthage
```

To integrate BlockHook into your Xcode project using Carthage, specify it in your `Cartfile`:

```ogdl
github "yulingtianxia/BlockHook"
```

Run `carthage update` to build the framework and drag the built `BlockHookKit.framework` into your Xcode project.

### Manual

After importing libffi, just add the two files `BlockHook.h/m` to your project.

## ❤️ Contributed

- If you **need help** or you'd like to **ask a general question**, open an issue.
- If you **found a bug**, open an issue.
- If you **have a feature request**, open an issue.
- If you **want to contribute**, submit a pull request.

## 👨🏻‍💻 Author

yulingtianxia, yulingtianxia@gmail.com

## 👮🏻 License

BlockHook is available under the MIT license. See the LICENSE file for more info.

Thanks to MABlockClosure and Aspects!

