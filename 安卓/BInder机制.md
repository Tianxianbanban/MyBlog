## BInder机制



**Linux内核的基础知识**

+ 进程隔离/虚拟地址空间

  操作系统当中为了保证进程间互不干扰，设计了进程隔离的技术，避免了一个进程去操作另一个进程的数据。进程隔离用到了虚拟地址空间，**不同进程的虚拟地址空间是不同**的，不同进程之间数据不共享，进程要和另一个进程通信就需要通过某种进程间的通信机制去进行，在Android当中就是通过**Binder机制**来完成。

+ 系统调用

  Linux内核当中有个很重要的概念叫**系统调用**，因为对内核会有某种保护机制来告诉应用程序只能访问某些许可资源，而不许可资源是不能被访问的，这就是把Linux的内核层和上层应用程序抽象分离开，也就是内核层和用户空间。用户可以**通过系统调用在用户空间访问内核的某些程序**。

+ Binder驱动

  在Android当中，**BInder运行在内核空间当中**的，各个用户进程通过Binder通信内核来进行交互的模块叫做**Binder驱动**。驱动程序一般指的是设备的驱动程序，是一种可以使计算机和设备通信的特殊程序，其实也是一种软件，但是又相当于硬件接口，操作系统可以通过这个接口来控制硬件设备，这就是Binder驱动，Binder可以理解为一种虚拟的物理设备。



**Binder通信机制**

+ 为什么使用Binder

  1. **Android使用Linux内核，拥有非常多的跨进程通信机制**，比如管道、Socket等等，为什么会添加一个BInder通信机制来作为Android特有的进程间通信机制呢？主要是因为**性能**与**安全**的优势。

  2. 性能

     移动设备上广泛使用跨进程通信肯定会对通信机制提出严格要求，而Binder比传统的方式更加高效。

  3. 安全

     传统的进程间通信对于通信双方的身份没有做出严格的验证，只有上层协议才会进行相关保证，比如说Socket通信的话IP地址是客户端手动填写的，可以进行人为伪造，而**Binder机制支持通信双方进行身份校验**，这是BInder在安全性方面所做的努力，这大大提高了应用程序的安全性，Binder的身份校验也是Android权限模型的基础。

+ Binder通信模型

  其实可以把跨进程通信的双方，一端称为服务进程，而另一端称为客户端进程，而由于进程隔离的存在，我们没有办法通过一般方式在客户端进程访问到服务端进程如果过不进行跨进程通信的话。

  两个运行在用户空间的进程要完成通信必须通过**内核**的帮助，而这个**运行在内核的程序就是Binder驱动**。它的功能类似现实生活中双方打电话所要借助的“通信基站”，那么通讯录就是ServiceManager。

  1. 电话基站：Binder驱动
  2. 通讯录：serviceManager

+ Binder通信机制原理

  通信的步骤：第一步是ServiceManager的建立，也就是首先要有一个进程向驱动提出申请ServiceManager，而内核驱动以后，ServiceManager就负责管理所有的“电话号码”也就是通信方，由于此时还没有需要通信的进程向它注册，所以最开始是没有内容的。第二步就是一方要通信另一方就需要将“自己联系的方式”向ServiceManager中注册。

  Binder是如何进行跨进程通信的呢？

  Server端在ServiceManager当中注册了它的方法，而Client端如果想要调用Server端的一个方法，如果到ServiceManager中去查找时候有这样的一个方法，这个时候ServiceManager会返回给Client一个代理对象的方法，而这个代理对象的方法是一个空方法，它的主要作用就是Client端调用方法的时候会返回给内核驱动，内核驱动接收到了代理对象的方法，知道了客户端想要调用的服务端方法，内核回去调用服务端的相应方法，然后服务端的方法调用完后，把结果返回给内核驱动层ServiceManager，ServiceManager才会把结果返回给客户端。这样才是一个完整的Binder跨进程通信机制的原理。也就是**客户端进程持有一个服务端进程的代理，通过代理对象协助驱动完成跨进程通信**，具体的跨进程通信都是通过代理完成的。



**所以什么是Binder？**

