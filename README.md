# GPS F3X Tracker for Ethos Version 1.8
Note: F3B tasks have not been tested till now!

### Installation guide and user manual

## Contents
1. [General Description](#GeneralDescription)
2. [Requirements](#Requirements)
3. [Known limitations](#Knownlimitations)
4. [Installation](#Installation)
5. [Configuration](#Configuration)
6. [Locations.lua](#Locations.lua)
7. [Event modes](#Eventmodes)
8. [Usage on a slope](#Usageonaslope)
9. [Logging](#Logging)
10. [Flight position correction](#Flight_position_correction)
11. [Management of course length difference](#Management_of_course_length_difference)
12. [Management of course direction difference](#Management_of_course_direction_difference)
13. [Change log](#Changelog)
14. [Development plan](#Developmentplan)
15. [License](#License)

<a name="GeneralDescription"></a>
## 1. General Description
GPS F3X Tracker for Ethos is the LUA application for FrSky transmitters with Ethos operating system. It supports training of slope racing events (F3F). Whole competition event is supported, starting with initial 30 sec countdown sequence, crossing base A line for the first time and counting and time measuring of 10 legs. Via a GPS sensor the turn lines are identified and an acoustic signal is given while crossing them. A supported GPS sensor must be placed in the model and connected and configured to telemetry subsystem.

There are also two modes for F3B available, where in case of speed task the time for 4 legs is measured, in case of distance task number of legs done in allocated time is counted.

The application is based on ideas of Frank Schreiber's F3F Tool for Jeti and on code of Alex Barnitzke's gpsF3XTracker for OpenTx. Its code is published under the MIT License.

This manual describes how to install, configure and use the GPS F3X Tracker.

<a name="Requirements"></a>
## 2. Requirements
- GPS sensor: RCGPS-F3x/RCGPS-F3x-Slim (https://www.cassini.vision/), SM-Modelbau GPS-Logger 3 (https://www.sm-modellbau.de/GPS-Logger-3), FrSky GPS ADV and generally any other GPS with data rate 10Hz are supported. We recommend RCGPS-F3x as it was designed for F3F and fully fits to the GPS F3X Tracker
- Ethos: versions 1.6.2 and newer are supported (previous versions have various issues in areas used by the application). Ethos versions 1.6.4 and 1.7.0 fix some other issues, which are but not critical for operation
- Transmitter: The application supports units with touchscreen resolution 800*480

<a name="Knownlimitations"></a>
## 3. Known limitations
- Due to accuracy of GPS sensors (e.g. FrSky GPS ADV has horizontal accuracy approx 2.5m CEP, RCGPS-F3x 1.5 m CEP with SBAS) and telemetry latency the turn signals are not 100% precise, but still give a good F3F-experience. Issue can be worse for F3B speed tasks
- Sometimes there is a GPS-drift of given start point. In this case the whole course might drift to left or right some meters, because the turn positions are calculated in relation to the start point
- The application supports GPS coordinates with 7 decimals to keep format provided by GPS units. Ethos but currently does not support full editing of such numbers, so editing of coordinates in the locations.lua file must be done in an external editor
- Max 14 fully defined and one "live" event sites are supported
- Application texts and menus are only in English. Speech announcements are given in language configured in the transmitter
- Application is resource demanding. There should not be many other widgets/system tools/tasks/sources running on transmitter otherwise accuracy of application can be compromised. It is valid also vice versa, so other applications can be affected by the GPS F3X Tracker

<a name="Installation"></a>
## 4. Installation
Unzip the installation package gpstrack.X.x.zip, downloaded from the repository (Releases gpsF3XTracker for Ethos), and place all files into directory /SCRIPTS on your transmitter. Folders gpstraca (keeping setup part) and gpstrack (keeping operation part) should not be changed. Please note all modules, excluding locations.lua, are in the compiled form (*.luac).
Start the transmitter and configure two widgets "GPS F3X Tracker Setup" and "GPS F3X Tracker" when a target model is selected. The application is capable to partly modify size of text to size of widget windows, however, for accommodation of all information properly it is recommended to use at least half height & full wide layout for both setup and main widget (in such case widget titles should be switched off), or full height & half wide layout:

<img width="398" height="194" alt="image" src="https://github.com/user-attachments/assets/24d7fa83-a41e-4873-aa1b-7b2aa2b11f38" />

<img width="397" height="171" alt="image" src="https://github.com/user-attachments/assets/92957e40-d2ac-43a9-9771-f433a39dbd48" />

Note: upgrade from a previous program version can be done simply by replacing of all program modules by new ones. It is strongly suggested to delete both widgets first and create them after replacement, checking and setting back all configuration items.

<a name="Configuration"></a>
## 5. Configuration
- GPS sensor: set data rate to 10Hz = 0.1s (or higher if possible without lost of accuracy). Crosscheck names of available GPS sensors and rename, if needed. The application expects sensors coordinates, speed and satellites, if supported, with any of following name:
	- Coord_names = {"GPS", "gps"}
	- Speed_names = {"GPS Speed", "GPS speed", "gps speed", "GSpd", "gspd"}
	- Sats_names = {"GPS Sats", "GPS sats", "gps sats", "GSats", "gsats", "GPS Satellites"}

- For individual telemetry units:
	- RCGPS-F3x: use firmware 0.0.3d or newer. Create a DIY sensor "GPS Sats" with Application ID is 0x5111. Sensors coordinates, speed and satellites are supported
	- SM-Modelbau GPS-Logger 3: be careful – due to error in the Logger firmware v1.31 will Ethos wrongly recognize a sensor with Application ID 0x0860 and with name "GPS Satellites". It is needed to delete such sensor and create a new DIY sensor with Application ID 0x0870! Sensors coordinates, speed and satellites are supported
	- FrSky GPS ADV: Sensors coordinates and speed are supported
	- Other GPS: Sensors coordinates and speed are supported

Note: Speed sensors must be configured in Ethos to give m/s!
Note: if the GPS sensor was bind (discovered) in the Ethos system version 1.6.1 or earlier, it is strongly recommended to delete it and discover again. It should fix various issues affecting Ethos before version 1.6.2
Note: not needed other telemetry values should be disabled in Ethos to speed up the telemetry transfer to transmitter. Also if possible, other telemetry sensors should not be used. A receiver polls 28 potential physical sensor units periodically, in an approximately 12 ms cycles (Physical IDs 0 - 27). One sensor unit can have multiple sensors types (Application IDs). Polling is dynamic, where active sensors are polled more often, without waiting for all 	the inactive ones, improving the data refresh rate. This means, if there is one active sensor it will be polled every 24ms, when there are two active sensors they will be polled every 36ms, etc. Lot of active sensors means problem for delivery of GPS coordinates as polling interval for individual sensor (= individual Physical and Application IDs) can be hundreds of milliseconds

- "GPS F3X Tracker Setup" widget configuration:
	- Event place: any item from list of places in locations.lua file
	- Course direction: course bearing from the left base to the right base in degrees (*)
	- Course difference: change of standard course length (*)
	- Competition type: any type from supported types (f3f_training, f3f_competition, f3b_distance, f3b_speed, f3f_debug) (*)
	- Base A is on left: set  true  if it is so (default status) (**)
 	- GPS sensor: any item from list of supported units
	- Lock GPS Home position switch: any 2-position switch or functional switch, mandatory
 	- Course length difference management: source for real-time change of course length, not mandatory, see chapter 11
 	- Course direction difference management: source for real-time change of course direction, not mandatory, see chapter 12

	(*) These items are available only for "Live Position & Direction event", otherwise are locked as they are determined by event information from locations.lua file

	(**) This item is available only for F3F event types, for F3B event types is Base A always on left

	<img width="393" height="206" alt="image" src="https://github.com/user-attachments/assets/f2bb632f-dedf-4ac2-acd9-b7753b31438d" />
	<img width="392" height="115" alt="image" src="https://github.com/user-attachments/assets/3aa0b767-86bd-4733-b853-805d663209ff" />

- "GPS F3X Tracker" widget configuration:
	- Start race switch: any 2-position switch or functional switch, mandatory
	- Logging: controls logging of event information
 	- Flight correction factor management: Source for setting of the Flight correction factor during flight, not mandatory, see chapter 10
 	- Flight correction factor: defines value for correction of flight position, 0 = no correction
	- Input debug GPS latitude and longitude: used for emulation of GPS input in debug mode  (suggested analog sources elevator and rudder), not mandatory

	<img width="392" height="197" alt="image" src="https://github.com/user-attachments/assets/36149c21-45d8-4919-a040-0509fc077e56" />

<a name="Locations.lua"></a>
## 6. Locations.lua
File Locations.lua, located in "/scripts/gpstrack/gpstrack" folder, keeps event location items in the format "Name of event site, home latitude, home longitude, course direction, course length difference, event type". Event types are:
- type 1: f3f training 
- type 2: f3f competition
- type 3: f3b distance
- type 4: f3b speed
- type 5: f3f debug

Default Location.lua looks like below. You can edit it as per your needs, but please do not delete the first row, which defines "live" event location. Default event type in such case is f3f training (type 1), however you can change it during configuration. Home position for “live” event location is defined from current GPS information. Please do not remove the last entry either (create new sites before that item) and keep its name "Last Entry". Max 15 event sites are supported:

    {name = "Live Position & Direction", lat = 0.0, lon = 0.0, dir = 0.0, dif = 0, comp = 1},
    {name = "Debug", lat = 53.550707, lon = 9.923472,dir = 9.0, dif = 20, comp = 5},
    {name = "Loechle", lat = 47.701974, lon = 8.3558498, dir = 152.0, dif = 10, comp = 2},
    {name = "F3B Distance site", lat = 53.333333, lon = 51.987654, dir = 19.9, dif = 0, comp = 3},
    {name = "F3B Speed site", lat = 53.555555, lon = 51.987654, dir = 10.9, dif = 0, comp = 4},
    {name = "Test site", lat = 31.212000, lon = 121.400000, dir =   0.1, dif = 1, comp = 1},
    {name = "Last Entry", lat = 0.0, lon = 0.0, dir = 0.0, dif = 0, comp = 1}

Notes:
- home latitude and longitude is for F3F events position of a center of the course
- home latitude and longitude is for F3B events position of a baseline A of the course
- course length of competition event types is defined as per F3X rules - F3F 100m and F3B 150m. F3F debug has its course length set to 30m. You can change this default course length for a particular event site via item "dif" if needed - course is longer when value is positive and course is shorter when value is negative. Difference is evenly split to both side of the course, that means for example difference in value of -1 shortens both left and right side of the course by 0.5 m

You can edit the file on a PC or via embedded editor:

<img width="393" height="206" alt="image" src="https://github.com/user-attachments/assets/ba7d9e94-d38b-4366-9b39-10c3e90868fd" />

Use button “Edit event place” at the bottom of the site configuration screen to enter the editor. The original screen will be replaced by a new screen allowing editing of all parameters:

<img width="396" alt="image" src="https://github.com/user-attachments/assets/c9244f6e-66f6-4ce4-acc4-27c22fc4f2e3" />

Button “Save” saves modified site as below:
    1. Creation of a new site from the “Live Position & Direction” event place – if list of sites is not full, new line with provided information will be created in the Locations table, just before the "Last Entry" line. Do not forget to change name of new event place. If list of sites is full, save operation will be refused and indicated by message "List of sites is full!"
    2. Editing of an existing site – changed parameter(s) will be written into relevant site line in the Locations table 

Editor can be closed by standard “back” widget Ethos button

Note: deletion of site lines in the Locations table and edition of parameters for “Live Position & Direction” event place is possible only via PC

<a name="Eventmodes"></a>
## 7. Event modes
The application supports F3F-competition, F3F-training, F3F-debug, F3B-speed and F3B-distance event types. They behave differently as below:
- F3F-competition: it follows F3F rules, so it begins with 30 sec timer and starts the run timer when the plane enters the event course from outside via base A toward base B for the fist time or when the initial timer expires. It then measures time for 10 laps between bases. Start in this mode is expected to be inside of the course, first crossing of a base A from inside the course to outside is indicated by beep
- F3F-training: it does not use the initial 30 sec timer and begins by entering the event course from outside via base A toward base B for the fist time. The event in this mode can be started when you are inside or outside of the course. When the event starts from inside, first crossing of base A from inside the course to outside is indicated by beep
- F3F-debug: similar to F3F-training, however it uses emulation of GPS input (via configured sources "Input debug GPS latitude and longitude") for simulation of flight around the set home position
- F3B-speed: at this moment only measures time for 4 laps since entering the competition place from outside via base A toward base B for the fist time. No check for the overall competition time is implemented
- F3B-distance: it measures number of laps made in 4 minutes since entering the competition place from outside via base A toward base B for the fist time 

The actual status is indicated by individual rows in the "GPS F3X Tracker" widget screen:
- Comp factor:  defines value for correction of flight position, 0 = no correction
- Comp: "waiting for start...", "started..." - just after switching the "Start race switch" on, "canceled..." - cancellation can be done by switching off/on/off of the "Start race switch", "start climbing..." - during initial event phase (so 30 sec max), "out of course" - plane between bases, "race timer started..." - initial 30 sec expired and plane between bases, "in course..." - plane outside of bases, "timer started..." - initial 30 sec expired and plane  outside of bases
- Runtime: time used for individual event
- Course: "center", "leftOutside", "leftInside", "rightOutside", "rightInside" - distance from the center is provided
- Spd, Dst: speed, distance from the center
- GPS: actual GPS position
- Runs: list of last events of the same type with their runtime

	<img width="194" alt="image" src="https://github.com/user-attachments/assets/3776fb41-35f4-4898-bc3e-7f8eeed1ecbd" />


Announcements and sounds: 
- Beep after switching the "Start race switch" on (600 Hz)
- Initial F3F timer countdown announcements: 30, 25, 20, 15, 10, 9, 8, 7, 6, 5, 4, 3, 2, 1, 0 sec
- F3B-distance timer countdown announcements: minutes and every 10 sec for last minute
- Beeps when crossing base, tone based on condition:
	- first crossing of base A from inside the course to outside in the competition mode (800 Hz)
	- first crossing of base A from inside the course to outside in the training mode (800 Hz), when an event starts inside of a course
	- first crossing of base A from outside the course to inside in the competition and training mode (1000 Hz)
	- any further crossing of bases from inside to outside in the competition and training mode (1200 Hz)
- Lap time announcements on even laps for F3F-traning event type
- Overall runtime at event end for F3F-x event types

<a name="Usageonaslope"></a>
## 8. Usage on a slope
- Switch on RC system and give the GPS sensor enough time for initiation and satellite detection - it can take 60+ seconds! For example for GPS-Logger 3 please wait till the orange LED is off and the green LED glows permanently or flashes. Open “GPS F3X Tracker Setup” widget and select “Live Position & Direction” event place or any pre-configured place.
- For F3F and "Live Position & Direction event place":
	- Set "Competition type" and "Base A is on left" configuration items
	- Go with your model to the center of the course
	- Wait for stable information in the "GPS" row in the "GPS F3X Tracker Setup" widget screen
 	- Take the cardinal direction from the left base perpendicular to the right base and set it to the "Course direction" item
	- Lock the position with the "Lock GPS Home position switch" - such status will be indicated by change of item name to "GPS Home lck" 
	- Now your flight configuration is ready, you do not start from the exact home place

- For F3B and "Live Position & Direction event place":
	- Set "Competition type" configuration item
	- Go with your model to the baseline A of the course
	- Wait for stable information in the "GPS" row in the "GPS F3X Tracker Setup" widget screen
	- Take the cardinal direction from baseline A perpendicular to baseline B and set it to the "Course direction" item
	- Lock the position with the "Lock GPS Home position switch" - such status will be indicated by change of item name to "GPS Home lck" 
	- Now your flight configuration is ready

- For other F3F/F3B event places (pre-configured in the Location.lua file):
	-Your flight configuration is ready - all parameters, excluding "Base A is on left" are set in the Location.lua file

- Go to the "GPS F3X Tracker" widget screen
	- The initial status is indicated by statement "waiting for start..." 

		<img width="194" alt="image" src="https://github.com/user-attachments/assets/b536919a-1589-4a6b-ac70-20bf087a35c7" />

- Start new event with the "Start race switch"

<a name="Logging"></a>
## 9. Logging
Logging of flight event information is supported. It should be used only if really necessary as it can affect performance of the application. Logging is disabled by default and has to be enabled in the "GPS F3X Tracker" widget configuration. Flight events are written into log files “YYYYMMDD-Log” located in the folder /scripts/gpstrack/gpstrack.

Recording begins when a flight is started by the “Start race switch” and ends when an event is concluded. An initial log row looks e.g. like:

_Start 16:06 Course direction:12.5°, Course difference:0m, luaRamAvailable:1758840B, Correction factor:0.0_

Next rows have format as below, where:

_comp.state, GPS:  latitude, longitude, Dist2home, Dir2home, Course distance, Speed, Sats_

- comp.state: provides information about current flight stage (5 – start overall timer, 10 – waiting for leaving the course, 13 – waiting for leaving the course for training events, 15 – waiting for entering the course from outside (training mode starts here), 20 – start competition timer, 25 – waiting for plane crossing right base from inside, 27 – waiting for plane crossing left base from inside, 30 – end of event)
- GPS: gives current position of the plane
- Dist2home: gives current distance between the plane and home point (center of the course) in meters (negative value means the plane is on the left from the home point, positive value means the plane is on the right)
- Dir2home: gives current angle between the plane and home point in rads
- Course distance: gives current distance between the plane and home point in meters, projected into the event geometry. It gives information about position of the plane in relation to bases A and B
- Speed: speed of flight in m/s
- Sats: number of visible and used GPS satellites (available only for sensors providing that information)

<a name="Flight_position_correction"></a>
## 10. Flight position correction
Due to latency of GPS sensor, telemetry latency and processing latency the turn signals are not 100% precise. The application has implemented basic mechanism aiming to compensate this issue.  The function is controlled via value of item “Flight correction factor” in the "GPS F3X Tracker" widget configuration:
- 0: correction is disabled
- 0.01 – 0.50: correction is enabled

Corrected position depends on a current fight speed and it is calculated as per formula below:
CorrectedDistance =  ReportedDistance + FlightcorrectionFactor * groundspeed

The best value of the Flight correction factor must be found as it depends on hardware and software conditions. For example the Flight correction factor = 0.1, compensates the overall position inaccuracy related to one delayed report from a GPS sensor set to 10 Hz, so ideally reporting each 0.1s. In a case of speed 100 km/h, compensation is 2.8m, with speed 150 km/s it is 4.2m. The Flight correction factor = 0.5 compensates 5 delayed reports and compensation is 14m respectively 21m for the same speeds.

You can set the Flight correction factor manually or you can set its best value during test flights:

- There is new configuration item of type Source "Flight correction factor management". If it isn’t configured, it is possible to change the configuration item "Flight correction factor" manually, otherwise the item "Flight correction factor" is managed by the configured source
- Value of the item "Flight correction factor management" must be in range 0-50%, I advise to create a Var, e.g.:

	<img width="393" alt="image" src="https://github.com/user-attachments/assets/5b9366b1-de47-44a1-8c0d-a8e72fdd5364" />

- Change of value is in the Tracker accepted only during status "waiting for start...". However value of the source can be changed anytime and the Tracker will accept it when goes again into the status "waiting for start..."!
- Change of value is announced by voice, be patient as Ethos chains voice messages
- Last value is stored in the item "Flight correction factor", you can see it in the operation widget
- After landing configure the item "Flight correction factor management" as empty – in this way you will fix the found best value of the correction factor.   It is expected to use this function only when a new model is configured or its hardware (sensors, receiver) has changed, so the "Flight correction factor management" configuration item is not mandatory and should stay generally empty to avoid unexpected change of the correction factor!

<a name="Management_of_course_length_difference"></a>
## 11. Management of course length difference
Standard course length for individual event can be permanently changed via configuration and reflected in the related event location item in the Locations.lua. For the “Live Position & Direction” event place there is possibility to change its course length in the range <-10, +10> meters during flight without landing.

For the feature to run you need to set a source for the “Course difference management” configuration item. It is suggested to use e.g. a free trim configured with Easy mode and Fine step – in such case each trim move will change the course by one meter in positive or negative manner. A Var with range -20%-20% with any suitable source is also good and universal solution.

Change of the course is possible only when the position is locked with the “Lock GPS Home position switch” – change is confirmed by a voice announcement and indicated on the “Course Difference” row of the “GPS F3X Tracker Setup” widget.

Notes:
- course length made by this feature is not considered as permanent and it isn’t recorded. If the change in the course length should be set permanently, configure it after landing in the Locations.lua
- if the “Course length difference management” source isn’t zero/neutral at a moment of locking with the “Lock GPS Home position switch”, its value is considered and reflected as course length difference
- course is longer when value is positive and course is shorter when value is negative. Difference is evenly split to both sides of the course, that means for example difference in value of -1 shortens both left and right side of the course by 0.5 m
- if the “Course length difference management” source changes its value after start of a “Live Position & Direction” event, the event is canceled and it goes into "waiting for start…" status
- it is expected management of course length will be used only rarely for previously unknown sites or for testing, so the “Course length difference management” configuration item is not mandatory and should stay generally empty to avoid unexpected change of the course length!

<a name="Management_of_course_direction_difference"></a>
## 12. Management of course direction difference
Accuracy of setting the course direction is crucial for correlation between actual position and the course frame (vertical planes at base A and B). Any mistake in setting of the course direction means wrong indication in crossing of bases. Depending on how far you fly in front from the center point of the course, inaccuracy in indication can be couple of meters even for relative small error in course direction (error 5° means 0.4m inaccuracy 5 meters from the center point, but 2.6m when you fly 30 meters from it). Wrong course direction can be identified by a permanent “move” of both bases to the left or right. When crossing of both bases is reported to the right of bases, try to decrease the course direction and vice versa.

Standard course direction for individual event can be permanently changed via configuration and reflected in the related event location item in the Locations.lua. For the “Live Position & Direction” event place there is possibility to change its course direction in the range <-10, +10> degrees during flight without landing.

For the feature to run you need to set a source for the “Course direction difference management” configuration item. It is suggested to use e.g. a free trim configured with Easy mode and Fine step – in such case each trim move will change the direction by one degree in positive or negative manner.  A Var with range -20%-20% with any suitable source is also very good and universal solution.

Change of the direction is possible only when the position is locked with the “Lock GPS Home position switch” – change is confirmed by a voice announcement and indicated on the “Course Direction” row of the “GPS F3X Tracker Setup” widget.

How to use this function: Fly near the edge of the slope, observe where the detection occurs, and then fly in the area far from the edge. Observe the difference and try to correct if necessary, until you feel you have found the correct course direction.

Notes:
- do not use this function without tuning of the Flight correction factor first! Otherwise you can face inaccurate position information!
- course direction made by this feature is not considered as permanent and it isn’t recorded. If the change in the course direction should be set permanently, configure it after landing in the Locations.lua
- if the source configured for the “Course direction difference management” item isn’t zero/neutral at a moment of locking with the “Lock GPS Home position switch”, its value is considered and reflected as course direction difference
- if the “Course direction difference management” source changes its value after start of a “Live Position & Direction” event, the event is canceled and it goes into "waiting for start…" status
- it is expected management of course direction will be used only rarely for previously unknown sites or for testing, so the “Course direction difference management” configuration item is not mandatory and should stay generally empty to avoid unexpected change of the course direction!

<a name="Changelog"></a>
## 13. Change log
V1.1:
- List of locations in the file Locations.lua has been enhanced by item "dif", which modifies default course length (F3F 100m, F3B 150m), positive number: course is longer, negative number: course is shorter
- Number of visible GPS satellites is available on “GPS F3X Tracker Setup” widget screen (only for SM-Modelbau GPS-Logger 3)
- Error in timestamp function has been fixed

V1.2: improved management of fonts

V1.3: created function for editing location table in the locations.lua

V1.4:
- Implemented logging of flight information into a local file 
- Function for flight position correction enabled
- Implemented management of course length difference during flight for the “Live Position & Direction” event
- Added support for RCGPS-F3x sensor
- List of sensors reviewed leaving only those needed for GPS F3X Tracker (Speed, GPS coordinates and Satellites)
- Deleted function for estimation of acceleration in axis Z for sensors without accelerometer 
- Deleted reading from an altitude sensor and its displaying

V1.5: Implemented management of the flight correction factor during flight

V1.6:
- Enhancement assuring readability of widget texts when the dark mode is disabled 
- Change the initial F3F timer countdown announcements to 30, 25, 20, 15, 10, 9, 8, 7, 6, 5, 4, 3, 2, 1, 0 sec
- Implemented management of course direction difference during flight for the “Live Position & Direction” event

V1.7:
- Implemented tone indication (800 Hz) for first crossing of base A from inside to outside the course in the training mode, when an event started inside of the course
- Direction in the Locations.lua for "Live Position & Direction" set to 0.0

V1.8: Change in identification of telemetry sensors with aim to mitigate issues with their names

<a name="Developmentplan"></a>
## 14. Development plan
It is probable there will be necessary to change or enhance some parts. Do not hesitate to comment and come with ideas, preferably via an Issue in the GitHub repository (New issue gpsF3XTracker for Ethos)

<a name="License"></a>
## 15. License
Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

Copyright © 2025 Milan Repik

