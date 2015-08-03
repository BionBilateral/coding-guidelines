##Objective-C

This document details coding style and coding conventions for Objective-C. This document assumes you are using Xcode.

##Coding Style

- Within _Preferences_ -> _Text Editing_ -> _Indentation_, set _Prefer indent using:_ to `Spaces` and make sure _Tab width:_ and _Indent width:_ are set to `4` spaces
- Within _Preferences_ -> _Text Editing_ -> _Indentation_, enable the `Wrap lines to editor width` option
- Optionally, within _Preferences_ -> _Text Editing_ -> _Indentation_, change the `Indent wrapped lines by:` option to something other than `0`

##Project Structure

The file structure of your Xcode project should be as flat as possible. Use groups and folder references within Xcode to organize appropriately. Not surprisingly, there are exceptions, just ask.

##Coding Conventions

###Class Names

- For each project, a meaningful class prefix should be chosen and used for everything, including categories (e.g. `BB` for Bion Bilateral)
- Class names should begin with the class prefix (e.g. `BBAddressBookPerson`)
- Category methods should begin with the class prefix (e.g. `BB_viewControllerForPresenting`)
- Model classes should not use the class prefix (e.g. `Person`), this ensure CoreData related classes are easily identifiable

###Constants

- Do not use `#define`, use properly typed constants as outlined below

###String Constants

- Public string constants should be declared using `extern NSString *const <name>;` in the header, with a corresponding `NSString *const <name> = <value>;` in the implementation
- Public string constants should begin with the class name, which includes the class prefix (e.g. `BBAddressBookManagerNotificationNameExternalChange`)
- If the string constant is a notification, the common suffix should be `Notification` (e.g. BBAddressBookManager**Notification**NameExternalChange)
- If the string constant is a dictionary key, the common suffix should be `Key` (e.g. BBFoundation**Key**Example)
- If the string constant is a user info key, the common suffix should be `UserInfoKey` (e.g. BBFoundation**UserInfoKey**Example)
- If the string constant is a user defaults key, the common suffix should be `UserDefaultsKey` (e.g. BBFoundation**UserDefaultsKey**Example)
- Private string constants should be declared using `static NSString *const <name> = <value>;` in the implementation
- Private string constants should begin with `k` (e.g. **k**BBFoundationKeyExample)
- Any string value that is user facing should be wrapped in `NSLocalizedString()` or one of its variants

###Primitive Constants

- Public variable constants should be declared using `extern <type> const <name>;` with a corresponding `<type> const <name> = <value>;` in the implementation
- Private variable constants should be declared using `static <type> const <name> = <value>;` in the implementation
    
###Enums

Use the `NS_ENUM` macro to define and list enum values. For example:

    /**
     Enum describing the authorization status of the AddressBook framework.
     */
    typedef NS_ENUM(NSInteger, BBAddressBookManagerAuthorizationStatus) {
        /**
         The status has not been determined for the calling application. The appropriate alert will be shown to grant access upon request.
         */
        BBAddressBookManagerAuthorizationStatusNotDetermined = kABAuthorizationStatusNotDetermined,
        /**
         The status has been restricted and the current user may not be able to modify it.
         */
        BBAddressBookManagerAuthorizationStatusRestricted = kABAuthorizationStatusRestricted,
        /**
         The user has denied access to the calling application. Prompt the user to adjust access in the Settings application.
         */
        BBAddressBookManagerAuthorizationStatusDenied = kABAuthorizationStatusDenied,
        /**
         The user has granted access to the calling application.
         */
        BBAddressBookManagerAuthorizationStatusAuthorized = kABAuthorizationStatusAuthorized
    };

The general pattern being:

    typedef NS_ENUM(<type>, <prefix>) {
        <prefix><suffix1> = <optional_value1>,
        <prefix><suffix2> = <optional_value2>,
        ...
    }

###Options

Use the `NS_OPTIONS` macro to define and list option values. For example:

    /**
     Flags describing the cache type that is used to store generated thumbnails.
     */
    typedef NS_OPTIONS(NSInteger, BBThumbnailGeneratorCacheOptions) {
        /**
         Caching is not enabled, the thumbnail will be generated each time it is requested.
         */
        BBThumbnailGeneratorCacheOptionsNone = 0,
        /**
         File caching is enabled, the generated thumbnail will be stored on disk.
         */
        BBThumbnailGeneratorCacheOptionsFile = 1 << 0,
        /**
         Memory caching is enabled, the generated thumbnail will be stored in memory.
         */
        BBThumbnailGeneratorCacheOptionsMemory = 1 << 1
    };

The general pattern being:

    typedef NS_OPTIONS(<type>, <prefix>) {
        <prefix><suffix1> = <value1>,
        <prefix><suffix2> = <value2>,
        ...
    }

##Class Interface, Extension, and Implementation

Do not use instance variables, use properties:

    @interface MyClass : NSObject
    
    @property (copy,nonatomic) NSString *myString;
    
    @end
    
    @interface MyClass ()
    @property (copy,nonatomic) NSString *myString;
    @end

