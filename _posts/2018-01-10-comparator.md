---
layout: post
# title:  "Java Comparable vs Comparator"
date:   2018-01-10 22:20:00 +0900
categories: Java
---
# 정렬알고리즘
 정렬 알고리즘을 풀 때 Collections.sort 메소드를 많이 쓴다.
 이 메서드는 퀵 정렬을 기반으로 정렬해주므로 정렬시 아주 유용하게 사용한다.
 하지만 Collections.sort만으로 해결할 수 없는 정렬들이 있다.
 ex) 2개의 원소가 쌍으로 되어있어 차레롤 정렬해야 하는 경우. 정렬 기준이 2가지 이상일 경우

 이런 특수한 경우에 사용하는 메서드가 Comparator다.
 
 **Comparator - 기본 정렬기준 외에 다른 기준으로 정렬하고자할 때 사용. **

하나의 예쩨로 백준 11650번을 할 때 사용했다.
x,y 한쌍으로 구성된 좌표들이 주어진다.
먼저 x값을 오름차순으로 정렬 후 같은 값이 있을 경우 y값의 오름차순으로 정렬한다.

2가지 이상의 기준이므로 Comparator을 사용하여 기준 정의를 해야한다.

``` java
		Collections.sort(num, new Comparator<XY>(){
			@Override
			public int compare(XY o1, XY o2){
				if(o1.x>o2.x){
					return 1;
				}else if(o1.x<o2.x){
					return -1;
				}
				else{
					if ( o1.y > o2.y ){
						return 1;
	                }
	                else if (o1.y < o2.y ){
	                    return -1;
	                }
	                else {
	                    return 0;
	                }
				}
			}
		});
```

[jekyll-gh]:   https://github.com/quarl894