1. 直观地说Binder是Android当中的一个类，**实现了IBinder接口**。
2. 通常意义下，从IPC角度看，Binder指的是Android当中的一种**跨进程的通信机制**，当然Binder跨线程也可以。
3. Binder还可以理解为是一种虚拟的物理设备，他的设备驱动是/dev/binder，这种通信方式在Linux当中是没有的；从Android Framework的角度来说，Binder是ServiceManager连接各种Manager（ActivityManager、WindowManager等）和相应ManagerService的桥梁，
4. 从Android应用层来说，**Binder是客户端和服务端进行通信的媒介**，当bindService的时候，服务端会返回一个包含了服务端业务调用的Binder对象，通过这个Binder对象，客户端就可以获取服务端提供的服务或者数据，这个服务包括了普通的服务和基于AIDL的服务。
5. 对于传输过程而言，Binder**是可以进行跨进程传递的对象**。Binder驱动会对具有跨进程传递能力的对象做特殊处理，自动完成代理对象和服务端对象的转换。
6. 对于Service进程来说，Binder指的是**Binder本地对象**/对于Client来说，Binder指的是**代理对象**（比如说如果Activity和Service处于不同进程中的话，最后获取的就是一个Binder代理对象）。



**从Framework层看Binder原理**

首先，Android利用Binder进行通信的话，肯定要首先获取Binder对象。

首先**系统服务**和我们自定义服务Binder对象的获取方式是不一样的，原因在于系统服务是在系统启动的时候就被注册到了**ServiceManager**，所以只需要通过**ServiceManager.getService(String name)**传入想要获得的系统服务的名字就能获得对应的Binder对象进行进程间通信了，系统已经将该Service所提供的服务全部写好了：系统启动，**init进程**会启动一个叫**ServiceManager**的进程，这个进程启动之后会做三件事请：`第一`通过**open**打开设备文件/dev/binder，将这个文件当中的内容通过**mmap**映射到本进程空间当中；`第二`是通过IO控制命令**BINDER_SET_CONTEXT_MGR**将当前进程注册到Binder驱动当中，然后Binder驱动就会为他在内核空间创建一个称为**binder_context_mgr_node**的节点，这个节点就是**ServiceManager在内核空间Binder实体**了，然后将它的句柄设置为0，它的创建时在系统启动的时候，这个节点在Binder驱动中是唯一的，所以也造成了ServiceManager区别于其他Server了，但是它仍然是运行在用户空间的；`第三`ServiceManager启动之后会无限**循环等待**Server和Client之间的进程间通信请求了；接着**Zygote进程会孵化一个子进程SystemService**，我们的大部分系统服务比如ActivityManagerService、WindowManagerService、MediaPlayService等等都是这个进程内的一个线程，这些服务都是会调用**ServiceManager.addService**方法注册添加到ServiceManager中的服务列表svlist中，这样系统服务就都注册到ServiceManager当中了。

ServiceManager是随着系统的启动而创建的，它不同于普通的Server，甚至可以理解为是普通Server的Server，**系统会将所要提供的服务注册到它的服务列表中**，服务列表中存储的就是服务名字和服务在Binder驱动中的句柄，这里的句柄可以理解为同一进程内部的引用，只不过现在是跨进程通信换了个名字而已，如果我们的应用想要使用某个系统服务的话，可以传入服务的名字调用ServiceManager.getService**得到这个服务对应的Binder对象**这种方式来进行获取，然后就可以使用这个Binder对象来进行相应操作了，具体来说这也是ContentProvider为我们封装的内容了。

Binder就是通过**客户端/服务端**模式实现的，关于Binder的实现，还需要区分几个概念：Server/Client/ServiceManager/Binder驱动，这个部分是Binder架构的核心，尤其是ServiceManager和Binder驱动，因为我们的应用程序安装到系统上的时候，都会被分配一个唯一的用户ID，它们是运行在独立进程当中的，Linux基于安全的考虑是不允许我们直接访问系统上服务的，因为它们属于不同的进程，而进程间的数据是不能共享的，那么要使用到这些系统服务就需要通过**跨进程通信**机制了，也就需要**内核空间**的参与，所以用到Binder驱动，它的作用相当于是桥梁。平常我们访问系统服务，比如媒体资源是使用ContentProvider，它的底层实现就是BInder，只是系统为我们做了封装。**一句话总结下Client和Server通信时，ServiceManager相当于通讯录，而Binder驱动就是通信基站**。

