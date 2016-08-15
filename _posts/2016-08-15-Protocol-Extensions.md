---
layout: post
title: Protocol Extensions. The Bad,The Good,The Ugly
---

An experiment with protocol extensions that goes frustratingly wrong.

##Spoilers
- The Ugly solution is the best. 
- Protocol extensions won't work for optional methods.

## The Bad

The bad in this post is life before protocol extensions. I'm a big fan of the WWDC 2015 session introducing [Protocol Oriented Programming](https://developer.apple.com/videos/play/wwdc2015/408) (POP). I remember being blown away when I saw it. I watched it again when I got home, and then again when I started doing more Swift development.

The primary feature that enables POP is the extension of protocols. In essence, protocol extensions are powerful because they let us provide a default implementation for a protocol. I'm going to talk about a real world use case I came across.

#### My Issue
I was trying to create the simplest possible UICollectionViewDelegate that implemented a focus functionality.
Briefly, this functionality is intended to focus on a particular cell after rotating the device:

- The first cell that was selected *and* visible prior to the rotation, or, failing that
- The first cell that was mostly visible prior to the rotation

#### The Bad Solution

Let's take a look at the relevant parts of the code that achieve this ([here's the complete file](https://github.com/kenthumphries/KHCollectionViewTest/blob/TheBad/CollectionViewTest/SimpleDelegate.swift)).

We create a concrete delegate that implements UICollectionViewDelegate and store which indexPath was last 'focussed on'.

    class SimpleDelegate: NSObject, UICollectionViewDelegate {
        var selectedIndexPath = NSIndexPath(forItem: 0, inSection: 0)
        var focussedIndexPath = NSIndexPath(forItem: 0, inSection: 0)
        ...

We need to keep the focussedIndexPath up to date following any scrolling events

    func scrollViewDidEndDragging(scrollView: UIScrollView, willDecelerate decelerate: Bool) {
      if !decelerate {
        self.scrollViewDidEndScrolling(scrollView)
      }
    }
    
    func scrollViewDidEndDecelerating(scrollView: UIScrollView) {
        self.scrollViewDidEndScrolling(scrollView)
    }
    
    func scrollViewDidEndScrolling(scrollView: UIScrollView) {
    
        guard let collectionView = scrollView as? UICollectionView,
            flowLayout = collectionView.collectionViewLayout as? UICollectionViewFlowLayout else {
            return
        }
        
        let visibleIndexPaths = collectionView.sortedIndexPathsForVisibleItems()
        
        for indexPath in visibleIndexPaths {
            if let center = flowLayout.centerForItemAtIndexPath(indexPath) where center > collectionView.contentOffset {
                focussedIndexPath = indexPath
                break;
            }
        }
    }

Then any time there is a rotation, we need to return the correct targetContentOffsetForProposedContentOffset (thanks to this [SO post](http://stackoverflow.com/questions/13780138/dynamically-setting-layout-on-uicollectionview-causes-inexplicable-contentoffset)).

    func collectionView(collectionView: UICollectionView, targetContentOffsetForProposedContentOffset proposedContentOffset: CGPoint) -> CGPoint {
        
        guard let flowLayout = collectionView.collectionViewLayout as? UICollectionViewFlowLayout else {
            return proposedContentOffset
        }
        
        return self.targetContentOffsetForProposedContentOffset(proposedContentOffset, flowLayout: flowLayout)
    }
    
    func targetContentOffsetForProposedContentOffset(proposedContentOffset: CGPoint, flowLayout: UICollectionViewFlowLayout) -> CGPoint {
        var contentOffset = proposedContentOffset
        
        // Determine whether to focus on selectedIndexPath or scrolledIndexPath
        let indexPathToFocusOn: NSIndexPath
        if let visibleIndexPaths = flowLayout.collectionView?.indexPathsForVisibleItems() where visibleIndexPaths.contains(selectedIndexPath) {
            indexPathToFocusOn = selectedIndexPath
        } else {
            indexPathToFocusOn = focussedIndexPath
        }
        
        if let frame = flowLayout.frameForItemAtIndexPath(indexPathToFocusOn) where frame != CGRectZero {
            let originX = max(0, frame.origin.x - flowLayout.minimumInteritemSpacing)
            let originY = max(0, frame.origin.y - flowLayout.minimumInteritemSpacing)
            contentOffset = CGPointMake(originX, originY)
        }
        
        return contentOffset
    }

This works as expected - you can download The Bad project (Swift 2.2) by checking out [this commit](https://github.com/kenthumphries/KHCollectionViewTest/commit/5b2b2ee8c83df753f87bab4254cefa581bd8270c).

## The Good

Now although The Bad works, it's not particularly sexy.

1. The focusing code is spread throughout a delegate that is doing multiple things
1. Each delegate that wants this ability to focus will need to implement all of the code shown above

#### The Good Solution

Protocol Extensions!

With Protocol extensions I can solve both of the above shortcomings.


1. A single protocol that contains all the code required for focusing
1. Default implementations that provide the functionality for free to any UICollectionViewDelegate

There are two parts to the UICollectionViewDelegateFlowLayoutFocusing protocol ([here's the complete file](https://github.com/kenthumphries/KHCollectionViewTest/blob/TheGood/CollectionViewTest/UICollectionViewDelegateFlowLayoutFocusing.swift)).

*A note on naming - this protocol only applies to UICollectionViews with flow layouts, hence the UICollectionViewDelegateFlowLayout part, and the Focusing suffix to show what this protocol does.*

    public protocol UICollectionViewDelegateFlowLayoutFocusing: UIScrollViewDelegate, UICollectionViewDelegate,     UICollectionViewDelegateFlowLayout {
        
        var focusedIndexPath: NSIndexPath { get set }
        
        // Must be called by collectionView
        func collectionViewDidEndScrolling(scrollView: UIScrollView)
        func focussedContentOffset(collectionView collectionView: UICollectionView) -> CGPoint?
        // Customisation point
        func indexPathToFocusOn(collectionView collectionView: UICollectionView, flowLayout: UICollectionViewFlowLayout) -> NSIndexPath?
    }

The protocol definition includes:

- a required variable for tracking which indexPath was last focused
- two required methods that should be called by the UICollectionViewDelegateFlowLayout
- a required method that is exposed for customisation (to change the algorithm for choosing which indexPath has focus)

    extension UICollectionViewDelegateFlowLayoutFocusing {
        
        func collectionViewDidEndScrolling(scrollView: UIScrollView) {
            
            guard let collectionView = scrollView as? UICollectionView,
                let flowLayout = collectionView.collectionViewLayout as? UICollectionViewFlowLayout else {
                return
            }
            
            if let indexPathToFocusOn = self.indexPathToFocusOn(collectionView: collectionView, flowLayout: flowLayout) {
                self.focusedIndexPath = indexPathToFocusOn
            }
        }
        
        func indexPathToFocusOn(collectionView collectionView: UICollectionView, flowLayout: UICollectionViewFlowLayout) -> NSIndexPath? {
            
            var indexPathToFocusOn = self.focusedIndexPath // If a new focus can't be found, default to last focus
            
            let visibleIndexPaths = collectionView.sortedIndexPathsForVisibleItems()
            
            if let selectedIndexPaths = collectionView.indexPathsForSelectedItems(),
                visibleSelectedIndexPath = NSArray(array: visibleIndexPaths).firstObjectCommonWithArray(selectedIndexPaths) as? NSIndexPath {
                // One of the selected cels is visible, so use it for the focus
                indexPathToFocusOn = visibleSelectedIndexPath
            } else {
                // No selected visible cells, find the first cell that's at least half visible
                for indexPath in visibleIndexPaths {
                    if let center = flowLayout.centerForItemAtIndexPath(indexPath) where center > collectionView.contentOffset {
                        indexPathToFocusOn = indexPath
                        break;
                    }
                }
            }
            
            return indexPathToFocusOn
        }
        
        func focussedContentOffset(collectionView collectionView: UICollectionView) -> CGPoint? {
            
            guard let flowLayout = collectionView.collectionViewLayout as? UICollectionViewFlowLayout,
                let frame = flowLayout.frameForItemAtIndexPath(focusedIndexPath) where frame != CGRectZero else {
                return nil
            }
            
            let originX = max(0, frame.origin.x - flowLayout.minimumInteritemSpacing)
            let originY = max(0, frame.origin.y - flowLayout.minimumInteritemSpacing)
            return CGPointMake(originX, originY)
        }
    }


The extension of UICollectionViewDelegateFlowLayoutFocusing protocol allows us to provide default implementations for the required methods. This is really powerful as all the complex logic for focusing on a cell is provided along with the protocol itself.

    extension UICollectionViewDelegate where Self: UICollectionViewDelegateFlowLayoutFocusing{
      
      func scrollViewDidEndDragging(scrollView: UIScrollView, willDecelerate decelerate: Bool) {
          
          if !decelerate {
              self.collectionViewDidEndScrolling(scrollView)
          }
      }
      
      func scrollViewDidEndDecelerating(scrollView: UIScrollView) {
          
          self.collectionViewDidEndScrolling(scrollView)
      }
      
      func collectionView(collectionView: UICollectionView, targetContentOffsetForProposedContentOffset proposedContentOffset: CGPoint) -> CGPoint {
          
          guard let focusedPoint = self.focussedContentOffset(collectionView: collectionView) else {
                  return proposedContentOffset
          }
          
          return focusedPoint
      }
    }


And for an added convenience, we can provide default implementations for some optional methods of UICollectionViewDelegate. Note that the extension only applies *where Self: UICollectionViewDelegateFlowLayoutFocusing* so these default implementations are scoped to delegates that are also implementing UICollectionViewDelegateFlowLayoutFocusing. 

The only obvious downside to this implementation is that if a delegate overrides any of these methods, the default implementations will not be called and focusing will no longer work. For that reason these methods are as simple as possible - in case this code needs to be included in an overridden version.

The reward for all this is a UICollectionViewDelegate that is incredibly simple:

    class SimpleDelegate: NSObject, UICollectionViewDelegateFlowLayoutFocusing // Ensure that the CollectionView is Focusing
    {
        var focusedIndexPath = NSIndexPath(forItem: 0, inSection: 0)
    }

This is the power of protocol extensions. As we have provided default implementations for all the functionality of UICollectionViewDelegateFlowLayoutFocusing, any delegate can add this functionality by implementing the protocol - simply defining a single focusedIndexPath variable.

You can download The Good project (Swift 2.2) by checking out [this commit](https://github.com/kenthumphries/KHCollectionViewTest/commit/de5dcc1534a655e425f1f91185bbe56b18392117). But first you should read about The Ugly solution.

## The Ugly

Now although The Good looks beautiful, and super Swifty, it doesn't actually work.

The wrinkle is that we're implementing optional methods on UICollectionViewDelegate. And in order for Swift to work with Objective-C, [all optional methods are declared with the @objc keyword](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/Protocols.html#//apple_ref/doc/uid/TP40014097-CH25-ID284). This enforced compatibility with objective-C also [causes our extension to be ignored in Swift](http://stackoverflow.com/a/32611453). Essentially, our protocol extension never receives the delegate callbacks.

#### The Ugly Solution

Fewer Protocol Extensions!
Inheritance!

We can still take advantage of the default implementations for UICollectionViewDelegateFlowLayoutFocusing - these are non-optional methods.

We need to delete the UICollectionViewDelegate extension 😢 and instead add this code directly to a delegate. **At least we made it as concise as possible!**

    class FocusingDelegate: NSObject, UICollectionViewDelegateFlowLayoutFocusing // Ensure that the CollectionView is Focusing
    {
        var focusedIndexPath = NSIndexPath(forItem: 0, inSection: 0)
    }
    
    // MARK: Call UICollectionViewDelegateFlowLayoutFocusing methods on scrolling or external contentOffset change
    extension FocusingDelegate {
        
        func scrollViewDidEndDragging(scrollView: UIScrollView, willDecelerate decelerate: Bool) {
            if !decelerate {
                self.collectionViewDidEndScrolling(scrollView)
            }
        }
        
        func scrollViewDidEndDecelerating(scrollView: UIScrollView) {
            self.collectionViewDidEndScrolling(scrollView)
        }
        
        func collectionView(collectionView: UICollectionView, targetContentOffsetForProposedContentOffset proposedContentOffset: CGPoint) ->     CGPoint {
            return self.focussedContentOffset(collectionView, proposedContentOffset: proposedContentOffset)
        }
    }

This solution is not *that Ugly*.

And we have two options going forward:

1. Subclass FocusingDelegate
⋅⋅* This is a neat solution, but could cause issues if a delegate needs to inherit from another class
2. Copy the code of FocusingDelegate into any focusing delegate

You can download The Ugly project (Swift 2.2) by checking out [this commit](https://github.com/kenthumphries/KHCollectionViewTest/commit/5a6a501d1898c8e5f7930fcc25945b4da61b2a6d).

-----

**Comments? [Tweet me](https://twitter.com/kentios).**
