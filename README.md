# Handler-
Handler在多线程间进行消息通信，维持handler机制的主要控件：Handler+Message+MessageQueue+Looper

Handler机制及其相关用法：
在andriod世界里handler的作用主要用来处理线程间通信，即我们常说的消息机制，那到底什么是handler机制呢？
假设在一个Activity中，有多个线程同时要更新UI，并且没有进行枷锁操作，想想会怎样？界面会很混乱，而且当我们一个线程已经更新完一个UI但是线程之间不是同步的，导致
另外一个线程更新不到最新的数据，这就在逻辑上必然出错，但是如果对更新UI操作进行枷锁处理的话，大大降低主线程进行UI更新效率，加入简简单单更新一个控件状态，却要对该控件
进行枷锁不仅降低我们主线程工作效率，带来的用户体验也是很差的，作为开发者对我们来说影响也很大，让我们逻辑变得臃肿，所以为了很好解决线程间这种通讯，google则
为我们提供这个消息机制，当然考虑到handler机制切换线程和处理消息逻辑相对负责，Google大牛又为我们封装了另外一个Api,相信大家知道AsyncTask,但是看过源码的小伙伴
就知道其实AsyncTask内部封装了Handler机制，对Handler进一步完善和封装，到底Handler机制是怎么更新主线程消息呢？？？ （废话多多）

（回到正题！！！）
我们要知道线程默认没有Looper的，如果需要使用handler就需要为线程创建Looper,我们经常提起主线程也叫UI线程（ActivityThread），
ActivityThread初始化的时候就已经创建了Looper，这也就解释了为什么我们在主线程可以直接使用Handler处理相关逻辑，事实上主线程已经为我们创建了Looper,
那么Looper是怎么使用呢？  一个线程只对应一个Looper,一个Looper只能对应一个MessageQueue,当我们创建Looper后，Looper内部为我们创建一个MessageQueue
当我们在子线程种使用handler,记得子线程没人没有Looper的哦，记得Looper.prepare()一下为当前线程创建Looper，不然会报错，那么接下来Looper.loop()不断进行
消息轮询，不断从messageQueue中取出消息，当没有消息时候就等待，这就导致阻塞线程，当MessageQueue里面没有消息我们可以手动quit()退出当前Looper,那么我们当前
的handler就不能发送消息了，在Looper内部进行消息循环时，MessageQueue通过next()将消息分别取出根据分发机制将消息分发给对应的Handler处理。

handler用来处理MessageQueue中message,通过handler的handleMessage(Message msg)处理相关逻辑，例如更新UI或者处理主线程逻辑，
一般来说，我们想执行一个耗时操作例如网络加载，大量图片下载或者庞大数据的访问，我们都知道主线程不能执行耗时操作，否则发生ANR，那么android在解决这个问题上
给我们很好一个答案“多线程”操作，我们通过在主线程中重新创建一个子线程用来处理耗时逻辑，这种异步线程机制很好解决线程阻塞的问题，其实看过源码的宝宝们就知道，
handler机制内部就是通过开启子线程实现不同线程之间消息通信。
当我们开启了一个子线程后将子线程处理得到信息想通知给主线程怎么办法，这个时候我们就需要用到message，message是消息类，我们通过Message.obtian（）拿到
message类，当然并不止这一种方法，但归根结底就是Message.obtian方法，顾名思义我们拿到message后就可以携带我们想要传递的消息，可以通过arg1和arg2传递
int型数据，也可以通过setData()向Bundle中传递复杂数据类型，当然message强大在与你可以直接传递一个Object对象给它，
当消息我们封装好了此时就需要将message发送出去，即用sendMessge(message)将消息发送到消息队列MessageQueue，sendMessage()又调用sendMessageDelay(),
sendMessageDelay又调用sendMessageAttime()，Handler中还有一系列的sendEmptyMessageXXX方法，而这些sendEmptyMessageXXX方法在其内部又分别调用了
其对应的sendMessageXXX方法。
兄台且慢，发送消息这里还有个postXXX方法，会发现postXXX方法在其内部又调用了对应的sendMessageXXX方法，可以看看源码。
那么我们又是怎么处理消息的呢？  还记得刚才我们说过的MessageQueue吗？ 对，就是它将我们消息收入囊中，大家应该也都能猜到他是干什么的，当我们象处理消息时
就从这个消息队列里面将其循环取出，怎么取出呢？？？  这就需要你去研究源码，handler内部维持了一个消息循环机制，按照这个机制在Looper推动下将消息一条一条
取出分别分发给handler，最终我们可以将拿到的消息处理我们的逻辑，这就是整个handler消息机制流程，由于鄙人才学疏浅讲的不是很透彻，大家可以多研究研究源码
，源码里面有很多google高精深的设计思想和设计模式。
