---
title: Merging bones on Blender
description: Quick notes about how to merge bones in Blender, since the process is far from obvious
published: true
date: 2022-02-07T03:36:27.974Z
tags: blender, armature, bones
editor: markdown
dateCreated: 2022-02-04T10:21:21.477Z
---

# Merging bones in Blender 3.x

The operation is needlessly hard on Blender, and the few methods that were available on previous versions, seem to be unuseable on recent versions.

# Using the UI

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

## Using Python

Here's a sample code that allows you to merge selected bones weights with the current active bone :

```python
import bpy

# Function to get the mesh associated with the
# armature.
# Since the linking is done through Modifiers,
# I'll check for the ArmatureModifier of each object,
# when applicable.
def get_associated_mesh(armature) -> bpy.types.Object:
    for o in bpy.context.selectable_objects:
        if o.type != 'MESH':
            continue
        if len(o.modifiers) == 0:
            continue
        for modifier in o.modifiers:
            if type(modifier) is not bpy.types.ArmatureModifier:
                continue
            if modifier.object != armature:
                continue
            return o
    return None

## Preparations

# Check if we're in 'Edit armature' mode.
# Else, I have no idea about how to get the selected
# bones, and their associated armature
if bpy.context.mode != 'EDIT_ARMATURE':
    quit('Wrong mode')

# Get the selected armature
armature = bpy.context.active_object

# Get the associated Mesh
mesh = get_associated_mesh(armature)

# Quit if we don't know which Mesh is associated
# with this armature
if mesh == None:
    quit('No associated Mesh. Forgot to add the Armature modifier ?')

# Get the current active bone
active_bone = bpy.context.active_bone

# Quit if there's no bone selected
if active_bone == None:
    quit('No bone selected')

# This will have to be defined through a UI...
target_vertex_group_name = active_bone.name

# Quit if the mesh have no such vertex group actually
if target_vertex_group_name not in mesh.vertex_groups:
    quit('The active bone has no vertex group associated. Create it before.')

# Get the targeted VertexGroup object
target_vertex_group = mesh.vertex_groups[target_vertex_group_name]

## Generate the vertex groups
# We'll manage the cache with a fixed size array
cached_groups = [set() for _ in range(len(mesh.vertex_groups))]

for v in mesh.data.vertices:
    for group_info in v.groups:
        group_index = group_info.group
        cached_groups[group_index].add((v.index, group_info.weight))

## Add the selected bones vertex groups weights to the target vertex group
selected_bones = bpy.context.selected_bones

for selected_bone in selected_bones:
    if selected_bone == active_bone:
        continue

    # Some bones have no associated vertex group.
    # Skip it if that's the case
    if selected_bone.name not in mesh.vertex_groups:
        continue

    # For each cached vertex data, add it to the target VertexGroup
    vertex_group = mesh.vertex_groups[selected_bone.name]
    for vertex_data in cached_groups[vertex_group.index]:
        target_vertex_group.add([vertex_data[0]], vertex_data[1], 'ADD')

```
