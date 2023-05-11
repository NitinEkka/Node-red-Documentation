FUNCTIONS
1: Updating Last active time stamp
2: Device Came online Alerts
3: Threshold Check for devices
4: Reset Alert
5: Ping Custom Devices
6: Get APIs
7: Farm name from ENV
8: UI Builder for Global Variables
9: Global Variables
10: Filter Devices GET API.
11: Switch ON/OFF Exhaust Devices
12: Feedback from Exhuast Devices

FUNCTION 1: Updating Last active time stamp
Key Points: 
Global Variable “devices” hold all devices value. These values can be fetched or altered by /devices API (“GET”,” POST”).
Global Variable “lastActiveTH” holds all the device’s last active timestamp and alert status code, this code changes whenever an alert is triggered.
PROCESS:
‘UPDATE DATA SENSOR’ node is being used for this. The node takes device’s ‘s_k’ from the mqtt topic. This node is subscribed to ‘+/TH’ topic. This node is filtering for all the thsensors and changing the timestamp and alertStatus field. alertStatus = 1 means, that device has received an alert from the threshold check node that it has not send any data for x mins. If alertStatus = 0, It means the device has not received and alerts. When ever this node detects any payload, it updates the device’s timestamp to current time and update the status code to 0.


FUNCTION 2: Device Came online Alerts

Key Points: 
‘UPDATE DATA SENSOR’ node is being used for this.
HTTP request node has been used to send Django alerts to mobile app.
Active thsensors will send payload on +/TH topic. “+” placeholder for device’s s_k. 
PROCESS:
This alert is triggered when the node receives any payload with topic like (th_110/TH) and if the alertStatus of this device (th_110) is 1. Alert Status 1 means that device had not send any data for x mins and was inactive.

FUNCTION 3: Threshold Check for devices (thSensors, irrigationDevices, exhaustDevices)

Key Points: 
“Detect No Data Sensor for THSensor” and “Detect No Data for Irrigation” these two nodes are being used.
There are 3 Global Variables which holds the threshold time value. ‘nothdataTime’ for thSensors, ‘noirrigationdataTime’ for irrigationDevices and ‘noexhaustdataTime’ for exhaustDevices.
Nodes are triggered every second to check if the no data time exceeds the threshold time.
Alerts are also sent based on the alertStatus in lastActiveTH array.

PROCESS:

Node checks the lastActiveTH array for active devices whose last active timestamp has exceed the device’s threshold. And if the current alertStatus of device in lastActiveTH is 0, It will send an alert to mobile app that the device has not sent any data for x mins.

FUNCTION 4: Reset Alert

Key Points:

Reset 3 Global Variables 'alertArray', 'lastActiveTimestamp', 'lastActiveTH'.

resetAlert node is being used.

PROCESS:

This node gets triggered everyday at 6 AM. And sends an "Info" level alert to mobile app with category "ALERT_RESET" and message "Previous alerts have been reset".


FUNCTION 5: Ping Custom Devices

Key Points:

Global variable 'count' and 'ping' are being used. 'count' works as a counter for variable ping interval. 'ping' holds the value of the ping interval, frequency for sending ping command to all custom devices.

Ping group is being used.

PROCESS:

Inject node is sending timestamp at every 1 second and every second the counter is incremented. When the counter reaches the ping value, 'ping -w 3 -c 1 {IP}' command is executed with an exec node and the counter is reset. Depending on the code given as a response from the exec node 'return code terminal' the alerts are being send to mobile app and the code attribute of custom devices are changed from 1 to 0, alerts are of 2 types came online and went offline. While intitiating a custom device , the starting value of code will always be one. 

FUNCTION 6: Get APIs

PROCESS:

There are 7 GET api's used for keeping track of all the Global Variables.
1: /device-alerts, keeps track of custom devices values like devicename, ip and code.
2: /get-threshold, keeps track of threshold time variables for different types of devices. 
3: /get-farmname, keeps track of the farmname which is being fetched from ENV.
4: /get-ping, keeps track of the ping interval time used for pinging custom devices.
5: /no-cycle, keeps track of alertArray global variable. To track for process cycle.
6: /no-data, keeps track of lastactivetimestamp.
7: /th-no-data, keeps track of the lastactive timestamp of thsensors and their alert status.
8: /get-mode, keeps track of the current mode.
9: /get-interval_On, keeps track of the switch ON interval of exhaust devices.
10: /get-interval_Off, keeps track of the switch OFF interval of exhaust devices.

FUNCTION 7: Farm name from ENV

Key Points:

Farm names will be saved on the server as environment variable by the name , FARM_ID.

Exec node can be used to check all the available Environment variables by "env" command.

PROCESS:

Environment variables are fetched using "env.get("FARM_ID")" command and are stored in "farmname" global variable.

FUNCTION 8: UI Builder for Global Variables

Key Points:

Configurable Variables are being set here.

PROCESS:

Variables such as threshold time, switch On/Off and custom devices are being configured from UI Builder.

FUNCTION 9: Global Variables

Key Points:

LIST OF GLOBAL VARIABLES:
1-devicesAlert: Keeps track of all custom devices and their status.
2-farmname: Holds the current farm name.
3-nothdataTime: Holds threshold alert time for thsensors.
4-noirrigationdataTime: Holds threshold alert time for irrigation devices.
5-noexhaustdataTime: Holds threshold alert time for exhaust devices.
6-ping: Holds Interval for pinging all custom devices.
7-mode: Holds the mode type for switching ON/OFF exhaust devices.
8-intervalon_Off: Holds delay time for switching ON exhaust devices.
9-interval_Off: Holds delay time for switching OFF exhaust devices.
10-exhaustAlert: Holds array of objects of current active exhaust devices.

FUNCTION 10: Filter Devices GET API.

Key Points:

Filter API for getting all devices.

PROCESS:

This API is being used for switching ON/OFF specific devices with specific modes. Currently there are 4 modes Even, Odd, Alternate Even, Alternate Odd. These are the designed wrt dome numbers. 1 Polyhouse has 14 domes and each dome have 1 irrigation device, 2 exhaust device and 1 temperature sensor. Both the exhaust fans are on different channels. Channels are used for balancing between similar kind of devices soo that the load on a particular device doesn't exceed.For Example, Even mode means [2,4,6,8,10,12,14] domes will get activated, currently only channel 1.


FUNCTION 11: Switch ON/OFF Exhaust Devices.

Key Points:

First Devices are turned off with respective switch OFF interval.

After switching OFF all the active devices, Switching ON devices will intitiate with respect to specific Switch ON interval time.

PROCESS:

For switching ON/OFF , "dd_001/+/switchOn" topic is being used with specific deviceID. This flows is triggered with mqtt topic "/switch" with payload {"polyhouse": "14_A", "deviceType": 3, "channel": 1}. Switch ON/OFF will depend upon the mode. The function will switch on only respective exhaust fans of specific dome. For handling error , undefined objects are ignored before sending message/topic.


FUNCTION 12: Feedback from Exhuast Devices.

Key Points:

Every device which is active will send different status code on different topics. Currently "dd_001/+/CA" and "dd_001/+/mode" topics are monitored.

For reference refer https://docs.google.com/spreadsheets/d/1FaLHGDIyuq77uaQQbnFck9CxWny9Zjq-ytIcU0K9qxQ/edit#gid=1045111749

PROCESS:

2 flows are monitoring these 2 topics. In 'dd_001/+/CA' 'err' field is being monitored, It can be either 0,2 (0="normal working", 2="outside configured current range"). In "dd_001/+/mode" 'st' field is being monitored, It can either be 0,1 (1="ON", 0="OFF"). Specific Django alerts are triggered when there is a transition in states.