Instead of:

    @interface MyClass : NSObject {
        NSString *_myString;
    }
    @end
    
    @interface MyClass () {
        NSString *_myString;
    }
    @end
    
    @implementation MyClass {
        NSString *_myString;
    }
    @end

###Properties

- Public properties should be declared before methods in the header
- Property attributes (e.g. `strong`, `assign`, etc.) should be ordered consistently, preferring: `@property (<assign,copy,strong,unsafe_unretained,weak>,<atomic,nonatomic>,<getter>=<getter_name>,<setter>=<setter_name>) <type> <name>;`
- The default property attributes are `readwrite`, `strong`, and `atomic`
- The `atomic` attribute is rarely needed, there is a huge difference between atomicity and thread safety
- Private properties should be declared in a class extension

        @interface MyClass ()
        @property (copy,nonatomic) NSString *myString;
        @end

- If a property should be read only publicly and readwrite privately, redefine it within the class extension

        @interface MyClass : NSObject
        @property (readonly,copy,nonatomic) NSString *myString;
        @end
        
        @interface MyClass ()
        @property (readwrite,copy,nonatomic) NSString *myString;
        @end
    
###Methods

- Use verbose method names, this is not vanilla C
- Do not prefix methods with _get_, this is not Java

        // Do this
        @interface MyClass : NSObject
        - (void)foo;
        @end
        
        // Not this
        @interface MyClass : NSObject
        - (void)getFoo;
        @end

- The exception to the above rule are methods that return multiple values by reference

        @interface UIColor : NSObject
        - (BOOL)getRed:(CGFloat *)red green:(CGFloat *)green blue:(CGFloat *)blue alpha:(CGFloat *)alpha;
        @end
    
- Methods that provide additional error information should do so using an `NSError` returned by reference and indicate success or failure using a boolean return value

        @interface MyClass : NSObject
        - (BOOL)fooBar:(NSError **)error;
        @end

- Action methods intended for attachment to UI controls should follow the below convention which allows Xcode to properly index the methods and allow for hookup within Interface Builder

        @interface MyClass : NSObject
        - (IBAction)myAction:(id)sender;
        @end

- Methods belonging to a data source or delegate protocol should always include the caller as the first parameter of the method

        @protocol MyClassDataSource
        - (NSInteger)numberOfThingsInMyClass:(MyClass *)myClass;
        - (id)myClass:(MyClass *)myClass thingAtIndex:(NSInteger)index;
        @end

- Private methods should be prefixed with `_`

        @interface MyClass ()
        - (void)_myPrivateMethod;
        @end

###Variables

- Use verbose variable names, this is not vanilla C
- Single letter variable names should be used where notation is appropriate

        for (NSInteger i=0; i<count; i++) {
            // do stuff
        }

- For primitive types, use the Apple provided typedefs
    - `int` becomes `NSInteger`
    - `unsigned int` becomes `NSUInteger`
    - `float` becomes `CGFloat`
    - `double` becomes `CGFloat`
    - `float` or `double` representing intervals of time becomes `NSTimeInterval`
    - location related values should use the typedefs located in _<CoreLocation/CLLocation.h>_

##Implementation Guidelines

###Subclassing

- Certain classes encourage subclassing
    - `UIViewController` is meant to be subclassed
    - `UIView` and its various descendants are meant to be subclassed
    - `UITableViewCell` is meant to be subclassed
    - `UIControl` is meant to be subclassed when developing a custom control that should not derive from an existing system control
    - `NSObject` is meant to be subclassed, it is the base class after all
- Before subclassing additional classes, determine whether your goal can be met by an alternative means (e.g. delegate methods)

###Notifications

Notifications should be used when the classes involved should remain loosely coupled or if multiple parties would be interested in the notification event. For example, multiple parties might be interested in knowing when an application becomes active on iOS, therefore the `UIApplicationDidBecomeActiveNotification` exists.

###Delegation

Delegation should be used when a tighter coupling should exist between classes, but the delegating class need not know anything about the delegate aside from whether it implements certain methods. For example, `UITableView` uses delegation to provide customization without having the know anything about the specific delegate.

###KVO/KVC

Key value observing and key value coding should be used instead of delegation when a tighter coupling should exist between the classes but multiple parties would be interested in the change, as delegation can only inform a single party. For example, `WKWebView` is key value observing compliant for its `loading` property, which can be observed by interested parties to know when the loading status has changed.

    static void *kObservingContext = &kObservingContext;
    
    @interface MyClass ()
    @property (strong,nonatomic) WKWebView *webView;
    @end
    
    @implementation MyClass
    
    - (void)dealloc {
        [self.webView removeObserver:self forKeyPath:@"loading" context:kObservingContext];
    }
    
    - (void)viewDidLoad {
        [self.webView addObserver:self forKeyPath:@"loading" options:0 context:kObservingContext];
        
        // classes we manage can also observe our WKWebView instance and react to changes
    }
    
    // override category method defined on NSObject to observe changes
    - (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary *)change context:(void *)context {
        if (context == kObservingContext) {
            if ([keyPath isEqualToString:@"loading"]) {
                // do stuff
            }
        }
        // call super if the observation is not meant for us
        else {
            [super observeValueForKeyPath:keyPath ofObject:object change:change context:context];
        }
    }
    
    @end

