---
layout: post
title:  "Simple OS X Sidebars"
date:   2016-02-12 14:47:51 +0000
categories: osx ios cocoa swift xcode
---

# Building a simple OS X Sidebar

I'm currently working on a OS X (Cocoa) application whose window is split (via NSSplitView) into separate content areas, navigation on the left, main editor in the centre and miscellaneous inspection sidebars on the right side. 

The aim of this post is to understand how an *Xcode* like sidebar could be built with minimal effort like the one seen below: 

![Xcode Sidebar]({{ site.url }}/assets/sample-xcode-sidebar.png){:height="400px" width="200px" :style="float: right;margin-right: 7px;margin-top: 7px;"} 

One would think that the Cocoa framework - as big as it is - would offer some inbuilt controller to come to our aid here. Sadly this doesn't appear to be the case, I've seen various Apple published examples [InfoBarStackView][1] being one but they've never quite worked as I wanted. In particular this relies on *NSViewController* inheritance and this is something I'm keen to avoid (in these anti-OOP days!).


[1]: https://developer.apple.com/library/mac/samplecode/InfoBarStackView/Introduction/Intro.html "InfoBarStackView"

## NSOutlineView 

It would seem obvious that *NSOutlineView* could step into the void here. If run in 'Source' mode then it kind of gives the functionality require such as headers with show hide capability. In my experience however it's not the ideal candidate for the type of sidebar I'm looking for here. 

*I'm not going to comment much on the very buggy nature of this component other than to say it's a complete pain in the backside at times*

## Rolling my Own



### Headers

### Content

### Lets think Protocols

In common with the view that Swift is a protocol oriented language let's see if we can define a few types/protocols that we're going to need in the implementation.

- Firstly, something to indicate the state of an individual sidebar entry. Only really 2 choices here.

{% highlight swift %}

    enum SidebarState {
        case Open
        case Collapsed
    }
    
{% endhighlight %}


- Given we want to be able to different headers datatypes for our sidebar components we need define a protocol which they all must adhere to. *note it has no specific API*

{% highlight swift %}
    protocol HeaderDetailPresenter{}
{% endhighlight %}


- Lets now take the case where we want to have a simple header with a title and a show/hide button (as per standard *NSOutlineView* source header). ***note we make this an immutable value type***
{% highlight swift %}
    struct SimpleLabelledSidebarHeader : HeaderDetailPresenter {
        let title       : String
        let showLabel   : String
        let hideLabel   : String
    }
{% endhighlight %}


- For the purposes of our example here lets define another simple header with only shows some textual content (will be obvious why when we get to define the view later).
{% highlight swift %}
    struct TextSidebarHeader  : HeaderDetailPresenter {
        let content : String
    }
{% endhighlight %}

#### Time to get involved with Views and Controllers now.

- Lets define the API of our Header elements.
    - We will need access to the view & controller
    - We need to some of indicating what action to take when we want to toggle between show and hide. We just use an optional function taking no parameters and returning no value.
    - We need some way of configuring the header (hence the previously defined presenter)
    - Allow the header state to be updated
    - Let the header deal with resizing (eg from a Split View) appropriately.
{% highlight swift %}
protocol SidebarHeaderElement : class {
    var headerView   : NSView {get}
    var controller   : NSViewController {get}
    var toggle       : (() -> ())? {get set}
    
    @warn_unused_result func configure(p: HeaderDetailPresenter) -> Bool
    func update(toViewState vs : SidebarState)
    func canvas(canvas: NSView, frameUpdated f: NSRect)
}
{% endhighlight %}

***we can provide a default implementation of the canvas updated for where no action is required***
{% highlight swift %}
extension SidebarHeaderElement {
    func canvas(canvas: NSView, frameUpdated f: NSRect) {} // default noop
}
{% endhighlight %}


- Now we can think about the API for the content views
{% highlight swift %}
protocol SidebarBodyElement : class {
    var contentView : NSView {get}
    var controller  : NSViewController {get}
    var sidebar     : SidebarElementContainer? {get}
    
    func contentWillCollapse()
    func contentDidCollapse()
    func contentWillExpand()
    func contentDidExpand()
    func canvas(canvas: NSView, frameUpdated f: NSRect)
}

{% endhighlight %}

***Again, we can provide some default implementations of our API functions***
{% highlight swift %}
extension SidebarBodyElement {
    func canvas(canvas: NSView, frameUpdated f: NSRect) {} // default noop
    func contentWillCollapse() {} // default noop
    func contentDidCollapse() {} // default noop
    func contentWillExpand() {} // default noop
    func contentDidExpand() {} // default noop
}
{% endhighlight %}





### Enter NSStackView

### Plugging it all together


You can find all the code here [OsxInfoBar][1]

[1]: https://github.com/plumhead/OsxInfoBar "InfoBar"
























