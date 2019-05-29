# Contents

* NotificationAdderListener.zip
  * Zipped directory with needed Node scripts + required modules
* serviceStatus.csv
  * Example output

# Details:

This NodeJS script does the following:

1. Creates a Notification handle for any changes to the `ServiceStatus` Qlik Sense Repository entity filtered on only the Engine Service. `POST /qrs/notification?name=ServiceStatus&filter=serviceType eq 3&changeType=2&PropertyName=timestamp`. For more details on the QRS Notification API refer to [this](https://community.qlik.com/t5/Qlik-Architecture-Deep-Dive-Blog/Qlik-Sense-Repository-Notification-API/ba-p/1582905) blog post.
2. Creates a listener to receive the push notification configured in step 1.
3. When it receives a notification, it does a `GET` operation on the specific `ServiceStatus` entity which belongs to an Engine Service who's `timestamp` value has changed. This can be due to the service restarting or due to the Qlik Sense Repository Service being unable to communicate with the Node's Engine Service.
4. Write the time at which the Engine went offline to `C:\Temp\serviceStatus.csv`

# Notes:

* This configuration will *not* persist after upgrading Qlik Sense either by a full upgrade or patch. The `services.conf` file is expected to be reverted on any upgrade or patch operation.

# Config steps for Script:

* Unzip the directory into the install path for Qlik Sense (by default `C:\Program Files\Qlik\Sense`) such that there is a directory named `NotificationAdderListener` with a file inside called `server.js` 
  * **Note**: While this can be installed to any node, installing this to the Central node is encouraged.
  * Example: `C:\Program Files\Qlik\Sense\NotificationAdderListener\server.js`
* Ensure that script runs manually:
  * Open up a command prompt with administrative rights
  * Navigate to the installation path (e.g. `cd C:\"Program Files"\Qlik\Sense\`)
  * Change the directory to the `ServiceDispatcher` service (e.g. `cd ServiceDispatcher`)
  * Launch the NodeJS script manually: `Node\node.exe ..\NotificationAdderListener\server.js`
  * Observe that it launches
  * Wait 15 seconds
  * Restart an Engine Service
    * Observe the output :
![Screenshot of a notification being registered](https://i.imgur.com/m6huS1r.png)
    * Observe that `serviceStatus.csv` is created in `C:\Temp`. Example format:
```
notificationTime,hostname,serviceStatusTimestamp,serviceState
2019-05-29T17:46:49.685Z,qliksenserim02,2019-05-29T17:48:11.368Z,NoCommunication
```
* Configure script to initialize with the Qlik Sense Service Dispatcher service:
  * Launch Notepad or similar text editor with administrator rights
  * Open the `services.conf` file in the install path for Qlik Sense\ServiceDispatcher (e.g. `C:\Program Files\Qlik\Sense\ServiceDispatcher\services.conf`)
  * At the very end, append with:
```
[notification]
Identity=qrs-notify
DisplayName=Notification Listener
ExePath=Node\node.exe
Script=..\NotificationAdderListener\server.js
```
  * Restart the Qlik Sense Service Dispatcher service
  * Wait 15 seconds
  * Restart an Engine Service
    * Observe the output :
![Screenshot of a notification being registered](https://i.imgur.com/m6huS1r.png)
    * Observe that `serviceStatus.csv` is created in `C:\Temp`. Example format:
```
notificationTime,hostname,serviceStatusTimestamp,serviceState
2019-05-29T17:46:49.685Z,qliksenserim02,2019-05-29T17:48:11.368Z,NoCommunication
```
