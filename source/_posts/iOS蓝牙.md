---
title: Bluetooth
date: 2022-08-24 18:25:41
tag: iOS
---
## 关于Core Bluetooth

### 概述

Core Bluetooth framework 提供了iOS和Mac app 与蓝牙设备交互的类。例如，你的app可以发现，检测外围蓝牙设备并与之通信。Mac和iOS设备也可以充当蓝牙外围设备，保存数据到其他设备，包括Mac 和 iOS 设备。蓝牙无线技术基于蓝牙4.0规格，Core Bluetooth framework抽象了蓝牙协议栈，隐藏了底层细节，让开发者更容易的开发蓝牙相关的app。
<!--more-->

### 中心和外围是蓝牙的两大角色

在低功耗蓝牙通信中，有两大角色：中心和外围，外围一般负责提供数据，中心负责获取外围提供的数据来完成任务，比如数码恒温器提供温度数据给app设备，app在界面上显示数据。每个角色执行不同的一系列工作，外围对外暴露自己；中心扫描周围感兴趣的外围设备，当发现这样的外围设备，则会请求连接，连接成功后，与外围的数据进行交互。外围设备以一个合适的方式应答中心的请求。

### 核心蓝牙库简化了常见的蓝牙任务

Core Bluetooth framework封装了蓝牙4.0规格的细节，意味着你的app实现蓝牙功能会变得更加简单，如果你开发一个实现中心角色的app，核心蓝牙库会让发现外围，连接和与外围角色数据交互变得更加简单。除此之外，你也可以很容易的让你的app实现外围角色。

### iOS app状态影响蓝牙功能

当你的app处于后台或者挂起状态，蓝牙相关的功能也会收到影响。默认的，如果app处于后台或者被挂起，蓝牙是不起作用的，如果你需要在app进入后台状态下执行蓝牙功能，你可以定义蓝牙后台执行模式。即使你设置了后台模式，但是app还是有可能被杀进程，比如内存不足。iOS7之后，Core Bluetooth支持保存状态信息并在app启动的时候重建之前的状态。你可以使用这个特性来支持蓝牙设备的长时间动作。

### 通过最佳实践来达到更好的用户体验

Core Bluetooth framework提供了跨越蓝牙通用功能的控制，通过最佳实践达到最好的用户体验。例如你使用无线电来传输信号来实现中心外围角色，因为你的设备的无线电与其他形式的无线通信共享而且无线电发送的使用会非常耗电，所以你需要尽可能少的使用无线电发送。

## Core Bluetooth 概览

需要注意的是：iOS10之后需要在info.plist文件中设置NSBluetoothPeripheralUsageDescription以访问蓝牙权限，否则程序会崩溃。

### 中心设备和外围设备在通信中负责的任务

外围通过广播一个广告包数据，改数据包含有外围提供的一些有用的信息例如名称，基本功能。另一方面，中心设备则扫描和监听外围设备，中心设备能够请求连接发现的外围。

#### 外围设备的数据构造

连接外围设备的目的是为了交互数据，在进行交互前，需要明白外围是如何组织数据的。外围设备可能包含一个或多个服务，或提供有关其连接信号强度的有用信息。服务是一些数据和实现设备（或设备的一部分）相关功能和特性的行为的集合。例如心率探测器的服务是为了显示传感器的心率数据。服务本身由特征或包含的服务(即对其他服务的引用)组成，特征（characteristic）提供了服务（service）的进一步的细节。
​

### 核心蓝牙库里如何表征中心，外围及外围数据

#### 中心设备探索并与外围设备进行数据交互(本地为中心，远程为外围)

当中心与外围设备建立连接成功后，可以发现全部的服务和特征，中心设备也可以通过读写数据与外围设备实现交互。在中心设备这边，一个中心设备由 CBCentralManager 来表征。这个对象用来管理发现和连接外围设备（用CBPeripheral对象来表征），包括扫描，发现，连接外围设备。

当你与外围设备（CBPeripheral对象）进行数据交互时，你处理的是他的服务（services）和特征（characteristics）。服务用CBService 对象来表征，特征用 CBCharacteristic 对象来表征。



#### 本地设备作为外围，远程是中心设备