具体的通信过程是这样的：

Server和Binder驱动通信**ServiceManager.addService(String name,IBinder service)**，传入的内容是Server的名字和Server在Binder驱动当中对应的对象：（1）Server首先将自己作为对象，并且附上一个句柄为0的值（用于访问ServiceManager），将这些内容封装成一个数据包，open有关Binder的设备文件/dev/binder，将封装好的这个数据包发送给Binder驱动；（2）Binder驱动在收到这个数据包之后，发现里面存在一个Server对象，首先会在Binder驱动自己内部新建该Server对应的Binder实体，并且赋予一个大于0的句柄，同时会将这个句柄也加入到数据包当中，接着会将数据包发送到句柄为0对应的对象，也就是ServiceManager了。（3）ServiceManager收到转发给自己的数据包之后，会查看其服务列表svlist中是否已经存在当前Server名字的服务，如果不存在的话，就会将当前服务+当前服务对应于Binder驱动中的句柄加入到列表当中。 这个过程，**系统服务就注册到ServiceManager**当中了。

Client和Binder驱动通信**ServiceManager.getService(String name)**，返回对应的所请求Binder对象，传入的参数是将要请求的服务的名字：（1）Client首先会将要获取的服务的名字以及一个句柄为0的值（为了访问ServiceManager）封装成一个数据包，open有关Binder的设备文件/dev/binder，将该数据包发送给Binder驱动；（2）Binder驱动在收到数据包之后发现里面有句柄为0的信息，就将该数据包转发给ServiceManager来进行处理了；（3）ServiceManager在收到数据包之后根据服务名字查看自己服务列表svlist，找到之后会将对应的在Binder驱动当中的句柄信息也封装成一个数据包；（4）该数据包也会通过Binder驱动被发送给Client端；（5）Client端在收到数据包之后，就得到了自己所请求的服务在Binder驱动中的句柄，它会**利用这个句柄信息在自己本地创建一个远程Server的代理**，以后Client发消息都是发给这个代理的，**随后的通信就都变成了代理通过Binder驱动与真正的Server进行交互，去完成跨进程通信**。

这就是系统服务怎么注册到ServiceManager内，以及怎么获取服务对应于Binder驱动的句柄也就是Binder对象的过程。

----

而对于**自定义的服务**想要实现进程间通信的话，Client端和Server端都是需要我们自己实现的，其中Server端的实现是借助于Service来完成的，Android就提供**AIDL帮助简化这个过程**，当然就算不用AIDL也是可以实现跨进程通信的，只是**序列化和反序列化**数据的封装和相关顺序问题需要注意。使用AIDL只需要**将Server进程想要提供给Client进程访问的方法定义在一个.aidl文件中**就可，假设IStudentService.aidl，那么系统会为这个AIDL文件自动生成对应的IStudentService.java文件，它的基本结构是这样的：（1）首先这个接口会继承于**android.os.IInterface**接口；（2）会声明一个名为**Stub**的内部抽象类，继承自**android.os.Binder**，也就是说是一个Binder类，并且实现remote.IStudentService本身这个接口；（3）Stub类包含了一些方法和一个**内部代理类Proxy**：

+ remote.IStudentService asInterface(android.os.IBinder obj)

+ android.os.IBinder asBinder()

+ boolean onTransact(int code, android.os.Parcel data, android.os.Parcel reply, int flags)

+ **class** Proxy implements remote.IStudentService 

  这个代理类实现了IStudentService的方法，但是并不是真正的逻辑实现，而是**通过transact进行的一些进程切换操作**。


