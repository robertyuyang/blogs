原文 https://medium.com/flutter/flutter-platform-channels-ce7f540a104e

“Nice UI. But how does Flutter deal with platform-specific APIs?”
Flutter invites you to write your mobile app in the Dart programming language and build for both Android and iOS. But Dart does not compile to Android’s Dalvik bytecode, nor are you blessed with Dart/Objective-C bindings on iOS. This means that your Dart code is written without direct access to the platform-specific APIs of iOS Cocoa Touch and the Android SDK.
This isn’t much of a problem as long as you are just writing Dart to paint pixels on the screen. The Flutter framework and its underlying graphics engine are very capable of that on their own. Nor is it a problem if everything you do besides painting pixels is file or network I/O and associated business logic. The Dart language, runtime, and libraries have you covered there.

“UI确实不错，但是Flutter要怎么跟平台相关API进行交互呢？”

Flutter允许你用Dart编程语言同时为Android系统和iOS系统编写你的移动端App，但是Dart不会编译成安卓的Dalvik字节码，你也别指望在iOS系统上Dart和Objective-C能有多亲密，这意味着你的Dart代码是没有对iOS Cocoa和Android SDK这些平台相关API直接访问的能力的。如果你只是想在屏幕上画上一些像素点，那刚才说的都不是什么大问题。Flutter框架和它底层的图形引擎自己就非常有能力处理这些工作。如果你除了画像素之外，还要做一些文件和网络的IO以及相关的业务逻辑，那这也不是什么问题，Dart语言，runtime和相关的库都能帮你搞定。

But non-trivial apps require deeper integration with the host platform:
notifications, app lifecycle, deep links, …
sensors, camera, battery, geolocation, sound, connectivity, …
sharing information with other apps, launching other apps, …
persisted preferences, special folders, device information, …
The list is long and broad and seems to grow with every platform release.
Access to all of these platform APIs could be baked into the Flutter framework itself. But that would make Flutter a lot bigger and give it many more reasons to change. In practice, that would likely lead to Flutter lagging behind the latest platform release. Or landing app authors with unsatisfactory “least common denominator” wrappings of platform APIs. Or puzzling newcomers with unwieldy abstractions to paper over platform differences. Or version fragmentation. Or bugs.
Come to think of it, probably all of the above.

但是，重要一点的App都需要与主宿平台进行更深层次的交互：

- 通知，App生命周期，deep links，...
- 传感器，摄像头，电池，地理信息，声音，连接性，...
- 与其他App共享信息，唤起其他App，...
- 持久化的偏好项，特定的目录，设备信息，...

这个列表随着每个系统平台版本的发布还会越来越长，范围越来越大。

所有这些对这些系统API的访问可以全部集成到Flutter框架内部。但是这回让Flutter变得非常臃肿、并且很多情况下都要进行改变。现实情况更是很有很可能让Flutter滞后于最新的系统发布版本，要么就系统API做一个让人难受的的“最小公分母”封装给代码开发者，要么就搞一些让初学者特别困惑的难以理解的抽象概念来掩盖平台版本的差异。也可能导致版本的脆弱和bug。

好好考虑一下上面提到的这些吧。


The Flutter team chose a different approach. It doesn’t do all that much, but it is simple, versatile, and completely in your hands.

First of all, Flutter is hosted by an ambient Android or iOS app. The Flutter parts of the app are wrapped in standard platform-specific components such as View on Android and UIViewController on iOS. So while Flutter invites you to write your app in Dart, you can do as much or as little in Java/Kotlin or Objective-C/Swift as you please in the host app, working directly on top of platform-specific APIs.

Second, platform channels provide a simple mechanism for communicating between your Dart code and the platform-specific code of your host app. This means you can expose a platform service in your host app code and have it invoked from the Dart side. Or vice versa.

And third, plugins make it possible to create a Dart API backed by an Android implementation written in Java or Kotlin and an iOS implementation written in Objective-C or Swift — and package that up as a Flutter/Android/iOS triple glued together using platform channels. This means you can reuse, share, and distribute your take on how Flutter should use a particular platform API.

Flutter团队则另辟蹊径，它不会做所有那些工作，而是保持简单和多样化，给你更多操作空间。

首先，Flutter会寄宿于一个容器性的Android或iOS应用。应用的Flutter部分会封装再一个平台相关组件中，比如iOS的UIViewController或者Android的View。这样当Flutter让你用Dart编写应用的时候，你可以随意愿在宿主应用中编写多少Java/Kotlin或者Objective-C/Swift代码，可以直接操作平台相关的API。

其次，平台通道（Platform channels）提供了一个可以让你的Dart代码和宿主应用中的平台相关代码之间一个简单的通信机制。这意味着你可以在宿主App里暴露一个平台服务，然后给你的Dart代码调用，反之亦然。

第三，用插件（Plugin）就有可能让我们让创建一个Dart API，而其背后是用Java/Kotlin实现的Android代码或Objective-C/Swift实现的iOS代码。然后用平台通道把他们包裹成一个Flutter/Android/iOS三方粘合的包。这意味着您可以重用、分享并且分发你用Flutter调用一个平台相关的API的片段。

This article is an in-depth introduction to platform channels. Starting from Flutter’s messaging foundations, I’ll introduce the message/method/event channel concepts, and discuss some API design considerations. There’ll be no API listings, but short code samples for copy-paste reuse instead. A brief list of usage guidelines is provided, based on my experience contributing to the flutter/plugins GitHub repository as a member of the Flutter team. The article concludes with a list of additional resources, including links to the DartDoc/JavaDoc/ObjcDoc reference APIs.

这篇文章是一个对平台通道的深入介绍。从Flutter的消息发送（messageing）功能讲起，我会介绍message/method/event channel的概念，并讨论一些API设计的思考。这里不会有API列表，而会有一些能够直接复制粘贴重用的短的代码例子。我会根据我作为Flutter团队一员给Flutter/plugins Github仓库的贡献代码经验，给出了一个简单的使用指南。这篇文章还会总结一些额外的资源，包括一些DartDoc/JavaDoc/ObjcDoc reference APIs的链接。

Table of contents

Platform channels API
Foundations: asynchronous, binary messaging
Message channels: name + codec
Method channels: standardized envelopes
Event channels: streaming

Usage guidelines
Prefix channel names by domain for uniqueness
Consider treating platform channels as intra-module communication
Don’t mock platform channels
Consider automated testing for your platform interaction
Keep platform side ready for incoming synchronous calls
Resources


### 平台通道API（Platform channels API）

For most use cases you would probably employ method channels for platform communication. But since many of their properties are derived from the simpler message channels and from the underlying binary messaging foundations, I’ll start there.

在绝大多数的用例中，你应该用method channel来做平台通信。但是既然它们很多的属性都是从更简单的message channel和更底层的binary messaging功能继承来的，那我们还是从这些开始讲起。

#### Foundations: asynchronous, binary messaging

基础：异步、二进制通信

