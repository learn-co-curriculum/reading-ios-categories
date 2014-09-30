---
tags: categories, objects
languages: objc
---

#Categories

###Categories allow us to add methods to an existing class without modifying its original implementation file. 

This is useful to us in a few ways:

1. It lets us separate the behaviors of a class into multiple files for
   organization.
2. By separating behaviors into multiple files, we can choose who gets to see
   which methods.
3. We can also add functionality to classes we have no control over, like
   Apple's standard libraries.

Since the first 2 benefits of categories can be somewhat application specific,
let's use categories to extend the functionality of Apple's NSString class!

Now I think NSString is just swell, but sometimes strings can get in the habit
of getting too long. Let's create a category on NSString that lets us
"ellipsize" a string after a specified number of characters. Ellipsize in this
case will mean to cut off the string after so many characters and replace the
rest with "...".

#####So this is what our category's header would look like:

```objc
//  NSString+Ellipsize.h

#import <Foundation/Foundation.h>

@interface NSString (Ellipsize)

-(NSString *)ellipsizeAfter:(NSUInteger)numCharacters;

@end
```

#####And this would be our implementation file:

```objc
//  NSString+Ellipsize.m

#import "NSString+Ellipsize.h"

@implementation NSString (Ellipsize)

-(NSString *)ellipsizeAfter:(NSUInteger)numCharacters
{
    if (numCharacters > self.length) {
        return self;
    }
    
    NSString *modifiedString = [self substringWithRange:NSMakeRange(0, numCharacters)];
    return [modifiedString stringByAppendingString:@"..."];
    
}

@end
```

#####And we can use it like this:

```objc
#import "NSString+Ellipsize.h"

NSLog(@"%@", [@"Hi there!" ellipsizeAfter:2]); // this prings "Hi..."
```
