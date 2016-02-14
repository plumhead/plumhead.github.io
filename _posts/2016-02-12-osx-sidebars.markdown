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

### NSOutlineView 

It would seem obvious that *NSOutlineView* could step into the void here. If run in 'Source' mode then it kind of gives the functionality require such as headers with show hide capability. In my experience however it's not the ideal candidate for the type of sidebar I'm looking for here. 

*I'm not going to comment much on the very buggy nature of this component other than to say it's a complete pain in the backside at times*

## Rolling my Own

This post works through a technique for providing sidebar capabilities using standard views, without view controller inheritance or complicated layout constraint mechanisms. It will utilise the NSStackView component as the 'host container' for the sidebar. 

First, we need to define a few types for our target domain.

### Lets think Protocols

Given that Swift is aimed at 'Protocol Oriented Programming' it seems fitting to work down that path.

>
We're going to allow headers to be built which can be configured to accept configuration information of differing types depending on a particular context. We need the following (empty) protocol to allow us to define domain types which can be passed as configuration to headers.
{% highlight swift %}
protocol HeaderDetailPresentable{}
{% endhighlight %}

>
Lets now define a protocol which will represent an element in the system which can toggle between collapsed (hidden) and expanded (visible) - we provide hooks so the system can, if necessary, add effects to the transition between these states.
{% highlight swift %}
protocol ExpandableElement {
    func contentWillCollapse()
    func contentDidCollapse()
    func contentWillExpand()
    func contentDidExpand()
}
{% endhighlight %}

>
Similar to the expandable element protocol, we define a protocol to represent something which can have it's canvas view resized. Equally applicable to header and content views and is typically the point where you can adjust the header/body content according to the 'host container' size.
{% highlight swift %}
protocol ResizableViewHost {
    func resized(canvas: NSView, toFrame f: NSRect)
}
{% endhighlight %}

>
We can now start to define what the 'header' element will look like. We restrict it to classes (normally NSViewController) and allow it to respond to host container resizing.
>
- Expose the controller behind the header
- Provide a 'toggle' function which any host can set to respond to actions
- Provide a means of configuring the header with presentable detail (eg title, button titles)
- Provide a means of updating the header depending on whether the target content is 'expanded' or 'collapsed'
{% highlight swift %}
protocol SidebarHeaderElement : class , ResizableViewHost{
    var controller   : NSViewController {get}
    var toggle       : (() -> ())? {get set}
    
    @warn_unused_result func configure(p: HeaderDetailPresentable) -> Bool
    func update(toViewState vs : SidebarState)
}
{% endhighlight %}

>
The content element is pretty similar to the header and provides functions for show/hide
{% highlight swift %}
protocol SidebarBodyElement : class , ExpandableElement, ResizableViewHost {
    var controller  : NSViewController {get}
    
    func show()
    func hide()
}
{% endhighlight %}



>Having defined what we want from the header and content elements we can build up a view of what a 'host' will look like. We use a typealias here to allow some configuration of the target host view (as will be seen below relating to stackviews) and make use of a container to hold a header/content combination (SidebarElementContainer).
{% highlight swift %}
protocol SidebarHost : class, ResizableViewHost {
    typealias ViewType
    var hostView        : ViewType {get}
    var hostController  : NSViewController {get}
    var content         : [String:SidebarElementContainer] {get set}
    
    func addSidebar(element : SidebarElementContainer) -> Bool
    func toggle(element: SidebarElementContainer)
    func show(element: SidebarElementContainer)
    func hide(element: SidebarElementContainer)
}
{% endhighlight %}

### Value & Class Types

Now we've defined the high level domain we can start to flesh out the detail.

> Start by  defining an enumerated type to capture the state of a particular sidebar element.
{% highlight swift %}
enum SidebarState {
    case Open
    case Collapsed
}
{% endhighlight %}

>We now define a container to hold associated headers/content
{% highlight swift %}
class SidebarElementContainer {
    let key     : String
    let header  : SidebarHeaderElement
    let body    : SidebarBodyElement
    var state   : SidebarState
    var hasSeparator : Bool
    
    init(key: String, header: SidebarHeaderElement, body: SidebarBodyElement, state: SidebarState, hasSeparator: Bool = false) {
        self.key = key
        self.header = header
        self.body = body
        self.state = state
        self.hasSeparator = hasSeparator
    }
}
{% endhighlight %}

