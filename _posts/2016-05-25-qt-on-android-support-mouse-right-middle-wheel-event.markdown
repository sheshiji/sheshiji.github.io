---
layout: post
category: "Qt"
title:  "修改Qt on android源码，支持鼠标右键，中键和滚轮事件"
tags: [Android]
summary: ""
---
## 第一步 定位源码

C++部分

```
$QTDIR\qtbase\src\plugins\platforms\android\androidjniinput.cpp
$QTDIR\qtbase\src\gui\kernel\qwindowsysteminterface.cpp
```

Java部分

```
$QTDIR\qtbase\src\android\jar\src\org\qtproject\qt5\android\QtNative.java
$QTDIR\qtbase\src\android\jar\src\org\qtproject\qt5\android\QtSurface.java
```

## 第二步 修改源码

**androidjniinput.cpp**新增内容

```
static void mouseStateDown(JNIEnv */*env*/, jobject /*thiz*/, jint /*winId*/, jint x, jint y, jint state) {
    if (m_ignoreMouseEvents)
        return;
    
    Qt::MouseButtons bts(Qt::LeftButton);
    if(state == 0x1) {
        bts = Qt::RightButton;
    } else if(state == 0x2) {
        bts = Qt::MidButton;
    }

    QPoint globalPos(x,y);
    QWindow *tlw = topLevelWindowAt(globalPos);
    m_mouseGrabber = tlw;
    QPoint localPos = tlw ? (globalPos - tlw->position()) : globalPos;
    QWindowSystemInterface::handleMouseEvent(tlw,localPos,globalPos,bts);
}

static void mouseStateMove(JNIEnv */*env*/, jobject /*thiz*/, jint /*winId*/, jint x, jint y, jint state) {
    if (m_ignoreMouseEvents)
        return;
    
    Qt::MouseButtons bts(Qt::LeftButton);
    if(state == 0x1) {
        bts = Qt::RightButton;
    } else if(state == 0x2) {
        bts = Qt::MidButton;
    }

    QPoint globalPos(x,y);
    QWindow *tlw = m_mouseGrabber.data();
    if (!tlw)
        tlw = topLevelWindowAt(globalPos);
    QPoint localPos = tlw ? (globalPos-tlw->position()) : globalPos;
    QWindowSystemInterface::handleMouseEvent(tlw,localPos,globalPos,bts);
}

static Qt::KeyboardModifiers mapAndroidModifiers(jint modifiers);
static void mouseWheel(JNIEnv */*env*/, jobject /*thiz*/, jint /*winId*/, jint x, jint y, jint axisVal, jint modifier) {
    if (m_ignoreMouseEvents)
        return;

    QPoint globalPos(x,y);
    QWindow *tlw = topLevelWindowAt(globalPos);
    m_mouseGrabber = tlw;
    QPoint localPos = tlw ? (globalPos - tlw->position()) : globalPos;
    QWindowSystemInterface::handleWheelEvent(tlw,localPos,globalPos,-axisVal * 8 * 15,Qt::Vertical,mapAndroidModifiers(modifier));
}

// JNI注册
static JNINativeMethod methods[] = {
    ...
    {"mouseStateDown", "(IIII)V", (void *)mouseStateDown},
    {"mouseStateMove", "(IIII)V", (void *)mouseStateMove},
    {"mouseWheel", "(IIIII)V", (void *)mouseWheel},
    ...
};
```

**QtNative.java**新增内容

```
static public void sendMouseEvent(MotionEvent event, int id) {
    int bs = event.getButtonState();
    int bt = 0x0;
    if(bs == event.BUTTON_SECONDARY) {
        bt = 0x1;
    } else if(bs == event.BUTTON_TERTIARY) {
        bt = 0x2;
    }
	
    switch (event.getAction()) {
        case MotionEvent.ACTION_UP:
            mouseUp(id, (int) event.getX(), (int) event.getY());
            break;

        case MotionEvent.ACTION_DOWN:
            mouseStateDown(id, (int) event.getX(), (int) event.getY(), bt);
                
            m_oldx = (int) event.getX();
            m_oldy = (int) event.getY();
            break;

        case MotionEvent.ACTION_MOVE:
            int dx = (int) (event.getX() - m_oldx);
            int dy = (int) (event.getY() - m_oldy);
            if (Math.abs(dx) > 5 || Math.abs(dy) > 5) {
                mouseStateMove(id, (int) event.getX(), (int) event.getY(), bt);
                m_oldx = (int) event.getX();
                m_oldy = (int) event.getY();
            }
            break;
    }
}

static public void sendMouseWheelEvent(MotionEvent event, int keyModifier, int id) {
    float av = event.getAxisValue(MotionEvent.AXIS_VSCROLL);
    if (av < 0.0f) {
        mouseWheel(id, (int) event.getX(), (int) event.getY(), -1, keyModifier);
    } else {
        mouseWheel(id, (int) event.getX(), (int) event.getY(), 1, keyModifier);
    }
}

// JNI声明
public static native void mouseStateDown(int winId, int x, int y, int state);
public static native void mouseStateMove(int winId, int x, int y, int state);
public static native void mouseWheel(int winId, int x, int y, int axisVal, int keyModifier);
```

**QtSurface.java**新增内容

```
import android.view.KeyEvent;
import android.view.InputDevice;

public class QtSurface extends SurfaceView implements SurfaceHolder.Callback {
    private int m_keyModifier = 0x0;
    private static boolean m_mouseTriggered = false;
    
    @Override
    public boolean onKeyDown(int keyCode,KeyEvent event) {  
        m_keyModifier = event.getMetaState();
        return true; 
    }  
	
    @Override
    public boolean onKeyUp(int keyCode,KeyEvent event) { 
        m_keyModifier = 0x0;
        return true;  
    }  

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        if(m_mouseTriggered) {
            // 鼠标事件
            QtNative.sendMouseEvent(event, getId());
        } else {
            // 触摸事件
            QtNative.sendTouchEvent(event, getId());
            m_gestureDetector.onTouchEvent(event);
        }
        return true;
    }
	
    @Override
    public boolean onGenericMotionEvent(MotionEvent event) {
        // 获取鼠标状态
        int ac = event.getAction();
        if(ac == MotionEvent.ACTION_HOVER_EXIT) {
            m_mouseTriggered = true;
        } else if(ac == MotionEvent.ACTION_HOVER_ENTER) {
            m_mouseTriggered = false;
        }
		
		// 获取鼠标滚轮状态
        if ((event.getSource() & InputDevice.SOURCE_CLASS_POINTER) != 0) {
            if(event.getAction() == MotionEvent.ACTION_SCROLL) {
                QtNative.sendMouseWheelEvent(event, m_keyModifier, getId());
            }
        }
		
        return true;
    }
}
```

重新编译源码。