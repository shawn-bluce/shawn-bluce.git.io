---
title: Python命令行日历 获取时间 格式化输出 
date: 2015-07-07 02:08
tags:
  - Python
  - Demo
---

实现简单日历
>读取当前时间并显示本月日历
>输出格式化

```python
#!/usr/bin/python2

#It's my first Python program

import time

#get date 获取当前日期
year  = time.strftime('%Y', time.localtime(time.time()))
year = (int)(year)
month = time.strftime('%m', time.localtime(time.time()))
day   = time.strftime('%d', time.localtime(time.time()))
day = (int)(day)
week  = time.strftime('%w', time.localtime(time.time()))

week = (int)(week)
for i in range(0, day - 1):
    week = week - 1
    if week == -1:
        week = 6

#judge leap 判断是否是闰年
if year % 4 == 0 and year % 400 != 0 or year % 400 == 0 :
    isLeap = True
else :
    isLeap = False

#all Day这个月一共有多少天
if month == '01' or month == '03' or month == '05' or month == '07' or month == '08' or month == '10' or month == '12' :
    allDay = 31
elif month == '02' and isLeap :
    allDay = 29
elif month == '02' and not isLeap :
    allDay = 28
else :
    allDay = 30

#print CAL  输出这个月的日历
print '            ' + str(year) + '   ' + str(month)
print ''
print 'Sun  Mon  Tue  Wed  Thu  Fri  Sat'

for i in range(week) :   #print space   输出空白部分
    print '    ',

for i in range(1, allDay + 1) :   #print everday  输出日期
    x = str(i)
    print '%-4s' % x,
    week = week + 1
    if week == 7 :
        week = 0
        print

raw_input()
```