>Now we define structures to hold configuration information for header types 
{% highlight swift %}
struct SimpleLabelledSidebarHeader : HeaderDetailPresentable {
    let title       : String
    let showLabel   : String
    let hideLabel   : String
}
{% endhighlight %}

{% highlight swift %}
struct TextSidebarHeader  : HeaderDetailPresentable {
    let content : String
}
{% endhighlight %}

That's it for our domain types. One initialiser and the rest simple type and protocol definitions.

### What can we Default?

The types defined above can easily be used to build up a working system but with the power of Swift Protocol Extensions we can go one better and start to provide default implementations for elements which, as you'll see, pretty much reduces the amount of code which needs to be defined in views and controllers to nothing (unless particular specialisations required).

> Lets provide default **noop** implementations where possible.
{% highlight swift %}
extension ExpandableElement {
    func contentWillCollapse() {} // default noop
    func contentDidCollapse() {} // default noop
    func contentWillExpand() {} // default noop
    func contentDidExpand() {} // default noop
}

extension SidebarHeaderElement {
    func resized(canvas: NSView, toFrame f: NSRect) {} // default noop
}

extension ResizableViewHost {
    func resized(canvas: NSView, toFrame f: NSRect) {} // default noop
}

{% endhighlight %}

> If we restrict protocol implementations to **NSViewControllers** then we can fill in more detail
{% highlight swift %}
extension SidebarHeaderElement where Self : NSViewController {
    var controller : NSViewController {return self}
}

extension SidebarBodyElement where Self : NSViewController {
    // If attached to a ViewController then we can provide default implementations
    // A more advanced show/hide could add effects (advanced animation etc)
    func show() {self.view.hidden = false}
    func hide() {self.view.hidden = true}
    var controller : NSViewController {return self}
}

{% endhighlight %}

>We can build up generic functionality for our sidebar host now
{% highlight swift %}
extension SidebarHost {
    func toggle(element: SidebarElementContainer) {
        switch element.state {
        case .Open: // move to collapse
            hide(element)
            element.state = .Collapsed
            
        case .Collapsed : // move to open
            show(element)
            element.state = .Open
        }
        
        element.header.update(toViewState: element.state)
    }
    
    func show(element: SidebarElementContainer) {
        element.body.contentWillExpand()
        element.body.show()
        element.body.contentDidExpand()
        element.header.update(toViewState: .Open)
    }
    
    func hide(element: SidebarElementContainer) {
        element.body.contentWillCollapse()
        element.body.hide()
        element.body.contentDidCollapse()
        element.header.update(toViewState: .Collapsed)
    }
    
    func resized(canvas: NSView, toFrame f: NSRect) {
        for (_,v) in content {
            v.header.resized(canvas, toFrame: f)
            v.body.resized(canvas, toFrame: f)
        }
    }
}

extension SidebarHost where Self : NSViewController {
    var hostController : NSViewController {return self}    
}

{% endhighlight %}


### Enter NSStackView

The obvious Cocoa view to use as the default host container is *NSStackView* and we can further extend our library by providing additional default implementation for this. This is where our protocol typealias comes in useful as it allows us to define the implementation constraint.

>Note we can make use of the NSBox component here to add a separation line (if required) between the components.
{% highlight swift %}
extension SidebarHost where ViewType == NSStackView {
    func addSidebar(element : SidebarElementContainer) -> Bool {
        // Make sure we haven't already added a container with the same key
        guard content[element.key] == nil else {return false}
        
        // Set the appropriate action for when the header view 'toggles state'
        element.header.toggle = {[unowned self] in
            self.toggle(element)
        }
        
        // If we want to add a default separator then setup here
        if element.hasSeparator {
            let box = NSBox()
            box.boxType = NSBoxType.Separator
            box.translatesAutoresizingMaskIntoConstraints = false
            hostView.addArrangedSubview(box)
        }
        
        // Add our header view
        hostView.addArrangedSubview(element.header.controller.view)
        
        // Add the main content view
        hostView.addArrangedSubview(element.body.controller.view)
        
        // Make sure the appropriate view controllers are added as children of the current controller
        hostController.addChildViewController(element.body.controller)
        hostController.addChildViewController(element.header.controller)
        
        // Fix the current state
        switch element.state {
        case .Open      : show(element)
        case .Collapsed : hide(element)
        }
        
        // Record the
        content[element.key] = element
        return true
    }
}
{% endhighlight %}


