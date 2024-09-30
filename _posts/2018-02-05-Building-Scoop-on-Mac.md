---
title: Building Scoop on Mac
date: 2023-08-23 07:45:10 -400
categories: [Documentation, Builds]
tags: [build, qt, sign] # TAG names should always be lowercase
author: mm
---

This is all a single command.  Complete a build of Scoop
```
/Users/martinmacdonald/Qt.old/5.15.2/clang_64/bin/macdeployqt /Users/martinmacdonald/Documents/GitHub/<NameOfBuild>/Launcher/Scoop.app -verbose=2 -codesign="Software Consulting Services" -dmg
```

```