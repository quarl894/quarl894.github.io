---
layout: post
# title:  "PokeUserActivity"
date:   2022-06-18 13:00:00 +0900
categories: Input, Android, Framework
tag: Android, Input
---

### [PokeUserActivity]

PokeUserActivity 역할<br>
휴대폰을 사용시에 Key,Motion등 사용하고 있음을 알려주어 Screen timeofff를 갱신시켜주게 PowerManager에게 알려주는 메소드이다.

### 1. 전달 프로세스

![Image](https://quarl894.github.io/assets/posts/20220618/dialog.jpeg)

#### 전달 조건
```java
void InputDispatcher::dispatchOnceInnerLocked(nsecs_t* nextWakeupTime) {
        // POLICY_FLAG_PASS_TO_USER가 있어야만 가능.
        if (mPendingEvent->policyFlags & POLICY_FLAG_PASS_TO_USER) {
            pokeUserActivityLocked(*mPendingEvent);
        }
}
```

#### pokeUserActivity 처리 조건 (flag, eventtype)

Motion,key event만 처리하며 DISABLE_USER_ACTIVITY가 있다면 처리 안함.
```java
void InputDispatcher::pokeUserActivityLocked(const EventEntry& eventEntry) {
    if (eventEntry.type == EventEntry::Type::FOCUS ||
        eventEntry.type == EventEntry::Type::POINTER_CAPTURE_CHANGED ||
        eventEntry.type == EventEntry::Type::DRAG) {
        // Focus or pointer capture changed events are passed to apps, but do not represent user
        // activity.
        return;
    }
int32_t displayId = getTargetDisplayId(eventEntry);
sp<WindowInfoHandle> focusedWindowHandle = getFocusedWindowHandleLocked(displayId);
if (focusedWindowHandle != nullptr) {
     const WindowInfo* info = focusedWindowHandle->getInfo();
     if (info->inputFeatures.test(WindowInfo::Feature::DISABLE_USER_ACTIVITY)) {
#if DEBUG_DISPATCH_CYCLE
            ALOGD("Not poking user activity: disabled by window '%s'.", info->name.c_str());
#endif
            return;
        }
    }
std::unique_ptr<CommandEntry> commandEntry = std::make_unique<CommandEntry>(&InputDispatcher::doPokeUserActivityLockedInterruptible);
    commandEntry->eventTime = eventEntry.eventTime;
    commandEntry->userActivityEventType = eventType;
    commandEntry->displayId = displayId;
    postCommandLocked(std::move(commandEntry));
}****
```

#### IMS통하여 PMS로 전달.

IMS_JNI로 전달.
```java
void InputDispatcher::doPokeUserActivityLockedInterruptible(CommandEntry* commandEntry) {
    mLock.unlock();
    mPolicy->pokeUserActivity(commandEntry->eventTime, commandEntry->userActivityEventType,commandEntry->displayId);
    mLock.lock();
}

PMS_JNI로 전달
void InputDispatcher::doPokeUserActivityLockedInterruptible(CommandEntry* commandEntry) {
    mLock.unlock();
    mPolicy->pokeUserActivity(commandEntry->eventTime, commandEntry->userActivityEventType,commandEntry->displayId);
    mLock.lock();
}

이후 PMS userActivityFromNative method call
void android_server_PowerManagerService_userActivity(nsecs_t eventTime, int32_t eventType,int32_t displayId) {
    if (gPowerManagerServiceObj) {
        ...
        // Tell the power HAL when user activity occurs.
        setPowerBoost(Boost::INTERACTION, 0);
        }
        JNIEnv* env = AndroidRuntime::getJNIEnv();

        env->CallVoidMethod(gPowerManagerServiceObj,
               gPowerManagerServiceClassInfo.userActivityFromNative,
               nanoseconds_to_milliseconds(eventTime), eventType, displayId, 0);
        checkAndClearExceptionFromCallback(env, "userActivityFromNative");
    }
}

```