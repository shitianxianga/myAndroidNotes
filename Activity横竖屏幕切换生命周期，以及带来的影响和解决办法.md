## **Activity横竖屏幕切换生命周期，以及带来的影响和解决办法**



### 1.生命周期

1. onPause
2. onStop
3. onDestroy
4. onCreate
5. onStart
6. onResume



可以看出切换的时候会将Activity重新构建一遍，这对于一些应用是致命的。

要解决这个问题有两个方法。

**1.在Manifest设置好Activity的screenOrientation属性，不管你怎么拿手机它都是横屏或者是竖屏幕。（横屏：landscape；竖屏：portrait）**

**2.设置Activity的configChanges属性（orientation|keyboardHidden|screenSize）低的版本前两项就可以。**