## Plugging it all together

In the provided example project we're looking to build a simple Cocoa window which contains an edit view as the main content and a sidebar which demonstrates usage of the code above. The main storyboard looks like the following:

![Xcode main content]({{ site.url }}/assets/main-edit-storyboard.png){:height="400px" width="600px" :style="float: right;margin-right: 7px;margin-top: 7px;"} 

>
Here the sidebar is simply an *NSStackView* embedded within a *NSScrollView* (in both cases these have origin flipped to be top left rather than bottom left).

### Define Content Views

Next, define whatever content views you require for your sidebar like so:

![Xcode sidebar example]({{ site.url }}/assets/sample-sidebar.png){:height="300px" width="400px" :style="float: right;margin-right: 7px;margin-top: 7px;"} 

The view controller for this particular example must implement the features of the *SidebarBodyElement* protocols but as we've already provided defaults for most of these functions the only item we need to provide is the actual container detail *SidebarElementContainer* as shown. 

**Note**, this particular example stores headers in a separate storyboard *Headers* but you can create these however you like.

{% highlight swift %}
class DocumentDetailSidebar: NSViewController , SidebarBodyElement{
    var sidebar         : SidebarElementContainer? 
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        let sb = NSStoryboard(name: "Headers", bundle: nil)
        let h = sb.instantiateControllerWithIdentifier("StandardHeader") as! StandardSidebarHeaderController         
        h.configure(SimpleLabelledSidebarHeader(title: "Document Options", showLabel: "show", hideLabel: "hide"))
        sidebar = SidebarElementContainer(key: "Document", header: h, body: self, state: .Open, hasSeparator: true)
        
        self.view.wantsLayer = true
        self.view.layer?.backgroundColor = NSColor.controlColor().CGColor
    }
}
{% endhighlight %}

There is however a minor change we can make to this view controller implementation and that is to only create the sidebar when required and not on view load. Move the sidebar creation to a *lazy* var which will ensure it is only created once on first usage. The slightly revised implementation looks like the following:


{% highlight swift %}
class DocumentDetailSidebar: NSViewController , SidebarBodyElement{
    lazy var _sidebar : SidebarElementContainer? = {
        let sb = NSStoryboard(name: "Headers", bundle: nil)
        guard let h = sb.instantiateControllerWithIdentifier("StandardHeader") as? StandardSidebarHeaderController else {
            return .None
        }
        
        h.configure(SimpleLabelledSidebarHeader(title: "Document Options", showLabel: "show", hideLabel: "hide"))
        return SidebarElementContainer(key: "Document", header: h, body: self, state: .Open, hasSeparator: true)
    }()
    
    var sidebar         : SidebarElementContainer? {return _sidebar}
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        self.view.wantsLayer = true
        self.view.layer?.backgroundColor = NSColor.controlColor().CGColor
    }
}
{% endhighlight %}

Unless you need any specialised behaviour then that's it for content view controller implementation.

#### Define Header Views

Now we can define a set of header views which can be used for the sidebar like so:

![Xcode header example]({{ site.url }}/assets/sample-header.png){:height="150px" width="400px" :style="float: right;margin-right: 7px;margin-top: 7px;"} 

The header view controller requires a little more code than content view controllers and will generally look like the following:

>
In this example we have a button which triggers a toggle between collapsed and expanded states - it doesn't handle the change of state itself but rather calls a *toggle* function provided by the host container. The mouse is tracked so that the button is shown only when the mouse pointer falls within the header view itself and hidden otherwise.

**note** due to the way this header is created the configure has the rather odd looking line *_ = self.view* which triggers the view load if not already loaded.

{% highlight swift %}
class StandardSidebarHeaderController : NSViewController {
    @IBOutlet weak var headerText: NSTextField!
    @IBOutlet weak var showHideBtn: NSButton!
    
    var _presenter   : SimpleLabelledSidebarHeader? {
        didSet {
            guard viewLoaded else {return}
            headerText.stringValue = _presenter?.title ?? ""
        }
    }
    
    var toggle       : (() -> ())? // will be set by the sidebar controller to an appropriate action
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        self.view.wantsLayer = true
        self.view.layer?.backgroundColor = NSColor.controlColor().CGColor
        