##Debugging

###Push Notifications

[NWPusher](https://github.com/noodlewerk/NWPusher) is a useful tool for debugging push notifications. Just pick a cert and device id and it allows you manually send push notifications with a desired payload.

##Dependency Management

###CocoaPods

Use [cocoapods](http://cocoapods.org) for dependency management. To setup `cocoapods` in a new project:

1. Install the `cocoapods` gem if necessary

        $ sudo gem install cocoapods

2. Create a file named _Podfile_ at the root of your project directory

        $ touch Podfile

3. Edit the _Podfile_

        $ open -e Podfile

4. Basic syntax for iOS project

        platform :ios, "8.0"
        
        pod "BBFrameworks"

5. Install the pods

        $ pod install

6. Open the created _.xcworkspace_ file instead of the _.xcodeproj_ file

Whenever you add or edit existing dependencies listed in the _Podfile_ update the pods:

    $ pod update

You should ensure the _Podfile_, _Podfile.lock_ and _Pods_ directory are checked into revision control.

####Pods

These pods have been vetted and their use is encouraged:

- [BBFrameworks](https://github.com/BionBilateral/BBFrameworks), provides separate subspecs covering a variety of use cases
- [AFNetworking](https://github.com/AFNetworking/AFNetworking), wraps `NSURLSession` and related classes
- [SDWebImage](https://github.com/rs/SDWebImage), library providing on disk and in memory caching of downloaded images
    - [BBFrameworks/BBThumbnail](https://github.com/BionBilateral/BBFrameworks) will also handle this and was modeled after the former
- [CocoaAsyncSocket](https://github.com/robbiehanson/CocoaAsyncSocket), library wrapping socket facilities provided in the `CFNetwork` framework
- [SocketRocket](https://github.com/square/SocketRocket), a WebSocket library
- [SVProgressHUD](https://github.com/samvermette/SVProgressHUD), a modal progress HUD
- [DACircularProgress](https://github.com/danielamitay/DACircularProgress), a circular progress indicator with an interface similar to `UIProgressView`
- [SVPullToRefresh](https://github.com/samvermette/SVPullToRefresh), provides infinite scrolling on `UIScrollView` to display a activity indicator when the user scrolls to the bottom of a scroll view and trigger loading additional content
- [ECSlidingViewController](https://github.com/edgecase/ECSlidingViewController), to implement the "Hamburger menu" pattern
- [ReactiveCocoa](https://github.com/ReactiveCocoa/ReactiveCocoa), one of, if not the most powerful third party framework available; provides functional reactive facilities to iOS/OSX
- [ReactiveViewModel](https://github.com/ReactiveCocoa/ReactiveViewModel), implements the View Model portion of the MVVM design pattern, built on top of `ReactiveCocoa`
- [AFNetworking-RACExtensions](https://github.com/CodaFi/AFNetworking-RACExtensions), provides extensions to the `AFNetworking` library built on top of `ReactiveCocoa`
- [SSZipArchive](https://github.com/soffes/ssziparchive), provides methods to interact with zip archives
- [SSKeychain](https://github.com/soffes/sskeychain), provides methods to interact with the device keychain
- [FormatterKit](https://github.com/mattt/FormatterKit), provides numerous `NSFormatter` subclasses to format units, time, and dates
- [CHCSVParser](https://github.com/davedelong/CHCSVParser), provides CSV parsing methods
- [TransformerKit](https://github.com/mattt/TransformerKit), provides numerous `NSValueTransformer` subclasses to transform strings
- [GPUImage](https://github.com/BradLarson/GPUImage), provides a large number of GPU accelerated image and video filters
- [SlackTextViewController](https://github.com/slackhq/SlackTextViewController), provides an expanding text view similar to the Messages application
- [pop](https://github.com/facebook/pop), provides methods to build realistic physics based animations; used extensively in the Facebook Paper application
- [CocoaLumberjack](https://github.com/CocoaLumberjack/CocoaLumberjack), the most comprehensive logging framework ever
- [CocoaHTTPServer](https://github.com/robbiehanson/CocoaHTTPServer), provide an embeddable HTTP server
- [SVGKit](https://github.com/SVGKit/SVGKit), provides methods to interact with SVG resources on iOS
- [CCHMapClusterController](https://github.com/choefele/CCHMapClusterController), provides map clustering behavior for the `MapKit` framework
- [GRMustache](https://github.com/groue/GRMustache), an iOS/OSX implementation of [Mustache templates](http://mustache.github.io/)
- [STTwitter](https://github.com/nst/STTwitter), a Twitter 1.1 API wrapper
- [XMLDictionary](https://github.com/nicklockwood/XMLDictionary), provides methods to transform a XML response into an NSDictionary
- [LiveFrost](https://github.com/radi/LiveFrost), performant real time blurring with support for iOS 6+ (more customizable than `UIVisualEffectView`)