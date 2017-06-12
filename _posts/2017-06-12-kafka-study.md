---
layout: post
title: kafka数据可靠性的深入研究
categories: 分布式
description: kafka
keywords: Kafka
---

## Kafka简介

**前世今生：**Kakfa起初是由LinkedIn公司开发的一个分布式的消息系统，后成为Apache的一部分，它使用Scala编写，以可水平扩展和高吞吐率而被广泛使用。目前越来越多的开源分布式处理系统如Cloudera、Apache Storm、Spark等都支持与Kafka集成。

Kafka凭借着自身的优势，越来越受到互联网企业的青睐，唯品会也采用Kafka作为其内部核心消息引擎之一。Kafka作为一个商业级消息中间件，消息可靠性的重要性可想而知。如何确保消息的精确传输？如何确保消息的准确存储？如何确保消息的正确消费？这些都是需要考虑的问题。本文首先从Kafka的架构着手，先了解下Kafka的基本原理，然后通过对kakfa的存储机制、复制原理、同步原理、可靠性和持久性保证等等一步步对其可靠性进行分析，最后通过benchmark来增强对Kafka高可靠性的认知。。

![](/images/kafka.png)
```vb
Sub 分离()
    Application.ScreenUpdating = False
    
    p = ThisWorkbook.Path & "/"
    f = p & "空白模板.doc"
    
    Dim myWS As Worksheet
    Set myWS = ThisWorkbook.Sheets(1) '存有数据的表格
    
    For i = 3 To 54    '遍历数据行
        FileCopy f, p & "test/" & myWS.Cells(i, 2).Text & ".doc"    '复制空模板并以某列数据为名命名新产生的文档
        Set wd = CreateObject("word.application")
        Set d = wd.documents.Open(p & "test/" & myWS.Cells(i, 2).Text & ".doc") '打开新文档
        
        d.tables(1).Cell(1, 2) = myWS.Cells(i, 2).Text '###
        '复制表格每列内容到文档，有多少项就有多少条
        d.tables(1).Cell(5, 4) = myWS.Cells(i, 20).Text '###
        
        d.Close
        wd.Quit
        Set wd = Nothing
    Next
    
    Application.ScreenUpdating = True
End Sub
```