```java
package remote;

/**
* 核心实现就是其中的内部Binder类Stub，和Stub的内部代理类Proxy。
*/
public interface IStudentService extends android.os.IInterface {
    /**
     * Default implementation for IStudentService.
     */
    public static class Default implements remote.IStudentService {
        @Override
        public remote.Student getStudentById(int id) throws android.os.RemoteException {
            return null;
        }

        @Override
        public android.os.IBinder asBinder() {
            return null;
        }
    }

    /**
     * Local-side IPC implementation stub class.
     */
    public static abstract class Stub extends android.os.Binder implements remote.IStudentService {
        private static final java.lang.String DESCRIPTOR = "remote.IStudentService";

        /**
         * Construct the stub at attach it to the interface.
         */
        public Stub() {
            this.attachInterface(this, DESCRIPTOR);
        }

        /**
         * Cast an IBinder object into an remote.IStudentService interface,
         * generating a proxy if needed.
         * ！！！！
         * 用于将服务端的Binder对象转换成客户端所需要的AIDL接口类型对象，这种转换过程是区分进程的，
         * 会根据传入的Binder对象来判断是跨进程通信还是进程内部通信：
         * 1. 如果是进程内部通信，返回的是IStudentService.Stub对象本身；
         * 2. 如果是跨进程通信，返回的就是remote.IStudentService.Stub.Proxy(obj)代理对象，
         * 		这个代理对象中的方法会通过调用这里传入的Binder#transact方法来进行内核态的切换。
         */
        public static remote.IStudentService asInterface(android.os.IBinder obj) {
            if ((obj == null)) {
                return null;
            }
            android.os.IInterface iin = obj.queryLocalInterface(DESCRIPTOR);
            if (((iin != null) && (iin instanceof remote.IStudentService))) {
                return ((remote.IStudentService) iin);
            }
            return new remote.IStudentService.Stub.Proxy(obj);
        }

        //返回当前Stub也就是Binder对象
        @Override
        public android.os.IBinder asBinder() {
            return this;
        }

        /**
         * 这个方法运行在服务端中的Binder线程池中，
         * 当客户端发起跨进程请求时，远程请求会通过系统底层封装后交由此方法处理。
         * 通过code确定客户端所请求的目标方法，接着从data中取出目标方法所需参数，然后执行目标方法。
         * 
         * 目标方法执行完毕，向reply写入返回值。
         * 而如果这个方法返回false，客户端的请求就会失败，所以也可以用这个特性来做权限验证。
         */
        @Override
        public boolean onTransact(int code, android.os.Parcel data, android.os.Parcel reply, int flags) throws android.os.RemoteException {
            java.lang.String descriptor = DESCRIPTOR;
            switch (code) {
                case INTERFACE_TRANSACTION: {
                    reply.writeString(descriptor);
                    return true;
                }
                case TRANSACTION_getStudentById: {
                    data.enforceInterface(descriptor);
                    int _arg0;
                    _arg0 = data.readInt();
                    remote.Student _result = this.getStudentById(_arg0);
                    reply.writeNoException();
                    if ((_result != null)) {
                        reply.writeInt(1);
                        _result.writeToParcel(reply, android.os.Parcelable.PARCELABLE_WRITE_RETURN_VALUE);
                    } else {
                        reply.writeInt(0);
                    }
                    return true;
                }
                default: {
                    return super.onTransact(code, data, reply, flags);
                }
            }
        }

        private static class Proxy implements remote.IStudentService {
            private android.os.IBinder mRemote;

            Proxy(android.os.IBinder remote) {
                mRemote = remote;
            }

            @Override
            public android.os.IBinder asBinder() {
                return mRemote;
            }

            public java.lang.String getInterfaceDescriptor() {
                return DESCRIPTOR;
            }

            /**
         	 * 运行在客户端，
         	 * 调用此方法时，首先创建所需输入型Parcel对象_data、输出型Parcel对象_reply和返回值对象List；
         	 * 然后把该方法的参数信息写入_data中（如果有参数的话）；
         	 * 然后内部调用transact方法发起RPC请求，进行远程过程调用，同时当前线程挂起；
         	 * 然后服务端的onTransact方法会被调用，直到RPC过程返回后，当前线程继续执行。
         	 * 并从_reply中取出RPC过程的返回结果；最后返回_reply中的数据。
     	     */
            @Override
            public remote.Student getStudentById(int id) throws android.os.RemoteException {
                android.os.Parcel _data = android.os.Parcel.obtain();
                android.os.Parcel _reply = android.os.Parcel.obtain();
                remote.Student _result;
                try {
                    _data.writeInterfaceToken(DESCRIPTOR);
                    _data.writeInt(id);
                    boolean _status = mRemote.transact(Stub.TRANSACTION_getStudentById, _data, _reply, 0);
                    if (!_status && getDefaultImpl() != null) {
                        return getDefaultImpl().getStudentById(id);
                    }
                    _reply.readException();
                    if ((0 != _reply.readInt())) {
                        _result = remote.Student.CREATOR.createFromParcel(_reply);
                    } else {
                        _result = null;
                    }
                } finally {
                    _reply.recycle();
                    _data.recycle();
                }
                return _result;
            }

            public static remote.IStudentService sDefaultImpl;
        }
				
      	//声明整形id用于标识方法getStudentById，标识在transact过程中客户端所请求的到底是哪一个方法。
        static final int TRANSACTION_getStudentById = (android.os.IBinder.FIRST_CALL_TRANSACTION + 0);

        public static boolean setDefaultImpl(remote.IStudentService impl) {
            if (Stub.Proxy.sDefaultImpl == null && impl != null) {
                Stub.Proxy.sDefaultImpl = impl;
                return true;
            }
            return false;
        }

        public static remote.IStudentService getDefaultImpl() {
            return Stub.Proxy.sDefaultImpl;
        }
    }

    public remote.Student getStudentById(int id) throws android.os.RemoteException;
}

```

