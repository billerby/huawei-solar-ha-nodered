# huawei-solar-ha-nodered
Description of my automation used to charge the battery during the cheapest hours, and discharge it during the most expensive.

### Prerequesites

* A Home Assistant installation 
* [HACS](https://hacs.xyz/) (to be able to install some custom components)
* [Nordpool ha-sensor](https://github.com/custom-components/nordpool)
* [Huawei solar integration](https://github.com/wlcrs/huawei_solar/) (you have to subscribe to the beta-channel, you need at least the 1.2.0 beta 3)
* [Node Red](https://community.home-assistant.io/t/home-assistant-community-add-on-node-red/55023)

* The following Node Red extensions:
  - [node-red-contrib-cron-plus](https://flows.nodered.org/node/node-red-contrib-cron-plus)
  - [node-red-contrib-power-saver](powersaver.no/)
  
### Things that should be considered
* Use fixed ip for your inverter dongle, we do not want the connection to drop if the dhcp hands out a different ip on restart.
* [Do not configure the lowest price node to start fetching before 00:00](https://github.com/billerby/huawei-solar-ha-nodered/issues/1), the data will not be accepted due to that the hours belongs to different days. I did not write the logic to split this corner case into multiple periods.

My current flow (work in progress) looks like this:
![image](https://user-images.githubusercontent.com/123237/210148616-4127067f-ba22-466e-bc75-9e9681451436.png)


The flow starts with a cron trigger that is set to fire each day at 14.00. It starts with a check if the prices for tomorrow have arrived, if not it waits 10 minutes and checks again (repeats unitl the prices have been updated)

It then fetches the lowest 5 hours (the time it takes for my luna to charge) prices the upcoming 24 hours and the highest prices for 14 hours (its some kind of average discharge time I've choosen). This should be changed to a dynamic config (https://powersaver.no/nodes/dynamic-config.html)

After that the two schedules for charge an discharged are combined into a payload understood by the inverter.


[Grab the code and import it into your NodeRed installation](tou-node-red.json)

Things you need change:

The ```device_id``` of your battery should be changed to match your own.

You can find it by navigating to Developer tools -> Services -> Choose Service "Huawei Solar: Set TOU Periods:
![image](https://user-images.githubusercontent.com/123237/210129402-b0da1b52-19c9-4316-b494-78c07707603b.png)

Then click GO TO YAML MODE and grab your device_id:
![image](https://user-images.githubusercontent.com/123237/210129516-46cfcf7a-1b43-4617-b077-77b235cca94f.png)




The power region you belong to should be changed in the Read Nordpool sensor state node:
![image](https://user-images.githubusercontent.com/123237/210046840-f4a06da5-e5a2-4ab4-8c9f-74958cc3c0ba.png)







