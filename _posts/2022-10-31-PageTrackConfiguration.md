---
title: PageTrack Configuration Documentation
date: 2022-10-31 16:06:25 -500
categories: [Documentation, PageTrack]
tags: [documentation, server, pagetrack] # TAG names should always be lowercase
author: mm
---
PageTrack configuration is now stored in INI format. Most of it can be edited via the `File -> Preferences` window
in PageTrack Product Manager.

## `namingconvention`
This tag defines how the names of the desktop publishing documents (i.e. InDesign documents)
are generated.

#### Labels
The namingconvention tag contains a number of labels, started with “[“ and ended with “]”. The first character
(which must be a number) in the label, immediately succeeding the “[“, defines how many characters in the
document name this particular label should occupy.

There are two types of labels: string labels and integer labels. The difference between them is what happens
if the value of the label is shorter or longer than the stipulated number of characters. A string label will
add blanks at the beginning if too short and truncate the end if too long. An integer label will add zeros
at the beginning and truncate the beginning if too long.

Examples:
The label [2pagenr] will evaluate to “02” if the page number is 2.
The label [2year] will evaluate to “04” if the year is 2004.
The label [4productname] will evaluate to “  AB” if the product name is “AB” and “ABCD” if the product name is “ABCDEF”.

The valid string labels are: productname, edition, section, deadline, department and responsible.
The valid integer labels are: pagenr, year, month, day, and version.

#### Suffix
If “.*” ends the naming convention, the suffix of the document file is copied from the document template file IF its suffix is one of the following: “.indd”, “.qxd”, “.qrk”. If the template file has another suffix or doesn’t have any suffix, the document file name will not get any suffix. This is a feature which makes it possible to use both Quark and InDesign documents (even within one product).

#### Other characters
All other characters in the naming convention, which are not a part of a recognized label or “.*” at the end, are static and will be preserved in the document name. It is, of course, not allowed to use any character which is not allowed in a windows filename. They are: \ " / : * ? < > |

### Example
The following namingconvention:
```
[4year][2month][2day][2productname]page[3pagenr][3department]_[1version].*
```

would generate this document name:
```
20040112ABpage024SPO_1.indd
``

if the date was January 1st, 2004, the product name was “AB”, page number 24, department was SPORT,
version number 1 and the document template “Sportpage.indd”

#### Critical labels
There are two labels whose absence will disable the use of the associated functions. They are edition and version.
If they are omitted, the use of editions and versions are disabled.

Other labels that you will have problems without are pagenr, some form of date (unless you will have unique product
names every day) and productname (unless you only produce one product).


## `productnames`
This tag contains a list of all the product names that can be selected in the drop down list in the Product Setu
dialog in Product Manager.

## `editionnames`
This tag contains a list of all the edition names that can be selected in the drop down list in the Product Setup
dialog in Product Manager.

## `DTPtemplateDirectory`
Contains the path to the directory where the document templates are stored.

## `DTPdocumentsDirectory`
Contains the path to the directory where the desktop publishing documents (i.e. InDesign documents) are stored.

## `StatusCodes`
This tag contains info about the statuses a document can have in the system. The syntax is:
```
StatusName1!statuscolour1,StatusName2!statuscolour2,...
```
The color, which is used for the frames on the thumbnails in the web view, is represented in six digit
RGB code with two hex numbers per color (the normal HTML standard for representing colors.)

If you are running an old style Quark/InDesign extension they will read this info from the `Pagetrack status` file.
That file is overwritten each time a product is created with the info from the PageTrack configuration.

## `PDFGeneratorInbox`
Contains the path to the directory where the Product Updater places a copy for the PDF generator of each saved document. The PDF generator shall place the PDF:s in PageTrack\InfoFiles\Picts\Out\ directory for further distribution by the Product Updater.

## `PDFArchivePath`
Tells the path to the PDF archive root. If you have this option enabled, all PDFs generated by the PDF generator
are copied to this archive as well as into the PageTrack web site.

## `enableAllEditionsInWeb`
If this is enabled, the open document link of the pages that exist in an earlier edition are enabled and the
thumbnails are fully visible.

In other case, the thumbnails have a sign “Earlier edition” on them and it is not possible to open document
unless you’re viewing the first edition the document appears in.

## `MultipleProductsHTML`
If this is enabled, there are several versions of the “Products.html” file generated, one for each product name.

The products are inserted in the “global” Products.html and in a certain “XXProducts.html”, where XX is the product
name. This can be used to limit access for users to only one or a few product types.

## `DeleteOldProducts`
If present, this tag tells the Product Manager to launch the Delete Product menu after each Create Product action
and show all products that are older than the number of days the tag specifies.

## `ArchivePath`
Contains the path to the root of the Product Filing archive.
