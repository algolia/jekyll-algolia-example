---
layout: post
title: 'iOS: When Automatic Reference Counting is a Bad Idea'
author:
  login: julien
  email: julien.lemoine@algolia.com
  display_name: julien
  first_name: Julien
  last_name: Lemoine
---

I started developing for iOS in 2009 by learning about the Objective-C
language. At that time ARC (Automatic Reference Counting) was not yet
available and developers were responsible for alloc/release/autorelease. I
found it pretty straightforward as it was very similar to C++ and the
resulting code was very self-explanatory.

When ARC was released at the end of 2011 it made a great impression on me and
looked like the perfect feature for any programmer coming from the Java world
who was not familiar with memory management. I started using ARC and
discovered a major flaw that completely changed my mind. Here is an example :

[Edit 28-Jan-2013] This post describes a bug in ARC that was fixed in Xcode
4.4.[/Edit]

Let's start with an example in C++, the sample contains a constructor that
allocates two vectors, a destructor that destroys them and a compute method
that just performs a swap between the two vectors. This code is perfectly
valid and a swap is the perfect operation when you have two sets of data to
maintain because this is very efficient (just two pointers copy, no data
copy).

    
    class Example {
    public:
      Example() {
        nextPositions = new std::vector<int>();
        currentPositions = new std::vector<int>();
      }
      ~Example() {
        delete nextPositions;
        delete currentPositions;
      }
    
      void compute() {
        // some code here
        std::swap(nextPositions, currentPositions);
        // some other code here
      }
    
    private:
      std::vector<int>* nextPositions;
      std::vector<int>* currentPositions;
    };

Before ARC the objective-C code was very similar and perfectly valid:

    
    // headers
    @interface Example : NSObject {
      NSMutableArray* nextPositions;
      NSMutableArray* currentPositions;
    }
    -(void) compute;
    @end
    
    // implementation
    @implementation Example
    -(id) init {
      self = [super init];
      if (!self)
        return nil;
      nextPositions = [[NSMutableArray alloc] init];
      currentPositions = [[NSMutableArray alloc] init];
      return self;
    }
    
    -(void) dealloc {
      [super dealloc];
      [nextPositions release];
      [currentPositions release];
    }
    
    -(void) compute
    {
      // some processing stuff here
      NSMutableArray* tmp = nextPositions;
      nextPositions = currentPositions;
      currentPositions = tmp;
      // some processing stuff here
    }
    @end

The semantics are very clear, it is exactly the same as in C++.

So when you start using ARC you tend to do exactly the same thing with less
code, and you just delete the dealloc method:

    
    // headers
    @interface Example : NSObject {
      NSMutableArray* nextPositions;
      NSMutableArray* currentPositions;
    }
    -(void) compute;
    @end
    
    // implementation
    @implementation Example
    -(id) init {
      self = [super init];
      if (!self)
        return nil;
      nextPositions = [[NSMutableArray alloc] init];
      currentPositions = [[NSMutableArray alloc] init];
      return self;
    }
    
    -(void) compute {
      // some processing stuff here
      NSMutableArray* tmp = nextPositions;
      nextPositions = currentPositions;
      // Wrong, at that step this is too late! nextPositions was deallocated
      currentPositions = tmp;
      // some processing stuff here
    }
    @end

The error does not come from a missing strong attribute, because instance
variables are strong by default. The problem is that ARC does not generate
code in the variable assignation. You should explicitely add properties and
use "self.variableName" notation like in the next example. I would encourage
ARC designers to read this excellent article by Joel Spolsky ["Making Wrong
Code Look Wrong"][1]. This ARC
usage is a perfect example of wrong code that looks perfect but leads me to
the conclusion that ARC is not trustworthy.

Here is the correct version:

    
    // headers
    @interface Example : NSObject {
      NSMutableArray* nextPositions;
      NSMutableArray* currentPositions;
    }
    -(void) compute;
    @property (strong, nonatomic) NSMutableArray* nextPositions;
    @property (strong, nonatomic) NSMutableArray* currentPositions;
    @end
    
    // implementation
    @implementation Example
    -(id) init {
      self = [super init];
      if (!self)
        return nil;
      nextPositions = [[NSMutableArray alloc] init];
      currentPositions = [[NSMutableArray alloc] init];
      return self;
    }
    
    -(void) compute {
      // some processing stuff here
      NSMutableArray* tmp = self.nextPositions;
      self.nextPositions = self.currentPositions;
      self.currentPositions = tmp;
      // some processing stuff here
    }
    @synthesize nextPositions;
    @synthesize currentPositions;
    @end

I am surprised Apple hasn't corrected that flaw yet. This is a major issue as
ARC does not generate code for variable affectation like it does for
properties (if one of you reads this post and has a complete explanation,
please tell me!):

  * Is it because of a performance issue? I would prefer to have no ARC at all than to see such a behavior.
  * Is this case too complex to be handled? The problem is it undermines ARC's utility and might get stuck at the prototype stage.

To me, this is a perfect example of a technology driven product. The engineers
know their product so well that they forget to step back and look at it with a
fresh eye to analyse the full complexity of their system.

And that leaves me with two important lessons we'll try to apply to our
products:

  * Always organize user tests, even when your users are developers themselves!
  * Always keep a fresh eye when trying to simplify a product (this one may prove particularly difficult!)

I hope you'll tell us when we will (inevitably) break these rules!


[1]: http://www.joelonsoftware.com/articles/Wrong.html
