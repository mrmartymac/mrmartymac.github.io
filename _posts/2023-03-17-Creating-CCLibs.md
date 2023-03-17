---
title: Creating CCLibs
date: 2023-03-17 14:36:30 -400
categories: [cat1, cat2]
tags: [tag1, tag2] # TAG names should always be lowercase
author: mm
---

# Creating new CCLIBS file {#creating-new-cclibs-file .Article-Heading}

Below is an outline of how to create a new CCLIBS (Creative Cloud
Library) file after chaning the paths to the images.

## Mac {#mac .Step-Heading}

1. Open Terminal
2. Change directories to the working directory you unzipped the original cclibs file.
3. Make the changes you need to make.
4. Run the following command to create the new cclibs file. Choose whatever name you want instead of \"ModifiedCCLibrary\". Be sure the suffix of the file is cclibs.zip

```bash
zip ModifiedCCLibrary.cclibs mimetype manifest \*/\*
```

## Windows {#windows .Step-Heading}

1. Right click on **mimetype**
2. Choose **Send to**
3. Choose **Compressed (zipped) folder**

![1.png](/images/creating-new-cclibs-file/media/ssimage1.png){: w="400" }

## Add manifest to the zip {#add-manifest-to-the-zip .Substep-Heading}

Drag **manifest** into **mimetype.zip**

![1.png](/images/creating-new-cclibs-file/media/ssimage2.png){: w="400" }

## Add all the subfolders to the zip {#add-all-the-subfolders-to-the-zip .Substep-Heading}

1. Select all of the folders ensuring you do NOT include the manifest and mimetype files again.
2. Drag them to **mimetype.zip**

![1.png](/images/creating-new-cclibs-file/media/ssimage3.png){: w="400" }

## Rename mimetype.zip {#rename-mimetype.zip .Substep-Heading}

Rename the mimetype.zip file to anything you want, just make sure the suffix is **.cclibs**

![1.png](/images/creating-new-cclibs-file/media/ssimage4.png){: w="400" }