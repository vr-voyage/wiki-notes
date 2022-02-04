---
title: Merging bones on Blender
description: Quick notes about how to merge bones in Blender, since the process is far from obvious
published: true
date: 2022-02-04T20:09:39.088Z
tags: blender, armature, bones
editor: markdown
dateCreated: 2022-02-04T10:21:21.477Z
---

# Merging bones on Blender 3.x

The operation is needlessly hard on Blender, and the few methods that were available on previous versions, seem to be unuseable on recent versions.

## Dissolve and Vertex Weight Mix

One way is to :
* **Dissolve** the child-bones to merge them with the parent
* Use the **Vertex Weight Mix** modifier to move the child-bones Vertex Groups weights to the parent one.

You can do these operations in any order (Dissolve then Vertex Weight, or Vertex Weight then Dissolve).

Anyway, the procedure is :

### Merge the child bones with Dissolve

* Select the **armature**
* Get into `Edit` mode
* Select all the child bones you want to merge with their parent
* In the *Bone* panel, check _Connected_ by `Alt` + `Left Click`-ing it

> Technically, you can do this from the outliner, without getting into `Edit` mode
{.is-info}

### Move the child weights to the parent with Vertex Weight Mix

Then, for each parent :

**In order to see the result before applying the changes**

* Select the **mesh**
* Get into `Weight paint` mode
* Select the vertex group corresponding to the parent to visualise the weights

**In any case**

* Add a **Vertex Weight Mix** modifier
  * In **Vertex Group A**, select the parent bone Vertex Group
  * In **Vertex Group B**, select one of the removed child bone Vertex Group
  * In **Vertex Set**, select **Vertex Group B**
  * In **Mix Mode**, select **Add**

Then if you need to copy other children weights :

* Duplicate the modifier and change **Vertex Group B** in the duplicates

Finally :

* Apply the **Vertex Weight Mix** modifiers

Once done, you can remove the removed bones Vertex Groups.
