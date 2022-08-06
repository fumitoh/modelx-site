---
layout: post
comments: true
title: "New MxDataView in spyder-modelx v0.13.0"
categories: modelx
---

<div style="text-align: center">
<iframe width="560" height="315" src="https://www.youtube.com/embed/U2VOWWJRIkg" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></div>


[Video transcript]

Spyder modelx version 0.13.0 has just been released.
This release includes a new improved MxDataView widget. 
This video demonstrates how new MxDataView works by comparing it against the previous version's.

This is how the previous MxDataView looks. By the way, the new improved MxDataView works only with Spyder 5, so if you’re using Spyder 4, you can still upgrade the plugin to version 0.13.0, but you get the previous version's MxDataView.

MxDataView is useful for examining the values of modelx objects. But the previous version has some limitations.

The first limitation of the previous version is the link between MxExplorer and MxDataView. When you examine an object in MxDataView, you first need to select it in MxExplorer. 
This behavior is inconvenient when you want to keep checking the same object by changing other objects’ formulas or values in MxExplorer.

For example, if you want to see what the value of this `result_pv`, you would select in the tree in MxExplorer by double clicking on it. And as you see here result_pv is now selected in MxDataView as well.

By clicking the update button to refresh MxDataView, you get the value of `result_pv`, which is, in this case, a pandas DataFrame object that contains the present values of cash flow components by model points.

Let's say you want to see how present values of cash flows will change depending on the maintenance expense assumption. You would select `result_pv`, to see the original values, and then you would select `expense_maint` by double clicking on it. At this moment the selected object in MxDataView changes to `expense_maint`. Then you would edit the formula of `expense_maint` in MxExplorer. And when you try to update the value of `result_pv` in MX DataView, you would end up showing the value `expense_maint` instead. So this behavior is obviously inconvenient.

Another limitation is that you can keep only one object open at a time. So if you want to examine multiple objects, you need to reuse the same pain in MxDataView, to show the value of multiple objects one after another.

Now let's look at the new version of Spyder modelx

This is the new MxDataView widget.
Initially the widget has one tab in it and no object is selected in the tab. 
And it has 2 new toolbar buttons near the top right corner of the widget. 
This button on the left is for creating a new tab, and this button on the right is for clearing the contents of the active tab.

MxExplorer is also updated. It has two new toolbar buttons located near the upper left corner of the widget. 
And also two new items "select in DataView" and "select in new DataView" are added in the context menu of MxExplorer.

Let's see how you can show the value of `result_pv` in MxDataView, as we did with the previous version.
Unlike the previous version, selection of an object in MxExplorer is not synced with MxDataView. You don't have to double-click on `result_pv` and select it in MxExplorer.  

To select it in MxDataView, just right-click on it and from the context menu, select the item, "Select in DataView".
Then `result_pv` is selected in MxDataViewer. 
Click *Update*, then the value of `result_pv` is shown in the widget. 

Let's say you want to see the value of another object, `results_cf`, but you want to keep the result of the value of `result_pv`. Right-click on `result_cf`, and this time select the item *Select in new DataView*. Then a new tab is created in MxDataView, and `result_cf` is selected in the new tab.

Leave the parameter box blank, because the object doesn't have any parameter. Click *update*, then you get the value of `result_cf` showing in the new tab. To see the contents of `result_pv` again, bring the previous tab back up by clicking on the `result_pv` tab.

Instead of selecting items from context menu as we just did, you can do the same operations by clicking one of the taskbar buttons in MxExplorer.


