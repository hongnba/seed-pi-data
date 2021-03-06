#Seed raspberry data service


######Please see details in the Developer notes.

##Project structure

   ``` 
│  manifest.yml
│  pom.xml
│  README.md  
├─src
│  ├─main
│  │  ├─java
│  │  │  └─com
│  │  │      └─pactera
│  │  │          └─predix
│  │  │              └─seed
│  │  │                  └─pi
│  │  │                      ├─bean
│  │  │                      │      DataNode.java
│  │  │                      │      DeviceEvent.java
│  │  │                      │      Dht.java
│  │  │                      │      EventValue.java
│  │  │                      │      Item.java
│  │  │                      │      ItemField.java
│  │  │                      │      ItemValue.java
│  │  │                      │      PageParams.java
│  │  │                      │      
│  │  │                      ├─boot
│  │  │                      │      AppConfiguration.java
│  │  │                      │      Application.java
│  │  │                      │      
│  │  │                      ├─dao
│  │  │                      │      DataDao.java
│  │  │                      │      
│  │  │                      └─service
│  │  │                              DataService.java
│  │  │                              
│  │  └─resources
│  │      │  application.properties
│  │      │  db.sql
│  │      │  
│  │      └─META-INF
│  │              spring.provides
│  │              
│  └─test
│      ├─java
│      └─resources
│              application-scratch.properties
│                          
   ``` 

