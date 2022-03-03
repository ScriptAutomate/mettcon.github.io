---
id: bxwfcvmffqjgfwebc0h768i
title: SharePoint
desc: ''
updated: 1646219074057
created: 1646216566525
---

## Metrics to observe
### Cache
#### Object Cache
- [ ]   taken from SharePoint 2013. Check Relevance for other versions
- Total Number of Cache Compactions  
ideally 0. 6 or more per hour means Object Cache is under stress. Allocate more memory

#### Page Output Cache
[Caching In for SharePoint Performance - BASPUG 2016](https://youtu.be/zkRB1D-8V-0?t=6325)  
- [ ] taken from SharePoint 2013. Check Relevance for other versions
- Memory Consumption ASP .NET Worker Process  
Page Output Cache is probably too granular
