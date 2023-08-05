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

