# Firepump

A system to control the pumps and monitor tank levels and environmental data.

## Aim

The aim of this project is to provide an automation system to operate fire fighting equipment around a rural property in the Adelaide hills.

During the 2019/12/19 Lobethal fire we were not at our house when the fire front came through, we did however have a fire sprinkler system on the roof of the house run by a petrol powered pump attached to a 100,000 liter rain water tank.

Thankfully the CFS attended the house and operated the pump. The original pump was destroyed by the fire after saving the house.

## Lessons learnt

1. The fire pump was not protected adequately
    * Place on concrete pad
    * Move away from outbuildings
    * Insulate from radiant heat
1. We are unlikely to be home during a catastrophic bushfire day and during working hours.
    * Provide remote start
    * Monitor environment and alert
1. Electricity is disconnected during fires
    * System must not require grid power
1. The water continued to siphon out of the tank after the pump had failed draining the complete 100,000 liters.
    * Provide solenoid water valve

### Decisions

The requirements are that the hardware used be accessible and inexpensive and that the solution be freely available to those that may want to replicate it.

* Battery start diesel pump
* Sensor network
* Cloud based provider (google app engine)
* Low bandwidth mqtt pub/sub
* IFTTT notifications
* Remote activation
* Battery backup 4G network
* LoRa WAN backup (things network)
* Mobile website / app

### Discounted

Due to expense and security concerns I have discounted a number of alternatives including:

1. Running a full *nix instance on a cloud provider with a more standard automation system (e.g. homeassistant, nodered etc.).
1. A local mqtt broker with some form of dynamic dns and firewall dmz.

These require maintenance and configuration and require a level of technical expertise that put them out of the reach of most users. It is understood that the chosen solution is still quite technical in nature.

## Google IOT

So the first set of examples I saw that made sense and that I was able to hack a lot of code from were:

* [talk](https://www.youtube.com/watch?v=RYaprBSDy8A)
* [code](https://github.com/GabeWeiss/GoogleIoTCoreApp)

It appears that both Amazon and Microsoft Azure offer IOT in a similar way (MQTT with PKI) but the above gave me the lowest barrier to entry, I would have amazon'd as I already pay for route66 but there wasn't the level of tutorial and sample code.

## Amazon green grass

This looks like a better system in some ways. It allows for a local Core device to make decisions for a network of local devices with intermittent connection to the cloud service. If we were to add more automation smarts and sensors on the property this would be the better solution. As we are, in the first instance, only remote controlling the extra complexity is unwarranted.

## MQTT

### Topics

We are going to use abbreviated topic names as longer names can form a significant data usage overhead for periodic update tasks.

We require **fire pump** , **water tank** and **sensor network**.

* fp/
* wt/
* sn/

There are some obvious candidates for second level.

1. Current status topic e.g. ```fp/cs```.
1. Configuration topic e.g. ```wt/cf```.
1. Top level error state e.g. ```sn/es``` (normally false e.g. 0).

## Monitoring

The following lists the sort of things to monitor.

* Tank water level
* Water pressure
* Battery power
* Temp/Humidity
* Thermal (IR/Pyro)
* Presence (bt/wifi)
* ? Visual (low rate mjpeg? on demand?)

## Hardware

### Fire pump

The fire pump is a diesel water pump connected to the main 100,000 liter water tank. I have not decided on whether to fix the throttle and use fuel cutoff as the engine control method or to include a throttle control.

#### Startup

* start ignition
* open fuel solenoid
* open throttle?
* open water solenoid

1. Ignition to on position
1. Fuel to on position
1. Water to on position
1. Throttle to on position
1. Delay 3 seconds
1. Starter engage for 5 seconds
1. Wait for oil pressure 5 seconds

On oil pressure success notify engine running

#### Running

* monitor oil pressure
* monitor water pressue
* monitor tank levels

On 1 minute of water pressure fail readings shutdown?
On 1 minute of failed oil pressure readings try restart?
On water tank empty ..?

#### Shutdown

* close fuel solenoid
* close throttle?
* stop ignition
* monitor oil pressure
* close water solenoid

### Solenoids

* [fuel solenoid](https://www.scintex.com.au/products/3-port-fuel-tank-selection-valve?variant=1270771211&currency=AUD&gclid=CjwKCAiAmNbwBRBOEiwAqcwwpfJ-OWstGg5RwaJ8cr4Gg1HaP5qT8d3JW9wu6q22A98V62j6yuyRLxoCGCgQAvD_BwE)
* [water solenoid](https://www.valvesonline.com.au/stainless-steel-general-purpose-zero-differential)

### Electronics

As I am using a wifi enabled LTE modem I have decided on an ESP32 as the base micro-controller, The relays to switch the 12 volts will be chosen for ease of use. I will either hack a 12V car usb module (5V) or use a low power draw 3.3V switching converter to power the electronics.

* [esp32](https://au.mouser.com/ProductDetail/Espressif-Systems/ESP32-DevKitC?qs=chTDxNqvsyn3pn4VyZwnyQ%3D%3D&vip=1&gclid=Cj0KCQiAgebwBRDnARIsAE3eZjSaMIxOQwbbzKJRoOLgDx2BNb10Zq_RORYd0BP7vRTPr6sH7-kzmIoaAlpFEALw_wcB)

Or

* [pi zero w](https://www.seeedstudio.com/Raspberry-Pi-Zero-W-p-4257.html)

* [relay](https://www.seeedstudio.com/Grove-Relay.html)
* [solar panel](https://www.amazon.com.au/ALLPOWERS-Flexible-connectors-Water-resistant-Applications/dp/B071JQGDXQ/ref=sr_1_2?dchild=1&keywords=flexible+solar+panel&qid=1590980796&sr=8-2)
* [solar regulator](https://github.com/danjulio/MPPT-Solar-Charger)
* [4G modem](https://www.telstra.com.au/internet/mobile-broadband/nighthawk-m2)

The 12V inputs will need to be voltage divided.

## Resources

1. [google mqtt howto](https://cloud.google.com/iot/docs/how-tos/mqtt-bridge)
1. [mqtt data optimization](https://blog.usejournal.com/how-to-optimize-data-usage-over-mqtt-792abebd2cd1)
1. [amazon green grass core](https://docs.aws.amazon.com/greengrass/latest/developerguide/what-is-gg.html)