![image](https://miro.medium.com/max/1400/1*BIknuDE2gMHmbYg7S8_F0Q.png)

At the most basic level, Flutter talks to platform code using asynchronous message passing with binary messages — meaning the message payload is a byte buffer. To distinguish between messages used for different purposes, each message is sent on a logical “channel” which is just a name string. The examples below use the channel name foo.


在最底层，Flutter是通过异步的传输二进制消息来完成与平台相关代码的通信的，这意味着消息的载体是一个字节缓冲。为了实现不同消息能用作不同的用途，每一个消息在一个逻辑“通道”上发送，这个“通道”实际就是一个字符串的名字。下面这个例子使用了名为foo的通道。


```
// Send a binary message from Dart to the platform.
final WriteBuffer buffer = WriteBuffer()
  ..putFloat64(3.1415)
  ..putInt32(12345678);
final ByteData message = buffer.done();
await BinaryMessages.send('foo', message);
print('Message sent, reply ignored');
```
On Android such a message can be received, as a java.nio.ByteBuffer, using the following Kotlin code:

在安卓系统里这样的一个消息可以用下面的Kotlin代码作为一个java.nio.ByteBuffer来接收。



```
// Receive binary messages from Dart on Android.
// This code can be added to a FlutterActivity subclass, typically
// in onCreate.
flutterView.setMessageHandler("foo") { message, reply ->
  message.order(ByteOrder.nativeOrder())
  val x = message.double
  val n = message.int
  Log.i("MSG", "Received: $x and $n")
  reply.reply(null)
}
```


The ByteBuffer API supports reading off primitive values while automatically advancing the current read position. The story on iOS is similar; suggestions for improving my weak Swift fu are very welcome:

ByteBuffer API支持从一个原始值里进行读取的同事自动的前进读取位置。在iOS上的情况是比较类似的。欢迎对如何提高我孱弱的Swift提出建议：


```
// Receive binary messages from Dart on iOS.
// This code can be added to a FlutterAppDelegate subclass,
// typically in application:didFinishLaunchingWithOptions:.
let flutterView =
  window?.rootViewController as! FlutterViewController;
flutterView.setMessageHandlerOnChannel("foo") {
  (message: Data!, reply: FlutterBinaryReply) -> Void in
  let x : Float64 = message.subdata(in: 0..<8)
    .withUnsafeBytes { $0.pointee }
  let n : Int32 = message.subdata(in: 8..<12)
    .withUnsafeBytes { $0.pointee }
  os_log("Received %f and %d", x, n)
  reply(nil)
}
```

Communication is bidirectional, so you can send messages in the opposite direction too, from Java/Kotlin or Objective-C/Swift to Dart. Reversing the direction of the above setup looks as follows:

通信往往是双向的，因此你也可以向反方向发消息。从Java/Kotlin或者Objective-C/Swift向Dart。反转上面的通信的话，代码是这个样子：



```
// Send a binary message from Android.
val message = ByteBuffer.allocateDirect(12)
message.putDouble(3.1415)
message.putInt(123456789)
flutterView.send("foo", message) { _ ->
  Log.i("MSG", "Message sent, reply ignored")
}

// Send a binary message from iOS.
var message = Data(capacity: 12)
var x : Float64 = 3.1415
var n : Int32 = 12345678
message.append(UnsafeBufferPointer(start: &x, count: 1))
message.append(UnsafeBufferPointer(start: &n, count: 1))
flutterView.send(onChannel: "foo", message: message) {(_) -> Void in
  os_log("Message sent, reply ignored")
}

// Receive binary messages from the platform.
BinaryMessages.setMessageHandler('foo', (ByteData message) async {
  final ReadBuffer readBuffer = ReadBuffer(message);
  final double x = readBuffer.getFloat64();
  final int n = readBuffer.getInt32();
  print('Received $x and $n');
  return null;
});
```


The fine print. Mandatory replies. Each message send involves an asynchronous reply from the receiver. In the examples above, there is no interesting value to communicate back, but the null reply is necessary for the Dart future to complete and for the two platform callbacks to execute.
Threads. Messages and replies are received, and must be sent, on the platform’s main UI thread. In Dart there is only one thread per Dart isolate, i.e. per Flutter view, so there is no confusion about which thread to use here.
Exceptions. Any uncaught exception thrown in a Dart or Android message handler is caught by the framework, logged, and a null reply is sent back to the sender. Uncaught exceptions thrown in reply handlers are logged.
Handler lifetime. Registered message handlers are retained and kept alive along with the Flutter view (meaning the Dart isolate, the Android FlutterView instance, and the iOS FlutterViewController). You can cut a handler’s life short by deregistering it: just set a null (or different) handler using the same channel name.

消息收到后被打印了出来，并给以合法的返回。任何一个消息发送都涉及到一个从接受方的异步回复。在上面的例子中，没有另一方感兴趣的值需要通信回去，但是对于Dart的Future来说也有必要回复null来执行跨平台的回调。

*线程*。消息和回复的发送和接受必须在平台的主线程中进行。在Dart环境里对每一个Dart isolate里只会存在一个线程，也就是Flutter View。因此这里使用哪个线程没什么好纠结的。

*异常*。任何一个在Dart或者Android消息处理中被抛出的异常都会被框架捕捉到，并记录下来，然后一个null回复会被发送回发送者那里。在回复处理中抛出的为捕捉异常也会被记录下来。

*处理程序的生命周期*。注册过的消息处理程序会被FlutterView（也就是Dart isolate，Android的FlutterView实例、iOS的FlutterViewController）持有并保持生命周期一致。想结束它的生命周期可以通过反注册的方式：只要给相同的通道名设置一个空的或者不同的消息处理。




Handler uniqueness. Handlers are held in a hash map keyed by channel name, so there can be at most one handler per channel. A message sent on a channel for which no message handler is registered at the receiving end is responded to automatically, using a null reply.
Synchronous communication. Platform communication is available in asynchronous mode only. This avoids making blocking calls across threads and the system-level problems that might entail (poor performance, risk of deadlocks). At the time of this writing, it is not entirely clear if synchronous communication is really needed in Flutter and, if so, in what form.
Working at the level of binary messages, you need to worry about delicate details like endianness and how to represent higher-level messages such as strings or maps using bytes. You also need to specify the right channel name whenever you want to send a message or register a handler. Making this easier leads us to platform channels:
A platform channel is an object that brings together a channel name and a codec for serializing/deserializing messages to binary form and back.

*处理程序唯一性*。处理程序会在一张HashMap里以通道名为key持有，这样基本上做到一个通道一个处理。消息如果发给一个没有任何处理程序注册的通道时，在接收端会自动使用null来回复。

*异步通信*。平台通信只能再异步模式下进行。这样可以避免跨线程的阻塞调用，也可以避免一些系统级别的问题（性能损失，死锁风险）。在写本篇的时候，在Flutter里是否真的需要异步通信，如果需要的话应该是什么形式，还不完全的清楚。

Working at the level of binary messages, you need to worry about delicate details like endianness and how to represent higher-level messages such as strings or maps using bytes. You also need to specify the right channel name whenever you want to send a message or register a handler. Making this easier leads us to platform channels:

A platform channel is an object that brings together a channel name and a codec for serializing/deserializing messages to binary form and back.

在二进制消息这个层面工作的话，你需要考虑很多非常细致的细节，比如字节序、如何用字节呈现一些像字符串、字典这样的高级别消息。同时每次发送消息或者注册处理处理程序时都要指定正确的通道名。想要让这些更简单的话，就需要引入平台通道：

*平台通道*是一个可以把通道名和能把消息与二进制格式之间序列化/反序列化的编解码器组合在一起的对象。

![image](https://miro.medium.com/max/1400/1*Sd6s3EDGkU8TBS9xLc4Zvw.png)

Suppose you want to send and receive string messages instead of byte buffers. This can be done using a message channel, a simple kind of platform channel, constructed with a string codec. The code below shows how to use message channels in both directions across Dart, Android, and iOS:

假如你想发送或接收字符串消息而不是二进制缓冲，可以使用消息通道，最简单的一种构造一个字符串编码的平台通道。下面的代码展示了如何在Dart、Android和iOS平台之间双向的使用消息通道：

```
// String messages
// Dart side
const channel = BasicMessageChannel<String>('foo', StringCodec());
// Send message to platform and receive reply.
final String reply = await channel.send('Hello, world');
print(reply);
// Receive messages from platform and send replies.
channel.setMessageHandler((String message) async {
  print('Received: $message');
  return 'Hi from Dart';
});

// Android side
val channel = BasicMessageChannel<String>(
  flutterView, "foo", StringCodec.INSTANCE)
// Send message to Dart and receive reply.
channel.send("Hello, world") { reply ->
  Log.i("MSG", reply)
}
// Receive messages from Dart and send replies.
channel.setMessageHandler { message, reply ->
  Log.i("MSG", "Received: $message")
  reply.reply("Hi from Android")
}

// iOS side
let channel = FlutterBasicMessageChannel(
    name: "foo",
    binaryMessenger: controller,
    codec: FlutterStringCodec.sharedInstance())
// Send message to Dart and receive reply.
channel.sendMessage("Hello, world") {(reply: Any?) -> Void in
  os_log("%@", type: .info, reply as! String)
}
// Receive messages from Dart and send replies.
channel.setMessageHandler {
  (message: Any?, reply: FlutterReply) -> Void in
  os_log("Received: %@", type: .info, message as! String)
  reply("Hi from iOS")
}
```


The channel name is specified only on channel construction. After that, calls to send a message or set a message handler can be done without repeating the channel name. More importantly, we leave it to the string codec class to deal with how to interpret bytes buffers as strings and vice versa.



These are noble advantages to be sure, but you’d probably agree that BasicMessageChannel doesn’t do all that much. Which is on purpose. The Dart code above is equivalent to the following use of the binary messaging foundations:

通道的名字只需要在通道构造函数里指定一次，之后所有发送消息或指定处理程序的操作就都不需要再重复的指定通道名了。更重要的是，我们我们把如何将字节缓冲和字符串互相转化的工作留给了字符串编解码类处理。

还有一些你未必同意的高级的好处：BasicMessageChannel刻意的不去做所有的工作。上面的Dart代码等同于下面的二进制消息通信功能：


```
const codec = StringCodec();
// Send message to platform and receive reply.
final String reply = codec.decodeMessage(
  await BinaryMessages.send(
    'foo',
    codec.encodeMessage('Hello, world'),
  ),
);
print(reply);
// Receive messages from platform and send replies.
BinaryMessages.setMessageHandler('foo', (ByteData message) async {
  print('Received: ${codec.decodeMessage(message)}');
  return codec.encodeMessage('Hi from Dart');
});

```

This remark applies to the Android and iOS implementations of message channels as well. There is no magic involved:

Message channels delegate to the binary messaging layer for all communication.

Message channels do not keep track of registered handlers themselves.

Message channels are light-weight and stateless.

Two message channel instances created with the same channel name and codec are equivalent (and interfere with each other’s communication).


这个描述也适用于Android和iOS的消息通道实现。这里没有什么魔法：
- 消息通道把所有的通信都托管给了二进制消息层面。
- 消息通道并不持有注册的处理程序。
- 消息通道是轻量级的状态无关的。
- 两个拥有相同名字和编解码器的通道实例是完全等同的（会干扰彼此的通信）
- 
For various historical reasons, the Flutter framework defines four different message codecs:

StringCodec Encodes strings using UTF-8. As we’ve just seen, message channels with this codec have type BasicMessageChannel<String> in Dart.

BinaryCodec Implementing the identity mapping on byte buffers, this codec allows you to enjoy the convenience of channel objects in cases where you don’t need encoding/decoding. Dart message channels with this codec have type BasicMessageChannel<ByteData>.

JSONMessageCodec Deals in “JSON-like” values (strings, numbers, Booleans, null, lists of such values, and string-keyed maps of such values). Lists and maps are heterogeneous and can be nested. During encoding, the values are turned into JSON strings and then to bytes using UTF-8. Dart message channels have type BasicMessageChannel<dynamic> with this codec.

StandardMessageCodec Deals in slightly more generalized values than the JSON codec, supporting also homogeneous data buffers (UInt8List, Int32List, Int64List, Float64List) and maps with non-string keys. The handling of numbers differs from JSON with Dart ints arriving as 32 or 64 bit signed integers on the platform, depending on magnitude — never as floating-point numbers. Values are encoded into a custom, reasonably compact, and extensible binary format. The standard codec is designed to be the default choice for channel communication in Flutter. As for JSON, Dart message channels constructed with the standard codec have type BasicMessageChannel<dynamic>.

因为各种各样的历史原因，Flutter框架定义了4种不同的消息编解码：

++StringCodec++ 用UTF-8编码字符串。像你已经看到的，消息这种编解码的消息通道在Dart里对应着BasicMessageChannel<String> 这个类型。

++BinaryCodec++ 实现了在字节缓冲里特定的映射，这种编码在你不需要编解码的时候可以享受到通道对象的便利。这种编码对应了BasicMessageChannel<ByteData>类。

++JSONMessageCodec++ 用来处理JSON样式的值（字符串，数字，布尔，null，存这些值的列表，以字符串为key的字典等）。列表和字典是复合类型且可以嵌套的。在编码过程中，值可以使用UTF-8编码转成JSON字符串再转成字节。对应类型BasicMessageChannel<dynamic>

++StandardMessageCodec++ 用来处理比JSON编码更广义一点的类型，支持一些非复合类型（UInt8List, Int32List, Int64List, Float64List) 和非字符串key的字典。数字的处理跟平台上的含有32位64位整形的JSON有所不同，而且不支持浮点数字。值会被编码成一些自定义的、合理压缩的、扩展的二进制格式。标准编解码器是为了给Flutter的通道通信提供一种默认的选择。对应的类跟JSON一样是BasicMessageChannel<dynamic>。

As you may have guessed, message channels work with any message codec implementation that satisfies a simple contract. This allows you to plug in your own codec, if you need to. You’ll have to implement compatible encoding and decoding in Dart, Java/Kotlin, and Objective-C/Swift.

你可能能猜到，任何满足了一定简单规则的消息编解码，消息通道都可以与之相配合。如果需要的话你可以插入任何你自己的编解码。你必须在Dart, Java/Kotlin, 和 Objective-C/Swift里实现与之相兼容的编码和解码。

The fine print. Codec evolution. Each message codec is available in Dart, as part of the Flutter framework, as well as on both platforms, as part of the libraries exposed by Flutter to your Java/Kotlin or Objective-C/Swift code. Flutter uses the codecs only for intra-app communication, not as a persistence format. This means that the binary form of messages may change from one release of Flutter to the next, without warning. Of course, the Dart, Android, and iOS codec implementations are evolved together, to ensure that what is encoded by the sender can be successfully decoded by the receiver, in both directions.


Null messages. Any message codec must support and preserve null messages since that is the default reply to a message sent on a channel for which no message handler has been registered on the receiving side.


编解码的演进。在作为Flutter框架一部分的Dart中，每一种消息的编解码器都是可用的，这种可用也包括Flutter在Java/Kotlin和Objective-C/Swift两个平台端暴露出来的库的一部分。Flutter只有在App内部通信时会用到这些编解码器，而不会运用到持久层。这意味着，随着FLutter不同版本的演进，消息的二进制格式可能无预警的发生变化。当然Dart，Android和iOS相关的编解码实现也会跟着一起进化，来确保发送端编码的内容再接收端能够正常的解码，反向也是。

空消息。当消息发送给一个没有注册任何对应处理程序的通道接收侧时，默认会回复一个空的消息，这种空消息必须是被编解码器正常的支持和保留的。

Static typing of messages in Dart. A message channel configured with the standard message codec gives type dynamic to messages and replies. You’d often make your type expectations explicit by assigning to a typed variable:

Dart消息里的静态类型。配置以标准编解码器的消息通道通常使```dynamic```类型来发送和回复消息。你应当通过把它赋值给一个有类型的变量来确保类型与期待一直。

```
final String reply1 = await channel.send(msg1);
final int reply2 = await channel.send(msg2);
```

But there’s a caveat when dealing with replies involving generic type parameters:


那么在处理泛型类型参数时就要特别注意了：

```
final List<String> reply3 = await channel.send(msg3);      // Fails.
final List<dynamic> reply3 = await channel.send(msg3);     // Works.
```

The first line fails at runtime, unless the reply is null. The standard message codec is written for heterogeneous lists and maps. On the Dart side, these have runtime types List<dynamic> and Map<dynamic, dynamic>, and Dart 2 prevents such values from being assigned to variables with more specific type arguments. This situation is similar to Dart JSON deserialization which produces List<dynamic> and Map<String, dynamic> — as does the JSON message codec.

Futures can get you into similar trouble:

除非返回的是null否则第一行代码会在运行时出错。标准消息编解码器是为了诸如列表、字典等复合类型服务的。在Dart侧，使用List<dynamic>和Map<dynamic, dynamic>这样的运行时类型。在Dart2会阻止这些类型呗赋值给更具体的类型。Dart JSON消息编解码器在生产List<dynamic> and Map<String, dynamic>的反序列化过程中也有类似的情况。

Future会给你带来类似的麻烦：

```
Future<String> greet() => channel.send('hello, world');    // Fails.
Future<String> greet() async {                             // Works.
  final String reply = await channel.send('hello, world');
  return reply;
}
```

The first method fails at runtime, even if the reply received is a string. The channel implementation creates a Future<dynamic> regardless of the type of the reply, and such an object cannot be assigned to a Future<String>.

Why the “basic“ in BasicMessageChannel? Message channels seem to be used only in rather restricted situations where you are communicating some form of homogeneous event stream in an implied context. Like keyboard events, perhaps. For most applications of platform channels, you’re going to need to communicate not only values, but also what you want to happen with each value, or how you’d like it to be interpreted by the receiver. One way to do that is to have the message represent a method call with the value as argument. So you’ll want a standard way of separating the method name from the argument in the message. And you’ll also want a standard way to distinguish between success and error replies. This is what method channels do for you. Now, BasicMessageChannel was originally named MessageChannel, but was renamed to avoid confusing MessageChannel with MethodChannel in code. Being more generally applicable, method channels kept the shorter name.

第一个方法在运行时会出错，即便返回的确实是一个字符串。不管通道返回的是什么类型，都会创造一个Future<dynamic>类型出来，而这个类型是没办法赋值给Future<String>的。

BasicMessageChannel为什么叫Basic呢？消息通道看上去只适合用在有默认上下文时传递一些事件流形式的信息这样一些有限的场景里，比如键盘事件。对于使用平台通道的多数应用来说，你要通信的可能不止是数值，还有这些数值应该发生什么动作，或者是你希望接收方如何与这些数值进行交互。一个实现方式是让消息去呈现一个方法和以及相关参数。因此你需要一个标准的方式去分离消息里的方法名和参数。你同样会希望一种标准的分离成功失败回复的方式。这就是*方法*通道能帮你做的。BasicMessageChannel原名叫MessageChannel，为了避免和MethodChannel里的MessageChannel冲突改名成了BasicMessageChannel。为了更广泛的适用性，方法通道保留了更短的名字。

### Method channels: standardized envelopes

方法通道：标准化的“信封”
![image](https://miro.medium.com/max/1400/1*ykNghfAKtx0xsZWedfgslg.png)

Method channels are platform channels designed for invoking named pieces of code across Dart and Java/Kotlin or Objective-C/Swift. Method channels make use of standardized message “envelopes” to convey method name and arguments from sender to receiver, and to distinguish between successful and erroneous results in the associated reply. The envelopes and supported payload are defined by separate method codec classes, similarly to how message channels use message codecs.

This is all that method channels do: combine a channel name with a codec.

方法通道是一种用来在Dart和Java/Kotlin，Objective-C/Swift中间执行的具名的代码段的平台通道。方法通道利用标准化的消息“信封”从发送者往接受者传达方法名和参数，并且能用相关联的返回来区分成功后有错误的结果。信封和支持的内容会使用单独的方法编解码类，跟消息通道使用消息编解码的方式类似。

这就是方法通道做的事情：组合一个通道名和编解码器。

In particular, no assumptions are being made about what code is executed on receipt of a message on a method channel. Even though the message represents a method call, you don’t have to invoke a method. You might just switch on the method name and execute a few lines of code for each case.

Side note. This lack of implied or automated binding to methods and their parameters might disappoint you. That’s fine, disappointment can be productive. I suppose you can build such a solution from scratch using annotation processing and code generation, or maybe you can reuse parts of an existing RPC framework. Flutter is open source, feel free to contribute! Method channels are available as a target for your code generation, if they fit the bill. In the mean time, they are useful on their own in “handcraft mode”.

Method channels were the Flutter team’s answer to the challenge of defining a workable communication API for use by the, at the time, non-existing plugin ecosystem. We wanted something that plugin authors could start using right away, without a lot of boilerplate or complicated build setup. I think the method channel concept makes a decent answer, but I’d be surprised if it remains the only answer.

特别要指出，并不会预设一个方法通道上的消息的接收方应该执行哪些代码。尽管如此这些消息还是代表了一次方法调用。你（译者注：接收方）并不一定要调用一个方法，你可能只需要切换一个方法名字，为每一个case执行几行简单的代码。

**旁注**。你可能会失望的发现，并不存在方法和相应的参数的自动绑定。没关系，失望会转化成动力。我相信你能构建一个通过注解处理和代码生成来实现这种绑定。或者你可以去重用现有的RPC框架的一部分。Flutter是开源的，欢迎来贡献代码。只要合适的话，方法通道可以为你的代码生成提供可用的手段。同时，纯手工模式他们也是非常有用的。

Here’s how you would use a method channel in the simple case of invoking a bit of platform code from Dart. The code is associated with the name bar which is not a method name in this case, but could have been. All it does is construct a greeting string and return it to the caller, so we can code this with the reasonable assumption that the platform invocation won’t fail (we’ll look at error handling further below):

下面是你应该怎么在一个从Dart执行一点平台代码的简单例子里去运用方法通道。这段代码跟名字bar相关联，这个名字之前可能是通道名字但在这个场景里不是。它所有的工作只是构造一个问候语并把它返回给调用者，这样我们可以合理的假设这里的平台调用不会出现失败。（在后面一点的位置我们会看到错误处理）：

```
// Invocation of platform methods, simple case.
// Dart side.
const channel = MethodChannel('foo');
final String greeting = await channel.invokeMethod('bar', 'world');
print(greeting);

// Android side.
val channel = MethodChannel(flutterView, "foo")
channel.setMethodCallHandler { call, result ->
  when (call.method) {
    "bar" -> result.success("Hello, ${call.arguments}")
    else -> result.notImplemented()
  }
}
// iOS side.
let channel = FlutterMethodChannel(
  name: "foo", binaryMessenger: flutterView)
channel.setMethodCallHandler {
  (call: FlutterMethodCall, result: FlutterResult) -> Void in
  switch (call.method) {
  case "bar": result("Hello, \(call.arguments as! String)")
  default: result(FlutterMethodNotImplemented)
  }
}
```

By adding cases to the switch constructs, we can easily extend the above to handle multiple methods. The default clause handles the situation where an unknown method is called (most likely due to a programming error).

The Dart code above is equivalent to the following:

通过在switch结构里增加更多的case，我们就可以轻易的把上面的代码扩展成可以处理多个方法。一个未知方法被调用时则会被default语句处理（这种情况往往由编程错误导致）。

上面的Dart代码等同于下面的：

```
const codec = StandardMethodCodec();
final ByteData reply = await BinaryMessages.send(
  'foo',
  codec.encodeMethodCall(MethodCall('bar', 'world')),
);
if (reply == null)
  throw MissingPluginException();
else
  print(codec.decodeEnvelope(reply));
```

The Android and iOS implementations of method channels are similarly thin wrappers around calls to the binary messaging foundations. A null reply is used to represent a “not implemented” result. This conveniently makes the behavior at the receiving end indifferent to whether the invocation fell through to the default clause in the switch, or no method call handler had been registered with the channel at all.

The argument value in the example is the single string world. But the default method codec, aptly named the “standard method codec”, uses the standard message codec under the hood to encode payload values. This means that the “generalized JSON-like” values described earlier are all supported as method arguments and (successful) results. In particular, heterogeneous lists support multiple arguments, while heterogeneous maps support named arguments. The default arguments value is null. A few examples:

Android和iOS对方法通道的实现都是类似于对调用二进制消息基础上的简单封装。一个空回复会用来代表“未实现”结果。这样就会比较方便的保证，无论是调用正常下滑到defualt语句汇总，还是对于这个通道根本就没有注册响应的方法处理程序，接收端都有一致的行为。

在上面的例子中参数值是一个字符串```world```。然而默认的方法编解码器，也就是名为“标准方法编解码器”在底层是用标准消息编解码来编码这些值的。这就意味着早先提到的JSON格式的值对于方法参数和返回值来说是完全支持的。特别要指出，列表可以支持多参数，字典可以支持具名参数。默认的参数值是null。另外一些例子：

```
await channel.invokeMethod('bar');
await channel.invokeMethod('bar', <dynamic>['world', 42, pi]);
await channel.invokeMethod('bar', <String, dynamic>{
  name: 'world',
  answer: 42,
  math: pi,
}));
```

The Flutter SDK includes two method codecs:

StandardMethodCodec which by default delegates the encoding of payload values to StandardMessageCodec. Because the latter is extensible, so is the former.

JSONMethodCodec which delegates the encoding of payload values to JSONMessageCodec.

Flutter SDK包含两个方法编解码：

++标准方法编解码++：用以把载体里的值编码成```标准消息编解码```。因为后者更可扩展性，而前者也是。
++JSON方法编解码：用来把值编码成JSON消息编解码。

You can configure method channels with any method codec, including custom ones. To fully understand what is involved in implementing a codec, let’s look at how errors are handled at the method channel API level by extending the example above with a fallible ```baz``` method:

你可以给方法通道配置任何编解码器，包括自定义的。为了完全理解实现一个编解码器都涉及到哪些东西。让我们用一个可出错的```baz```方法来扩展上面的例子，从而在API层面了解一下方法通道是如何处理错误的。

```
// Method calls with error handling.
// Dart side.
const channel = MethodChannel('foo');
// Invoke a platform method.
const name = 'bar'; // or 'baz', or 'unknown'
const value = 'world';
try {
  print(await channel.invokeMethod(name, value));
} on PlatformException catch(e) {
  print('$name failed: ${e.message}');
} on MissingPluginException {
  print('$name not implemented');
}
// Receive method invocations from platform and return results.
channel.setMethodCallHandler((MethodCall call) async {
  switch (call.method) {
    case 'bar':
      return 'Hello, ${call.arguments}';
    case 'baz':
      throw PlatformException(code: '400', message: 'This is bad');
    default:
      throw MissingPluginException();
  }
});

// Android side.
val channel = MethodChannel(flutterView, "foo")
// Invoke a Dart method.
val name = "bar" // or "baz", or "unknown"
val value = "world"
channel.invokeMethod(name, value, object: MethodChannel.Result {
  override fun success(result: Any?) {
    Log.i("MSG", "$result")
  }
  override fun error(code: String?, msg: String?, details: Any?) {
    Log.e("MSG", "$name failed: $msg")
  }
  override fun notImplemented() {
    Log.e("MSG", "$name not implemented")
  }
})
// Receive method invocations from Dart and return results.
channel.setMethodCallHandler { call, result ->
  when (call.method) {
    "bar" -> result.success("Hello, ${call.arguments}")
    "baz" -> result.error("400", "This is bad", null)
    else -> result.notImplemented()
  }
}
// iOS side.
let channel = FlutterMethodChannel(
  name: "foo", binaryMessenger: flutterView)
// Invoke a Dart method.
let name = "bar" // or "baz", or "unknown"
let value = "world"
channel.invokeMethod(name, arguments: value) {
  (result: Any?) -> Void in
  if let error = result as? FlutterError {
    os_log("%@ failed: %@", type: .error, name, error.message!)
  } else if FlutterMethodNotImplemented.isEqual(result) {
    os_log("%@ not implemented", type: .error, name)
  } else {
    os_log("%@", type: .info, result as! NSObject)
  }
}
// Receive method invocations from Dart and return results.
channel.setMethodCallHandler {
  (call: FlutterMethodCall, result: FlutterResult) -> Void in
  switch (call.method) {
  case "bar": result("Hello, \(call.arguments as! String)")
  case "baz": result(FlutterError(
    code: "400", message: "This is bad", details: nil))
  default: result(FlutterMethodNotImplemented)
}
```

Errors are triples (code, message, details) where the code and message are strings. The message is intended for human consumption, the code for, well, code. The error details is some custom value, often null, which is constrained only by the kinds of value that the codec supports.

The fine print. Exceptions. Any uncaught exception thrown in a Dart or Android method call handler is caught by the channel implementation, logged, and an error result is returned to the caller. Uncaught exceptions thrown in result handlers are logged.

错误是一个三元组 (code, message, details)，这里面code和message是字符串。message是一个人类能读懂的串，code是指，嗯，就是code。错误的detail是一些自定义的值，经常是null，并且需要是被编解码器支持的类型。

**打印**。*异常*。任何呗Dart或者Android方法调用抛出而未被捕捉的异常都会被通道的实现所捕捉，并记录，然后一个错误结果会返回给调用者。在结果的处理程序中跑出的未捕捉的异常会被记录。

Envelope encoding. How a method codec encodes its envelopes is an implementation detail just like how message codecs convert messages to bytes. As an example, a method codec might use lists: method calls can be encoded as a two-element lists [method name, arguments]; success results as one-element lists [result]; error results as three-element lists [code, message, details]. Such a method codec can then be implemented simply by delegation to an underlying message codec that supports at least lists, strings, and null. The method call arguments, success results, and error details would be arbitrary values supported by that message codec.

*信封编码*。方法编解码器如何去编码它的信封是一个类似于消息编解码器吧消息转成字节的实现。举个例子：一个方法编解码器可以会用到列表：方法调用可以被编码成一个两个元素的数组[方法名，参数列表]；成功结果可以编码成一个单元素列表[结果]；错误结果是一个三元素列表[code，message, detail]。这样一个方法编解码器就可以简单的把视线委托给一个至少支持列表、字符串、null的底层消息编解码器。方法调用参数、成功结果、错误细节都可以是任何值，只要被消息编解码器支持。

API differences. The code examples above highlight that method channels deliver results very differently across Dart, Android, and iOS:

On the Dart side, invocation is handled by a method returning a future. The future completes with the result of the call in success cases, with a PlatformException in error cases, and with a MissingPluginException in the not implemented case.

On Android, invocation is handled by a method taking a callback argument. The callback interface defines three methods of which one is called, depending on the outcome. Client code implements the callback interface to define what should happen on success, on error, and on not implemented.

On iOS, invocation is similarly handled by a method taking a callback argument. But here, the callback is a single-argument function which is given either a FlutterError instance, the FlutterMethodNotImplemented constant, or, in case of success, the result of the invocation. Client code provides a block with conditional logic to handle the different cases, as needed.

*API差异*。上面的例子代码指出了方法通道在Dart，Android，iOS上传递结果是非常不一样的：

- 在Dart侧，调用是处理成被一个返回future的方法。成功的话future会带着调用结果完成。失败的话会带着一个```PlatformException```，未实现的情况下会带着一个```MissingPluginException```.
- 在Android侧，调用是处理成一个带回调参数的方法。毁掉接口会定义三个方法，根据结果决定调用哪一个。客户代码实现了回调函数，并定义了在成功、失败和未实现的情况下应该发生什么。
- 在iOS侧。调用简单的被处理成一个带着回调参数的方法，但是这里，回调函数是一个单参数函数，这个入参要么是一个```FlutterError```实例，要么是一个```FlutterMethodNotImplemented```，或者，在成功时是调用的返回值。客户代码则提供了一个带有条件逻辑处理不同分支的block。

These differences, mirrored also in the way message call handlers are written, arose as concessions to the styles of the programming languages (Dart, Java, and Objective-C) used for the Flutter SDK method channel implementations. Redoing the implementations in Kotlin and Swift might remove some of the differences, but care must be taken to avoid making it harder to use method channels from Java and Objective-C.

这些差异，在消息调用处理程序的编写上同样存在。取决于这些编程语言(Dart, Java, 和 Objective-C) 在使用FlutterSDK方法通道时的编码风格。在Kotlin和Swift中重写这些实现可能会消除其中的一些差异，但是必须非常谨慎的保证它们不会比在Java和Objective-C中使用方法通道更复杂。

### Event channels: streaming
### 事件通道：流

![image](https://miro.medium.com/max/1400/1*jd9Thys5_k-jbkKM7P69Ng.png)

An event channel is a specialized platform channel intended for the use case of exposing platform events to Flutter as a Dart stream. The Flutter SDK currently has no support for the symmetrical case of exposing Dart streams to platform code, though that could be built, if the need arises.

Here’s how you would consume a platform event stream on the Dart side:

事件通道是一种用来把平台事件作为Dart流的方式暴露给Flutter的特殊的平台通道。Flutter SDK现在并没有对称的支持向平台代码暴露Dart流。不过如果有必要的话也可以支持。

这是你应该如何在Dart侧构建一个平台事件流：

```
// Consuming events on the Dart side.
const channel = EventChannel('foo');
channel.receiveBroadcastStream().listen((dynamic event) {
  print('Received event: $event');
}, onError: (dynamic error) {
  print('Received error: ${error.message}');
});
```

The code below shows how to produce events on the platform side, using sensor events on Android as an example. The main concern is to ensure that we are listening to events from the platform source (the sensor manager in this case) and sending them through the event channel precisely when 1) there is at least one stream listener on the Dart side and 2) the ambient Activity is running. Packaging up the necessary logic in a single class increases the chance of doing this correctly:

下面的代码展现了如何在平台测去制造时间，以Android平台的感应器事件为例。主要的关注点是确保我们再监听平台来源的事件（这个case里就是sensorManager）并且准确的通过事件通道把他们发送出去，发送的条件有两个：1）在Dart侧有至少一个流监听者。2） 周围的Activity正在运行。把必要的逻辑封装在一个单独的呃类里有助于提高代码的正确性。

```
// Producing sensor events on Android.
// SensorEventListener/EventChannel adapter.
class SensorListener(private val sensorManager: SensorManager) :
  EventChannel.StreamHandler, SensorEventListener {
  private var eventSink: EventChannel.EventSink? = null

  // EventChannel.StreamHandler methods
  override fun onListen(
    arguments: Any?, eventSink: EventChannel.EventSink?) {
    this.eventSink = eventSink
    registerIfActive()
  }
  override fun onCancel(arguments: Any?) {
    unregisterIfActive()
    eventSink = null
  }

  // SensorEventListener methods.
  override fun onSensorChanged(event: SensorEvent) {
    eventSink?.success(event.values)
  }
  override fun onAccuracyChanged(sensor: Sensor?, accuracy: Int) {
    if (accuracy == SensorManager.SENSOR_STATUS_ACCURACY_LOW)
      eventSink?.error("SENSOR", "Low accuracy detected", null)
  }
  // Lifecycle methods.
  fun registerIfActive() {
    if (eventSink == null) return
    sensorManager.registerListener(
      this,
      sensorManager.getDefaultSensor(Sensor.TYPE_GYROSCOPE),
      SensorManager.SENSOR_DELAY_NORMAL)
  }
  fun unregisterIfActive() {
    if (eventSink == null) return
    sensorManager.unregisterListener(this)
  }
}
// Use of the above class in an Activity.
class MainActivity: FlutterActivity() {
  var sensorListener: SensorListener? = null

  override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    GeneratedPluginRegistrant.registerWith(this)
    sensorListener = SensorListener(
      getSystemService(Context.SENSOR_SERVICE) as SensorManager)
    val channel = EventChannel(flutterView, "foo")
    channel.setStreamHandler(sensorListener)
  }

  override fun onPause() {
    sensorListener?.unregisterIfActive()
    super.onPause()
  }

  override fun onResume() {
    sensorListener?.registerIfActive()
    super.onResume()
  }
}
```

If you use the android.arch.lifecycle package in your app, you could make SensorListener more self-contained by making it a LifecycleObserver.

The fine print. Life of a stream handler. The platform side stream handler has two methods, onListen and onCancel, which are invoked whenever the number of listeners to the Dart stream goes from zero to one and back, respectively. This can happen multiple times. The stream handler implementation is supposed to start pouring events into the event sink when the former is called, and stop when the latter is called. In addition, it should pause when the ambient app component is not running. The code above provides a typical example. Under the covers, a stream handler is of course just a binary message handler, registered with the Flutter view using the event channel’s name.

如果你在你的app里使用``` android.arch.lifecycle```package，你可以把```SensorListener```转换成```LifecycleObserver```让它能够更加的自我管理。

**打印**。*流处理程序的生命周期。*平台侧的流处理程序有两个方法：```onListen```和```onCancel```，只要在Dart流的监听者的数量从0到1或者从1到0时这两个方法就会被分别调用。可能会发生多次。当前者被调用时，流处理程序的实现会开始往事件槽（event sink）里灌输事件，当后者被调用时就停止。另外。当容器App组件不再运行时就应该暂停。上面的代码提供了一个典型的例子。表层的下面，一个过程的流处理程序就是一个二进制消息的处理程序，使用事件通道的名字和Flutter View注册的消息处理程序。

Codec. An event channel is configured with a method codec, allowing us to distinguish between success and error events in the same way that method channels are able to distinguish between success and error results.

Stream handler arguments and errors. The onListen and onCancel stream handler methods are invoked via method channel invocations. So we have control method calls from Dart to the platform and event messages in the reverse direction, all on the same logical channel. This setup allows arguments to be relayed to both control methods and any errors to be reported back. On the Dart side, the arguments, if any, are given in the call to receiveBroadcastStream. This means they are specified only once, regardless of the number of invocations of onListen and onCancel happening during the lifetime of the stream. Any errors reported back are logged.

End of stream. An event sink has an endOfStream method that can be invoked to signal that no additional success or error events will be sent. The null binary message is used for this purpose. On receipt on the Dart side, the stream is closed.

Life of a stream. The Dart stream is backed by a stream controller fed from the incoming platform channel messages. A binary message handler is registered using the event channel’s name to receive incoming messages only while the stream has listeners.


*编解码器*。一个事件通道的编解码器是用一个方法的编解码器配置的。这就让我们能够用方法通道区分成功失败结果相同的方式来区分成功和失败的事件。

*流处理参数和错误*。在```onListen```和```onCancel```流处理方法是通过方法通道的调用来实现调用的。这样我们就能够控制从Dart到平台的方法调用，和反方向的事件消息，这些都会在相同的逻辑通道上。这些设置就允许参数可以再两个控制方法中都能被返回，任何的错误信息也能够被上报。在Dart侧，如果有任何参数的话，可以在调用```receiveBroadcastStream```时给出。这就意味着，在流的声明周期中，不管```onListen```和```onCancel```被调用几次， 他们都是会只被设置一次。任何错误信息都会被记录下来。

*流结尾*(End of stream)。一个事件槽有一个```endOfStream```方法，这个方法可以用来标记“不会再发送任何附加的成功或错误事件”这个信息。null二进制消息也会被用在这个用途上。Dart侧接收到这个方法后流就会关闭。

*流声明周期*。Dart流是背后是平台通道消息供给的流控制器。只有在流有监听者的时候，二进制消息处理程序会用事件通道的名字注册，用来接收进入的消息。

### Usage guidelines

##### Prefix channel names by domain for uniqueness

Channel names are just strings, but they have to be unique across all channel objects used for different purposes in your app. You can accomplish that using any suitable naming scheme. However, the recommended approach for channels used in plugins is to employ a domain name and plugin name prefix such as some.body.example.com/sensors/foo for the foo channel used by the sensors plugin developed by some.body at example.com. Doing so allows plugin consumers to combine any number of plugins in their apps without risk of channel name collisions.

### 使用指南

##### 域做通道名前缀保持唯一性

通道名只是字符串，但是它们必须是在所有在你app中用于各种不同用途的通道对象中保持唯一性。要达到这一点你可以使用任何合适的命名法则。尽管如此，对于用作插件的通道的命名，比较推荐的方法是使用域名和插件名前缀，比如对于```some.body```在```example.com```开发的用作```sensors```的通道```foo``` 使用这个名字：```some.body.example.com/sensors/foo```。这样可以让插件使用者在他们的app里使用任意数量的插件都不会有命名冲突的风险。


##### Consider treating platform channels as intra-module communication
Code for invoking remote procedure calls in distributed systems look superficially similar to code using method channels: you invoke a method given by a string and serialize your arguments and results. Since distributed system components are often developed and deployed independently, robust request and reply checking is critical, and usually done in check-and-log style on both sides of the network.

##### 把对待平台通道当成模块内通信

在一个分布式系统里调用一个远程过程的的代码表面上跟使用方法通道的代码非常相似：你给出一个字符串并序列化你的参数和结果然后调用一个方法。因为分布式系统的组件经常是独立的开发和部署的，健壮性和回复检查是特别重要的，通常是在网络双边的检查-记录过程中实现的。

Platform channels on the other hand glue together three pieces of code that are developed and deployed together, in a single component.

Java/Kotlin ↔ Dart ↔ Objective-C/Swift

In fact, it very often makes sense to package up a triad like this in a single code module, such as a Flutter plugin. This means that the need for arguments and results checking across method channel invocations should be comparable to the need for such checks across normal method calls within the same module.

另一方面平台通道把三块代码粘合到一起进行开发和部署，形成一个单独的组件。

Java/Kotlin ↔ Dart ↔ Objective-C/Swift

事实上，更合理的做法经常是把这些打包成一个类似三合一的单独代码模块，比如Flutter Plugin。这就意味着，在方法通道调用中需要的参数和结果检查和在相同模块中进行的普通方法调用的检查是基本类似的。

Inside modules, our main concern is to guard against programming errors that are beyond the compiler’s static checks and go undetected at runtime until they blow things up non-locally in time or space. A reasonable coding style is to make assumptions explicit using types or assertions, allowing us to fail fast and cleanly, e.g. with an exception. Details vary by programming language of course. Examples:

If a value received over a platform channel is expected to have a certain type, immediately assign it to a variable of that type.

If a value received over a platform channel is expected to be non-null, either set things up to have it dereferenced immediately, or assert that it is non-null before storing it for later. Depending on your programming language, you may be able to assign it to a variable of a non-nullable type instead.

在模块中，我们主要的关注点是防止程序错误，这些错误是可以绕过编译器静态检查并且在运行时保持不被检测到，然后会在时间或空间上把事情都破坏的。在一个合理的编程风格里，应当尽量显式的使用类型和断言，这能让我们更清晰和更快的发现问题，比如使用异常。当然使用不同的编程语言会出现非常多样化的编程细节。比如：
- 如果一个从平台通道接收到的值有确定的类型，那就马上把它赋值给一个该类型的变量。
- 如果一个从平台通道接收到的值是一个非空类型，要么快速把信息设置好后将它解引用，或者再存储它之前进行非空断言来确保后面的使用。你或许也可以根据你的编程语言将之赋值给一个不可为空的类型。
- 
Two simple examples:

两个简单的例子：

```
// Dart: we expect to receive a non-null List of integers.
for (final int n in await channel.invokeMethod('getFib', 100)) {
  print(n * n);
}
// Android: we expect non-null name and age arguments for
// asynchronous processing, delivered in a string-keyed map.
channel.setMethodCallHandler { call, result ->
  when (call.method) {
    "bar" -> {
      val name : String = call.argument("name")
      val age : Int = call.argument("age")
      process(name, age, result)
    }
    else -> result.notImplemented()
  }
}
:
fun process(name: String, age: Int, result: Result) { ... }

```

The Android code exploits the generically typed <T> T argument(String key) method of MethodCall which looks up the key in the arguments, assumed to be a map, and casts the value found to the target (call site) type. A suitable exception is thrown, if this fails for any reason. Being thrown from a method call handler, it would be logged, and an error result sent to the Dart side.

Android代码利用```MethodCall```方法的泛型类型```<T> T argument(String key)```来查找参数中的key，设置给一个map，然后把值转换成调用侧的目标类型。如果因为任何原因失败了，会有一个响应的异常被抛出。这个异常会被从一个方法处理程序中抛出，并会被记录，一个错误结果会被发回给Dart端。

#### Don’t mock platform channels

(Pun intended.) When writing unit tests for Dart code that uses platform channels, a knee jerk reaction may be to mock the channel object, as you would a network connection.

####  不要给平台通道打桩

（双关语哦）当你为使用了平台通道的Dart代码编写单元测试时，给一个通道对象打桩会是一件很奇怪的事情。


You can certainly do that, but channel objects don’t actually need to be mocked to play nicely with unit tests. Instead, you can register mock message or method handlers to play the role of the platform during a particular test. Here is a unit test of a function hello that is supposed to invoke the bar method on channel foo:

你当然也可以这么做，但是通道对象在单元测试中实际上不需要通过打桩来达到目的。你应该注册一个打桩的消息或者是方法处理程序在特定的测试中充当平台的角色。这有一个关于函数```hello```的单元测试，这个方法应该再```foo```通道上调用```bar```方法：

```
test('gets greeting from platform', () async {
  const channel = MethodChannel('foo');
  channel.setMockMethodCallHandler((MethodCall call) async {
    if (call.method == 'bar')
      return 'Hello, ${call.arguments}';
    throw MissingPluginException();
  });
  expect(await hello('world'), 'Platform says: Hello, world');
});
```

To test code that sets up message or method handlers, you can synthesize incoming messages using BinaryMessages.handlePlatformMessage. At present, this method is not mirrored on platform channels, though that could easily be done as indicated in the code below. The code defines a unit test of a class Hello that is supposed to collect incoming arguments of calls to method bar on channel foo, while returning greetings:

为了测试设置消息和方法的处理程序的代码，你可以用```BinaryMessages.handlePlatformMessage```合成输入的消息。现在，这个方法不再反映在平台通道上，但是这些也可以轻松的通过下面的代码指示的方式做。这段代码定义了一个单元测试类```Hello```，这个类主要能收集在```foo`·`通道上调用```bar```方法时传进来的参数，同时返回问候语：

```
test('collects incoming arguments', () async {
  const channel = MethodChannel('foo');
  final hello = Hello();
  final String result = await handleMockCall(
    channel,
    MethodCall('bar', 'world'),
  );
  expect(result, contains('Hello, world'));
  expect(hello.collectedArguments, contains('world'));
});
// Could be made an instance method on class MethodChannel.
Future<dynamic> handleMockCall(
  MethodChannel channel,
  MethodCall call,
) async {
  dynamic result;
  await BinaryMessages.handlePlatformMessage(
    channel.name,
    channel.codec.encodeMethodCall(call),
    (ByteData reply) {
      if (reply == null)
        throw MissingPluginException();
      result = channel.codec.decodeEnvelope(reply);
    },
  );
  return result;
}
```

Both examples above declare the channel object in the unit test. This works fine — unless you worry about the duplicated channel name and codec — because all channel objects with the same name and codec are equivalent. You can avoid the duplication by declaring the channel as a const somewhere visible to both your production code and the test.

What you don’t need is to provide a way to inject a mock channel into your production code.



上面两个简单的例子都在单元测试里声明了通道对象。这个没有问题——除非你担心重复的通道名和编解码器——因为所有相同名字和编解码器的通道对象被认为是等价的。你可以通过在你产品代码和测试代码的某些地方显式的把通道声明成```const```来避免重复的问题。

而你完全不需要做的是想办法在你的产品代码里注入一个桩。


Consider automated testing for your platform interaction

Platform channels are simple enough, but getting everything working from your Flutter UI via a custom Dart API backed by a separate Java/Kotlin and Objective-C/Swift implementation does takes some care. And keeping the setup working as changes are made to your app will, in practice, require automated testing to guard against regressions. This cannot be accomplished with unit testing alone because you need a real app running for platform channels to actually talk to the platform.

#### 考虑自动测试你的平台互动

平台通道已经足够简单，但是如果想所有从你的FlutterUI上调用一个背后用Java/Kotlin和Objective-C/Swift实现的自定义Dart API时所有的步骤都奏效还是需要一定的考虑的。并且保持在你的应用进行修改时，初始设置的代码也能跟着生效，在实践上需要自动测试来堤防功能的倒退。这些无法单纯用单元测试完成，因为你需要一个真实的在平台通道上运行的应用实际的跟平台对话。

Flutter comes with the flutter_driver integration test framework that allows you to test Flutter applications running on real devices and emulators. But flutter_driver is not currently integrated with other frameworks to enable testing across Flutter and platform components. I am confident this is one area where Flutter will improve in the future.

In some situations, you can use flutter_driver as is to test your platform channel usage. This requires that your Flutter user interface can be used to trigger any platform interaction and that it is then updated with sufficient detail to allow your test to ascertain the outcome of the interaction.

If you are not in that situation, or if you are packaging up your platform channel usage as a Flutter plugin for which you want a module test, you can instead write a simple Flutter app for testing purposes. That app should have the characteristics above and can then be exercised using flutter_driver. You’ll find an example in the Flutter GitHub repo.


Flutter中配套的```flutter_driver```互动测试框架可以让我们测试在真实设备上或模拟器上运营的Flutter应用。但是flutter_driver现在还不能与其他框架进行交互来执行跨Flutter和平台组件的测试。我相信这是未来Flutter一定会改进的领域。

在某些情况下你可以使用flutter_driver测试你的平台通道。这需要你的FLutter UI可以用来触发任何平台交互以及之后有足够的细节信息更新，这样让你来探明这些互动的输出结果。

如果你不是这种情况，或者如果你把你的平台通道包装成了一个Flutter插件，你需要对这个插件进行模块测试，那么你可以写一个简单的Flutter应用来进行测试。这个应用应该也有上述的特点并且能够使用```flutter_driver```执行。你可以在[Flutter GitHub repo](https://github.com/flutter/flutter/tree/master/dev/integration_tests/platform_interaction)里找到一个例子。

Keep platform side ready for incoming synchronous calls

Platform channels are asynchronous only. But there are quite a few platform APIs out there that make synchronous calls into your host app components, asking for information or help or offering a window of opportunity. One example is Activity.onSaveInstanceState on Android. Being synchronous means everything must be done before the incoming call returns. Now, you might like to include information from the Dart side in such processing, but it is too late to start sending out asynchronous messages once the synchronous call is already active on the main UI thread.

The approach used by Flutter, most notably for semantics/accessibility information, is to proactively send updated (or updates to) information to the platform side whenever the information changes on the Dart side. Then, when the synchronous call arrives, the information from the Dart side is already present and available to platform side code.

#### 让平台侧为同步调用做好准备

平台通道只能是一步的，但是也存在一些平台API会把同步调用带入到你的宿主应用组件里，请求信息或者帮助，或者提供一个机会窗口。一个例子是Android上的```Activity.onSaveInstanceState```。同步意味着所有事情都必须在调用返回之前完成。现在你可以想在Dart侧的这个处理中包含信息，但是一旦同步调用在主UI线程里被激活，再发送一步消息就已经来不及了。

用Flutter的正确方法，多数都是在Dart侧有任何信息修改的时候，向平台侧发送已经更新的（或即将更新的）语义或可达性信息。因此当同步调用到来时，从Dart侧的消息已经展现并且已经对平台侧代码可见了。


