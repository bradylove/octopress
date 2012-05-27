---
layout: post
title: "Create Gradients With Ease In Ruby Motion"
date: 2012-05-27 13:01
comments: true
categories: [Ruby, Ruby Motion, iOS]
---
Lately I have been messing around with Ruby Motion and have been learning alot about the way things are done in iOS. Along the way there have been many things I wanted to do but had no clue how (Having very little expirience with Objective-C). One of the things I wanted to figure out how to do was to create gradients programatically so I would not have to images. After about 3 hours of trying to figure it out and failing miserably I gave up and used images anyway. But the next day it suddenly hit me and I was able to figure it out and it was alot easier then what I was trying to do the night before. So I figured someone else might find this useful.

In the first part of this example I am going to show you how add a gradient background on a UIView. So first thing we need to do is create our Ruby Motion project and setup our view controller.

```
motion create GradientExample
cd GradientExample
```

Then open up the Rakefile created by the generator and add the QuartzCore framework.

```ruby Rakefile
$:.unshift("/Library/RubyMotion/lib")
require 'motion/project'

Motion::Project::App.setup do |app|
  # Use `rake config' to see complete project settings.
  app.name = 'EasyGradients'
  app.frameworks << "QuartzCore"
end
```

Next we will create our UIWindow and add a view controller to it. So open up your app_delegate.rb and make it look like this.

```ruby app/app_delegate.rb
class AppDelegate
  def application(application, didFinishLaunchingWithOptions:launchOptions)
    @window = UIWindow.alloc.initWithFrame(UIScreen.mainScreen.bounds)
    @window.rootViewController = GradientViewController.new
    @window.rootViewController.wantsFullScreenLayout = true
    @window.makeKeyAndVisible

    true
  end
end
```

Then create the file for our GradientViewController.

```sh
touch app/gradient_view_controller.rb
```

Open up the gradient_view_controller.rb. This is were we are going to work our gradient magic. 

```ruby app/gradient_view_controller.rb
class GradientViewController < UIViewController
  def viewDidLoad
    gradient = CAGradientLayer.layer
    gradient.frame = view.bounds
    gradient.colors = [UIColor.blackColor.CGColor, UIColor.whiteColor.CGColor]

    view.layer.addSublayer(gradient)
  end
end
```

Then back in your console you can go ahead and run the `rake` command to launch the app in the iOS Simulator and should have something like this.

![image](https://img.skitch.com/20120527-gk3926patwmn7b9q42ebjs7ts8.jpg)

If may notice that gradient.colors just accepts an array so we can easily add more colors to the array.
```ruby app/gradient_view_controller.rb
gradient.colors = [UIColor.blackColor.CGColor, UIColor.redColor.CGColor, 
                   UIColor.whiteColor.CGColor, UIColor.blueColor.CGColor]
```
And you will end up with this

![image](https://img.skitch.com/20120527-dwuxbbhnyrewgne2m7wpk7sdmq.jpg)
For more control over the locations of the gradients you can add an array to gradient locations with a Float making their location. 0 being top 1 being bottom.
```ruby app/gradient_view_controller.rb
gradient.locations = [0.1, 0.2, 0.8, 0.9]
```
Which will end up someting like this 

![](https://img.skitch.com/20120527-p3si97dy86x9bskaxsad8eea8s.jpg)

Well thats about it, you can easily use these gradients for backgrounds for almost all UI objects. Hope this helps someone that is as lost as me. There are some more iOS/Ruby Motion tricks I have been learning so check back soon on write ups for them.
