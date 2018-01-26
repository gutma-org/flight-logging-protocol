# UAV/Operator Flight logging Exchange Protocol

_A protocol designed to describe a logged flight from an UAV/Drone

**LICENSE**

 Licensed under the Apache License, Version 2.0 (the "License");
 you may not use this file except in compliance with the License.
 
 You may obtain a copy of the License at: http://www.apache.org/licenses/LICENSE-2.0

 Unless required by applicable law or agreed to in writing, software
 distributed under the License is distributed on an "AS IS" BASIS,
 WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 See the License for the specific language governing permissions and
 limitations under the License.
  

##Data types
 
### Dates & Times

Dates and times will follow the ISO-8601 formatting standard. Local times are not supported; all times must be in UTC or have a time zone offset specified.

No support is made for just a date or just a time. For comparison of time ranges, the start time is regarded as included in the time range, where the end time is excluded.

The format pattern for date times is YYYY-MM-DDTHH:mm:ss.sssZ where Z is either the character Z to represent UTC, or the +/- timezone offset from UTC. If a message is received with no timezone offset, it should be regarded as an error and the correct error code returned.

The requirement to specify times or dates does not apply to the specification, so all temporal data will be defined as a datetime.

No guarantees that a timezone offset will be preserved should be made.


### Geospatial Data

Geospatial data must be described using a geometry object as defined in the GeoJSON specification with the following specific requirements

Using the default CRS - geographic coordinate reference system, using the WGS84 datum, and with longitude and latitude units of decimal degrees

Bounding box is not expected

Latitude and Longitude should not be specified to more than 8 decimal places. This gives an accuracy of approximately 1.1 mm at the equator making any further digits superfluous.

### Distances

All distances (both horizontal and vertical) are specified in metres.

### Speed

All speeds are specified in meters/seconds

## Message structure

The message is divided into three main parts: Description of devices used (optional),  logging data (mandatory) and informations about the file itself (optional).

Here is a short message example:

    {
        "exchange": {
            "exchange_type": "flight_logging",
            "message": {
                "flight_data": {
                    "aircraft": {
                        "firmware_version": "2.01b",
                        "hardware_version": "1.00B",
                        "manufacturer": "senseFly",
                        "model": "eBee",
                        "name": "John doe Drone",
                        "serial_number": "EB-99-01807"
                    },
                    "gcs": {
                        "manufacturer": "senseFly",
                        "model": "eMotion",
                        "version": "1.1"
                    },
                    "payload": [
                        {
                            "firmware_version": "1.23",
                            "hardware_version": "0",
                            "model": "WX RGB",
                            "serial_number": "2352342141"
                        }
                    ],
                    "project": "Projet test"
                },
                "flight_logging": {
                    "flight_logging_items": [
                        [ 0.5, 6.5431337999999997, 46.687659199999999, 100, 0, 0, 0, 0 ],
                        [ 1, 6.5429424000000003, 46.6879116, 110, 2, 0, 0, 0 ],
                        [ 1.5, 6.5429424000000003, 46.6879116, 100, 0, 0, 0, 0 ]
                    ],
                    "flight_logging_keys": [
                        "timestamp", "gps_lon", "gps_lat", "gps_altitude", "speed", "speed_vx", "speed_vy", "battery_voltage"
                    ],
                    "event": [
                        {
                            "event_type": "CONTROLER_EVENT",
                            "event_info": "TAKE_OFF",
                            "event_timestamp": "0.5"
                        }
                    ],
                    "altitude_system": "WGS84",
                    "logging_start_dtg": "2017-05-16T13:19:25.250Z",
                    "tick_increment_modulo": "1",
                    "tick_increment_seconds": "1",
                    "uom_system": "Metric"
                },
    
                "file":  {
                    "logging_type": "GUTMA_DX_JSON",
                    "filename": "EB-99-01807_0069",
                    "creation_dtg": "2017-05-23T08:38:41.306Z",
                    "version": "1.0.0"
                },
                "message_type": "flight_logging_submission"
            }
        }
    }


### flight_data section

    "flight_data": {
    	"aircraft": {
    		"firmware_version": "2.01b",
    		"hardware_version": "1.00B",
    		"manufacturer": "senseFly",
    		"model": "eBee",
    		"name": "John doe Drone",
    		"serial_number": "EB-99-01807"
    	},
    	"gcs": {
    		"manufacturer": "senseFly",
    		"model": "eMotion",
    		"version": "1.1"
    	},
    	"payload": [
    		{
    			"firmware_version": "1.23",
    			"hardware_version": "0",
    			"model": "WX RGB",
    			"serial_number": "2352342141"
    		}
    	],
    	"project": "Projet test"
    }
    
This section  contains informations about hardware devices used during the flight.
Aircraft section contains fields which a describing the aircraft used during the flight. An aircraft is uniquely identified with his manufacturer/model and serial number.
gcs describes ground control station.
Payload is used to declare what devices are embedded during the flight. There can be several payloads like gimbal, camera etc...
Project is used to "tag" the flight or indicate if this flight is part of project.

### flight_logging section

    "flight_logging": {
    	"flight_logging_items": [
    		[ 0.5, 6.5431337999999997, 46.687659199999999, 100, 0, 0, 0, 0 ],
    		[ 1, 6.5429424000000003, 46.6879116, 110, 2, 0, 0, 0 ],
    		[ 1.5, 6.5429424000000003, 46.6879116, 100, 0, 0, 0, 0 ]
    	],
    	"flight_logging_keys": [
    		"timestamp", "gps_lon", "gps_lat", "gps_altitude", "speed", "speed_vx", "speed_vy", "battery_voltage"
    	],
    	"event": [
    		{
    			"event_type": "CONTROLER_EVENT",
    			"event_info": "TAKE_OFF",
    			"event_timestamp": "0.5"
    		}
    	],
    	"altitude_system": "WGS84",
    	"logging_start_dtg": "2017-05-16T13:19:25.250Z"
    }

This is the main part of the file. The format is self describing i.e. it contains the types that are logged. For the moment, mandatory fields are timestamp, gps\_lon, gps\_lat, gps\_altitude. speed and battery\_voltage are  also taken into account, but they are optional.
Many types can be added, it will simply not be analysed, just stored.

* **Timestamp** : number of the seconds elapsed since logging_start_dtg. It is a float, with max 3 decimals (so precision is milliseconds). By extension, the last timestamp will be equal to the duration flight in seconds.
* **gps\_lon, gps\_lat, gps\_altitude**: GPS coordinates
*  **speed**: ground speed in m/s (float)
* **battery\_voltage**: voltage in volt (float)
* **logging\_start\_dtg**: describes the beginning of the flight. It is mandatory. 
* **altitude\_system**: indicates the type of altitude reported: "AGL", "MSL" or "WGS84".

Event is used for notify events during the flight. It can be take off, gps lost, obstacle detection etc..

### file section

    "file":  {
    	"logging_type": "GUTMA_DX_JSON",
    	"filename": "EB-99-01807_0069",
    	"creation_dtg": "2017-05-23T08:38:41.306Z"
    }
            
This section contains informations about the file itself. All fields are optional.

* **logging_type**: type of logging
* **filename**: filename, can be with extension
* **creation_dtg**: file date creation
* **version**: protocol version