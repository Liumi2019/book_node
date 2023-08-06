# Android 服务开发

## 1. 认识服务
新建一个继承自Service的类MyService，然后在AndroidManifest.xml里注册这个Service。

## 2. 启动服务的方法
### 2.1 startService/stopService
启动服务类似于启动活动，多次启动也只能启动一次，必须手动关闭。
```java
startService(new Intent(getBaseContext(), MyService.class));
stopService(new Intent(getBaseContext(), MyService.class));
```

### 2.2 使用 Binder 的方法启动服务
#### 2.2.1 Binder 是什么，有什么用
参考 [学习Binder](./学习Binder.md) 

#### 2.2.2 使用 Binder 传递 Service 实例
1. 新建一个继承自Service的类MyService，然后在AndroidManifest.xml里注册这个Service
2. 新建一个继承自Binder的类MyBinder，并在 Binder 类内获取 MyService 的实例
3. 在MyService里实例化一个MyBinder对象mBinder，并在onBind回调方法里面返回这个mBinder对象
4. 在Activity里面实现 ServiceConnection 实现拿到 MyBinder 的实例
5. 调用 MyBinder 的实例方法获取 MyService 的实例，即可调用 Service 的方法

## 3. 服务和活动在不同进程时只能通过 Binder 传递服务
### 3.1 跨进程启动服务
自定义的MyBinder不具有跨进程的能力，绑定Service的时候无法得到Binder。需要使用AIDL生成一个可以跨进程的Binder，然后用这个可跨进程的Binder替换MyBinder。

定义 ADIL 文件:
1. 定义 ADIL 文件使用 Android studio 新建 adil 文件向导 
2. 所有的类必须导入，不管是不是同一个包内
3. 所有要传递的数据都必须继承 Parcelable，并定义相同包路径的 ADIL 文件

定义一个接口
``` adil
package com.example.myfristlib;

interface IMyAidlInterface {
    String getPackagename();
}

```

定义一个自定义数据类，自定义 Rect 类型

```java

package com.example.mypackage;

import android.os.Parcel;
import android.os.Parcelable;

public final class Rect implements Parcelable {
    public int left;
    public int top;
    public int right;
    public int bottom;

    public static final Parcelable.Creator<Rect> CREATOR = new Parcelable.Creator<Rect>() {
        public Rect createFromParcel(Parcel in) {
            return new Rect(in);
        }

        public Rect[] newArray(int size) {
            return new Rect[size];
        }
    };

    public Rect() {
    }

    private Rect(Parcel in) {
        readFromParcel(in);
    }

    public void writeToParcel(Parcel out, int flags) {
        out.writeInt(left);
        out.writeInt(top);
        out.writeInt(right);
        out.writeInt(bottom);
    }

    public void readFromParcel(Parcel in) {
        left = in.readInt();
        top = in.readInt();
        right = in.readInt();
        bottom = in.readInt();
    }

    public int describeContents() {
        return 0;
    }
}

```

自定义类型的 adil 文件，包名必须一致，使用自定类型必须使用关键字：in、out、inout标识数据传输方向。
```adil
package com.example.mypackage;

parcelable Rect;
```

### 3.2 ADIL 回调接口

1. 定义一个回调接口，在客户端定义，服务端调用。
```adil
package com.zhonghong.airconditione.aidl;

interface NotifyCallBack{
    void notifychanged(int event);
}
```

2. 服务的adil文件，调用了回调接口 NotifyCallBack。
```adil
package com.zhonghong.airconditione.aidl;

import com.zhonghong.airconditione.aidl.NotifyCallBack;

interface HealthyServer{
    int[] getHealthyInfo();
    int[] getWheelPressure();
    void registerCallBack(NotifyCallBack callback);
    void unregisterCallBack(NotifyCallBack callback);
}
```

服务端可以获得多个客户端的注册，需要使用容器装载回调接口对象，并实现加载和卸载操作。
```java
// 定义一个泛型容器
private RemoteCallbackList<NotifyCallBack> mCallBacks =
    new RemoteCallbackList<NotifyCallBack>();

@Override
public void registerCallBack(NotifyCallBack callback) throws RemoteException {
    mCallBacks.register(callback);
}

@Override
public void unregisterCallBack(NotifyCallBack callback) throws RemoteException {
    mCallBacks.unregister(callback);
}

try {
    int len = mCallBacks.beginBroadcast();
    Log.i(TAG, "len:"+len);
    for(int i =0;i<len;i++){    
        mCallBacks.getBroadcastItem(i).notifychanged(COM_MSG_5E0H);
    };
} catch (RemoteException e) {
    e.printStackTrace();
}
Log.i(TAG, "finish braodcast!");
mCallBacks.finishBroadcast();
```

3. 客户端定义回调内容
```java
NotifyCallBack callback = new NotifyCallBack.Stub() {
    @Override
    public void notifychanged(int event) throws RemoteException {
        handler.sendEmptyMessage(0x01);
    }
};

/** 这个是在ServiceConnection里的绑定成功的回调方法*/
public void onServiceConnected(ComponentName name, IBinder service) {
    mServer = HealthyServer.Stub.asInterface(service);
    try {
        mServer.registerCallBack(callback);
    } catch (RemoteException e) {
        e.printStackTrace();
    }
    refresh();

    if(healthyData==null){
        shareHealthy();
    }

    if(wheelPressure==null){
        shareWheel();
    }
}

```
















