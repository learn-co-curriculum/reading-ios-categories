# Categories

Categories allow us to add methods to an existing class without modifying its original implementation file. This is also called "extension". 

**Note:** *Categories can only extend a class via methods. Properties cannot be added to a class in a category file. If your problem is best solved by adding a property, you must create a subclass instead.*

Categories are primarily useful for adding functionality to a class that the current project does not own. This can be a class belonging to an Apple framework (such as `NSString`), a third-party framework (such as a custom matcher for Expecta), or a dynamically generated class owned by Core Data.

### Example: String Elision

As an example, let's take a situation in which we would need to shorten a string, perhaps in order to fit it into a label. We can truncate the string to a certain length and then append an ellipsis character (`…`) to it. This a process called *elision,* or *to elide*. Since this is functionality that we wish to add to the foundation class `NSString`, it is a good candidate for writing a category.

To create a category file in Objective-C, create a new Cocoa Touch Class and give it the name of the base class plus (`+`) a name for the extension you are writing. In our case we're going call this category `FISElide`, so the file names will use `NSSting+FISElide`.

In the `@interface` section of the category's `.h` header file, write the name of the base class (`NSString` in our case) followed by the name of the extension in parentheses:

```objc
//  NSString+FISElide.h

#import <Foundation/Foundation.h>

@interface NSString (FISElide)

@end
```

Now, let's declare a method that we're going to add to `NSString`. Let's call it `fis_elideStringToLength:` and pass in an `NSUInteger` argument called `length`. This `length` argument will allow the user to determine how long the returned string should be including the ellipsis: 

```objc
//  NSString+FISElide.h

#import <Foundation/Foundation.h>

@interface NSString (FISElide)

- (NSString *)fis_elideStringToLength:(NSUInteger)length;

@end
```

In our implementation, we should return the whole string if the `length` argument is greater than or equal to the string's length. However, if the string is longer than the maximum length, we should shorten the string to one character less than the maximum length (to make room for the ellipsis character), and then append the ellipsis character:

```objc
//  NSString+FISElide.m

#import "NSString+FISElide.h"

@implementation NSString (FISElide)

- (NSString *)fis_elideStringToLength:(NSUInteger)length {
    if (length >= self.length) {
        return self;
    }
    
    NSUInteger index = length - 1;
    NSString *truncated = [self substringToIndex:index];
    NSString *elision = [truncated stringByAppendingString:@"…"];
    
    return elision;
}

@end
```

### Calling a Category Method

The category files are then just like any other pair of class files, with the exception that they are depend on the base class in order to work. We can import our category file by using its name, which then makes the category methods within it available to the class. Using our category method within `FISAppDelegate`'s `application:didFinishLaunchingWithOptions:` method might look like this:

```objc
// FISAppDelegate.m

#import "FISAppDelegate.h"
#import "NSString+FISElide.h"  // import the category's header

@implementation

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {

    NSString *brightStar = @"Bright star, would I were stedfast as thou art";

    NSString *elision = [brightStar fis_elideStringAtIndex:20];

    NSLog(@"%@", elision);
    
    return YES;
}

@end
```
This will print: `Bright star, would …`.