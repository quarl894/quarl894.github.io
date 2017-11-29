---
layout: post
# title:  "Android Thread 사용방법"
date:   2017-10-27 22:41:58 +0900
categories: android
---
이번에 설명할 내용은 Thread이다.
Thread는 개발을 하면서 누구나 한번쯤은 듣고 써봤을 기능이다.

간략하게 Thread란 무엇인가?
쉽게 말해 멀티 작업을 하기 위한 기능 이라고 생각 하면 된다.
예를 들자면 메시지를 보내면서 음악을 들을 수 있는 기능이라고 생각 하면 이해 하기가 편하다.
스레드의 수에 따라 싱글스레드와 멀티스레드로 나눌 수 있다.

스레드 활용의 대표적인 활용은 밑에와 같다
* 타이머
* 프로그레스바
* 알람
* 다운로드 로딩

이번에 내가 적용한 것은 프로그레스바다.

먼저 Android에서의 Thread 특징이 몇가지 있다.

1. 일회성이다.(재사용이 되지 않는다)
	-> 타이머 객체의 메소드 중 Task를 비워주는 것도 있지만 비워준다해도 재사용이 되지 않는다.

2. UI 업데이트는 Main Thread에서만 가능하다.
3. 만약 다른 Thread에서 UI를 업데이트를 해야한다면 Handler를 이용해서 Main Thread와 통신해야한다.


나는 첫번째 특징을 까먹고 멍청하게 oncreate에 THread 객체를 생성해주면서 재사용을 하려고 했다..(시간 엄청 잡아먹음..)

일회성이므로 스레드를 할 때마다 객체를 새로 생성해줘야 한다.
나는 클릭 이벤트가 있으면 프로그레스바를 실행, 일시정지, 초기화를 해야하므로 onclick안에 생성했다.

그리고 Thread에서 onStop(), onResume(), onWait() 이런거 쓰지말자...
효율성과 충돌해서 안될뿐더러 이걸 이용하려고 하다가 스트레스 엄청 받는다..

일시정지를 하기 위해서는 sleep()을 주는 방법이 가장 편하다.

++** stop, work 는 일시정지 및 초기화를 위해 설정해 줘야 한다.(정지 기능이 없다면 필요없다)**++
```java
private class P_Thread extends Thread{
        private int progressStatus = 0;
        public boolean stop = false;
        public boolean work = true;
        public void run() {
            while(progressStatus <60 && !Thread.currentThread().isInterrupted()) {
                if (work) {
                    try {
                        progressStatus++;
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    // Update the progress bar
                    handler3.post(new Runnable() {
                        public void run() {
                            p_gradient.setProgress(progressStatus);
                            start_time.setText("00:" + String.format("%02d", progressStatus));
                            end_time.setText("00:" + String.format("%02d", 60 - progressStatus));
                        }
                    });
                } else {
                    if (Thread.currentThread().getState().equals(State.RUNNABLE)) {
                        try {
                            Thread.sleep(800);
                        } catch (Exception e) {
                        }
                    }
                    if(stop){
                        Log.e("stop", " " + Thread.currentThread().getState());
                        progressStatus =0;
                        handler3.post(new Runnable() {
                            public void run() {
                                p_gradient.setProgress(progressStatus);
                                start_time.setText("00:" + String.format("%02d", progressStatus));
                                end_time.setText("00:" + String.format("%02d", 60 - progressStatus));
                            }
                        });
                        break;
                    }
                }
            }
            Log.e("status", "end");
            progressStatus = 0;
            stop = false;
            work = true;
        }
    }
```

추가로 해야할 것들.
* 파일 저장 위치 변경 및 복사
* 퍼미션 확인하기

해당 코드는 <https://github.com/quarl894/Yapp_Dev> 이 프로젝트에 있다.(MainActivity)

[jekyll-gh]:   https://github.com/quarl894
