# Service相关知识

### 1.Service生命周期：

1.onCreate

2.onStartCommand

3.onDestroy

4.onBind

5.onUnbind

6.onRebind

### 2.生命周期调用

- 启动服务：

  startService执行的生命周期： oncreate->onstartCommand->onDestory

  1. 启动服务：startService

     单次：oncreate->onStartCommand

     多次：oncreate->onStartCommand->onStartCommand....

  2. 停止服务：stopService

     onDestory()

- 绑定服务：

  bindService执行的生命周期：onCreate->onBind->onUnBind->onDestory

  1. 绑定服务：bindService

     oncreate->onBind

  2. 解绑服务：unBindService

     onUnbind->onDestroy

- 先绑定后启动服务

  先bindService后startService的生命周期：onCreate->onBind->onStartCommand->onUnBind->onDestroy

  1. 绑定启动服务：bindService,startService

     onCreate->onBInd->onStartCommand

  2. 解绑停止服务：unBindService，stopService

     onUnBind->onDestory

- 先解除绑定后绑定服务：

  unBindService，bindService

  onUnBind(true)->onReBind()

- 先启动后绑定服务

  生命周期：onCreat->onStartCommand->onBind->onUnBind->onDestroy

  1. 启动绑定服务：startService,bindService

     onCreate->onStartCommand->onBInd

  2. 停止解绑服务：stopService,unBindService

     onUnBind->onDestory

### 3.IntentService和Service的区别

1.首先IntentService继承自Service所以service该有的它都有，包括生命周期等等。它相当于一个异步定制化的Service。

2.IntentService多了一个方法是onHandleIntent，一般在这里写耗时操作。那什么时候调用这个方法呢？在oncreate的时候IntentService新建了一个线程，并且获取这个线程的looper新建了一个ServiceHandler，在onStart的时候调用了Handler的sendMessage方法。而在这个Handler的hanlerMessage里调用了onHandleIntent。

```java
private final class ServiceHandler extends Handler {
    public ServiceHandler(Looper looper) {
        super(looper);
    }

    @Override
    public void handleMessage(Message msg) {
        onHandleIntent((Intent)msg.obj);
        stopSelf(msg.arg1);
    }
}
```

```java
public void onCreate() {
    // TODO: It would be nice to have an option to hold a partial wakelock
    // during processing, and to have a static startService(Context, Intent)
    // method that would launch the service & hand off a wakelock.

    super.onCreate();
    HandlerThread thread = new HandlerThread("IntentService[" + mName + "]");
    thread.start();

    mServiceLooper = thread.getLooper();
    mServiceHandler = new ServiceHandler(mServiceLooper);
}

@Override
public void onStart(@Nullable Intent intent, int startId) {
    Message msg = mServiceHandler.obtainMessage();
    msg.arg1 = startId;
    msg.obj = intent;
    mServiceHandler.sendMessage(msg);
}
```

总的来说就是IntentService就是自带异步执行的Service。但是需要注意的是，每次startService都会去调用一次耗时操作，并且是每次只能执行一个。