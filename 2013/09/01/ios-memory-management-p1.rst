public: yes
tags: [ios, memory, concept]
summary: |
    Basic memory management and the difference between strong and weak property attributes

=========================================
iOS Memory Management I - Strong vs Weak
=========================================

Although I'm quite behind the curve in terms of jumping into iOS development, this seems to be as good a time as ever. iOS 7 changes things up quite a bit and it feels like the playing field is leveling for new entrants like me, even if only by a tiny bit. Also, I'm learning a lot more now than I did a few years ago, and I feel like it's mostly thanks to Automatic Reference Counting, or ARC. When I tried my hand at learning iOS development a few years ago, having no prior programming experience whatsoever, I found it really hard to wrap my mind around memory management and eventually gave up. This time around, not having to worry about retaining, releasing, and deallocating objects made it easier to understand the fundamentals. But once I got the basics down, I started to wonder what ARC was doing in the background. Why am I  assigning an object a property of strong? What's the difference between atomic and nonatomic? This series of posts is an attempt to understand these concepts better, starting with strong versus weak.

Before ARC
-------------

In order for me to understand why I was assigning a strong attribute to a property and what that meant, I had to look at how things were done before ARC was introduced. Pre-ARC, everyone followed a 'manual-retain-release', or MRR, model of memory management. The basic rules are as follows:

- You own any object you create
- You can take ownership of an object using retain
- When you no longer need it, you must relinquish ownership of an object you own
- You must not relinquish ownership of an object you do not own

Under these rules, if an object is created (using a method whose name beings with "alloc", "new", "copy", or "mutableCopy"), it must be relinquished when it is no longer needed. To do this you send the object a ``release`` or ``autorelease`` message. Let's look at an example:

.. sourcecode:: Objective-C

    {
        Car *aCar = [[Car alloc] init];
        // ...
        NSString *model = aCar.model;
        // ...
        [aCar release]
    }

We create the Car object using the ``alloc`` method, so to relinquish ownership once we're done with it, we send it a ``release`` message. We don't take ownership of the string pointing to the Car model, so we don't bother with sending it a release message.

Object Properties
------------------

If any of your classes have properties that are objects, you have to make sure that any object that is set as the value is not deallocated when using it. To prevent this, you have to take ownership of the object when it is set, and relinquish ownership of any currently held values of said object. Easier explained in code. Here, we're assigning an attribute of retain to the property.

.. sourcecode:: Objective-C

    @interface Car: NSObject

    @property (nonatomic, retain) NSString *model;

    @end;

To understand what ``retain`` does as an attribute, let's look at the property's getter and setter methods.

.. sourcecode:: Objective-C

    - (NSString *)string {
        return _string;
    }

    - (void)setString:(NSString *)newString{
        [newString retain];
        [_string release];
        _string = newString;
    }

In the getter method, we're just returning the instance variable, so there's no need to retain or release. In the setter method, since we want the new string to persist after the method call, we have to take ownership of the object by sending it a retain message. Simultaneously, we relinquish ownership of the old string by sending it the release message. 

Now, to further complicate things, objects can have many owners. As objects get passed around, you can have multiple references to the same object. To keep track of all this, you take care to *reference count* (or retain count) - using the retain method. This is how it works:

- When you create an object, it has a retain count of 1.
- When you send an object a retain message, its retain count is incremented by 1.
- When you send an object a release message, its retain count is decremented by 1.
- Once an object's retain count is reduced to zero, it is deallocated.

As long as your retains and releases are balanced, you'll be fine. 

Why do we do all this? For the sake of proper memory management. Incorrect memory usage can result in two types of problems:

- Freeing or overwriting data that is still in use. This can corrupt your memory, crash your application and even corrupt your data.
- Memory leaks. A memory leak occurs when memory is not freed up even though it is no longer in use. An application that has memory leaks uses ever increasing amounts of memory, which can result in slow performance and the app being terminated.

Strong
-------
Automatic reference counting, or ARC, introduced in iOS 5, takes care of all of this for you. So instead of having to remember to use retain, release and autorelease, ARC evaluates the lifetime of your objects and inserts the appropriate memory management calls for you. To do this, ARC introduced ``strong`` and ``weak`` as new declared property attributes. Strong is a synonym for retain, so the following declarations are identical:

.. sourcecode:: Objective-C

    @property (retain) Person *aPerson;
    @property (strong) Person *aPerson;

Under ARC, strong is the default for all types unless explicitly specified otherwise. In a strong relationship, like with retain, one object assumes ownership of another and it can share this ownership with yet another object. If in ``FirstViewController.m``, I had the following property:

.. sourcecode:: Objective-C

    @property (nonatomic, strong) NSMutableArray *items;

I can pass this items array to a ``SecondViewController`` which assumes shared ownership of the object. Retaining and releasing are done in the background by ARC.

Weak
-----
In contrast to strong, we have weak references. In a strong relationship, an object cannot be deallocated until all of its references are released. With weak references, the source object does not retain the object to which it has a reference, i.e., it is a non-owning relationship. Under the MRR model, this was achieved by assigning a property attribute of ``assign``. For the most part, ``assign`` and ``weak`` achieve the same objective, except that on releasing, weak sets the pointer to ``nil`` whereas assign does not. This prevents the app from crashing.

This pairing for strong and weak helps avoid what is known as a strong reference cycle (previously called retain cycle). Let's look at the following illustration to understand what that means.

.. image:: /static/images/reference_cycle_1.png
   :align: center
   :alt: Strong reference cycle

In this example, we have a ViewController, the parent, and its child, a TableViewController. If the relationship between the two were both strong, then neither one could be deallocated because it is always owned by the other. To solve this problem, we substitute one of the strong references for a weak reference. This weak reference does not imply ownership and therefore doesn't keep the object alive. 

.. image:: /static/images/Reference_Cycle_2.png
   :align: center
   :alt: No strong reference cycle

As illustrated above, this is how a parent-delegate pattern works as well. The parent view has a weak relationship to its delegate:

.. sourcecode:: Objective-C

    UITableView
    @property (weak) id delegate;

While the delegate object has a strong one

.. sourcecode:: Objective-C

    Delegate object
    @property (strong) UITableView *tableView;

.. figure:: /static/images/relationships.png
   :align: center
   :alt: Strong and weak relationships in a delegate pattern

   Diagram from `Apple Docs <https://developer.apple.com/library/ios/documentation/cocoa/conceptual/ProgrammingWithObjectiveC/EncapsulatingData/EncapsulatingData.html>`_

This means that once the delegate object is deallocated, it releases the strong reference on ``NSTableView``. 

That should cover the basics of memory management using strong and weak property attributes. Next, I'm going to try and tackle atomic and nonatomic.