自定义服务进行跨进程通信的过程：

（1）创建一个AIDL文件，内部定义了服务端进程想要提供给客户端进程的方法列表，然后系统会生成对应的一个同名Java文件（AndroidStudio中右键选项生成AIDL文件，才能编译出Java文件）。（AIDL文件不是必须的，它只是方便自动生成代码，我们也可以手写Binder。）

（2）自定义服务通过Service实现，在里面实现提供给客户端的方法的具体逻辑，之后客户端bindService后，在Service的onBind方法返回这个服务端对应的Binder对象，然后客户端就能在**ServiceConnection中onServiceConnected**这个回调方法当中获得**服务端对应的Binder对象**；

（3）客户端获得了Binder对象之后会调用例如IStudentService.**Stub.asInterface(IBinder service)**将Binder对象传入，获取remote.IStudentService对象，从这里开始就跟跨进程通信分开了。**如果是跨进程通信asInterface返回的是IStudentService.Stub.Proxy代理对象**，以后客户端调用服务端的方法实际上就是调用IStudentService.Stub.Proxy代理对象里声明的服务端的方法，**通过transact陷入内核态来进行实际上的进程间通信去调用服务端的onTransact方法，在onTransact方法中会根据传入标志调用不同的服务端方法**； 而如果是同进程通信的话，asInterface返回的是IStudentService.Stub对象，然后直接调用服务端的方法，而不必陷入内核态执行了，可以是在Service类当中继承IStudentService.Stub抽象类实现抽象方法然后在onBind（）方法返回这个实现类的实例。

![image-ipc_aidl](/Users/chenying/Documents/学习笔记/TyporaImages/image-ipc_aidl.jpg)

在使用AIDL实现Binder通信的过程中，无论服务端方法还是客户端方法都是运行在各自的Binder线程池中的，如果我们想要更新UI的话，就需要用到Handler进行切换操作。



**AIDL**

Binder通信机制的具体应用。



**常用系统服务**

| 传入的Name              | 返回的对象          | 说明                   |
| ----------------------- | ------------------- | ---------------------- |
| WINDOW_SERVICE          | WindowManager       | 管理打开的窗口程序     |
| ACTIVITY_SERVICE        | ActivityManager     | 管理应用程序的系统状态 |
| NOTIFICATION_SERVICE    | NotificationManager | 状态栏服务             |
| LAYOUT_INFLATER_SERVICE | LayoutManager       | 取得XML里面定义的View  |
| ALARM_SERVICE           | AlarmManager        | 闹钟服务               |
| POWER_SERVICE           | PowerManager        | 电源服务               |

获取示例：

WindowManager windowManager= Context.getSystemService(Context.WINDOW_SERVICE)；



**参考**

[BInder系列开篇](http://gityuan.com/2015/10/31/binder-prepare/)

[Binder原理剖析](https://juejin.im/post/5acccf845188255c3201100f)未看完