iOS6， macOS 10.9之后，设备可以作为外围，实现蓝牙外围设备的行为与外界通信。给其他设备提供数据。ios或macOS充当外围设备时，用 CBPeripheralManager 对象表征。改对象来管理公开服务和特征（从设备数据库里面获取），广告到远程的中心设备（用 CBCentral对象来表征）。 CBPeripheralManager也用来响应远程中心设备的读写操作。


当你设置或者与本地外围设备（CBPeripheralManager）数据进行交互时，你处理的是他的服务（services）和特征（characteristics）。本地外围服务用CBMutableService对象来表征，特征用CBMutableCharacteristic对象来表征。


## 执行常见的中心角色任务

在你的本地设备实现中心角色，你需要实现一下步骤：

1.设置中心管理对象。

2.发现和连接广播的外围设备。

3.连接后，检测外围设备的数据。

4.对外围设备服务的特征值发送读写请求。

5.订阅特征值，以便当特征值更新时收到通知。

### 设置中心管理者

CBCentralManager 为本地中心设备的对象表征。初始化CBCentralManager

```
CBCentralManager *myCentralManager = [[CBCentralManager alloc] initWithDelegate:self queue:nil options:nil];
```
self设置为代理，用来接收蓝牙事件，queue设置为nil时，蓝牙事件会在主线程中执行。当你创建一个CBCentralManager，需要实现 centralManagerDidUpdateState:代理来确保蓝牙可以使用。更多的代理方法可查阅：CBCentralManagerDelegate Protocol Reference.

### 发现广播的外围设备

通过调用CBCentralManager的 scanForPeripheralsWithServices:options: 方法来发现设备，

```
[myCentralManager scanForPeripheralsWithServices:nil options:nil];
```

注意：当你传的参数为nil时，CBCentralManager会返回所有发现的外围设备。不管是否支持相关服务。通常在app里面，外围服务以 CBUUID 对象来唯一指定，传 CBUUID 数组来表示你感兴趣的设备，会返回只有对应服务的外围设备。

每次中心管理者发现一个外围设备，就会调用 centralManager:didDiscoverPeripheral:advertisementData:RSSI: 方法，新发现的设备会返回 CBPeripheral 对象，如果你想要连接某个发现的外围对象（CBPeripheral），需要对该对象保持强引用。比如：

```
-(void)centralManager::(CBCentralManager *)central didDiscoverPeripheral:(CBPeripheral *)peripheral advertisementData:(NSDictionary *)advertisementData RSSI:(NSNumber *)RSSI {
 NSLog(@"Discovered %@", peripheral.name);
 self.discoveredPeripheral = peripheral;
}
```

如果你需要连接多个外围（CBPeripheral）你需要用一个NSArray来持有这些对象，当你已经获取完了这些外围，你需要调用stopScan方法来停止扫描，以减少电量的消耗。

```
[myCentralManager stopScan]
```

### 连接外围设备

发现外围之后，调用 connectPeripheral:options: 方法来进行连接。

```
[myCentralManager connectPeripheral:peripheral options:nil];
```

连接成功后会收到CBCentralManager的回调 centralManager:didConnectPeripheral: ，在回调方法中，你需要设置连接成功的CBPeripheral的代理用来接收回调。

```
- (void)centralManager:(CBCentralManager *)central didConnectPeripheral:(CBPeripheral *)peripheral {
 NSLog(@"Peripheral connected");
 peripheral.delegate = self;
 ...
}
```

### 发现你所连接的外围所提供的服务

在连接了外围设备之后，你可以探索数据了，探索外围数据的第一步是发现有效的服务。因为外设可以发布的数据量有大小限制，你可能通过发现会获取更多的服务数据，通过调用discoverServices:方法来发现服务。

```
[peripheral discoverServices:nil];
```

如果传nil则会返回该外围设备的所有服务， 一般来说，你可以传你感兴趣的服务。当指定的服务被发现后，CBPeripheral 会回调 peripheral:didDiscoverServices: 代理方法。你可以实现该方法来访问服务数组。

```
- (void)peripheral:(CBPeripheral *)peripheral didDiscoverServices:(NSError *)error {
 for (CBService *service in peripheral.services) {
 NSLog(@"Discovered service %@", service);
 ...
}
```

