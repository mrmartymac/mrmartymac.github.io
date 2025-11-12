---
title: Rebuild SCS/Track lucene indexes
date: 2025-11-12 14:16:25 -400
categories: [Cheatsheets, SCS/Track]
tags: [track, indexes, lucene] # TAG names should always be lowercase
author: mm
---

Check the directory below, if it is empty the indexes for Track need to be rebuilt.
```
/u/track/index/adinquiry/index
```

To rebuild the Track indexes as a background process run:
```
/u/track/script/track_lucene.sh adinquiry --rebuild-type clean &
```

> To check on a backgrounded process you can run the `fg` command.  
{: .prompt-info }

To view the log for the building on the index.
```
cd /u/track/debug/
tail -f track lucene.1og
```

To check the status of the index build run:
```
ll /u/track/index/adinquiry/
```

> The result of the listing should show you the directories **index**, **scratch**, and **rebuild**
{: .prompt-info }

