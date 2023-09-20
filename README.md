# equipment-scheduling
This project is for a backend service where various equipment can call and get their current settings. Equipment settings are provided as a json struct containing a number of fields. There is also a schedule that powers the actual settings provided.

## Settings
A client (a piece of equipment) requests its settings using a REST api. The REST api returns the settings as the response. The client can optionally provide a timestamp with the request, and the API shall respond with the settings for that timestamp. A client identifies itself using one of two ID:s, either a GUID string or a serial number (also encoded as a string).

## Schedule
A schedule can have a piece of equipment either "in schedule", or "out of schedule". Each schedule should dictate settings for one specific piece of equipment, or a group of several pieces of equipment. The schedule should have a start date and an end date. In addition to the start- and end dates, there should be a list of exceptions when the schedule should not be active. A schedule should span over the days of the week (Mon-Sun). On each weekday, there should be a number of start and end times describing if the schedule is active or not. A schedule should also have two json structs describing the configuration settings for when the equipment is in respectively out of schedule.

An example schedule data pseudo data model can look like this:
```
{
  "serial": [ "AAAA-01-276-02", "BBBB-01-276-03" ],
  "group_id": "1a56fa...",

  "start_date": Date("2023-01-09"),
  "end_date": Date("2023-07-01"),

  "mon": [ { "start": "08:30", "end": "09:30" }, { "start": "15:30", "end": "17:30" } ],
  "tue": [ { "start": "08:30", "end": "09:30" }, { "start": "15:30", "end": "17:30" } ],
  "wed": [ { "start": "09:30", "end": "10:15" }, { "start": "15:30", "end": "17:30" } ],
  "thu": [ { "start": "08:30", "end": "09:30" }, { "start": "15:30", "end": "17:30" } ],
  "fri": [ { "start": "08:30", "end": "09:30" }, { "start": "14:30", "end": "16:30" } ],
  "sat": [],
  "sun": [],

  "exceptions": [ Date("2023-04-01"), Date("2023-05-01") ],

  "in_schedule_settings": {
    "active": true, 
    "alert": true,
    "lower_limit": 25, 
    "upper_limit": 30 
  },
  "out_of_schedule_settings": { 
    "active": true, 
    "alert": true,
    "lower_limit": 25, 
    "upper_limit": 30 
  }
}
```
When a client requests its settings, the correct schedule is found by first matching the given serial number or group id. After this the active schedule should be retrieved by comparing the data part of the timestamp given by the client. Lastly, the service need to decide whether the equipment is in or out of its schedule by looking at the time part of the given timestamp. Then the service responds with the correct settings depending on if the equipment is in oru out of its schedule.

The service should support hundreds of equipment serial numbers, hundreds of different schedules schedules and tens of different groups.

## Infrastructure
The service shall run as a google cloud function, or alternateively as a google cloud run service. Authentication of clients is done using the authentication mechanisms of google cloud functions or google cloud run. Schedules should be stored in a database backend (mongodb atlas  preferred), or alternatively as json documents in a google cloud storage bucket. It is preferred that infrastructure provisioning is done in code through terraform or similar.

## Extra frontend work
As an extra piece of functionality, there can be a frontend deployed in retool to manage schedules. This should work directly towards the data backend chosen above - without the need for going through the service.
