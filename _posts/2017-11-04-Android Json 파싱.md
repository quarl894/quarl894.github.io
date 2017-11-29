---
layout: post
# title:  "Android Json 파싱"
date:   2017-11-04 00:41:58 +0900
categories: android
---
Android에서 공공기관이나 여러 기관에서 공개된 API를 사용할때 파싱을 해야합니다.
파싱에 대표적인 것으로 XML과 JSON이 있는데요. 
오늘은 JSON 파싱에 대해서 알려드리겠습니다.
기본적으로 JSON은 []대괄호와 {}중괄호로 이루어져 있습니다.

```json
 {"DATA": [
  {
   "title": "서울 강남구 헌릉로757길 27",
   "lat": 37.4696880,
   "lon": 127.1200550,
   "topic": "가야자동차정비공업사(주)",
   "tel": "02-3414-1710"
  }]}
```
이런식으로 JSON이 이루어져있습니다.
**[]대괄호는 JSONArray 를 이용하여 구별하고 {}중괄호는 JSONObject 를 이용하여 구별합니다.**

제가 2년전에 했던건 카센터 위치를 JSON으로 받았던건데요.
카센터 위치는 API가 없어서 직접 크롤링을 통해 JSON파일을 만들어서 파싱했습니다.

저는 raw 파일로 파싱을 한거라 인터넷에서 파싱을 하려면 URL을 통해 데이터를 가져오신 다음 따라오시면 됩니다.

```java
        private void readItems() throws JSONException {
                try{
                        InputStream is= getResources().openRawResource(R.raw.great_hospital);
                        int size= is.available();
                        byte[] buffer=new byte[size];
                        is.read(buffer);
                        is.close();
                        String result=new String(buffer,"utf-8");

                        JSONObject json =new JSONObject(result);
                        JSONArray jarr=json.getJSONArray("DATA");

                        for(int i=0; i<jarr.length();i++){
                                json=jarr.getJSONObject(i);
                                double name=json.getDouble("LAT");
                                double address=json.getDouble("LNG");
                                String title = json.getString("HOSP_NM");
                                String topic = json.getString("topic");
                                String tel = json.getString("tel");
                                items.add(new MyItem(name, address, title, topic, tel));
                                mClusterManager.addItem(new MyItem(name, address, title,topic, tel));
                        }
                } catch (Exception e) {
                        e.printStackTrace();
                }
        }
```

JSON 파싱시에는 try catch를 반드시 해줘야합니다.
JSONArray로 []로 묶어놓은 DATA를 구별한뒤 JSONObject로 {}안에 있는 값들을 나누는 작업을 합니다.
그 다음으로 Array에 집어넣어 알맞게 활용하시면 됩니다.
[jekyll-gh]:   https://github.com/quarl894