### 发现一个服务的特征

当你发现服务后，接下来就是要发现该服务的特征，调用 peripheral 的 discoverCharacteristics:forService: 方法来发现特征。

```
[peripheral discoverCharacteristics:nil forService:interestingService];
```
同理，如果你传nil参数，则会获取到所有的特征对象，一般来说，你应该指定UUIDs来代表你想要发现的特征。

当获取完特征后，CBPeripheral对象会调用代理方法 peripheral:didDiscoverCharacteristicsForService:error: 来返回特征数组。

```
- (void)peripheral:(CBPeripheral *)peripheral didDiscoverCharacteristicsForService:(CBService *)service error:(NSError *)error {
 for (CBCharacteristic *characteristic in service.characteristics) {
 NSLog(@"Discovered characteristic %@", characteristic);
 ...
 }
}
```

### 获取特征数据

一个特征（characteristic）包含一个信号值来表征外围的服务信息，比如一个健康温度计服务的温度测量特征会有一个值来表示温度，你可以通过直接读取或订阅更新来获得这个特征值。

#### 读取特征值

调用 readValueForCharacteristic: 方法来读取特征值。

```
[peripheral readValueForCharacteristic:interestingCharacteristic];
```

当你尝试读取特征值时，peripheral对象通过调用 peripheral:didUpdateValueForCharacteristic:error: 方法来使得它的代理对象获取对应的值。

```
- (void)peripheral:(CBPeripheral *)peripheral didUpdateValueForCharacteristic:(CBCharacteristic *)characteristic error:(NSError *)error {
 NSData *data = characteristic.value;

 // parse the data as needed

 ...
}
```

并不是所有的特征值都是可读的，你可以检查包含CBCharacteristicPropertyRead 的属性来决定是否可读。如果你去读取一个不可读的特征， peripheral:didUpdateValueForCharacteristic:error: 代理方法将会被调用。

#### 订阅特征值

如果是静态的特征值，采取读取的方式比较高效，但是如果特征值是动态的，则采取订阅的方式。当特征值发生变化时，将会收到外围设备的通知。通过调用 setNotifyValue:forCharacteristic: 方法，第一个参数传YES来实现订阅特征值。

```
if(interestingCharacteristic.isNotifying == NO){
 [peripheral setNotifyValue:YES forCharacteristic:interestingCharacteristic];
}
```
当你订阅或取消订阅一个特征值时，peripheral会发送 peripheral:didUpdateNotificationStateForCharacteristic:error: 方法给代理对象。如果订阅失败，可以通过error获得失败结果。

```
- (void)peripheral:(CBPeripheral *)peripheral didUpdateNotificationStateForCharacteristic:(CBCharacteristic *)characteristic error:(NSError *)error {
 if (error) {
 NSLog(@"Error changing notification state: %@",
 [error localizedDescription]);
 }
}
```
并不是所有的特征都提供订阅，你可以通过 properties 的 CBCharacteristicPropertyNotify 或 CBCharacteristicPropertyIndicate 常量来决定改特征是否提供订阅。

当你成功订阅一个特征时，当特征值发生变化时，peripheral会调用 peripheral:didUpdateValueForCharacteristic:error: 方法来通知代理对象。在实现的代理方法中通过直接读取特征的方法来获得特征值。

### 写入特征数据

有些场景需要写数据到蓝牙模块，调用peripheral的 writeValue:forCharacteristic:type: 方法来实现对相应的特征写入数据。

```
[peripheral writeValue:dataToWrite forCharacteristic:interestingCharacteristic type:CBCharacteristicWriteWithResponse];
```
注意type参数，如果传CBCharacteristicWriteWithResponse，则peripheral会通过 peripheral:didWriteValueForCharacteristic:error: 代理方法来告知是否写入成功。

```
- (void)peripheral:(CBPeripheral *)peripheral didWriteValueForCharacteristic:(CBCharacteristic *)characteristic error:(NSError *)error {
 if (error) {
 NSLog(@"Error writing characteristic value: %@", [error localizedDescription]);
 }
 ...
}
```

如果传的是CBCharacteristicWriteWithoutResponse，则不会告知是否写入成功，也不一定保证写入成功，根据性能来决定。

