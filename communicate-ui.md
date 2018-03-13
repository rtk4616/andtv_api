# Communicating with the UI Thread

## This lesson teaches you to

<span style="background-color:Cyan">[本課教你如何]</span>

  1. Define a Handler on the UI Thread

<span style="background-color:Cyan">[在UI線程上定義一個處理程序]</span>

  2. Move Data from a Task to the UI Thread

<span style="background-color:Cyan">[將數據從任務移至UI線程]</span>

## You should also read

<span style="background-color:Cyan">[你也應該閱讀]</span>

  * [Processes and Threads [ 進程和線程](https://developer.android.com/guide/components/processes-and-threads.html)

## Try it out

[Download the sample](https://developer.android.com/shareables/training/ThreadSample.zip)

<span style="background-color:Cyan">[試試看\n下載樣本]</span>

ThreadSample.zip

<span style="background-color:Cyan">[ThreadSample.zip]</span>

In the previous lesson you learned how to start a task on a thread managed by [ThreadPoolExecutor](https://developer.android.com/reference/java/util/concurrent/ThreadPoolExecutor.html).

<span style="background-color:Cyan">[在上一課中，您學習瞭如何在由ThreadPoolExecutor管理的線程上啟動任務。]</span>

This final lesson shows you how to send data from the task to objects running on the user interface (UI) thread.

<span style="background-color:Cyan">[本最後一課向您展示瞭如何將數據從任務發送到在用戶界面（UI）線程上運行的對象。]</span>

This feature allows your tasks to do background work and then move the results to UI elements such as bitmaps.

<span style="background-color:Cyan">[此功能允許您的任務執行後台工作，然後將結果移至UI元素（如位圖）。]</span>

Every app has its own special thread that runs UI objects such as [View](https://developer.android.com/reference/android/view/View.html) objects; this thread is called the UI thread.

<span style="background-color:Cyan">[每個應用程序都有自己的特殊線程來運行UI對象，比如View對象，這個線程被稱為UI線程。]</span>

Only objects running on the UI thread have access to other objects on that thread.

<span style="background-color:Cyan">[只有在UI線程上運行的對象才能訪問該線程上的其他對象。]</span>

Because tasks that you run on a thread from a thread pool _aren't_ running on your UI thread, they don't have access to UI objects.

<span style="background-color:Cyan">[因為您在線程池中的線程上運行的任務未在UI線程上運行，所以他們無法訪問UI對象。]</span>

To move data from a background thread to the UI thread, use a [Handler](https://developer.android.com/reference/android/os/Handler.html) that's running on the UI thread.

<span style="background-color:Cyan">[要將數據從後台線程移動到UI線程，請使用在UI線程上運行的Handler。]</span>

## Define a Handler on the UI Thread

<span style="background-color:Cyan">[在UI線程上定義一個處理程序]</span>

* * *

[Handler](https://developer.android.com/reference/android/os/Handler.html) is part of the Android system's framework for managing threads.

<span style="background-color:Cyan">[Handler是Android系統管理線程框架的一部分。]</span>

A [Handler](https://developer.android.com/reference/android/os/Handler.html) object receives messages and runs code to handle the messages.

<span style="background-color:Cyan">[Handler對象接收消息並運行代碼來處理消息。]</span>

Normally, you create a [Handler](https://developer.android.com/reference/android/os/Handler.html) for a new thread, but you can also create a [Handler](https://developer.android.com/reference/android/os/Handler.html) that's connected to an existing thread.

<span style="background-color:Cyan">[通常，你為一個新線程創建一個Handler，但是你也可以創建一個連接到一個現有線程的Handler。]</span>

When you connect a [Handler](https://developer.android.com/reference/android/os/Handler.html) to your UI thread, the code that handles messages runs on the UI thread.

<span style="background-color:Cyan">[將處理程序連接到UI線程時，處理消息的代碼在UI線程上運行。]</span>


Instantiate the [Handler](https://developer.android.com/reference/android/os/Handler.html) object in the constructor for the class that creates your thread pools, and store the object in a global variable.

<span style="background-color:Cyan">[在創建線程池的類的構造函數中實例化Handler對象，並將該對象存儲在全局變量中。]</span>

Connect it to the UI thread by instantiating it with the [Handler(Looper)](https://developer.android.com/reference/android/os/Handler.html#Handler\(android.os.Looper\)) constructor.

<span style="background-color:Cyan">[通過使用Handler（Looper）構造函數實例化它，將它連接到UI線程。]</span>

This constructor uses a [Looper](https://developer.android.com/reference/android/os/Looper.html) object, which is another part of the Android system's thread management framework.

<span style="background-color:Cyan">[這個構造函數使用Looper對象，它是Android系統線程管理框架的另一部分。]</span>

When you instantiate a [Handler](https://developer.android.com/reference/android/os/Handler.html) based on a particular [Looper](https://developer.android.com/reference/android/os/Looper.html) instance, the [Handler](https://developer.android.com/reference/android/os/Handler.html) runs on the same thread as the [Looper](https://developer.android.com/reference/android/os/Looper.html).

<span style="background-color:Cyan">[當您基於特定的Looper實例實例化Handler時，Handler與Looper在同一線程上運行。]</span>

For example:

<span style="background-color:Cyan">[例如：]</span>



     private PhotoManager() {
    ...
        // Defines a Handler object that's attached to the UI thread
        mHandler = new Handler(Looper.getMainLooper()) {
        ...

Inside the
[Handler](https://developer.android.com/reference/android/os/Handler.html),
override the
[handleMessage()](https://developer.android.com/reference/android/os/Handler.html#handleMessage\(android.os.Message\))
method.

<span style="background-color:Cyan">[在Handler內部，重寫handleMessage（）方法。]</span>

The Android system invokes this method when it receives a new message for a thread it's managing; all of the [Handler](https://developer.android.com/reference/android/os/Handler.html) objects for a particular thread receive the same message.

<span style="background-color:Cyan">[當Android系統收到它正在管理的線程的新消息時，將調用此方法，特定線程的所有Handler對像都會收到相同的消息。]</span>

For example:

<span style="background-color:Cyan">[例如：]</span>



            /*
             * handleMessage() defines the operations to perform when
             * the Handler receives a new Message to process.
             */
            @Override
            public void handleMessage(Message inputMessage) {
                // Gets the image task from the incoming Message object.
                PhotoTask photoTask = (PhotoTask) inputMessage.obj;
                ...
            }
        ...
        }
    }

The next section shows how to tell the [Handler](https://developer.android.com/reference/android/os/Handler.html) to move data.

<span style="background-color:Cyan">[下一節將介紹如何告訴Handler移動數據。]</span>


## Move Data from a Task to the UI Thread

<span style="background-color:Cyan">[將數據從任務移至UI線程]</span>


* * *

To move data from a task object running on a background thread to an object on the UI thread, start by storing references to the data and the UI object in the task object.

<span style="background-color:Cyan">[要將數據從後台線程上運行的任務對象移動到UI線程上的對象，請首先將對數據和UI對象的引用存儲在任務對像中。]</span>

Next, pass the task object and a status code to the object that instantiated the [Handler](https://developer.android.com/reference/android/os/Handler.html).

<span style="background-color:Cyan">[接下來，將任務對象和狀態碼傳遞給實例化Handler的對象。]</span>

In this object, send a [Message](https://developer.android.com/reference/android/os/Message.html) containing the status and the task object to the [Handler](https://developer.android.com/reference/android/os/Handler.html).

<span style="background-color:Cyan">[在此對像中，將包含狀態和任務對象的消息發送到處理程序。]</span>

Because [Handler](https://developer.android.com/reference/android/os/Handler.html) is running on the UI thread, it can move the data to the UI object.

<span style="background-color:Cyan">[因為Handler在UI線程上運行，所以它可以將數據移動到UI對象。]</span>


### Store data in the task object

<span style="background-color:Cyan">[將數據存儲在任務對像中]</span>


For example, here's a [Runnable](https://developer.android.com/reference/java/lang/Runnable.html), running on a background thread, that decodes a [Bitmap](https://developer.android.com/reference/android/graphics/Bitmap.html) and stores it in its parent object `PhotoTask`.

<span style="background-color:Cyan">[例如，下面是一個運行在後台線程上的Runnable，它解碼一個Bitmap並將其存儲在其父對象PhotoTask中。]</span>

The [Runnable](https://developer.android.com/reference/java/lang/Runnable.html) also stores the status code `DECODE_STATE_COMPLETED`.

<span style="background-color:Cyan">[Runnable還存儲狀態碼DECODE_STATE_COMPLETED。]</span>



    // A class that decodes photo files into Bitmaps
    class PhotoDecodeRunnable implements Runnable {
        ...
        PhotoDecodeRunnable(PhotoTask downloadTask) {
            mPhotoTask = downloadTask;
        }
        ...
        // Gets the downloaded byte array
        byte[] imageBuffer = mPhotoTask.getByteBuffer();
        ...
        // Runs the code for this task
        public void run() {
            ...
            // Tries to decode the image buffer
            returnBitmap = BitmapFactory.decodeByteArray(
                    imageBuffer,
                    0,
                    imageBuffer.length,
                    bitmapOptions
            );
            ...
            // Sets the ImageView Bitmap
            mPhotoTask.setImage(returnBitmap);
            // Reports a status of "completed"
            mPhotoTask.handleDecodeState(DECODE_STATE_COMPLETED);
            ...
        }
        ...
    }
    ...

`PhotoTask` also contains a handle to the [ImageView](https://developer.android.com/reference/android/widget/ImageView.html) that displays the [Bitmap](https://developer.android.com/reference/android/graphics/Bitmap.html).

<span style="background-color:Cyan">[PhotoTask還包含顯示Bitmap的ImageView句柄。]</span>

Even though references to the [Bitmap](https://developer.android.com/reference/android/graphics/Bitmap.html) and [ImageView](https://developer.android.com/reference/android/widget/ImageView.html) are in the same object, you can't assign the [Bitmap](https://developer.android.com/reference/android/graphics/Bitmap.html) to the [ImageView](https://developer.android.com/reference/android/widget/ImageView.html), because you're not currently running on the UI thread.

<span style="background-color:Cyan">[即使對Bitmap和ImageView的引用位於同一個對像中，也不能將Bitmap分配給ImageView，因為您目前不在UI線程上運行。]</span>


Instead, the next step is to send this status to the `PhotoTask` object.

<span style="background-color:Cyan">[相反，下一步是將此狀態發送到PhotoTask對象。]</span>


### Send status up the object hierarchy

<span style="background-color:Cyan">[向對象層次發送狀態]</span>


`PhotoTask` is the next higher object in the hierarchy.

<span style="background-color:Cyan">[PhotoTask是層次結構中的下一個更高級的對象。]</span>

It maintains references to the decoded data and the [View](https://developer.android.com/reference/android/view/View.html) object that will show the data.

<span style="background-color:Cyan">[它保持對解碼數據和將顯示數據的View對象的引用。]</span>

It receives a status code from `PhotoDecodeRunnable` and passes it along to the object that maintains thread pools and instantiates [Handler](https://developer.android.com/reference/android/os/Handler.html):

<span style="background-color:Cyan">[它從PhotoDecodeRunnable接收狀態碼，並將其傳遞給維護線程池並實例化Handler的對象：]</span>



    public class PhotoTask {
        ...
        // Gets a handle to the object that creates the thread pools
        sPhotoManager = PhotoManager.getInstance();
        ...
        public void handleDecodeState(int state) {
            int outState;
            // Converts the decode state to the overall state.
            switch(state) {
                case PhotoDecodeRunnable.DECODE_STATE_COMPLETED:
                    outState = PhotoManager.TASK_COMPLETE;
                    break;
                ...
            }
            ...
            // Calls the generalized state method
            handleState(outState);
        }
        ...
        // Passes the state to PhotoManager
        void handleState(int state) {
            /*
             * Passes a handle to this task and the
             * current state to the class that created
             * the thread pools
             */
            sPhotoManager.handleState(this, state);
        }
        ...
    }

### Move data to the UI

<span style="background-color:Cyan">[將數據移動到用戶界面]</span>


From the `PhotoTask` object, the `PhotoManager` object receives a status code and a handle to the `PhotoTask` object.

<span style="background-color:Cyan">[從PhotoTask對像中，PhotoManager對象接收狀態碼和PhotoTask對象的句柄。]</span>

Because the status is `TASK_COMPLETE`, creates a [Message](https://developer.android.com/reference/android/os/Message.html) containing the state and task object and sends it to the [Handler](https://developer.android.com/reference/android/os/Handler.html):

<span style="background-color:Cyan">[由於狀態為TASK_COMPLETE，因此會創建一個包含狀態和任務對象的消息並將其發送給處理程序：]</span>



    public class PhotoManager {
        ...
        // Handle status messages from tasks
        public void handleState(PhotoTask photoTask, int state) {
            switch (state) {
                ...
                // The task finished downloading and decoding the image
                case TASK_COMPLETE:
                    /*
                     * Creates a message for the Handler
                     * with the state and the task object
                     */
                    Message completeMessage =
                            mHandler.obtainMessage(state, photoTask);
                    completeMessage.sendToTarget();
                    break;
                ...
            }
            ...
        }

Finally, [Handler.handleMessage()](https://developer.android.com/reference/android/os/Handler.html#handleMessage\(android.os.Message\)) checks the status code for each incoming [Message](https://developer.android.com/reference/android/os/Message.html).

<span style="background-color:Cyan">[最後，Handler.handleMessage（）檢查每個傳入消息的狀態碼。]</span>

If the status code is `TASK_COMPLETE`, then the task is finished, and the `PhotoTask` object in the [Message](https://developer.android.com/reference/android/os/Message.html) contains both a [Bitmap](https://developer.android.com/reference/android/graphics/Bitmap.html) and an [ImageView](https://developer.android.com/reference/android/widget/ImageView.html).

<span style="background-color:Cyan">[如果狀態碼是TASK_COMPLETE，則任務完成，並且Message中的PhotoTask對象同時包含位圖和ImageView。]</span>

Because [Handler.handleMessage()](https://developer.android.com/reference/android/os/Handler.html#handleMessage\(android.os.Message\)) is running on the UI thread, it can safely move the [Bitmap](https://developer.android.com/reference/android/graphics/Bitmap.html) to the [ImageView](https://developer.android.com/reference/android/widget/ImageView.html):

<span style="background-color:Cyan">[因為Handler.handleMessage（）在UI線程上運行，所以它可以安全地將位圖移動到ImageView：]</span>



        private PhotoManager() {
            ...
                mHandler = new Handler(Looper.getMainLooper()) {
                    @Override
                    public void handleMessage(Message inputMessage) {
                        // Gets the task from the incoming Message object.
                        PhotoTask photoTask = (PhotoTask) inputMessage.obj;
                        // Gets the ImageView for this task
                        PhotoView localView = photoTask.getPhotoView();
                        ...
                        switch (inputMessage.what) {
                            ...
                            // The decoding is done
                            case TASK_COMPLETE:
                                /*
                                 * Moves the Bitmap from the task
                                 * to the View
                                 */
                                localView.setImageBitmap(photoTask.getImage());
                                break;
                            ...
                            default:
                                /*
                                 * Pass along other messages from the UI
                                 */
                                super.handleMessage(inputMessage);
                        }
                        ...
                    }
                    ...
                }
                ...
        }
    ...
    }


<span style="background-color:Cyan">[上一页  arrow_back  Running Code on a Thread Pool Thread ](https://developer.android.com/training/multiple-threads/run-code.html)


