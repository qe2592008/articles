NavigableMap继承自SortedMap，在拥有排序功能的基础上提供了一些导航性的方法。

方法罗列：
    
    返回大于（或大于等于）给定key的最小Entry
    返回小于（或小于等于）给定key的最大Entry
                   返回大于（或大于等于）给定key的最小Key
                   返回小于（或小于等于）给定key的最大Key
                   返回（并删除）首个（末个）Entry
                   返回当前集合的逆序集合
                   返回当前集合的逆序的keySet
                   返回各种子Map
                   
               TreeMap实现了这个接口。