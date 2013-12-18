public: yes
tags: [ios]
summary: |
    Using CocoaPods to manage dependencies in an Xcode project

=======================================================
Managing Dependencies in Objective-C Projects
=======================================================

When working with Objective-C projects in Xcode, you will occasionally want to work with third party libraries. This can be a bit cumbersome as Xcode doesn't have a clean solution. You can either link a static library, which involves setting up the header paths to point to the correct location, or you can download the source files from the library and include it in your project. Neither of these options are great, especially if you want to keep your libraries up to date. 

`CocoaPods <http://beta.cocoapods.org/>`__ is an Objective-C dependency manager and offers a great solution to this problem. If you are familiar with Ruby, this works much like a Ruby gem (I'm not familiar with Ruby and don't have a clue what that means). It is built with Ruby and can be installed using the default Ruby on OS X. To understand how CocoaPods simplifies the process of including third party libraries, we're going to create an Xcode project and add a few different libraries to it. Let's start by installing CocoaPods:

.. sourcecode:: bash
    
    $ sudo gem install cocoapods

When using the default Ruby install you have to use ``sudo`` when installing a gem (what the heck is `sudo <http://en.wikipedia.org/wiki/Sudo>`__?). If I understand correctly, you don't have to use ``sudo`` when using the Ruby Version Manager or RVM, but that's outside the scope of this post. You now have CocoaPods installed on your system.

To search for libraries you can run:

.. sourcecode:: bash 

    $ pod list

That shows you all libraries and that probably doesn't help you very much, since there are, at the writing of this post, 3136 pods. You can also be more specific and search for pods. Running the following command searches for pods with 'cup' in the name and returns a pod called CupertinoYankee:

.. sourcecode:: bash

    $ pod search cup

    -> CupertinoYankee (1.0.0)
       An NSDate Category With Locale-Aware Calculations for Beginning & End of Day, Week, Month, and Year.
       pod 'CupertinoYankee', '~> 1.0.0'
       - Homepage: https://github.com/mattt/CupertinoYankee
       - Source:   https://github.com/mattt/CupertinoYankee.git
       - Versions: 1.0.0, 0.1.1, 0.1.0 [master repo]



Adding Libraries to Xcode Projects
------------------------------------

Let's create a simple Xcode project. Everything looks the same, we have our basic files.

.. image:: /static/images/sample-project-structure.png
    :align: center
    :alt: Sample project structure

I want to add the following libraries to the project: `CupertinoYankee <https://github.com/mattt/CupertinoYankee>`__, `GroundControl <https://github.com/mattt/GroundControl>`__ and `AFNetworking <https://github.com/mattt/AFNetworking>`__. To do this, in the main Xcode project directory, create a file called ``Podfile`` and add the following code:

.. sourcecode:: ruby

    platform :ios, '7.0'
    pod 'CupertinoYankee', '~> 1.0.0'
    pod 'GroundControl', '~> 2.0.0'
    pod 'AFNetworking', '~> 2.0.3'

A Podfile is similar to a Gemfile. All we're doing is specifying the platform and listing each dependency along with the version required. If you don't know how to list the dependency, on the `CocoaPods website <http://beta.cocoapods.org/>`__, just start typing the name of the library. When you find the library you are looking for, click the clipboard icon to copy the necessary text. Then it's as easy as running:

.. sourcecode:: bash
    
    $ pod install

When you run this command CocoaPods creates a workspace, adds our current project to the workspace and adds all the pods we specified into a Pods project which it then statically links. Once the command executes it reminds you to use the workspace, instead of opening up our project directly.

.. sourcecode:: bash    

    [!] From now on use `SampleProject.xcworkspace`.

If we go ahead and open up our workspace, we can see the Pods project that was generated, along with the libraries and relevant source files.

.. image:: /static/images/pods-project.png
    :align: center
    :alt: Pods project

In our project directory, you will notice a new ``Pods.xcconfig`` file that specifies the relevant header search paths automatically. 

.. image:: /static/images/header-search-paths.png
    :align: center
    :alt: Header search paths

So let's go ahead and use these libraries in our project. Navigate to the ``SampleViewController.m`` file and add import the following headers:

.. sourcecode:: objective-c

    #import "AFNetworking.h"
    #import "NSDate+CupertinoYankee.h"
    #import "NSUserDefaults+GroundControl.h"

And that's all you need to do! An easy and clean way to manage any dependencies in your Objective-C projects.