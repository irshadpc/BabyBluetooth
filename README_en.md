![](http://images.jumppo.com/uploads/BabyBluetooth_logo.png)

The easiest way to use Bluetooth (BLE )in ios, even baby can use.  CoreBluetooth wrap.

#feature

- 1: CoreBluetooth wrap，simple and easy for use.
- 2: CoreBluetooth is a dependency on the delegate, and most times, call method at delegate then go into a delegate, and over and over, it's messy.BabyBluetooth favor to using block.
- 3: call method in a serial，it's simple and elegant syntax.
- 4: using channel switch blocks in a group.
- 5: convenience tools class
- 6: comprehensive documentation and active project
- 7: more star library for ios Bluetooth in github（not PhoneGap and SensorTag）
- 8: include demo and tutorial
- 9: works with both the central model and peripheral model

current version 0.7.0

[update histroy](https://github.com/coolnameismy/BabyBluetooth/wiki/verision)

![](.res/qrcode.jpg)

# Contents

* [QuickExample](#user-content-QuickExample)
* [how to install](#user-content-how-to-install)
* [how to use](#user-content-how-to-use)
* [demo explain](#user-content-demo-explain)
* [Compatibility](#user-content-Compatibility)
* [plan for update](#user-content-plan-for-update)
* [Community with US](#user-content-community with us)
* [wish](#user-content-wish)
 
# QuickExample

## central model 
>  let you apps be Central and contact other BLE4.0 peripheral 

```objc

//import head files
#import "BabyBluetooth.h"
//define var
BabyBluetooth *baby;

-(void)viewDidLoad {
    [super viewDidLoad];

    //init BabyBluetooth
    baby = [BabyBluetooth shareBabyBluetooth];
    //set delegate
    [self babyDelegate];
    //direct use，no longer wait for status Of CBCentralManagerStatePoweredOn
    baby.scanForPeripherals().begin();
}

//set babybluetooth delegate
-(void)babyDelegate{

    //when scanfor perihpheral
    [baby setBlockOnDiscoverToPeripherals:^(CBCentralManager *central, CBPeripheral *peripheral, NSDictionary *advertisementData, NSNumber *RSSI) {
        NSLog(@"搜索到了设备:%@",peripheral.name);
    }];
   
    //filter
    //discover peripherals filter
    [baby setFilterOnDiscoverPeripherals:^BOOL(NSString *peripheralName, NSDictionary *advertisementData, NSNumber *RSSI) {
        //最常用的场景是查找某一个前缀开头的设备 most common usage is discover for peripheral that name has common prefix
        //if ([peripheralName hasPrefix:@"Pxxxx"] ) {
        //    return YES;
        //}
        //return NO;
        //设置查找规则是名称大于1 ， the search rule is peripheral.name length > 1
        if (peripheralName.length >1) {
            return YES;
        }
        return NO;
    }];
}
  
```
more method and delegate please see : [wiki](https://github.com/coolnameismy/BabyBluetooth/wiki)

Central model deme see:[BabyBluetoothAppDemo](https://github.com/coolnameismy//BabyBluetoothExamples/BabyBluetoothAppDemo)


## peripheral model 
>  let you apps be a BLE4.0 peripheral


mock a peripheral which has 2 services and 6 characteristics together

````objc
//import head files
#import "BabyBluetooth.h"
//define var
BabyBluetooth *baby;

-(void)viewDidLoad {
    [super viewDidLoad];

    //config first service
    CBMutableService *s1 = makeCBService(@"FFF0");
    //config s1‘s characteristic
    makeCharacteristicToService(s1, @"FFF1", @"r", @"hello1");//can read
    makeCharacteristicToService(s1, @"FFF2", @"w", @"hello2");//can write
    makeCharacteristicToService(s1, genUUID(), @"rw", @"hello3");//can read,write ,uuid be automatically generate
    makeCharacteristicToService(s1, @"FFF4", nil, @"hello4");//default property is rw
    makeCharacteristicToService(s1, @"FFF5", @"n", @"hello5");//can notiy
    //config seconed service s2
    CBMutableService *s2 = makeCBService(@"FFE0");
    //a static characteristic and has cached vuale ,it must be only can read.
    makeStaticCharacteristicToService(s2, genUUID(), @"hello6", [@"a" dataUsingEncoding:NSUTF8StringEncoding]);
   
    //init BabyBluetooth
    baby = [BabyBluetooth shareBabyBluetooth];
    //config delegate
    [self babyDelegate];
    //let peripheral add services and start advertising
    baby.bePeripheral().addServices(@[s1,s2]).startAdvertising();
}

//set baby peripheral model delegate
-(void)babyDelegate{

     //设置添加service委托 | set didAddService block
    [baby peripheralModelBlockOnPeripheralManagerDidUpdateState:^(CBPeripheralManager *peripheral) {
        NSLog(@"PeripheralManager trun status code: %ld",(long)peripheral.state);
    }];
    
    //设置添加service委托 | set didAddService block
    [baby peripheralModelBlockOnDidStartAdvertising:^(CBPeripheralManager *peripheral, NSError *error) {
        NSLog(@"didStartAdvertising !!!");
    }];
    
    //设置添加service委托 | set didAddService block
    [baby peripheralModelBlockOnDidAddService:^(CBPeripheralManager *peripheral, CBService *service, NSError *error) {
        NSLog(@"Did Add Service uuid: %@ ",service.UUID);
    }];

    //.....
}

````
  
more method and delegate please see : [wiki](https://github.com/coolnameismy/BabyBluetooth/wiki)

peripheral model demo see :[BluetoothStubOnIOS](https://github.com/coolnameismy//BabyBluetoothExamples/BluetoothStubOnIOS)

# how to install

##1 manual
step1: let src folder‘s files import your project

step2:import .h

````objc
#import "BabyBluetooth.h"
````

##2 cocoapods
step1:add the following line to your Podfile:
````
pod 'BabyBluetooth','~> 0.6.0'

````

step2:import header files
````objc
#import "BabyBluetooth.h"
````

# how to use
[please see wiki](https://github.com/coolnameismy/BabyBluetooth/wiki)


# demo explain
**BabyBluetoothExamples/BabyBluetoothAppDemo** :similar to lightblue ,implement by BabyBluetooch

functionality
- 1：scan for nearby Bluetooth peripheral
- 2：cannect and discover peripheral’s services and characteristic
- 3：read characteristic and characteristic‘s value, and descriptors or descriptors' value
- 4：write 0x01 to characteristic
- 5：subscription/unsubscription characteristic

**BabyBluetoothExamples/BluetoothStubOnIOS **: an iOS app ， when they launch, they will start a BLE4,0 peripheral and provides two services and together with six characteristics. it's Babybluetooth peripheral model develope example


**BabyBluetoothExamples/BabyBluetoothOSDemo **: mac osx app，osx and ios are not different on CoreBluetooth，so BabyBluetooth can use in both ios and osx 。
functionality
- 1：scanfor peripheral, connect peripheral 、read characteristic，read characteristic 's value, discover descriptors and descriptors' value，the message all in nslog, this app none UI

**BabyBluetoothExamples/BluetoothStubOnOSX **: mac os app 。it can act a Bluetooth peripheral.this app solve that learning Bluetooth but without periphera。and it also can be a demo and learning peripheral model program 。it write by swift

functionality
- 1：be a Bluetooth peripheral，and can be scanned, connected, written, read, subscribe by CBCentralManager
- 2：provide a service and contain three characteristics,each has one of reading,write, subscribe permission

# compatibility
- Bluetooth 4.0，another name is BLE，ios6 free access
- both os and ios
- in ios app, it's can be used in a simulator

# plan for the update
 
- improve English code note 
- babyBluetooth test application
- swift babyBluetooth develop

history verison，see wiki

# Community with the US
QQ Group Number:168756967. [QQ is an IM from china](http://im.qq.com/)

Or Mail me:coolnameismy@hotmail.com

*nearest update：* be support for BLE peripheral model 


# wish
  - coding BabyBluetooth is work hard, wish audience star BabyBluetooch for support 
  - if find bug or function is not enough, please issues me
  - wish everyone join BabyBluetooth and active pull requests 。my code is only just beginning and it has a lot of area for improvements。
  - wish to learn, communicating, growing with yours by pull requests in BabyBluetooch 
  - English grammar issue, please help for fix
  - thanks
 
