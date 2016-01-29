#Categories

## Objectives

1. Learn the purpose of categories, and when they make more sense than subclasses.
2. Recognize the limitations and caveats of categories.
3. Identify the anonymous category (AKA the class extension) and its purpose.


## What is a Category?

In most of the code we've seen so far, there has been just one `@interface` and `@implementation` per class. Categories provide a way to spread the implementation and interface of a class across multiple files, and even across time. Most often, **categories are used to add methods to an existing class** without modifying (or even without access to) its original implementation file. Categories can also be used to split the definition of a class into multiple files for organization or to gain more fine-grained control over visibility.

## Syntax

Categories consist of an `@interface`-`@implementation` pair, just like the definition of a class itself, *but* they have one small addition -- a category name in parentheses after the name of the class. Categories are typically declared in files named `<ClassName>+<CategoryName>.{h,m}`. Let's look at an example.

```objc
// NSString+FISUtils.h

#import <Foundation/Foundation.h>


@interface NSString (FISUtils)

-(NSArray *)characters;

@end
```

```objc
// NSString+FISUtils.m

#import "NSString+FISUtils.h"


@implementation NSString (FISUtils)

-(NSArray *)characters
{
    NSMutableArray *characters = [[NSMutableArray alloc] init];
    for(NSUInteger i = 0; i < self.length; i++) {
        NSRange rangeForIthCharacter = NSMakeRange(i, 1);
        NSString *ithCharacter = [self substringWithRange:rangeForIthCharacter];
        [characters addObject:ithCharacter];
    }
    
    return characters;
}

@end
```

We have just written a category, `FISUtils`, on `NSString`. The **name of a category** is not important, but it must be unique and shared between the interface and its corresponding implementation. For that reason, we recommend prefixing your category names.


Some things to note:

1. Even though `NSString` is already declared elsewhere, this is valid code. **Categories do not declare a class, they only expand the contents of an existing one.** So, we are not are re-declaring `NSString` (which would be a compiler error).
2. You refer to the receiver in a category method as `self`, just as you would in a normal method of a class.
3. Just as in a class, to be visible to other files, a method must be in the header of a category and that header must be imported where it is used.


## Use Cases

Let's discuss the uses of categories in turn.

### Dividing the Implementation

Since categories allow us to have multiple files that collectively implement a class, we could write the implementation of our class across various `.h` and `.m` files. This is not a very common use in application code, though some of the larger built-in classes (`NSString`, `NSArray`, `NSDictionary`, and many UI classes) do this.


### Fine-grained Visibility

We could also relegate methods that aren't *exactly* public and not *exactly* private to a particular category. For example, some languages have a "protected" visibility for methods and properties that are visible only to subclasses of a given class. We can emulate this in Objective-C by making a category intended solely for consumption by subclasses. Apple has taken this approach in a few places, most notably with the `UIGestureRecognizerProtected` category in `UIGestureRecognizerSubclass.h`. Again, this is not a very common use of categories, but it is good to be aware of it.

### Adding Behavior

This is the most common use of categories, and the only one we will focus on in this course. By implementing a category on an existing class, we can add functionality to classes we have no control over, like Apple's standard libraries.

You're probably thinking, "Aren't subclasses for adding functionality to existing classes?" They are, but the trouble with subclasses is that you have to use them. That is, if you make an `NSString` subclass, *only* instances of your subclass will have the new behavior you add. The trouble is, no amount of coaxing is going to make string literal syntax (`@""`) return instances of your custom class. The same goes for third-party (and Apple) libraries -- they use straight `NSString` and `NSArray` classes throughout and won't be convinced to do otherwise.

This is the beauty of categories -- they allow us to add functionality to *all* instances of a pre-existing class, even built-in ones like `NSString`. We already wrote the implementation of a `characters` in the code sample above. Let's see what using it would look like. 

```objc
#import "NSString+FISUtils.h"

-(void)testCategory
{
    NSString *state = @"Wyoming";
    NSLog(@"%@", [state characters]);  // Prints the letters in "Wyoming" as an array
}
```

Our new method `characters` is available on all instances of `NSString`!


## The Class Extension

In some new projects or online, you may have seen an `@interface` in the `.m` file of a class. It typically looks like this: `@interface FISPet ()`. This particular category is called **the anonymous category** or **the class extension**.

The class extension is typically used to make private properties. Since it is declared in the `.m`, only the methods in the `.m` will be able to see the properties declared in it.


## Caveats

### Properties

**There is no easy way to add `readwrite` properties in a category.** There are hacky ways. Avoid them. (Note that the class extension is exempt from this restriction; it is *intended* to contain properties.)

Adding `readonly` properties is possible, however, since they are essentially just syntactic sugar for getter methods. In fact, our `characters` method could have been declared in the category header as `@property (nonatomic, readonly) NSArray *characters`.


### Name Collisions

If a method in a category has the same name as a method on the class, the behavior is undefined. That is, there's no way of knowing whether your implementation or the original one will win. **Never declare a method in a category with the same name as one in the class.** To avoid accidental collisions with private methods, some sources (including Apple) recommend prefixing your category method names with a lowercase prefix and an underscore, so that our `characters` method from earlier would in fact be called `fis_characters`.

<p data-visibility='hidden'>View <a href='https://learn.co/lessons/reading-ios-categories' title='Categories'>Categories</a> on Learn.co and start learning to code for free.</p>