## Installation
 - clone repository  
    `>git clone https://github.com/zhigangyu/seed-pi-data.git`
    
 - Create a postgres service instance:
 
 	`>cf create-service postgres shared-nr db`
 	
 - Create a redis service instance:
 
 	`>cf create-service redis-1 shared-vm redis1`
 	
 - Update manifest.yml to change the application name
 	```
 	---
applications:
  - name: data-pi-data # change this to your application name
    buildpack: java_buildpack
    path: target/pi-data-0.0.1.jar
services:
    - db
    - redis1
 	
 	```
 	
 - Build project
 
 	`> mvn clean package`
 	
 - Push to the cloud.
 
 	`> cf push`
 	
 - Using Cloud Foundry CLI, find the database information by looking up the VCAP service
 	```
 	> cf env seed-pi-data
Getting env variables for ...
OK

System-Provided:
{
 "VCAP_SERVICES": {
  "postgres": [
   {
    "credentials": {
     "ID": 0,
     "binding_id": "1724fc58-e2f0-4841-a1b8-ae6255ca2065",
     "database": "d328edf88d4fc43689ef2d170e84d887c",
     "dsn": "host=10.72.6.134 port=5432 user=u328edf88d4fc43689ef2d170e84d887c password=93c309d1c8454c35afbea0c2bb68fa50 dbname=d328edf88d4fc43689ef2d170e84d887c connect_timeout=5 sslmode=disable",
     "host": "10.72.6.134",
     "instance_id": "5c60f1ad-dc4b-4ff0-a14a-75797e189df2",
     "jdbc_uri": "jdbc:postgres://u328edf88d4fc43689ef2d170e84d887c:93c309d1c8454c35afbea0c2bb68fa50@10.72.6.134:5432/d328edf88d4fc43689ef2d170e84d887c?sslmode=disable",
     "password": "93c309d1c8454c35afbea0c2bb68fa50",
     "port": "5432",
     "uri": "postgres://u328edf88d4fc43689ef2d170e84d887c:93c309d1c8454c35afbea0c2bb68fa50@10.72.6.134:5432/d328edf88d4fc43689ef2d170e84d887c?sslmode=disable",
     "username": "u328edf88d4fc43689ef2d170e84d887c"
    },
    "label": "postgres",
    "name": "db",
    "plan": "shared-nr",
    "provider": null,
    "syslog_drain_url": null,
    "tags": [
     "rdpg",
     "postgresql"
    ]
   }
 	```
 	
 - use [PgStudio](https://studio.run.aws-usw02-pr.ice.predix.io/) to create table 
 
 	```
 	create table t_pi(
		n_id SERIAL primary key,
		d_dateline	timestamp, 
		c_name varchar(100),
		c_category varchar(50),
		c_value varchar(50),
		c_quality varchar(50),
		c_address character varying(200)
	);
	
	create table t_device(
		n_id SERIAL primary key,
		d_dateline	timestamp, 
		c_name varchar(100),
		c_value decimal(10,3),
		c_device varchar(50)
	);
 	```
 	
 - Configure [AppConfiguration.java](https://github.com/zhigangyu/seed-pi-data/blob/master/src/main/java/com/pactera/predix/seed/pi/boot/AppConfiguration.java) to set database infomation
 
 	```
 	dataSource.setUrl("jdbc:postgres://u328edf88d4fc43689ef2d170e84d887c:93c309d1c8454c35afbea0c2bb68fa50@10.72.6.134:5432/d328edf88d4fc43689ef2d170e84d887c?sslmode=disable");
	dataSource.setUsername("u328edf88d4fc43689ef2d170e84d887c");
	dataSource.setPassword("93c309d1c8454c35afbea0c2bb68fa50");
 	```
 	
 - use maven to build project
 
 	`> mvn clean package`
 	
 - use cf push command to deploy App  
 ```
> cf push
 ```
    
## Generate Predix Machine container && Configure Machine Services: 
 - Configure the mod bus config xml to set up the Connection node ("dataNodeConfig") and a Subscription node ("datasubscriptionconfig")
 - Replace the contents of this file $PREDIX_MACHINE_HOME/configuration/machine/com.ge.dspmicro.machineadapter.modbus-0.xml with the text below.
 - ![image](http://7xrn7f.com1.z0.glb.clouddn.com/16-5-31/66219558.jpg) 
 - Configure the http river config file
$PREDIX_MACHINE_HOME/configuration/machine/com.ge.dspmicro.httpriver.send-0.config
   ```
# [Required] A friendly and unique name of the HTTP River.
com.ge.dspmicro.httpriver.send.river.name="Http Sender Service"


# [Required] Route to the river receive application. (e.g. myapp.mycloud.com)
com.ge.dspmicro.httpriver.send.destination.host="seed-pi-data.run.aws-usw02-pr.ice.predix.io"
   ```

#### modify the contents of this file $PREDIX_MACHINE_HOME/configuration/machine/com.ge.dspmicro.predixcloud.identity.config according to your UAA Service.

   ```
com.ge.dspmicro.predixcloud.identity.oauth.authmode="CLIENT_CREDENTIALS"

#
# [Required] The Predix cloud URL of an OAuth2 authorization endpoint. This is the UAA URL for 
# the technician to log into the cloud.
#
com.ge.dspmicro.predixcloud.identity.oauth.authorize.url=""

#
# [Required] Predix Cloud enrollment endpoint url
#
com.ge.dspmicro.predixcloud.identity.uaa.enroll.url=""

#
# [Required] Predix Cloud UAA token endpoint
#
com.ge.dspmicro.predixcloud.identity.uaa.token.url="https://8b1123fa-67c8-47eb-9cf4-b38448aafa71.predix-uaa.run.aws-usw02-pr.ice.predix.io/oauth/token"

#
# Predix Cloud UAA client credentials
#
com.ge.dspmicro.predixcloud.identity.uaa.clientid="predix"
com.ge.dspmicro.predixcloud.identity.uaa.clientsecret="predix"
com.ge.dspmicro.predixcloud.identity.uaa.clientsecret.encrypted=""

#
# Predix device identity.
# deviceid must contain only lower case letters (a-z), numbers (0-9), and : - _. Must begin with a letter or number.
#
com.ge.dspmicro.predixcloud.identity.deviceid="pi01"
# [Required if using JWT AuthMode] Predix cloud internal device id
com.ge.dspmicro.predixcloud.identity.asdid=""
com.ge.dspmicro.predixcloud.identity.tenantid=""

#
# [Required if using JWT AuthMode] Predix device MAC address used to pin certificate to device. 
# Acceptable formats are 6 bytes of case insensitive hex with each byte separated by ':','-' or none
#   example: xx:xx:xx:xx:xx:xx or xx-xx-xx-xx-xx-xx or XxXxXxXxXxXx   
#
com.ge.dspmicro.predixcloud.identity.mac="b8-27-eb-b8-ee-2f"

#
# [Optional] Predix cloud upload URL - This is used for uploading configuration command from the device.
# The device id will be appended automatically to the end of the URL if not set.
# When the device is enrolled this value will be set automatically. 
#
com.ge.dspmicro.predixcloud.identity.cloud.upload.url=""

#
# [Optional] Predix cloud Yeti signature URL - The cloud service for validating install packages.
# When the device is enrolled this value will be set automatically. 
#
com.ge.dspmicro.predixcloud.identity.yeti.signature.url=""

# 
# [Required if using JWT AuthMode] Shared secret required for certificate based enrollment.
#
com.ge.dspmicro.predixcloud.identity.enroll.sharedSecret=""
com.ge.dspmicro.predixcloud.identity.enroll.sharedSecret.encrypted=""

   ```
   
# Raspberry Pi 3
![image](https://dri1.img.digitalrivercontent.net/Storefront/Company/msintl/images/English/en-INTL-Raspberry-Pi-3-16GB-10-Class-with-NOOBS-QK9-00028/en-INTL-L-Raspberry-Pi-3-16GB-10-Class-with-NOOBS-QK9-00028-mnco.jpg)
## Connect the DHT22
![image](http://7xuwcw.com1.z0.glb.clouddn.com/20150830000751994.jpg)
![image](http://7xuwcw.com1.z0.glb.clouddn.com/20150830002549737.jpg)

##install pymodbus
   ```
git clone git://github.com/bashwork/pymodbus.git
cd pymodbus
python setup.py install
   ```
   
## Modbus Server
   ```
#!/usr/bin/env python
#---------------------------------------------------------------------------# 
# import the various server implementations
#---------------------------------------------------------------------------# 
from pymodbus.server.sync import StartTcpServer

from pymodbus.device import ModbusDeviceIdentification
from pymodbus.datastore import ModbusSequentialDataBlock
from pymodbus.datastore import ModbusSlaveContext, ModbusServerContext

from pymodbus.transaction import ModbusRtuFramer
#---------------------------------------------------------------------------# 
# configure the service logging
#---------------------------------------------------------------------------# 
import logging
logging.basicConfig()
log = logging.getLogger()
log.setLevel(logging.DEBUG)


store = ModbusSlaveContext(
    di = ModbusSequentialDataBlock(0, [17]*100),
    co = ModbusSequentialDataBlock(0, [17]*100),
    hr = ModbusSequentialDataBlock(0, [17]*100),
    ir = ModbusSequentialDataBlock(0, [17]*100))
context = ModbusServerContext(slaves=store, single=True)

#---------------------------------------------------------------------------# 
# initialize the server information
#---------------------------------------------------------------------------# 
# If you don't set this or any fields, they are defaulted to empty strings.
#---------------------------------------------------------------------------# 
identity = ModbusDeviceIdentification()
identity.VendorName  = 'Pymodbus'
identity.ProductCode = 'PM'
identity.VendorUrl   = 'http://github.com/bashwork/pymodbus/'
identity.ProductName = 'Pymodbus Server'
identity.ModelName   = 'Pymodbus Server'
identity.MajorMinorRevision = '1.0'

#---------------------------------------------------------------------------# 
# run the server you want
#---------------------------------------------------------------------------# 
# Tcp:
StartTcpServer(context, identity=identity, address=("10.10.23.30", 502))

   ```
## start Modbus TCP Server
```

sudo ./modbus-server.py
```	
## DHT22 modbus client
   ```
#!/usr/bin/python

from pymodbus.client.sync import ModbusTcpClient as ModbusClient

import sys
import time
import urllib
import urllib2 
import Adafruit_DHT

import logging
logging.basicConfig()
log = logging.getLogger()
log.setLevel(logging.DEBUG)

client = ModbusClient('10.10.23.30', port=502)

# Parse command line parameters.
sensor = Adafruit_DHT.DHT22
pin = 18

while True:
	humidity, temperature = Adafruit_DHT.read_retry(sensor, pin)

	if humidity is not None and temperature is not None:
		print('Temp={0:0.1f}*  Humidity={1:0.1f}%'.format(temperature, humidity))
		client.connect()
		client.write_register(1, temperature)
		client.write_register(2, humidity)
		client.close()
	else:
		print('Failed to get reading. Try again!')

	time.sleep(60*5)
    
sys.exit(1)

   ```
	
## start DHT22 modbus client
 	`>sudo ./modbus-client.py`

## PM2.5
####Connect the sensor to Raspberry Pi 3

![image](http://7xuwcw.com1.z0.glb.clouddn.com/pm25.PNG)



####Python Code
```
#!/usr/bin/python
# -*- coding: utf-8 -*-
from pymodbus.client.sync import ModbusTcpClient as ModbusClient
import time
import serial
import logging

#check the checksum value
def check_value(data, length):
    if(0 == len(data) or length != 32):
        print "check_value input error!!!!!"
        return False
    else:
        #get the checksum from the two bytes of the data
        csum = int('%02x'%ord(data[length - 2]) + '%02x'%ord(data[length - 1]), 16)
        print "csum is %d" % csum
        tmp = 0
        
        #calculate the checksum
        for i in range(length - 2):
            tmp += int('%02x'%ord(data[i]), 16)
        print "tmp is %s" % tmp
        
        #compare the checksum
        if(tmp == csum):
            return True
        else:
            return False

#read the aqi data from the sensor
def read_pms_data():
    logging.basicConfig()
    log = logging.getLogger()
    log.setLevel(logging.DEBUG)
    
    #head data
    head = ''
    
    #aqi data
    data = ''
    
    #create the modbus client to send the aqi data to the modbus server.
    try:
        client = ModbusClient('10.10.23.30', port=502)
    except Exception as e:
        logging.exception(e)
        
    #serial instance
    ser = None
    
    try:
        #open serial
        ser = serial.Serial("/dev/ttyAMA0", 9600, timeout=1.5)
    except Exception as e:
        logging.exception(e)
        
    while True:
        try:
            #read the head data
            head = ser.read(1)
            
            #the head data must be 42
            if('42' == '%02x'%ord(head)):
                print "find head!!!!!"
                
                #read the aqi data
                data = ser.read(31)
                print "read len is [%d][%s]" % (len(data), '%02x'%ord(data[0]))
                
                #judge the length of the data and check the checksum of the data
                if('4d' == '%02x'%ord(data[0]) and 31 == len(data) and check_value(head + data, 32)):
                    length = 2
                    print "%s%s" % ('%02x'%ord(head), '%02x'%ord(data[0]))
                    print_data = '%02x'%ord(head) + '%02x'%ord(data[0])
                    
                    #serialize the data to output
                    while length < 32:
                        print_data = print_data + '%02x'%ord(data[length - 1]) + '%02x'%ord(data[length])
                        length = length + 2
                    print print_data
                    #print_data = "pm1.0 value is %s, " + str(int('%02x'%ord(data[9]) + '%02x'%ord(data[9]), 16))
                    pm1p0 = int('%02x'%ord(data[9]) + '%02x'%ord(data[10]), 16)
                    pm2p5 = int('%02x'%ord(data[11]) + '%02x'%ord(data[12]), 16)
                    pm10 = int('%02x'%ord(data[13]) + '%02x'%ord(data[14]), 16)
                    print_data = "pm1.0 value is %d μg/m³" % pm1p0 + \
                                 "\t\tpm2.5 value is %d μg/m³" % pm2p5 + \
                                 "\t\tpm10 value is %d μg/m³" % pm10
                    
                    print print_data
                    client.connect()
                    client.write_register(3, pm1p0)
                    client.write_register(4, pm2p5)
                    client.write_register(5, pm10)
                    pm1p0_value = client.read_holding_registers(3,1)
                    pm2p5_value = client.read_holding_registers(4,1)
                    pm10_value = client.read_holding_registers(5,1)
                    log.debug("send pm1.0 value = %s[%s]" % (str(pm1p0_value.registers[0]), type(pm1p0_value.registers[0])))
                    log.debug("send pm2.5 value = %s[%s]" % (str(pm2p5_value.registers[0]), type(pm2p5_value.registers[0])))
                    log.debug("send pm10 value = %s[%s]" % (str(pm10_value.registers[0]), type(pm10_value.registers[0])))
                    #client.write_register(4, int('%02x'%ord(data[11]) + '%02x'%ord(data[12]), 16))
#                     client.write_register(5, int('%02x'%ord(data[13]) + '%02x'%ord(data[14]), 16))
                    client.close()
                    #print "read r3 is %d[%s]" % (client.read_register(3), type(client.read_register(3)))
                    #print "read r4 is %d[%s]" % (client.read_register(4), type(client.read_register(4)))
#                     print "read r5 is %d[%s]" % (client.read_register(5), type(client.read_register(5)))
                    time.sleep(5)
                else:
                    print "read data error!!!!![%d][%s]" % (len(data), '%02x'%ord(data[0]))
                    data = ''
            else:
                print "head error!!!!![%s]" % ('%02x'%ord(head))
                head = ''
                continue
        except Exception as e:
                logging.exception(e)
'''
    if check == tmp:
      print "temperature :", temperature, "*C, humidity :", humidity, "%"
      client.connect()
      client.write_register(1, temperature)
      client.write_register(2, humidity)
      client.close()
    else:
      print "wrong"
      print "temperature :", temperature, "*C, humidity :", humidity, "% check :", check, ", tmp :", tmp
    GPIO.cleanup()
    time.sleep(60)
'''

if __name__=='__main__':
    read_pms_data()
```
	
## start predix machine
 	`>sudo ./predixmachine clean`
	
	
## use [PgStudio](https://studio.run.aws-usw02-pr.ice.predix.io/) to view the time series data 
![image](http://7xuwcw.com1.z0.glb.clouddn.com/t_pi.png)
 
#### Use curl command in liunx/mac terminal  to query data: 
``` 
curl -X POST --header "Content-Type: application/json" --data '{"page":1,"pageSize":20,"from":"2015-05-30","to":"2016-06-01"}' "https://seed-pi-data.run.aws-usw02-pr.ice.predix.io/api/pi/dht"
```
#### query current value by location (https://seed-pi-data.run.aws-usw02-pr.ice.predix.io/item/{location})
```
curl -X GET --header "Content-Type: application/json" "https://seed-pi-data.run.aws-usw02-pr.ice.predix.io/item/dl"
```

#### query min value by location (https://seed-pi-data.run.aws-usw02-pr.ice.predix.io/item/min/{location})
```
curl -X GET --header "Content-Type: application/json" "https://seed-pi-data.run.aws-usw02-pr.ice.predix.io/item/min/dl"
```

#### query max value by location (https://seed-pi-data.run.aws-usw02-pr.ice.predix.io/item/max/{location})
```
curl -X GET --header "Content-Type: application/json" "https://seed-pi-data.run.aws-usw02-pr.ice.predix.io/item/max/dl"
```
 
#### Developer notes:

 - To load in eclipse you may use [SpringSource Tool Suite - STS](https://spring.io/tools/sts/all)  
  ```
  >mvn eclipse:clean eclipse:eclipse  
  
  open eclipse and use the following commands:
  File/Import/General/Existing Projects/Browse to seed-pi-data dir   
  ```  