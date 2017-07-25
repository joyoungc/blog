---
layout: post
title:  "Map 안의 내용을 뒤지기 위한 반복 유형"
date:   2014-04-10 15:07:00
categories: java
tags: java map
author: Joyoungc
---

## Iterator, for, while, forEach

```java
public static void main(String[] args) {

        Map<string, string> map = new HashMap<string, string>();
         
        map.put("key1", "value1");
        map.put("key2", "value2");
        map.put("key3", "value3");
        map.put("key4", "value4");

        // Pattern 1 : entrySet을 이용한  Enhanced For-Loops
        for( Map.Entry<string, string> elem : map.entrySet() ){
            System.out.println( "key : " + elem.getKey() + ", value : " +  elem.getValue()) );
        }

        // Pattern 2 : keySet을 이용한  Enhanced For-Loops
        for( String key : map.keySet() ){
            System.out.println( "key : " + key + ", value : " +  map.get(key)) );
        }
         
        // Pattern 3 : Iterator를 이용한  While-Loops
        Iterator<string> keys = map.keySet().iterator();
        while( keys.hasNext() ){
            String key = keys.next();
            System.out.println( "key : " + key + ", value : " +  map.get(key)) );
        }

        // Pattern 4 : Java 8 forEach를 이용 (2017-07-25 추가)
        map.forEach((k,v)->System.out.println("key : " + k + " , value : " + v) );

}
```
