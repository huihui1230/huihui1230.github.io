---
title: Java8 Stream操作
date: 2018-07-09 18:01:50
tags: 
- Java8
- Stream
categories:
- 后端  
- Java
---
## 需求
* 一个JSON数组，每个JSON对象都有一个key为id  
    
``` JavaScript
[
    {
        "material1": {
            "label": "素材一",
            "type": "file"
        },
        "materialPath1": {
            "label": "素材一路径",
            "type": "text"
        },
        "id": "3"
    },
    {
        "titlexi": {
            "label": "标题",
            "type": "text",
            "regExp": "",     
            "placeholder": "请输入一个标题",   
            "de-value": "默认标题",   
            "readonly": "true"     
        },
        "select": {
            "label": "类型",
            "type": "select",
            "value": [
                {
                    "key": "11519",
                    "value": "11519"
                },
                {
                    "key": "1",
                    "value": "一"
                }
            ],
            "de-value": "1"    
        },
        "id": "18"
    }
]
```    


* 一个JavaBean列表，每个对象都有属性id和属性name  



* 得到JSON数组中的每个id和id对应的name  
  
## for循环解决方案  


## Stream解决方案  

原本需要返回的数据格式是`List<Map<String, String>>`，即`map.put("id", id); map.put("name", name)`。在Java8具体操作时，遇到了点问题。  

 

第一次map之后返回的流是整型值，第二次得到的流不再是template对象的流，无法获取name。  
  
只好将返回的数据格式改为`List<Map<Integer, String>>`，即`map.put(id, name)`。    

