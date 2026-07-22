---
title: Count articles by year in eLibrary
date: 2026-07-22 12:41:12 -400
categories: [Cheatsheets, eLibrary]
tags: [sql, query, count] # TAG names should always be lowercase
author: mm
---

```bash
SELECT
    EXTRACT(YEAR FROM date)::integer AS year,
    COUNT(*) AS record_count
FROM article_texts
WHERE date IS NOT NULL
GROUP BY EXTRACT(YEAR FROM date)
ORDER BY year;
```