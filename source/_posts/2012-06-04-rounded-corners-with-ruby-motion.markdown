---
layout: post
title: "Rounded Corners With Ruby Motion"
date: 2012-06-04 20:08
comments: true
categories: [Ruby, Ruby Motion, iOS]
---
Another trick I learned with Objective-C/Ruby Motion is giving a view is rounded corners. When I first set out to learn how to do rounded corners with Ruby Motion, I quickly found a one liner calling layer.cornerRadius= on a view, which I thought was awesome! I remember thinking to myself "that was too easy but awesome!". I soon found out I was right, it was too easy. So I set out again to find a better solution and found a similar question on stack overflow for someone trying to accomplish the same thing with Objective-C. I find myself using this alot now so I thought I would share it and hope it helps someone else as well. Before I get started with the details. I have already created a project with "motion create AppName", created a window and initilized the RootViewController in my app_delegate.rb.
And here is the starting code for the RootViewController which just has a single UIView, 200x200 in size, centered and colored green.

``` ruby root_view_controller.rb
class RootViewController < UIViewController
  def viewDidLoad
    view.backgroundColor = UIColor.lightGrayColor

    @rounded_rect = UIView.alloc.initWithFrame([[view.size.width / 2 - 100, view.size.height / 2 - 100], [200, 200]])
    @rounded_rect.backgroundColor = UIColor.greenColor


    view.addSubview @rounded_rect
  end
end
```
So you should have something that looks like this.


![](http://content.screencast.com/users/BLove/folders/Jing/media/955d5e1d-e52a-420d-9a2e-9ac3a8afd81a/00000005.png)


## The Easy Way
Okay so real quick I thought I would show you the easy way and show you the problem I found with it. To quickly achieve this we just need to set the cornerRadius on the UIView's layer. So after setting the backgroundColor for @rounded_rect I am going to add this line.

```ruby root_view_controller.rb
@rounded_rect.layer.cornerRadius = 30.0
```
Awesome! We now have some rounded corners! 


![](http://content.screencast.com/users/BLove/folders/Jing/media/f33fc52a-9c7d-42dd-8f3a-f1df10f283b4/00000006.png)

## The Problem
Okay so this works out great for us if we need to quickly add rounded corners to a view. But the problem comes when we add subviews that are going to be in the area of those rounded corners. To show you this I am going to add another UIView as a subview to our @rounded_rect view. We will call this @non_rounded_subview, I am going to make this subview the same size as its parent and color it red by adding the following.

```ruby root_view_controller.rb
    @non_rounded_subview = UIView.alloc.initWithFrame(@rounded_rect.bounds)
    @non_rounded_subview.backgroundColor = UIColor.redColor

    @rounded_rect.addSubview @non_rounded_subview
```
OH NOES! Where did our rounded corners go?

![](http://content.screencast.com/users/BLove/folders/Jing/media/c74b0c0f-b854-4195-a660-f7919c577bf3/00000007.png)

Our rounded corners are still there, however the parent views rounded corners do not clip the child view. To show this better I will set the backgroundColor of @non_rounded_subview to have a slight transparent to better show this.

```ruby root_view_controller.rb
@non_rounded_subview.backgroundColor = UIColor.colorWithRed(1.0, green: 0.0, blue: 0.0, alpha: 0.5)
```

![](http://content.screencast.com/users/BLove/folders/Jing/media/1e3ffeab-3e07-42aa-81c1-0670b3bb88f5/00000008.png)

## The Solution
So the easy way works great if you just need some simple rounded corners and arent going to be placing anything over them. However in my case I needed the rounded corners of the parent view to clip the child view. The solution I came up with ended up being even greater then I expected, not only cliping child views but giving me alot more control over the corners. To accomplish this I used UIBezierPath and CAShapeLayer to create a layer mask and then applied the layer mask to the parent view (@rounded_rect). To do this I removed the following line

```ruby root_view_controller.rb
@rounded_rect.layer.cornerRadius = 30.0
```

and replaced it with this

```ruby root_view_controller.rb
mask_path = UIBezierPath.bezierPathWithRoundedRect(@rounded_rect.bounds,
                                                   byRoundingCorners: UIRectCornerAllCorners,
                                                   cornerRadii:       CGSizeMake(30.0, 30.0))
mask_layer = CAShapeLayer.layer
mask_layer.frame = @rounded_rect.bounds
mask_layer.path = mask_path.CGPath
@rounded_rect.layer.mask = mask_layer
```
And our rounded corners are back and better than ever!!!


![](http://content.screencast.com/users/BLove/folders/Jing/media/87b459f3-3893-4d5f-89f0-95484f871fe3/00000009.png)

The cornerRadii for the UIBezierPath is where we set how much extreme we want our corners rounded. The first value of the CGSizeMake is a float for the width of your corner radius, the other being a float for the height of your corner radius. So you can change those values to something like

```ruby root_view_controller
cornerRadii:       CGSizeMake(40.0, 150.0))
```

And you will end up with something a little different.

![](http://content.screencast.com/users/BLove/folders/Jing/media/8a5a9051-4ff0-431f-8036-726f440d4d4c/00000010.png)

You also have more control in which corners actually get the rounding by editing the "byRoundingCorners" value. I currently have that set to UIRectCornerAllCorners. Which we can set to UIRectCornerAllCorners, UIRectCornerTopLeft, UIRectCornerTopRight, UIRectCornerBottomLeft, UIRectCornerBottomRight or any combonation of these seperated by a pipe ( | ). For example to round only the top left and bottom right corners I would change the following.

```ruby root_view_controlelr.rb
byRoundingCorners: UIRectCornerAllCorners,
```
to
```ruby root_view_controlelr.rb
byRoundingCorners: UIRectCornerTopLeft | UIRectCornerBottomRight,
```
Ending up with,

![](http://content.screencast.com/users/BLove/folders/Jing/media/207d53ff-8e1e-4dd1-afa8-6979ed78f54d/00000011.png)

Well thats about it, in the end my code ended up looking like this incase you missed something. Or you can check it out on my Github at [https://github.com/bradylove/RoundedCorners-RubyMotion](https://github.com/bradylove/RoundedCorners-RubyMotion)

```ruby root_view_controller.rb
class RootViewController < UIViewController
  def viewDidLoad
    view.backgroundColor = UIColor.lightGrayColor

    @rounded_rect = UIView.alloc.initWithFrame(
      [[view.size.width / 2 - 100, view.size.height / 2 - 100], [200, 200]])
    @rounded_rect.backgroundColor = UIColor.greenColor

    mask_path = UIBezierPath.bezierPathWithRoundedRect(@rounded_rect.bounds,
                                                       byRoundingCorners: UIRectCornerTopLeft | UIRectCornerBottomRight,
                                                       cornerRadii:       CGSizeMake(40.0, 100.0))
    mask_layer = CAShapeLayer.layer
    mask_layer.frame = @rounded_rect.bounds
    mask_layer.path = mask_path.CGPath
    @rounded_rect.layer.mask = mask_layer

    @non_rounded_subview = UIView.alloc.initWithFrame(@rounded_rect.bounds)
    @non_rounded_subview.backgroundColor = UIColor.colorWithRed(1.0, green: 0.0, blue: 0.0, alpha: 0.5)

    @rounded_rect.addSubview @non_rounded_subview
    view.addSubview @rounded_rect
  end
end
```

Hope you enjoyed. I have not done a whole lot of documentation or blogging but it is something I want to get better at and do more of so any input is more than welcome. Also if anyone has a good suggestion for a screenshot app that does uploads that would be helpful too. Thank you.
