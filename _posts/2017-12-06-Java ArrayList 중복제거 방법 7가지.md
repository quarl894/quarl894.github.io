---
layout: post
# title:  "Java ArrayList 중복 제거 방법 7가지"
date:   2017-12-06 21:20:00 +0900
categories: Java
---

- dataList는 생성되어 있음(생략).


# 1번 Logic

```java
   resultList = new ArrayList<String>();
            for (int i = 0; i < dataList.size(); i++) {
                if (!resultList.contains(dataList.get(i))) {
                    resultList.add(dataList.get(i));
                }
            }
```

# 2번 HashSet

 ```java
 HashSet<String> distinctData = new HashSet<String>(dataList);
            dataList = new ArrayList<String>(distinctData);
```

# 3번 TreeSet

 ```java
TreeSet<String> distinctData = new TreeSet<String>(dataList);
            dataList = new ArrayList<String>(distinctData);
```

# 4번 Google guava

 ```java
 resultList = Lists.newArrayList(Sets.newHashSet(dataList));

```

# 5번 Lambdas

 ```java
  resultList = dataList.parallelStream().distinct().collect(Collectors.toList());

```

# 6번 LinkedHashSet

 ```java
temp = new LinkedHashSet<String>();
            temp.addAll(dataList);
            dataList.clear();
            dataList.addAll(temp);

```

# 7번 Logic2

 ```java
 for (int i = 0; i < dataList.size(); i++) {
                for (int j = 0; j < dataList.size(); j++) {
                    if (i == j) {
                    } else if (dataList.get(j).equals(dataList.get(i))) {
                        dataList.remove(j);
                    }
                }
            }
```

## 속도 비교
### 1번 > 7번 > HashSet > LinkedHashSet > TreeSet > Google guava > Lambdas

참조

[ArrayList 중복제거](http://selfinvestfriends.tistory.com/22)

[jekyll-gh]:   https://github.com/quarl894
