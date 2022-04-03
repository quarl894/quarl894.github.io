---
layout: post
# title:  "TouchInputMapper(3)"
date:   2021-04-03 19:10:00 +0900
categories: Input, Android, Framework
tag: Android, Input
---

### [Mapper에서 event 처리과정]

앞서 InputDevic에 add된 Mapper들에게 event를 전달하고 InputDispatcher thread를 깨우는것까지 설명했습니다.
이번에는 Mapper들에서 어떻게 event가 가공되어 전달되는지 알아보도록 하겠습니다.

## TouchInputMapper
결론적으로 프로세스 과정을 말씀드리면 아래와 같습니다.(아래는 Gestures 기준)
==(cookAndDispatch에서 DeviceMode에 따라 dispatchMotion까지 과정이 달라지긴 함)==
Process -> sync -> processRawTouches -> cookAndDispatch -> dispatchPointerUsage -> dispatchPointerGestures ->  preparePointerGestures -> dispatchMotion -> getListener()->notifyMotion(&args);

### process
   rawEvent에 SYN_REPORT를 통해 이벤트의 종료를 알 수 있고 sync를 함.

```java
void TouchInputMapper::process(const RawEvent* rawEvent) {
    mCursorButtonAccumulator.process(rawEvent);
    mCursorScrollAccumulator.process(rawEvent);
    mTouchButtonAccumulator.process(rawEvent);

    if (rawEvent->type == EV_SYN && rawEvent->code == SYN_REPORT) {
        sync(rawEvent->when, rawEvent->readTime);
    }
}
```

### sync
 새로운 RawState를 만든 후 Sync후에 processRawTouches로 이동.
   
```java
void TouchInputMapper::sync(nsecs_t when, nsecs_t readTime) {
    // Push a new state.
    mRawStatesPending.emplace_back();
    RawState& next = mRawStatesPending.back();
    
    // Sync button state.
    next.buttonState =
            mTouchButtonAccumulator.getButtonState() | mCursorButtonAccumulator.getButtonState();
    // Sync scroll
    next.rawVScroll = mCursorScrollAccumulator.getRelativeVWheel();
    next.rawHScroll = mCursorScrollAccumulator.getRelativeHWheel();
    mCursorScrollAccumulator.finishSync();

    // Sync touch
    syncTouch(when, &next);
	processRawTouches(false /*timeout*/);
```

### processRawTouches

앞서 DeviceMode::DISABLED에 저장한 touch event를 mCurrentRawState에 복사 후 cookAndDispatch전달

```java
   for (count = 0; count < N; count++) {
        const RawState& next = mRawStatesPending[count];
        // All ready to go.
        clearStylusDataPendingFlags();
        mCurrentRawState.copyFrom(next);
        if (mCurrentRawState.when < mLastRawState.when) {
            mCurrentRawState.when = mLastRawState.when;
            mCurrentRawState.readTime = mLastRawState.readTime;
        }
        cookAndDispatch(mCurrentRawState.when, mCurrentRawState.readTime);
   }
```

### cookAndDispatch

 DeviceMode::POINTER 면 pointerUsage를 선언하고 dispatchPointerUsage로 전달.
 아니라면 updateTouchSpots()후 각dispatch상황에 전달.
```java
void TouchInputMapper::cookAndDispatch(nsecs_t when, nsecs_t readTime) {
 // Dispatch the touches either directly or by translation through a pointer on screen.
    if (mDeviceMode == DeviceMode::POINTER) {
    	// Stylus takes precedence over all tools, then mouse, then finger.
        PointerUsage pointerUsage = mPointerUsage;
        if (!mCurrentCookedState.stylusIdBits.isEmpty()) {
            mCurrentCookedState.mouseIdBits.clear();
            mCurrentCookedState.fingerIdBits.clear();
            pointerUsage = PointerUsage::STYLUS;
        } else if (!mCurrentCookedState.mouseIdBits.isEmpty()) {
            mCurrentCookedState.fingerIdBits.clear();
            pointerUsage = PointerUsage::MOUSE;
        } else if (!mCurrentCookedState.fingerIdBits.isEmpty() ||
                   isPointerDown(mCurrentRawState.buttonState)) {
            pointerUsage = PointerUsage::GESTURES;
        }
    	
    	dispatchPointerUsage(when, readTime, policyFlags, pointerUsage);
    } else {
        updateTouchSpots();

        if (!mCurrentMotionAborted) {
            dispatchButtonRelease(when, readTime, policyFlags);
            dispatchHoverExit(when, readTime, policyFlags);
            dispatchTouches(when, readTime, policyFlags);
            dispatchHoverEnterAndMove(when, readTime, policyFlags);
            dispatchButtonPress(when, readTime, policyFlags);
        }
```

### dispatchTouches

 Touch 상황에 따라 action값을 넣고 dispatchMotion으로 전달.
```java
void TouchInputMapper::cookAndDispatch(nsecs_t when, nsecs_t readTime) {
    if (currentIdBits == lastIdBits) {
        if (!currentIdBits.isEmpty()) {
            // No pointer id changes so this is a move event.
            // The listener takes care of batching moves so we don't have to deal with that here.
            dispatchMotion(...);
        }
    } ...
 	 // Dispatch pointer up events.
        while (!upIdBits.isEmpty()) {
            uint32_t upId = upIdBits.clearFirstMarkedBit();
            dispatchMotion(...);
        }
```

### dispatchMotion

 action 정의나 POINTER일 경우 cursor 좌표 등 설정 후 listener를 통해 notifyMotion(&args) 전달.
```java
void TouchInputMapper::dispatchMotion(

    if (changedId >= 0 && pointerCount == 1) {
        // Replace initial down and final up action.
        // We can compare the action without masking off the changed pointer index
        // because we know the index is 0.
        if (action == AMOTION_EVENT_ACTION_POINTER_DOWN) {
            action = AMOTION_EVENT_ACTION_DOWN;
        } else if (action == AMOTION_EVENT_ACTION_POINTER_UP) {
            if ((flags & AMOTION_EVENT_FLAG_CANCELED) != 0) {
                action = AMOTION_EVENT_ACTION_CANCEL;
            } else {
                action = AMOTION_EVENT_ACTION_UP;
            }
        } else {
            // Can't happen.
            ALOG_ASSERT(false);
        }
    }
    float xCursorPosition = AMOTION_EVENT_INVALID_CURSOR_POSITION;
    float yCursorPosition = AMOTION_EVENT_INVALID_CURSOR_POSITION;
    if (mDeviceMode == DeviceMode::POINTER) {
        auto [x, y] = getMouseCursorPosition();
        xCursorPosition = x;
        yCursorPosition = y;
    }
    
NotifyMotionArgs args(getContext()->getNextId(), when, readTime, deviceId, source, displayId, policyFlags, action, actionButton, flags, metaState, buttonState, MotionClassification::NONE, edgeFlags, pointerCount, pointerProperties, pointerCoords, xPrecision, yPrecision, xCursorPosition, yCursorPosition, downTime, std::move(frames));

getListener()->notifyMotion(&args);
```