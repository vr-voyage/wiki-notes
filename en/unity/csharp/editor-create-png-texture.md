---
title: Create a texture save it as a PNG file
description: Describes how to create a texture, inside an editor script, and save it to PNG format. This also applies for various other formats too.
published: true
date: 2022-03-13T04:29:25.613Z
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

Creating a [**Texture2D**](https://docs.unity3d.com/ScriptReference/Texture2D.html) just require to use the following [constructor](https://docs.unity3d.com/ScriptReference/Texture2D-ctor.html) :

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

To set the pixels colors, the simple way is to create an array of `width` * `height` Color or Color32 objects, and then either use [**texture.SetPixels**](https://docs.unity3d.com/ScriptReference/Texture2D.SetPixels.html) or [**SetPixels32**](https://docs.unity3d.com/ScriptReference/Texture2D.SetPixels32.html).  
When using a more complex texture format, where you understand the pixel format, you can use [**SetPixelsData**](https://docs.unity3d.com/ScriptReference/Texture2D.SetPixelData.html).  
This method won't be shown here, however.

```csharp
Color32 colors = new Color32[texture.width * texture.height];
// Use some some code to fill the array, like a "for" loop for example.
texture.SetPixels32(colors);
```

## Convert the texture to PNG and save it to disk

To convert the texture to a Unity known format, use one of the following extension function :
  * [**EncodeToEXR**](https://docs.unity3d.com/ScriptReference/ImageConversion.EncodeToEXR.html) to export to EXR format
  * [**EncodeToJPG**](https://docs.unity3d.com/ScriptReference/ImageConversion.EncodeToJPG.html) to export to JPEG format
  * [**EncodeToPNG**](https://docs.unity3d.com/ScriptReference/ImageConversion.EncodeToPNG.html) to export to PNG format
  * [**EncodeToTGA**](https://docs.unity3d.com/ScriptReference/ImageConversion.EncodeToTGA.html) to export to TGA format
  
All export functions are used like this :

```csharp
texture.EncodeToPNG();
```

> The JPEG format can take an optional `quality` argument, indicating the level of lossy compression desired. Not passing this argument is equivalent to passing `75`.  
> So  `texture.EncodeToJPG()` is the same as `texture.EncodeToJPG(75)`
{.is-info}


The Encode functions will return a byte array that can then written on the disk to generate the appropriate file.  
We cannot use the [**AssetDatabase**](https://docs.unity3d.com/ScriptReference/AssetDatabase.html) to generate such files, since it only works for Unity objects, and here we got a simple C# `byte[]` System object.  
So, instead, we have to rely on System I/O functions like [**File.WriteAllBytes**](https://docs.microsoft.com/en-us/dotnet/api/system.io.file.writeallbytes?view=net-6.0) but this require to know the actual absolute path of the file we want to create.

When using an editor script, [**Application.dataPath**](https://docs.unity3d.com/ScriptReference/Application-dataPath.html) provides the absolute path to the **Assets** folder used in the project.

> Even on Windows, this path is written using forward-slashes.  
> Forward-slashes paths are supported on Windows since Windows XP.
{.is-info}

By combining the **Assets** absolute directory path, with an asset relative file path, we get the asset absolute file path :

```csharp
string saveFilePath = $"{Application.dataPath}/{assetsRelativeFilePath}"
```

Once the filepath determined, saving the exported content can be done with `File.WriteAllBytes(string filepath, byte[] content)`.

```csharp
string relativeFilePath = "GeneratedTexture.png";
File.WriteAllbytes($"{Application.dataPath}/{relativeFilePath}", texture.EncodeToPNG());
```

# Extra

## Refresh the Database

When using `File.WriteAllBytes()`, Unity won't refresh the Assets Database, and the file won't be seen in the `Project` tab until the database is actually refreshed.  
This can be done by calling [**AssetDatabase.Refresh**](https://docs.unity3d.com/ScriptReference/AssetDatabase.Refresh.html).

## Setup the import settings

Since the PNG file is not generated using Unity functions, Unity can't link the generated texture to the generated PNG file.  
Hence the file will inherit default import settings once imported into Unity database (`AssetDatabase.Refresh()`).

The import settings of any asset can be accessed through a script editor, by using [**AssetImporter.GetAtPath**](https://docs.unity3d.com/ScriptReference/AssetImporter.GetAtPath.html).

This function allows to retrieve the import settings of a specific asset.  
The function returns an [**AssetImporter**](https://docs.unity3d.com/ScriptReference/AssetImporter.html) object, which is a class inherited by various importers objects.  
In order to access the Texture import settings, the object needs to be casted into a specicialized [**TextureImporter**](https://docs.unity3d.com/ScriptReference/TextureImporter.html) object, using the following code :

```csharp
TextureImporter textureImportSettings =
    AssetImporter.GetAtPath($"Assets/{relativeFilePath}") as TextureImporter;
```

Various fields of the texture importer share the same name as **Texture2D** objects fields, which make it easier to replicate settings.  
Though, for the most parts, this just boils down to setting up the sampler default filter mode, and the amount of mipmaps.  
Once the import settings modified, call [**importer.SaveAndReimport**](https://docs.unity3d.com/ScriptReference/AssetImporter.SaveAndReimport.html) to save them.

```csharp
textureImportSettings.filterMode    = texture.filterMode;
textureImportSettings.mipmapEnabled = texture.mipmapCount > 1;
textureImportSettings.SaveAndReimport();
```

## Reload the texture from disk

You can reload the PNG texture from the disk using :

```csharp
Texture2D reloadedTexture = AssetDatabase.LoadAssetAtPath<Texture2D>($"Assets/{relativeFilePath}");
```

Note that, the first **Texture2D** was saved to the disk without using Unity functions.  
Hence, Unity cannot link the first **Texture2D** to the PNG file generated out of it.  
This has multiple implications.  
If you generate a [**Material**](https://docs.unity3d.com/ScriptReference/Material.html) and setup a texture field by passing the first **Texture2D** to [**Material.SetTexture**](https://docs.unity3d.com/ScriptReference/Material.SetTexture.html), the material will lose its reference when saved to disk using [**AssetDatabase.CreateAsset**](https://docs.unity3d.com/ScriptReference/AssetDatabase.CreateAsset.html).

However, passing the PNG file path to [**AssetDatabase.LoadAssetAtPath**](https://docs.unity3d.com/ScriptReference/AssetDatabase.LoadAssetAtPath.html) will return a new **Texture2D** object, that will be linked to this file.

Meaning that, this time, if you generate a new **Material**, setup one its texture property with the reloaded texture, using **Material.SetTexture** and save the material with **AssetDatabase.CreateAsset**; the saved material will keep the reference to the actual PNG file.

> This new Texture2D object will follow the file Import settings.
{.is-info}


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
                /* The filter mode is only setup to showcase
                 * import settings modification afterwards. */
                texture.filterMode = FilterMode.Point;
                int textureSize = texture.width * texture.height;
                Color[] colors = new Color[textureSize];

                /* This variable is just used to generate a rainbow pattern */
                float max = textureSize;

                for (int i = 0; i < textureSize; i++)
                {
                    colors[i] = Color.HSVToRGB(i / max, 1, 1);
                }

                texture.SetPixels(colors);

                File.WriteAllBytes(
                    $"{Application.dataPath}/{relativeFilePath}",
                    texture.EncodeToPNG());

                AssetDatabase.Refresh();

                TextureImporter textureImportSettings =
                    AssetImporter.GetAtPath($"Assets/{relativeFilePath}") as TextureImporter;
                textureImportSettings.filterMode = texture.filterMode;
                textureImportSettings.mipmapEnabled = texture.mipmapCount > 1;
                textureImportSettings.SaveAndReimport();
            }
        }
    }
}

#endif
```