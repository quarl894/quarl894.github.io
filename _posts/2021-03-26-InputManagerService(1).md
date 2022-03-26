---
layout: post
# title:  "InputManagerService"
date:   2021-03-26 15:20:00 +0900
categories: Input, Android, Framework
tag: Android, Input
---

### [개요]

InpuFW대해서 전반적인 Input event가 어떻게 동작하는지에 대해 정리하고자 합니다.

## InputMangerService Servie 등록부터 시작

EventHub에서 event를 받게 되면 InputReader로 전달되게 됩니다.
이 때, InputReader, InputDispatcher thread를 깨우는 과정이 시작됩니다.

* Android가 시작되면 SystemServer에서는 각종 Service들이 시작됩니다.
IMS 또한 SystemServer에서 IMS를 등록하는 것을 알 수 있습니다.

```java
// SystemServer.java
private void startOtherServices(@NonNull TimingsTraceAndSlog t) {
  inputManager = new InputManagerService(context);

 ServiceManager.addService(Context.WINDOW_SERVICE, wm, /* allowIsolated= */ false,
     DUMP_FLAG_PRIORITY_CRITICAL | DUMP_FLAG_PROTO);
 ServiceManager.addService(Context.INPUT_SERVICE, inputManager,
     /* allowIsolated= */ false, DUMP_FLAG_PRIORITY_CRITICAL);
 // Start InputManager
 inputManager.setWindowManagerCallbacks(wm.getInputManagerCallback());
 inputManager.start();
}
```
* IMS.java에서 호출을 받으면, JNI를 통하여 native를 호출한다.
    추가로 Watchdog monitor를 추가시키는 것을 볼 수 있다.

```java
// InputManagerService.java
public void start() {
        Slog.i(TAG, "Starting input manager");
        nativeStart(mPtr);

        // Add ourself to the Watchdog monitors.
        Watchdog.getInstance().addMonitor(this);
```
* JNI단에서는 InputManager를 호출한다.

```java
// com_android_server_input_InputManagerService.cpp
static void nativeStart(JNIEnv* env, jclass /* clazz */, jlong ptr) {
    NativeInputManager* im = reinterpret_cast<NativeInputManager*>(ptr);

    status_t result = im->getInputManager()->start();
    if (result) {
        jniThrowRuntimeException(env, "Input manager could not be started.");
    }
}
```
* 이미 thread가 있으면 종료하고 없으면 InputDispatcher,Reader thread를 생성한다.

```java
status_t InputManager::start() {
    status_t result = mDispatcher->start();
    if (result) {
        ALOGE("Could not start InputDispatcher thread due to error %d.", result);
        return result;
    }

    result = mReader->start();
    if (result) {
        ALOGE("Could not start InputReader due to error %d.", result);

        mDispatcher->stop();
        return result;
    }

    return OK;
}
```
* Looper->wake, EventHub->wake 시키면서 Thread가 정상적으로 동작할 수 있게 셋팅 끝.

```java
// InputDispatcher.cpp
status_t InputDispatcher::start() {
    if (mThread) {
        return ALREADY_EXISTS;
    }
    mThread = std::make_unique<InputThread>(
            "InputDispatcher", [this]() { dispatchOnce(); }, [this]() { mLooper->wake(); });
    return OK;
}
// InputReader.cpp
status_t InputReader::start() {
    if (mThread) {
        return ALREADY_EXISTS;
    }
    mThread = std::make_unique<InputThread>(
            "InputReader", [this]() { loopOnce(); }, [this]() { mEventHub->wake(); });
    return OK;
}
```
* 이후 InputReader에서 Event 처리 시작.

```java
// InputReader.cpp
void InputReader::loopOnce() {
    size_t count = mEventHub->getEvents(timeoutMillis, mEventBuffer, EVENT_BUFFER_SIZE);

    if (count) {
         processEventsLocked(mEventBuffer, count);
    }
```



