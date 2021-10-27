---
title: grok-PasteBin
date: 2021-10-26 09:29:35
tags:
---

# What is Paste Bin - requirement

- customer enter a text on webpage
- generate url for the text
- expire time
- title / cutome uRL

## additional
- analytics 
- user login
- public private setting
- password crypted
- grammer high light

# What is the throughput - capacity & volumn calcute

- text size limit - 10MB
- url size
- write volumn - 1M/DAY = 12/s
- access & read volumn = 60/s

band width
- WRITE = 12 * 10 = 120 KB / S
- READ = 600 KB /S

Storage

assume average 10K
- total : 10KB * 1M  = 10 GB/day = 36TB/10year
- key for each data: 1M * 365 * 10 = 36B < 64^6
- key size: 6bytes * 36B = 22GB 

Cache 20% of read
- 5M read * 0.2 * 10KB = 10GB

# API design - function design

1. basic create url on text
    - input (api_dev_key,text) optional(title, custom url, id, expiredate)
    - output [url]
2. get paste
3. delete paste

# DB design

key | text | title | expire date | user id | url

seperate user

key | title | create date | expire date

userid | email | create date

S3 store object data

# Component design

application server
- get text
- generate key
- store
- return url

possible error - duplicate key - regenerate

key generate service - pre generate key & keep track used key

# DB detail

- partition & replica
- purge DB clean up
  - handle expire 
  - expire program
- Cache
  - 20%
  - chose what to cache - LRU
  - update

# Security and Permissions

