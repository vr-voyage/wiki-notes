---
title: Create a texture save it as a PNG file
description: Describes how to create a texture, inside an editor script, and save it to PNG format. This also applies for various other formats too.
published: true
date: 2022-03-13T03:46:21.944Z
tags: unity, editor, csharp
editor: markdown
dateCreated: 2022-03-06T03:05:30.511Z
---

# Create a PNG texture file with a Unity Editor script

The operation is pretty simple and consist in :

* Creating a Texture2D object
* Setting the pixels colors, through an array of Color or Color32 pixels.
* Saving the texture converted to PNG in the application **Data Path**, which is the **Assets** folder when using an Editor Script.

## Creating a Texture2D object

Create a Texture2D just require to use the following constructor :

```csharp
new Texture2D(width, height);
```

This will create a RGBA32 Texture, with mipmaps but no compression.

For more control, the following constructors can be used :

```csharp
Texture2D(int width, int height, TextureFormat textureFormat, bool mipChain, bool linear)
```

```csharp
Texture2D(int width, int height, GraphicsFormat format, int mipCount, TextureCreationFlags flags)
```

> With the TextureCreationFlags, you can enable CrunchCompression.
> {.is-info}

## Set the pixels Colors

To set the pixels colors, the simple way is to create an array of `width` * `height` Color or Color32 objects, and then either use `texture.SetPixels` or `SetPixels32`.  
When using a more complex texture format, where you understand the pixel format, you can use `SetPixelsData<T>`. This method won't be shown here, however.

```csharp
Color32 colors = new Color32[texture.width * texture.height];
// Use some some code to fill the array, like a "for" loop for example.
texture.SetPixels32(colors);
```

## Convert the texture to PNG and save it to disk

To convert the texture to a Unity known format, use one of the following function :
  * `EncodeToEXR` to export to EXR format
  * `EncodeToJPG` to export to JPEG format
  * `EncodeToPNG` to export to PNG format
  * `EncodeToTGA` to export to TGA format
  
All export functions are used like this :

```csharp
texture.EncodeToPNG();
```

> The JPEG format can take an optional `quality` argument, indicating the level of lossy compression desired. Not passing this argument is equivalent to passing `75`.  
> So  `texture.EncodeToJPG()` is the same as `texture.EncodeToJPG(75)`
{.is-info}


The Encode functions will return a `byte[]` array that can then written on the disk to generate the appropriate file.  
We cannot use the `AssetDatabase` to generate such files, since it only works for Unity objects.  
Instead, we can use functions like `File.WriteAllBytes` but this require to know the actual absolute path of the file we want to create.

When using an editor script, `Application.dataPath` provides the absolute path to the **Assets** folder used in the project.

> Even on Windows, this path is written using forward-slashes.  
> Forward-slashes paths are supported on Windows since Windows XP.
{.is-info}

By combining the **Assets** absolute directory path, with an asset relative file path, we get the asset absolute file path :

```csharp
string saveFilePath = $"{Application.dataPath}/{assetsRelativeFilePath}"
```

Once the filepath determined, saving the exported content can be done with `File.WriteAllBytes(string filepath, byte[] content)`

```csharp
string relativeFilePath = "GeneratedTexture.png";
File.WriteAllbytes($"{Application.dataPath}/{relativeFilePath}", texture.EncodeToPNG());
```

# Extra

## Refresh the Database

When using `File.WriteAllBytes()`, Unity won't refresh the Assets Database, and the file won't be seen in the `Project` tab until the database is actually refreshed.  
This can be done by calling `AssetDatabase.Refresh()`.

## Setup the import settings

Since the PNG file is not generated using Unity functions, Unity can't link the generated texture to the generated PNG file.  
Hence the file will inherit default import settings once imported into Unity database (`AssetDatabase.Refresh()`).

The import settings of any asset can be accessed through a script editor, by using [**AssetImporter.GetAtPath**](https://docs.unity3d.com/ScriptReference/AssetImporter.GetAtPath.html).

This function allows to retrieve the import settings of a specific asset.  
The function returns an **AssetImporter** object, which is a class inherited by various importers objects.  
In order to access the Texture import settings, the object needs to be casted into a specicialized [**TextureImporter**](https://docs.unity3d.com/ScriptReference/TextureImporter.html) object, using the following code :

```csharp
TextureImporter textureImportSettings =
    AssetImporter.GetAtPath($"Assets/{relativeFilePath}") as TextureImporter;
```

Various fields of the texture importer share the same name as **Texture2D** objects fields, which make it easier to replicate settings.  
Though, for the most parts, this just boils down to setting up the sampler default filter mode, and the amount of mipmaps.

```csharp
textureImportSettings.filterMode    = texture.filterMode;
textureImportSettings.mipmapEnabled = texture.mipmapCount > 1;
```

# Full simple example

```csharp
#if UNITY_EDITOR

using System.IO;

using UnityEditor;
using UnityEngine;

namespace Myy
{
    class GenerateTextureExample : EditorWindow
    {

        SerializedObject serialO;
        SerializedProperty savePathSerialized;
        public string savePath;

        private void OnEnable()
        {
            serialO = new SerializedObject(this);
            savePathSerialized = serialO.FindProperty("savePath");
        }


        [MenuItem("Voyage / Save generated texture to PNG")]
        public static void ShowWindow()
        {
            GetWindow(typeof(GenerateTextureExample), false, "Constraints");
        }
        private void OnGUI()
        {
            bool everythingOK = true;

            serialO.Update();
            EditorGUILayout.PropertyField(savePathSerialized);
            if (savePath == null)
            {
                EditorGUILayout.HelpBox(
                    "Please type where to save the new Asset",
                    MessageType.Error);
                everythingOK = false;
            }
            serialO.ApplyModifiedProperties();

            if (!everythingOK) return;

            if (GUILayout.Button("Create Texture"))
            {

                string relativeFilePath =
                    (savePath.EndsWith(".png") ? savePath : savePath + ".png");


                Texture2D texture = new Texture2D(256, 256);
                int textureSize = texture.width * texture.height;
                Color[] colors = new Color[textureSize];

                float max = textureSize;

                for (int i = 0; i < textureSize; i++)
                {
                    colors[i] = Color.HSVToRGB(i / max, 1, 1);
                }

                texture.SetPixels(colors);


                File.WriteAllBytes(
                    $"{Application.dataPath}/{relativeFilePath}.png",
                    texture.EncodeToPNG());

                AssetDatabase.Refresh();

            }
        }
    }
}

#endif
```