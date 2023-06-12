---
title: Google Pages - Google Analytics
date: 2023-06-12 10:17:20 -400
categories: [FoundInfo, Jekyll]
tags: [jekyll, ga, google, analytics] # TAG names should always be lowercase
author: https://morotsman.github.io/blog,/google/analytics,/jekyll,/github/pages/2020/07/07/add-google-analytics.html
---

So I decided to add Google analytics to my blog and now I can blog about it, so very meta of me :-)

The blog is hosted by Github Pages using Jekyll by the way.

## Get Google Analytics account

### Get the account

The first thing you have to do is to get a Google analytics account: [Google analytics](https://analytics.google.com/). This is pretty self explaining (+ I forgot to take a screenshot) so I will not elaborate on this.

### Copy some stuff you will need to setup Jekyll

After you have create an account you will receive a Tracking Id that looks like this: UA-XXXXXXXXX-X. If you for some reason forget the Tracking Id it is available at: Admin/Tracking info/Tracking code

You will also need the code snippet in Global Sit Tag:

```
<script async src="https://www.googletagmanager.com/gtag/js?id=UA-XXXXXXXXX-X"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());

  gtag('config', 'UA-XXXXXXXXX-X');
</script>
```

If you copy paste from this post, please remember to replace UA-XXXXXXXXX-X (two places) with the Tracking id for your site.

![Tracking code](https://morotsman.github.io/assets/adding_ga/Tracking_code.png)

## Setup Jekyll

### Update \_config.yml

Add the following row:

```
google_analytics: UA-XXXXXXXXX-X
```

![Tracking code](https://morotsman.github.io/assets/adding_ga/config.png)

You now know the drill about replacing UA-XXXXXXXXX-X with the Tracking Id from Google analytics, I will not mention it again (but you will have to do that in the next step as well).

## Add google-analytics.html

Create a new file called google-analytics.html in the \_includes directory (I also had to create the directory)

![Tracking code](https://morotsman.github.io/assets/adding_ga/Directory.png)

And then add the code snippet from Google Analytics like this:

```
<script async src="https://www.googletagmanager.com/gtag/js?id=UA-XXXXXXXXX-X"></script>
<script>
window.dataLayer = window.dataLayer || [];
function gtag(){dataLayer.push(arguments);}
gtag('js', new Date());

gtag('config', 'UA-XXXXXXXXX-X');
</script>
```

And that’s it, the setup is completed!

## Head back to Google analytics

Time to celebrate and observe that the blog has zero visitors, but hey: it’s in realtime!

![Tracking code](https://morotsman.github.io/assets/adding_ga/visitors.png)

## Conclusion

Ok, so now that you have added google analytics to your blog, why not stay for a while, relax and maybe read a couple of blog posts about [covariance](https://morotsman.github.io/java,/covariance,/the/liskov/substitution/principle/2020/07/12/java-covariance.html), [contravariance](https://morotsman.github.io/java/contravariance/the/liskov/substitution/principle/2020/07/17/java-contravariance.html) or even [functional bliss](https://morotsman.github.io/scala/finagle/finch/2021/03/28/finagle-finch.html)? There will be quotes from Lord of the rings, funny gif’s and much more.

![Alt Text](https://media.giphy.com/media/vsW15HzddQRZ2KiJ5j/giphy.gif)

## Github

The source code for my blog is available at [github](https://github.com/morotsman/morotsman.github.io).

___