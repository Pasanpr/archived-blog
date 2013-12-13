public: yes
tags: [ios, memory, concept]
summary: |
    Basic memory management and the difference between strong and weak property attributes

=======================================================
iOS Memory Management: Automatic Reference Counting
=======================================================

In our previous `post <http://www.pasanpremaratne.com/2013/09/01/ios-memory-management-p1/>`__ we went over the basics of memory management and the manual-retain-release model that was used before the introduction of ARC. With the introduction of ARC, most of the practices under the manual-retain-release model were automated, handing off the task of memory management to the compiler. ARC automatically implements memory management of objects and blocks in Objective-C without the programmer having to explicitly insert retains and releases like before. It does this by introducing new declared property attributes.


Strong
-------
The first property attribute we're going to look at is `strong`. Strong is a synonymous with retain under the manual-retain-release model, so the following declarations are identical:

.. sourcecode:: objective-c

    @property (retain) Person *aPerson;
    @property (strong) Person *aPerson;

Under ARC, strong is the default for all types unless explicitly specified otherwise. In a strong relationship, like with retain, one object assumes ownership of another and it can share this ownership with yet another object. If in ``FirstViewController.m``, I had the following property:

.. sourcecode:: objective-c

    @property (nonatomic, strong) NSMutableArray *items;

I can pass this items array to a ``SecondViewController`` which assumes shared ownership of the object. With the usage of strong, ARC automatically increases and decreases the reference count when ownership is shared or relinquished so that memory is properly allocated.

Weak
-------

In contrast to strong we have the weak property attribute. While strong relationships are owning relationships, i.e., an object cannot be deallocated until all of its references are released, with weak relationships, the source object does not retain the object to which it has a reference. This weak reference to an object helps avoid what are known as **strong reference cycles** (previously called retain cycles). Let's take a brief segue to talk about reference cycles and why they are important.

.. image:: /static/images/reference_cycle_1.png
   :align: center
   :alt: Strong reference cycle

In this example, we have a ViewController, the parent, and its child, a TableViewController. If the relationship between the two were both strong, then neither one could be deallocated because it is always owned by the other. To solve this problem, we substitute one of the strong references for a weak reference. This weak reference does not imply ownership, since it's a non-owning relationship, and therefore doesn't keep the object alive. 

.. image:: /static/images/reference_cycle_2.png
   :align: center
   :alt: No strong reference cycle

This is also how parent-delegate patterns work as well. The parent view has a weak relationship to its delegate:

.. sourcecode:: objective-c

    UITableView
    @property (weak) id delegate;

While the delegate object has a strong one

.. sourcecode:: objective-c

    Delegate object
    @property (strong) UITableView *tableView;

.. figure:: /static/images/relationships.png
   :align: center
   :alt: Strong and weak relationships in a delegate pattern

   Diagram from `Apple Docs <https://developer.apple.com/library/ios/documentation/cocoa/conceptual/ProgrammingWithObjectiveC/EncapsulatingData/EncapsulatingData.html>`_

This means that once the delegate object is deallocated, it releases the strong reference on ``UITableView``. 

So there you have it. Instead of having to remember to manually increase and decrease the reference count of objects, ARC simplifies the process with the use of the strong and weak property attributes. Why do we do all this again? For the sake of proper memory management. Incorrect memory usage can result in two types of problems:

- Freeing or overwriting data that is still in use. This can corrupt your memory, crash your application and even corrupt your data.
- Memory leaks. A memory leak occurs when memory is not freed up even though it is no longer in use. An application that has memory leaks uses ever increasing amounts of memory, which can result in slow performance and the app being terminated.

Even though it's a lot simpler to program with ARC watching over you, you still have to remember to use the right property attributes - ARC does not look out for strong reference cycles. As the docs say, "judicious use of weak relationships will help to ensure you don't create cycles".

Happy coding!