特征可能只支持某些类型的写入。通过 properties 的CBCharacteristicPropertyWriteWithoutResponse or CBCharacteristicPropertyWrite 常量来决定。

## 在本地设备执行常见的外围任务

要在本地设备实现常见的外围设备任务，有以下步骤：

1.设置外围管理者对象。

2.在你本地外围设备设置服务和特征。

3.发布你的服务和特征到你设备本地的数据库。

4.广播服务。

5.回应连接中心的读写请求。

6.发送更新的特征值到订阅的中心设备。

### 设置外围管理对象

```
myPeripheralManager = [[CBPeripheralManager alloc] initWithDelegate:self queue:nil options:nil];
```
如果queue参数传nil，则事件默认运行在主线程。当创建完成后，CBPeripheralManager会调用代理方法 peripheralManagerDidUpdateState:，需要实现该代理方法确保你的设备是否支持本地外围设备。

### 设置服务与特征

在前面的图可以知道，本地外围的服务和特征是以树状的形式来组织的，所以设置的时候也要以树状的形式来组织，首先要明白的是服务和特征是怎么被识别的。

#### 服务和特征通过 UUIDs 来被识别

外围设备的服务和特征是通过128位的蓝牙规格指定的 UUIDs 来标记，通过核心蓝牙库的 CBUUID 对象来表征。虽然不是所有的蓝牙服务和特征都是有蓝牙兴趣组（SIG）预定义的，蓝牙SIG定义并发布了许多常用的uuid，为了方便，这些uuid被缩短为16位。例如，Bluetooth SIG预定义了16位UUID，该UUID将心率服务标识为180D.这个uuuid是从等效的128位UUID (0000180D-0000-1000-8000-00805F9B34FB)缩短而来的，它基于蓝牙4.0规范(卷3,F部分，第3.2.1节)中定义的蓝牙基础UUID。

CBUUID类提供了工厂方法，这使得在开发应用程序时更容易处理长uuid。例如，与其在代码中传递心率服务的128位UUID的字符串表示形式，不如使用UUIDWithString方法从服务预定义的16位UUID中创建一个CBUUID对象，如下所示:

```
CBUUID *heartRateServiceUUID = [CBUUID UUIDWithString: @"180D"];
```
当您从预定义的16位UUID创建一个CBUUID对象时，核心蓝牙预先填充了其他128位UUID。

#### 给你自己定义的服务和特征创建UUIDs

可能蓝牙预定义的UUIDs里面没有你的服务和特征，这时候，你需要自己创建预定义的UUIDs，你可以在终端输入命令行：uuidgen 获取到UUID,如：

```
$ uuidgen
71DA3FD1-7E10-41C1-B16F-4430B506CDE7
```

接下来，使用uuid来创建 CBUUID 对象，

```
CBUUID *myCustomServiceUUID = [CBUUID UUIDWithString:@"71DA3FD1-7E10-41C1-B16F-4430B506CDE7"];
```
#### 构建服务特征树

有了CBUUID来代表服务和特征后，你可以创建可变的服务和特征来构建服务特征树。如，你有一个特征uuid，你可以建立如下：

```
myCharacteristic = [[CBMutableCharacteristic alloc] initWithType:myCharacteristicUUID properties:CBCharacteristicPropertyRead value:myValue permissions:CBAttributePermissionsReadable];
```
你可以设置里面的 properties， permission， value，设置是否可以被中心设备读取。如果你指定一个特征的值，这个值会被缓存起来，properties和permissions将会被设置为可读，所以如果你想让中心设备写入值，或者想让这个值在它所属的服务的生命周期内发生变化，你必须指定这个值为nil。

接下来创建可变的服务，并且让特征与服务联系起来。

``` 
myService = [[CBMutableService alloc] initWithType:myServiceUUID primary:YES];
```

参数 primary 区别是主要服务还是辅助服务，比如心率探测器的主要服务为读取心率数据的服务，读取电池数据服务为辅助服务。

把服务与特征联系起来：

```
myService.characteristics = @[myCharacteristic];
```
### 发布你的服务和特征

完成服务特征树之后，你就可以发布服务特征到设备的数据库了，如下步骤：

1.添加服务

```
[myPeripheralManager addService:myService];
```

