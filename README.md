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
![image](https://user-images.githubusercontent.com/123237/210081422-c533bd12-9802-4efb-a9c2-b77867691a1e.png)


The flow starts with a cron trigger that is set to fire each day at 14.00. It starts with a check if the prices for tomorrow have arrived, if not it waits 10 minutes and checks again (repeats unitl the prices have been updated)

It then fetches the lowest 5 hours (the time it takes for my luna to charge) prices the upcoming 24 hours and the highest prices for 14 hours (its some kind of average discharge time I've choosen). This should be changed to a dynamic config (https://powersaver.no/nodes/dynamic-config.html)

After that the two schedules for charge an discharged are combined into a payload understood by the inverter.


[Grab the code and import it into your NodeRed installation](tou-node-red.json)

Things you need change:

The ```device_id``` of your battery should be changed to match your own.

The power region you belong to should be changed in the Read Nordpool sensor state node:
![image](https://user-images.githubusercontent.com/123237/210046840-f4a06da5-e5a2-4ab4-8c9f-74958cc3c0ba.png)







