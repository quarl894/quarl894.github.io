---
layout: post
# title:  "PointerIcon"
date:   2021-05-05 14:30:00 +0900
categories: Input, Android, Framework
tag: Android, Input
---

### [PointerIcon]

이번편은 PointerIcon을 어떻게 변경시키는지 동작에 대해 알아보려 한다.
연관 파일
1. PointerController.cpp
2. MouseCursorController.cpp
3. com_android_server_input_InputManagerService.cpp
4. InputManagerService.java

## Icon Resource들을 load하는 부분.
loadPointerIcon
loadPointerResources
loadAdditionalMouseResources

먼저 PointerIcon을 사용하기 위해 view에서 Resocure들을 native로 가져오는 선행작업들.

앱단에서 PointerIcon 변경 위해 호출하면서 시작 setPointerIconType(int iconId).

### 1.InputManager.java
```java
    public void setPointerIconType(int iconId) {
        try {
            mIm.setPointerIconType(iconId);
        } catch (RemoteException ex) {
            throw ex.rethrowFromSystemServer();
        }
    }
```

### 2. InputManagerService.java
```java
    // Binder call
    @Override
    public void setPointerIconType(int iconId) {
        nativeSetPointerIconType(mPtr, iconId);
    }
```

### 3. com_android_server_input_InputManagerService.cpp (setPointerIconType)
```java
void NativeInputManager::setPointerIconType(int32_t iconId) {
    AutoMutex _l(mLock);
    std::shared_ptr<PointerController> controller = mLocked.pointerController.lock();
    if (controller != nullptr) {
        controller->updatePointerIcon(iconId);
    }
}
```

###4. PointerController.cpp (updatePointerIcon)
```java
void PointerController::updatePointerIcon(int32_t iconId) {
    std::scoped_lock lock(mLock);
    mCursorController.updatePointerIcon(iconId);
}
```

###5. MouseCursorController.cpp (updatePointerIcon)
```java
void MouseCursorController::updatePointerIcon(int32_t iconId) {
    std::scoped_lock lock(mLock);
	// iconId가 변경되었으면 update
    if (mLocked.requestedPointerType != iconId) {
        mLocked.requestedPointerType = iconId;
        mLocked.updatePointerIcon = true;
        updatePointerLocked();
    }
}
```

###6. MouseCursorController.cpp (updatePointerLocked)
```java
void MouseCursorController::updatePointerLocked() REQUIRES(mLock) {
    if (!mLocked.viewport.isValid()) {
        return;
    }
    sp<SpriteController> spriteController = mContext.getSpriteController();
    spriteController->openTransaction();

    mLocked.pointerSprite->setLayer(Sprite::BASE_LAYER_POINTER);
    mLocked.pointerSprite->setPosition(mLocked.pointerX, mLocked.pointerY);
    mLocked.pointerSprite->setDisplayId(mLocked.viewport.displayId);

    if (mLocked.pointerAlpha > 0) {
        mLocked.pointerSprite->setAlpha(mLocked.pointerAlpha);
        mLocked.pointerSprite->setVisible(true);
    } else {
        mLocked.pointerSprite->setVisible(false);
    }

// updatePointerIcon을 true인 상태로 와야 변경이 됨.
/*
mLocked.updatePointerIcon = true 시켜주는 부분.
1. updatePointerIcon(int32_t iconId)
2. setCustomPointerIcon(const SpriteIcon& icon)
3. loadResourcesLocked(bool getAdditionalMouseResources)
4. getAdditionalMouseResources
*/

    // additionalMouseResources는 IMS_JNI보면 처음에 모든 리소스를 다 로드해놓음.
    // 요청값이 기본값이라면 mLocked.pointerIcon로 바로 set해주고 아니라면 리소스에서 찾아서 셋해줌.
    if (mLocked.updatePointerIcon) {
        if (mLocked.requestedPointerType == mContext.getPolicy()->getDefaultPointerIconId()) {
            mLocked.pointerSprite->setIcon(mLocked.pointerIcon);
        } else {
            std::map<int32_t, SpriteIcon>::const_iterator iter =
                    mLocked.additionalMouseResources.find(mLocked.requestedPointerType);
            if (iter != mLocked.additionalMouseResources.end()) {
                std::map<int32_t, PointerAnimation>::const_iterator anim_iter =
                        mLocked.animationResources.find(mLocked.requestedPointerType);
                if (anim_iter != mLocked.animationResources.end()) {
                    mLocked.animationFrameIndex = 0;
                    mLocked.lastFrameUpdatedTime = systemTime(SYSTEM_TIME_MONOTONIC);
                    startAnimationLocked();
                }
                mLocked.pointerSprite->setIcon(iter->second);
            } else {
                ALOGW("Can't find the resource for icon id %d", mLocked.requestedPointerType);
                mLocked.pointerSprite->setIcon(mLocked.pointerIcon);
            }
        }
        mLocked.updatePointerIcon = false;
    }

    spriteController->closeTransaction();
}
```