2.添加后，myPeripheralManager 会回调 peripheralManager:didAddService:error: 方法给代理，实现代理方法：

```
- (void)peripheralManager:(CBPeripheralManager *)peripheral didAddService:(CBService *)service error:(NSError *)error {
 if (error) {
 NSLog(@"Error publishing service: %@", [error localizedDescription]);
 }
 ...
}
```

### 广播你的服务和特征

把服务和特征写入到本地数据库之后，你可以广播你的服务了，调用以下方法：

```
[myPeripheralManager startAdvertising:@{CBAdvertisementDataServiceUUIDsKey : @[myFirstService.UUID, mySecondService.UUID] }];
```
同理需要实现代理方法来得知是否广播成功：

```
- (void)peripheralManagerDidStartAdvertising:(CBPeripheralManager *)peripheral error:(NSError *)error {
 if (error) {
 NSLog(@"Error advertising: %@", [error localizedDescription]);
 }
 ...
}
```

当广告成功后，中心设备可以发现并连接。

### 响应中心设备的读写

#### 处理读取请求

1.当连接的中心设备读取特征数据时，peripheral manager会调用代理方法 peripheralManager:didReceiveReadRequest: ，如下：

```
- (void)peripheralManager:(CBPeripheralManager *)peripheraldidReceiveReadRequest:(CBATTRequest *)request {
 if ([request.characteristic.UUID isEqual:myCharacteristic.UUID]) {
 ...
 }
}
```
2.如果特征的uuid是匹配的，下一步就是确保读取的偏移位置并没有超出特征值的边界。如：

```
if (request.offset > myCharacteristic.value.length) {
 [myPeripheralManager respondToRequest:request withResult:CBATTErrorInvalidOffset];
 return;
}
```

3.假设偏移位置是正确的，这时设置特征值。

```
request.value = [myCharacteristic.value subdataWithRange:NSMakeRange(request.offset, myCharacteristic.value.length - request.offset)];
```

4.回传request，调用 respondToRequest:withResult: 方法，

```
[myPeripheralManager respondToRequest:request withResult:CBATTErrorSuccess];
```
每次收到 peripheralManager:didReceiveReadRequest: 方法时，准确的调用 respondToRequest:withResult: 方法。当uuid没有匹配或者发生其他错误的时候，返回错误信息， 核心库里CBATTError Constants 里定义了各种错误。

#### 处理写请求

处理读请求也是同样的简单，同理，当收到写请求时，peripheral manager会调用peripheralManager:didReceiveWriteRequests: 方法，利用 myCharacteristic.value = request.value;来设置写入的值。在写入特征值时，一定也要考虑请求的偏移属性。

同理，当写完成后调用 respondToRequest:withResult: 方法来告知中心设备。但是request参数需要传peripheralManager:didReceiveWriteRequests:中的第一个。

### 发送更新的特征值到订阅的中心设备

1.当中心设备要订阅一个特征值时，peripheral manager 会调用代理方法： peripheralManager:central:didSubscribeToCharacteristic:，

```
- (void)peripheralManager:(CBPeripheralManager *)peripheral central:(CBCentral *)central didSubscribeToCharacteristic:(CBCharacteristic *)characteristic {
 NSLog(@"Central subscribed to characteristic %@", characteristic);
 ...
}
```

2.当订阅的值发生改变时

```
// fetch the characteristic's new value
NSData *updatedValue = BOOL didSendValue = [myPeripheralManager updateValue:updatedValue forCharacteristic:characteristic onSubscribedCentrals:nil];
```

如果 onSubscribedCentrals 参数传nil，所有的连接并订阅的中心设备将会收到通知。改方法会返回一个布尔值来表明是否发送成功，如果底层的发送队列已经满了，则会返回NO。当队列有空间并有效时，会调用代理方法 peripheralManagerIsReadyToUpdateSubscribers: 你可以实现该方法调用 updateValue:forCharacteristic:onSubscribedCentrals: 重新发送。

需要注意的是，取决于特征值数据包大小，有些特征值不一定完全返回，中心设备收到通知后，可用 CBPeripheral 的 readValueForCharacteristic: 方法来读取所有的数据包。

![hh1](158.jpeg)