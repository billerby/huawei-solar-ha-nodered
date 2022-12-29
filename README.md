# huawei-solar-ha-nodered
Description of my automation

### Prerequesites

* A Home Assistant installation 
* [HACS](https://hacs.xyz/) (to be able to install some custom components)
* [Nordpool ha-sensor](https://github.com/custom-components/nordpool)
* [Huawei solar integration](https://github.com/wlcrs/huawei_solar/) (you have to subscribe to the beta-channel, that is you need at least 1.2.0 beta 3)
* [Node Red](https://community.home-assistant.io/t/home-assistant-community-add-on-node-red/55023)
* The following Node Red extensions:
  - [node-red-contrib-cron-plus](https://flows.nodered.org/node/node-red-contrib-cron-plus)
  - [node-red-contrib-power-saver](powersaver.no/)
  
### Things that should be considered
* Use fixed ip:s for inverter dongle, and Home Assistant

My current flow (work in progress) looks like this:

![image](https://user-images.githubusercontent.com/123237/209923869-7aa084eb-0297-4e92-b55c-c2d44e752257.png)

Explanations:

The flow starts with a cron trigger that is set to fire each day at 14.00 (when we are pretty confident that tomorrows prices have arrived).

Two paths are taken, the first fetches the prices for tomorrow (remember to change to your preferred power region if not SE3)
This part of the flow continues by using the price receiver from the power saver node and then splits into to different paths to get the lowest prices (for charge) and the highest prices (for discharge). Following the lowest price path I have configured it to fetch the 5 hours with the lowest prices between 00:00-07:00 (I have choosen to have a consecutive on-period) since I want to extend the flow with car charging later on, and don't want that to be too complex. 

Following the highest price path, I start with negating all the prices (this is because the power saver for the moment lacks a highest price node). The configuration for the highest price node is configured to get the 14 hours with the highest prices. I also uncheck the consecutive on-period box to be able to construct multiple periods since prices normally drop mid-day.

The flow continues on both paths by removing the schedules received that contains intervals that are already overdue in time. Ater this the upcoming time intervals that should be used are parsed and prepared in the time format expected by the Huawei integation. Here I also add the dayOfWeek to the pattern by using the current dayOfWeek +1 to indicate that this schedule is only for tomorrow. After that the charge/discharge character are appended on both paths.

The first subpaths joins here and the charging periods are merged: We are now here: 
![image](https://user-images.githubusercontent.com/123237/209925671-35bb3af3-c2f1-45e9-b974-f826a554888f.png)

Lets look at the other subpath taken from the starting trigger.

When calling the set_tou_periods service in the inverter, the whole schedule is overwritten. So when fetching at 14.00 and doing all the logic above the schedule of today would be overwritten. Thats why we need this other path.

We start with "Get TOU" by fetching the current schedule from the inverter, we prepare the period data and put it into an existingPeriods array and the joins with the first path that supplies an array "periodsToAdd" on the same format. 

Finally we do some logic where with combine the existing and upcoming periods and then we call the inverter with the new periods to set.


[Grab the code and import it into NodeRed]([Contribution guidelines for this project](tou-node-red.json))

Things to change:

The ```device_id``` of your battery should be change to match your own.
The power region you belong to should be changed in the nordpool integration.





