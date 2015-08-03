# microgear-nodejs

microgear-nodejs คือ client library ภาษา Node.js ที่ทำหน้าที่เป็นตัวกลางในการเชื่อมโยง application code หรือ hardware เข้ากับบริการของ netpie platform เพื่อการพัฒนา IOT application รายละเอียดเกี่ยวกับ netpie platform สามารถศึกษาได้จาก http://netpie.io

## การติดตั้ง

```
npm install microgear
```

ตัวอย่างการเรียกใช้
```js
var MicroGear = require('microgear');

const APPKEY    = <APPKEY>;
const APPSECRET = <APPSECRET>;
const APPID     = <APPID>;

var microgear = MicroGear.create({
    gearkey : APPKEY,
    gearsecret : APPSECRET
});

microgear.on('connected', function() {
    console.log('Connected...');
    microgear.setname("mygear");
    setInterval(function() {
        microgear.chat('mygear', 'Hello world.');
    },1000);
});

microgear.on('message', function(topic,body) {
    console.log('incoming : '+topic+' : '+body);
});

microgear.on('closed', function() {
    console.log('Closed...');
});

microgear.connect(APPID);
```
## การใช้งาน library
---
**microgear create(config)**

**arguments**
* *config* เป็น json object ที่ที่มี attribute ดังนี้
  * *gearkey* `string` - เป็น key สำหรับ gear ที่จะรัน ใช้ในการอ้างอิงตัวตนของ gear
  * *gearsecret* `string` - เป็น secret ของ key ซึ่งจะใช้ประกอบในกระบวนการยืนยันตัวตน
  * *scope* `string` - เป็นการระบุขอบเขตของสิทธิ์ที่ต้องการ

**scope**
เป็นการต่อกันของ string ในรูปแบบต่อไปนี้ คั่นด้วยเครื่องหมาย comma
  * [r][w]:</topic/path> - r และ w คือสิทธิ์ในการ publish ละ subscribe topic ดังที่ระบุ เช่น rw:/outdoor/temp
  *  name:<gearname> - คือสิทธิ์ในการตั้งชื่อตัวเองว่า <gearname>
  *  chat:<gearname> - คือสิทธ์ในการ chat กับ <gearname>
ในขั้นตอนของการสร้าง key บนเว็บ netpie.io นักพัฒนาสามารถกำหนดสิทธิ์ขั้นพื้นฐานให้แต่ละ key ได้อยู่แล้ว หากการ create microgear อยู่ภายใต้ขอบเขตของสิทธิ์ที่มี token จะถูกจ่ายอัตโนมัติ และ microgear สามารถเชื่อมต่อ netpie platform ได้ทันที แต่หาก scope ที่ร้องขอ มากเกินกว่าที่กำหนด นักพัฒนาจะได้รับการแจ้ง ให้พิจารณาอนุมัติ microgear เป็นรายๆไป ข้อควรระวัง หาก microgear มีการกระทำการเกินกว่าสิทธิ์ที่ได้รับ เช่น publish ไปยัง topic ที่ไม่มีสิทธิ์ broker จะตัดการเชื่อมต่อของ microgear โดยอัตโนมัติ ในกรณีที่ใช้ APPKEY เราสามารถละเว้น attribute นี้ได้ เพราะ APPKEY จะได้สิทธิ์ทุกอย่างโดย default อยู่แล้ว 

```
var microgear = MicroGear.create({
    gearkey : "sXfqDcXHzbFXiLk",
    gearsecret : "DNonzg2ivwS8ceksykGntrfQjxbL98",
    scope : "r:/outdoor/temp,w:/outdoor/valve,name:logger,chat:plant"
});
```
---
## microgear
**void microgear.connect(*appid*)**

**arguments**
* *appid* `string` - คือกลุ่มของ application ที่ microgear จะทำการเชื่อมต่อ 

```
microgear.connect("happyfarm");
```
---
**void microgear.setname(*gearname*)**
microgear สามารถตั้งชื่อตัวเองได้ (ตามสิทธิ์ของ gearkey) ซึ่งสามารถใช้เป็นชื่อเรียกในการใช้ฟังก์ชั่น chat()

**arguments**
* *gearname* `string` - ชื่อของ microgear นี้   

```
microgear.setname("plant");
```
---
**void microgear.chat(*gearname*, *message*)**

**arguments**
* *gearname* `string` - ชื่อของ microgear ที่ต้องการจะส่งข้อความไปถึง 
* *message* `string` - ข้อความ

```
microgear.chat("valve","I need water");
```
---
**void microgear.publish(*topic*, *message*)**
ในการณีที่ต้องการส่งข้อความแบบไม่เจาะจงผู้รับ สามารถใช้ฟังชั่น publish ไปยัง topic ที่กำหนดได้ (ขึ้นอยู่กับสิทธิ์ของ gearkey) ซึ่งจะมีแต่ microgear ที่ subscribe topoic นี้เท่านั้น ที่จะได้รับข้อความ

**arguments**
* *topic* `string` - ชื่อของ topic ที่ต้องการจะส่งข้อความไปถึง 
* *message* `string` - ข้อความ

```
microgear.publish("/outdoor/temp","28.5");
```
---
**void microgear.subscribe(*topic*)**
microgear อาจจะมีความสนใจใน topic ใดเป็นการเฉพาะ เราสามารถใช้ฟังก์ชั่น subscribe() ในการบอกรับ message ของ topic นั้นได้ (ขึ้นอยู่กับสิทธิ์ของ gearkey ด้วย)

**arguments**
* *topic* `string` - ชื่อของ topic ที่ต้องการจะส่งข้อความไปถึง 

```
microgear.subscribe("/outdoor/temp");
```
---
**void microgear.unsubscribe(*topic*)**
ยกเลิกการ subscribe

**arguments**
* *topic* `string` - ชื่อของ topic ที่ต้องการจะส่งข้อความไปถึง 

```
microgear.unsubscribe("/outdoor/temp");
```




---
## Events
application ที่รันบน microgear จะมีการทำงานในแบบ event driven คือเป็นการทำงานตอบสนองต่อ event ต่างๆ ด้วยการเขียน callback function ขึ้นมารองรับ
```
miucrogear.on(eventname,function(param) {
	/* do something */
});
```
netpie platform เวอร์ชั่นปัจจุบัน มี event ดังต่อไปนี้

**Event: 'connected'**
เกืดขึ้นเมื่อ microgear library เชื่อมต่อกับ platform สำเร็จ

**Event: 'closed'**
เกืดขึ้นเมื่อ microgear library ตัดการเชื่อมต่อกับ platform

**Event: 'message'**
เมื่อมี message เข้ามา จะเกิด event นี้ข้น พร้อมกับข้อมูลเกี่ยวกับ message นั้น
```
microgear.on("message", function(topic,msg) {
	console.log("Incoming message: "+mesage);
});
```

**Event: 'present'**
event นี้จะเกิดขึ้นเมื่อมี microgear ใน appid เดียวกัน online เข้ามาเชื่อมต่อ netpie
```
microgear.on("present", function(event) {
	console.log("New friend found: "+event.gearkey);
});
```
**Event: 'absent'**
event นี้จะเกิดขึ้นเมื่อมี microgear ใน appid เดียวกัน offline หายไป
```
microgear.on("absent", function(event) {
	console.log("Friend lost: "+event.gearkey);
});
```
