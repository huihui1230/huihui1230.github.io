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
## 需求 ##
* 一个JSON数组，每个JSON对象都有一个key为id  
    
```json
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

```java
public class AdTemplateDO {
    private Integer id;

    private String templateName;

    private Byte deleted;

    private Date gmtCreate;

    private Date gmtModified;

    private String templateJson;

    ...
}
```

* 得到JSON数组中的每个id和id对应的name  
  
## for循环解决方案 ##
```java
public List<Map<String, String>> getTemplateByOrderItemId(Integer orderItemId) {
    String slotJsonStr = this.getSlotJsonByOrderItemId(orderItemId);
    JSONArray slotJson = JSONArray.fromObject(slotJsonStr);
    List<AdTemplateDO> adTemplateDOList = adTemplateService.selectDOList();
    List<Map<String, String>> templateMapList = new ArrayList<>();

    for (Object templateObject : slotJson) {
        JSONObject templateJson = (JSONObject) templateObject;
        
        for (AdTemplateDO adtemplateDO : adTemplateDOList) {
			if (templateJson.get("id").toString().equals(adTemplateDO.getId().toString())) {
				Map<Integer, String> templateMap = new HashedMap();
				templateMap.put("id", adTemplateDO.getId());
				templateMap.put("name", adTemplateDO.getName());
				templateMapList.add(templateMap);
			}
		}
    }

    return templateMapList;
}  
```

## Stream解决方案 ##

原本需要返回的数据格式是`List<Map<String, String>>`，即`map.put("id", id); map.put("name", name)`。在Java8具体操作时，遇到了点问题。  
```java
public List<Map<String, String>> getTemplateByOrderItemId(Integer orderItemId) {
    String slotJsonStr = this.getSlotJsonByOrderItemId(orderItemId);
    JSONArray slotJson = JSONArray.fromObject(slotJsonStr);
    List<AdTemplateDO> adTemplateDOList = adTemplateService.selectDOList();
    List<Map<String, String>> templateMapList = new ArrayList<>();

    for (Object templateObject : slotJson) {
            JSONObject templateJson = (JSONObject) templateObject;
            Map<Integer, String> templateMap = new HashedMap();

            //找到对应模板，将id和name放入templateMap中
            adTemplateDOList.stream().filter(adTemplateDO -> adTemplateDO.getId().toString().equals(templateJson.get("id").toString())).findFirst()
                    .map(template -> templateMap.put("id", template.getId()));
			//.map(template -> templateMap.put("id", template.getId())).map(template -> templateMap.put("name", template.getTemplateName()))

            templateMapList.add(templateMap);
        }

    return templateMapList;
}  
```  

第一次map之后返回的流是整型值，第二次得到的流不再是template对象的流，无法获取name。  
  
只好将返回的数据格式改为`List<Map<Integer, String>>`，即`map.put(id, name)`。    

```java
public List<Map<Integer, String>> getTemplateByOrderItemId(Integer orderItemId) {
    String slotJsonStr = this.getSlotJsonByOrderItemId(orderItemId);
    JSONArray slotJson = JSONArray.fromObject(slotJsonStr);
    List<AdTemplateDO> adTemplateDOList = adTemplateService.selectDOList();
    List<Map<Integer, String>> templateMapList = new ArrayList<>();

    for (Object templateObject : slotJson) {
        JSONObject templateJson = (JSONObject) templateObject;
        Map<Integer, String> templateMap = new HashedMap();

        //找到对应模板，将id和name放入templateMap中
        adTemplateDOList.stream().filter(adTemplateDO -> adTemplateDO.getId().toString().equals(templateJson.get("id").toString())).findFirst()
                .map(template -> templateMap.put(template.getId(), template.getTemplateName()));

        templateMapList.add(templateMap);
    }

    return templateMapList;
}    
```  