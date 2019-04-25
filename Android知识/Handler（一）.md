# Handler

## 定义
一套 Android 消息传递机制 / 异步通信机制。

## 作用
在多线程的应用场景中，将工作线程中需更新UI的操作信息传递到UI主线程，从而实现工作线程对UI的更新处理，最终实现异步消息的处理。

## 意义
使用Handler的原因：将工作线程需操作UI的消息传递到主线程，使得主线程可根据工作线程的需求更新UI，从而避免线程操作不安全的问题

## 使用方式
Handler使用方式 因发送消息到消息队列的方式不同而不同
共分为2种：使用Handler.sendMessage（）、使用Handler.post（）

## 具体使用
### 一、Handler.sendMessage()

```
/** 
  * 方式1：新建Handler子类（内部类）
  */

    // 步骤1：自定义Handler子类（继承Handler类） & 复写handleMessage（）方法
    class mHandler extends Handler {

        // 通过复写handlerMessage() 从而确定更新UI的操作
        @Override
        public void handleMessage(Message msg) {
         ...// 需执行的UI操作
            
        }
    }

    // 步骤2：在主线程中创建Handler实例
        private Handler mhandler = new mHandler();

    // 步骤3：创建所需的消息对象
        Message msg = Message.obtain(); // 实例化消息对象
        msg.what = 1; // 消息标识
        msg.obj = "AA"; // 消息内容存放

    // 步骤4：在工作线程中 通过Handler发送消息到消息队列中
    // 可通过sendMessage（） / post（）
    // 多线程可采用AsyncTask、继承Thread类、实现Runnable
        mHandler.sendMessage(msg);

    // 步骤5：开启工作线程（同时启动了Handler）
    // 多线程可采用AsyncTask、继承Thread类、实现Runnable


/** 
  * 方式2：匿名内部类
  */
   // 步骤1：在主线程中 通过匿名内部类 创建Handler类对象
            private Handler mhandler = new  Handler(){
                // 通过复写handlerMessage()从而确定更新UI的操作
                @Override
                public void handleMessage(Message msg) {
                        ...// 需执行的UI操作
                    }
            };

  // 步骤2：创建消息对象
    Message msg = Message.obtain(); // 实例化消息对象
  msg.what = 1; // 消息标识
  msg.obj = "AA"; // 消息内容存放
  
  // 步骤3：在工作线程中 通过Handler发送消息到消息队列中
  // 多线程可采用AsyncTask、继承Thread类、实现Runnable
   mHandler.sendMessage(msg);

  // 步骤4：开启工作线程（同时启动了Handler）
  // 多线程可采用AsyncTask、继承Thread类、实现Runnable

```

### 二、使用Handler.post()

```
// 步骤1：在主线程中创建Handler实例
    private Handler mhandler = new mHandler();

    // 步骤2：在工作线程中 发送消息到消息队列中 & 指定操作UI内容
    // 需传入1个Runnable对象
    mHandler.post(new Runnable() {
            @Override
            public void run() {
                ... // 需执行的UI操作 
            }

    });

    // 步骤3：开启工作线程（同时启动了Handler）
    // 多线程可采用AsyncTask、继承Thread类、实现Runnable
```


