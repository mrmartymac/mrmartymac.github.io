---
title: Google Analytics
date: 2023-02-20 15:32:13 -500
categories: [Cheatsheet, Google Analytics]
tags: [google analytics, ga] # TAG names should always be lowercase
author: mm
---
# Setting up Google Analytics in WebEdit and eLibrary

## eLibrary

### New file
Add a file called `gtag.js` to `/opt/elibrary/web` contents are

```js
<!-- Google tag (gtag.js) replace ########## with actual G-tag-->
<script async src="https://www.googletagmanager.com/gtag/js?id=G-############"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());

  gtag('config', 'G-##########');
</script>
```

### Changes to index.php
In `index.php` add the line just after the opening `<?php` making the addition the second line.

```shell
include_once('gtag.js'); 
```

The result should look like
```shell
<?php
include_once('gtag.js');
require_once('db.php');
```

In the <head> section of index.php locate the `Content-Security-Policy` to make changes.  The conetn may vary, but the goal is to locate or add the `connect-src` argument. Ensuring the argument `connect-src 'self' analytics.google.com;` is present is the key.  The URL may vary.  Make sure the URL you are adding matches the URL specified in the `gtag.js` file.

#### Example before
```js
<meta http-equiv="Content-Security-Policy" content="default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'; img-src 'self' data:;">
with
```
#### Modified Example
```js
<meta http-equiv="Content-Security-Policy" content="default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline'; img-src 'self' data:; connect-src 'self' analytics.google.com;">
```