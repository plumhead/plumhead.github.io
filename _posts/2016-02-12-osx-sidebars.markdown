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


## NSOutlineView 


## Why not a Custom Sidebar

### Headers

### Content

### Protocols & Things

{% highlight swift %}

    enum SidebarState {
        case Open
        case Collapsed
    }

    protocol HeaderDetailPresenter{}

    struct SimpleLabelledSidebarHeader : HeaderDetailPresenter {
        let title       : String
        let showLabel   : String
        let hideLabel   : String
    }
    
{% endhighlight %}


### Enter NSStackView

### Plugging it all together

 