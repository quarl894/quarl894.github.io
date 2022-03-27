---
layout: post
# title:  "InputReader(2)"
date:   2021-03-27 14:20:00 +0900
categories: Input, Android, Framework
tag: Android, Input
---

### [개요]

InputReader에서 event 가공/전달하는 법을 알아보겠습니다.

==주요 동작은 EventHub에서 올라온 Linux event(RawEvent)를 디코딩하고 Android event code로 변환하는 과정입니다.==(입력 기기 구성, 키보드 레이아웃 파일 등...)

## InputReader 동작

1. 앞서 InputReader Thread를 동작시키면 loopOnce()부터 시작된다.
   ProcessEventsLocked으로 가공할 event 전달.

```java
void InputReader::loopOnce() {
    size_t count = mEventHub->getEvents(timeoutMillis, mEventBuffer, EVENT_BUFFER_SIZE);

    { // acquire lock
        std::scoped_lock _l(mLock);
        mReaderIsAliveCondition.notify_all();

        if (count) {
            processEventsLocked(mEventBuffer, count);
        }
    } // release lock

    // Send out a message that the describes the changed input devices.
    if (inputDevicesChanged) {
        mPolicy->notifyInputDevicesChanged(inputDevices);
    }

    mQueuedListener->flush();
}

```
2. add,remove,scan이 아닌 event들은 InputDevice로 전달
	맞다면 해당 동작을 실행한다.

```java
void InputReader::processEventsLocked(const RawEvent* rawEvents, size_t count) {
    for (const RawEvent* rawEvent = rawEvents; count;) {
		...
        if (type < EventHubInterface::FIRST_SYNTHETIC_EVENT) {
            processEventsForDeviceLocked(deviceId, rawEvent, batchSize);
        } else {
            switch (rawEvent->type) {
                case EventHubInterface::DEVICE_ADDED:
                    addDeviceLocked(rawEvent->when, rawEvent->deviceId);
                    break;
                case EventHubInterface::DEVICE_REMOVED:
                    removeDeviceLocked(rawEvent->when, rawEvent->deviceId);
                    break;
                case EventHubInterface::FINISHED_DEVICE_SCAN:
                    handleConfigurationChangedLocked(rawEvent->when);
                    break;
                default:
                    ALOG_ASSERT(false); // can't happen
                    break;
            }
        }
        count -= batchSize;
        rawEvent += batchSize;
    }
}

void InputReader::processEventsForDeviceLocked(int32_t eventHubId, const RawEvent* rawEvents, size_t count) {
 device->process(rawEvents, count);
}
```
1. Device에 add된 Mapper에게 전달 후 각 Mapper에서 Android event로 가공
   (Notify 통해 InputDispatcher 전달)
Mapper는 addDevice할 때 addDeviceLocked()-> CreateDevice() -> addHubDevice 에서 Mapping.

```java
void InputDevice::process(const RawEvent* rawEvents, size_t count) {
    for (const RawEvent* rawEvent = rawEvents; count != 0; rawEvent++) {
        if (mDropUntilNextSync) {
        } else {
            for_each_mapper_in_subdevice(rawEvent->deviceId, [rawEvent](InputMapper& mapper) {
                mapper.process(rawEvent);
            });
        }
}

// ex) Send down. #TouchInputMapper.cpp
NotifyMotionArgs args(getContext()->getNextId(), when, readTime, getDeviceId(), mSource,
displayId, policyFlags, AMOTION_EVENT_ACTION_DOWN, 0, 0,metaState, mCurrentRawState.buttonState, MotionClassification::NONE, AMOTION_EVENT_EDGE_FLAG_NONE, 1,
&mPointerSimple.currentProperties, &mPointerSimple.currentCoords, mOrientedXPrecision, mOrientedYPrecision, xCursorPosition, yCursorPosition, mPointerSimple.downTime, /* videoFrames */ {});

getListener()->notifyMotion(&args);
```
1. mArgsQueue에 event를 넣고, NotifyKeyArgs를 InputListenerInterface에 알립니다.
InputReader::loopOnce mQueuedListener->flush()의 마지막 호출은 mArgsQueue를 전달하게 됩니다.

InputDispatcher는 InputDispatcherInterface를 상속하고
InputDispatcherInterface는 InputListenerInterface를 상속하므로
InputDispatcher의 notifyMotion을 호출하고 NotifyMotionArgs를 InputDispatcher에 전달하여 처리됩니다.

```java
void QueuedInputListener::notifyMotion(const NotifyMotionArgs* args) {
    traceEvent(__func__, args->id);
    mArgsQueue.push_back(new NotifyMotionArgs(*args));
}

void QueuedInputListener::flush() {
    size_t count = mArgsQueue.size();
    for (size_t i = 0; i < count; i++) {
        NotifyArgs* args = mArgsQueue[i];
        args->notify(mInnerListener);
        delete args;
    }
    mArgsQueue.clear();
}

struct NotifyArgs {
    int32_t id;
    nsecs_t eventTime;
    inline NotifyArgs() : id(0), eventTime(0) {}
    inline explicit NotifyArgs(int32_t id, nsecs_t eventTime) : id(id), eventTime(eventTime) {}
    virtual ~NotifyArgs() { }
    virtual void notify(const sp<InputListenerInterface>& listener) const = 0;
};
```
1. BeforeQueueing을 동작하고 inputfilter check 후 InboundQueue에 넣고
   InputDispatcher Thread를 깨우면서 시작.

```java
void InputDispatcher::notifyMotion(const NotifyMotionArgs* args) {
    uint32_t policyFlags = args->policyFlags;
    policyFlags |= POLICY_FLAG_TRUSTED;

    mPolicy->interceptMotionBeforeQueueing(args->displayId, args->eventTime, /*byref*/ policyFlags);

  { // acquire lock
        mLock.lock();

        if (shouldSendMotionToInputFilterLocked(args)) {
         	event.initialize(...);
             policyFlags |= POLICY_FLAG_FILTERED;
            if (!mPolicy->filterInputEvent(&event, policyFlags)) {
                return; // event was consumed by the filter
            }
        }
        // Just enqueue a new motion event.
        std::unique_ptr<MotionEntry> newEntry = std::make_unique<MotionEntry>(...);
        needWake = enqueueInboundEventLocked(std::move(newEntry));
        mLock.unlock();
    } // release lock

    if (needWake) {
        mLooper->wake();
    }
```



