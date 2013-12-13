public: yes
tags: [ios, memory, concept]
summary: 

===========================================================
iOS Memory Management: The Manual-Retain-Release Model
===========================================================

**Edit:** This post was rewritten to be a bit more clear and was originally posted on the `Treehouse Blog <http://blog.teamtreehouse.com/ios-memory-management-part-1>`__. 

_____________________________________________________________________________________________________________________________________

Being a Business teacher at Treehouse, where our mission is to teach people to code, it’s lame to be someone who doesn’t know how to actually code. About a year ago, I decided it was about time and set about to teach myself. I wanted to use Treehouse videos to do this because not only was it a great resource I already had, but I also got to test our curriculum and provide feedback to the relevant teachers if I found any holes. I picked iOS development because I’m pretty much an Apple fanboy and because I’ve always wanted to make apps. In this first post, I’m going to outline some of the stuff I’ve learned so far. Be gentle, the training wheels are still on. :)

When I tried my hand at learning iOS development a few years ago, having no prior programming experience whatsoever, one of my biggest mental blocks was in regards to iOS memory management. Retaining, releasing and deallocating objects made no sense to me and I soon gave up.

Since then ARC, or Automatic Reference Counting, has been introduced and has made the job much easier. But learning with ARC has been somewhat of a black box. Why am I assigning an object a property of strong? What’s the difference between atomic and nonatomic? Before we go into what ARC is and how it made learning easier, let’s understand why we need it in the first place.

Memory Management
---------------------

Devices have a limited amount of memory and in order for apps to function efficiently, we need to distribute ownership of this limited memory among our app data and code. The concept of memory management is “the process of allocating memory during your program’s runtime, using it, and freeing it when you are done with it”. There are two ways you can accomplish this:

- The Manual-Retain-Release method, where you explicitly manage memory by keeping track of the objects you own using a method called reference counting
- Using ARC, or Automatic Reference Counting, which still uses reference counting, but does it automatically by inserting the appropriate memory management method calls at compile-time

In this post, the first of several posts, we are going to take a look at the Manual-Retain-Release model and how memory used to be managed so that we have an understanding of what ARC is doing in the background.

Manual-Retain-Release
------------------------

Before the introduction of ARC, everyone followed a ‘manual-retain-release’, or MRR, model of memory management. The basic rules are as follows:

- You own any object you create
- You can take ownership of an object using retain
- When you no longer need it, you must relinquish ownership of an object you own.
- You must not relinquish ownership of an object you do not own

Under these rules, if an object is created (using a method whose name beings with ``alloc``, ``new``, ``copy``, or ``mutableCopy``), it must be relinquished when it is no longer needed. To do this you send the object a release or autorelease message. Let’s look at an example:

.. sourcecode:: objective-c

    {
       Car *aCar = [[Car alloc] init];
       // ...
       NSString *model = aCar.model;
       // ...
       [aCar release]
    }


Once we’re done with the Car object, we relinquish ownership by sending a release or autorelease message. As such, relinquishing ownership of an object is typically referred to as *releasing* the object. In our example, we don’t take ownership of the string pointing to the Car model, so we don’t bother with sending it a release message.

Let’s look at another example of a method.

.. sourcecode:: objective-c

    - (NSString *)model
      {
        NSString *string = [[[NSString alloc] initWithFormat:@"%@ %@", self.carMaker, self.modelName] autorelease];
        return string;
      }

Since we use the ``alloc`` method to create the object, according to Rule #1, we own the string returned by alloc. It is our responsibility now, to relinquish ownership of this object before we lose reference to it. In this case, if were to relinquish it using ``release``, the string will be deallocated before it is returned and the method would return an invalid object.

Finally, you can relinquish ownership of your objects by implementing the ``dealloc`` method. The NSObject class defines a dealloc method that is implemented automatically when an object has no owners. From the docs: “The role of the dealloc method is to free the object’s own memory, and to dispose of any resources it holds, including ownership of any object instance variables.”

In the following example, we create the instance variables ``carMaker`` and ``modelName``. To dispose of them when we are done and free up memory, we release them in our dealloc method.

.. sourcecode:: objective-c

    @interface Car : NSObject

    @property (retain) NSString *carMaker;
    @property (retain) NSString *modelName;

    @end

    @implementation Person
    // ...
    - (void)dealloc
      {
        [_carMaker release];
        [_modelName release];
        [super dealloc];
      }
    @end


Using MRR techniques
----------------------

If any of your classes have properties that are objects, you have to make sure that when an object is set its value it is not deallocated while in use. To do this you have to claim ownership of the object and relinquish or release it when you are done with it. Take the following code:

.. sourcecode:: objective-c

    @property (nonatomic, retain) NSString *string;

Properties declare two accessor methods – the setter and getter. This is done automatically for you, but let’s look under the hood so we can understand what’s going on.

In the ‘get’ accessor or getter, all we’re doing is returning the synthesized instance variable. We don’t create a new object using ``alloc``, ``new``, ``copy``, or ``mutableCopy`` so we don’t have to retain or release it.

.. sourcecode:: objective-c

    - (NSString *)string {
      return _string;
    }

In the ‘set’ method or setter, the value we assign could be disposed of at any time, so we have to take ownership of the object to make sure this won’t happen.

.. sourcecode:: objective-c

    - (void)setString:(NSString *)newString 
    {
      [newString retain];
      [_string release];
      // Make the new assignment.
      _string = newString;
    }

We take ownership of the newString object using retain, relinquish ownership of the old string using release and then make the new assignment.

We can use these accessor methods and our MRR rules to make sure we manage our memory properly. A final example:

.. sourcecode:: objective-c

    - (void)change 
    {
      NSString *anotherString = [[NSString alloc] initWithString:@"some string"];
      [self setString:anotherString];
      [anotherString release];
    }

We create anotherString with alloc so we balance it at the end with release. We also use our setter accessor method to set the value of our string property. Within our setter methods we take ownership of any relevant objects and release it when we don’t need, making sure we manage our memory effectively.

Ownership Policy
------------------
Using retain and release to manage our memory uses a model called reference counting. Each object has a retain count and this is used to keep track of ownership.

- When you create an object, it has a retain count of 1.
- When you send an object a retain message, its retain count is incremented by 1.
- When you send an object a release message, its retain count is decremented by 1.
- When you send an object a autorelease message, its retain count is decremented by 1 at the end of the current autorelease pool block.

Once an object’s retain count is reduced to zero, it is deallocated.

That should wrap up the basics of memory management using the manual-retain-release model. An understanding of this forms the foundation for building upon ARC, which we’ll tackle in our next post in the series.