        // Track mouse movement within the header so we can show or hide the show/hide button
        let tracker = NSTrackingArea(rect: self.view.bounds, options: [NSTrackingAreaOptions.MouseEnteredAndExited, NSTrackingAreaOptions.ActiveAlways, NSTrackingAreaOptions.InVisibleRect], owner: self , userInfo: nil)
        self.view.addTrackingArea(tracker)
    }
    
    @IBAction func showHidePressed(sender: AnyObject) {
        toggle?()
    }
    
    override func mouseEntered(theEvent: NSEvent) {
        super.mouseEntered(theEvent)
        showHideBtn.hidden = false
    }
    
    override func mouseExited(theEvent: NSEvent) {
        super.mouseExited(theEvent)
        showHideBtn.hidden = true
    }
}

//MARK: - SidebarHeaderElement implementation
extension StandardSidebarHeaderController : SidebarHeaderElement {
    func configure(p: HeaderDetailPresentable) -> Bool {
        guard let pr = p as? SimpleLabelledSidebarHeader else {return false}
        _ = self.view
        _presenter = pr
        return true
    }
    
    func update(toViewState vs : SidebarState) {
        switch vs {
        case .Open: showHideBtn.title = _presenter?.hideLabel ?? ""
        case .Collapsed: showHideBtn.title = _presenter?.showLabel ?? ""
        }
    }
}
{% endhighlight %}

### Define the Sidebar Host

Finally, it's time to build the component which manages the set of sidebar elements. As mentioned previously, it's based on a view controller which manages an *NSStackView* and in the example project looks like the following:

>
We need to provide storage for the **content**, a reference to the **hostView** and an implementation of **resized** (if required) and that's it. The other code is simply loading elements into the view.

>The CanvasDetailController shown here shows how a view can be configured using different headers. See main source for details on this.

{% highlight swift %}
class SidebarController: NSViewController , SidebarHost {
    @IBOutlet weak var scroller: FlippedScroll!
    @IBOutlet weak var stack: FlippedStack!
    
    //MARK:- Sidebar delegate
    typealias ViewType = NSStackView
    var content : [String:SidebarElementContainer] = [:]
    var hostView : NSStackView {return stack}
    
    //MARK: - View Initialisation
    override func viewDidLoad() {
        super.viewDidLoad()
        
        let sb = NSStoryboard(name: "Sidebars", bundle: nil)
        let sb1 = sb.instantiateControllerWithIdentifier("DocumentSidebar") as? DocumentDetailSidebar
        
        let sb2 = sb.instantiateControllerWithIdentifier("CanvasSidebar") as? CanvasDetailController
        
        let sb3 = sb.instantiateControllerWithIdentifier("TableSidebar") as? TableDetailController
        
        let sb4 = sb.instantiateControllerWithIdentifier("CollectionSidebar") as? CollectionDetailController
 
        let sb5 = sb.instantiateControllerWithIdentifier("CanvasSidebar") as? CanvasDetailController
        sb5?.useSidebar1 = false
        
        _ = [sb1?.sidebar, sb4?.sidebar, sb2?.sidebar, sb3?.sidebar, sb5?.sidebar]
            .flatMap{$0}
            .map{ addSidebar($0) }
        
        NSNotificationCenter.defaultCenter().addObserver(self , selector: Selector("viewFrameChanged:"), name: NSViewFrameDidChangeNotification, object: self.view)
    }
    
    @IBAction func viewFrameChanged(obj: AnyObject) {
        resized(self.view, toFrame: self.view.frame)
    }
}
{% endhighlight %}

### Just a word on Constraints

For the system to function as expected it is required that you define appropriate view layout constraints - in general this means a view which is flexible in its width but fixed in its height. These can generally be defined through Xcode - see the sample project for examples on how to do this.

Flexibility in content height can be achieved but it requires that you calculate and set the expected height of your content on resize - see the CollectionDetailController sample for an example of how to do this.

## What does it look like?

The sample code provided creates a sidebar looking like the following:

![Running Example]({{ site.url }}/assets/sample-running-sidebar.png){:height="500px" width="300px" :style="float: right;margin-right: 7px;margin-top: 7px;"} 

>
Ok, not the latest in UI design but that's up to you to define! Have fun.

You can find all the code here [OsxInfoBar][1]

[1]: https://github.com/plumhead/OsxInfoBar "InfoBar"
























