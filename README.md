# Nokē Core API Documentation

## Overview

The Nokē Core API is a quick and simple way to integrate Nokē products with your existing software systems. This documentation covers the initial closed beta release. 

* [Locks](#locks)
  * [POST /lock/](#post-lock)
  * [POST /unlock/](#post-unlock)*
  * [POST /unshackle/](#post-unshackle)
  * [POST /keys/](#post-keys)
  * [Quick Clicks](#quick-clicks)
    * [POST /qc/issue/](#post-qcissue)
    * [POST /qc/revoke/](#post-qcrevoke)
    * [POST /qc/display/](#post-qcdisplay)
    * [POST /qc/](#post-qc)
* [Activity](#activity)
  * [POST /upload/](#post-upload)*
  * [POST /activity](#post-activity)
* [API Versions](#api-versions)
* [Firmware Update](#firmware-update)
  * [POST /fwupdate/](#post-fwupdate)

 \* these two calls should be implemented first and of these two /unlock/ is most critical.

## Locks
---
### ```POST /lock/```

Used to view information about locks. Supports finding a single lock, multiple locks, or all locks associated with a company. Also supports finding locks by serial number or by both mac or serial number. At least one array must be present but both maybe included. Locks that fit either array will be returned.


#### HEADERS


|Key|Value|
|--|--|
|Content-Type | application/json  |
|Authorization | Bearer *api_key* |


#### BODY

|Parameter|Description|
|--|--|
|```macs``` | An array of mac addresses from the lock(s) _optional_|
|```serials``` | An array of serial numbers for the lock(s) _optional_|


##### Single Lock

```json
{
    "macs": [
		"XX:XX:XX:XX:XX"
	]
}
```
##### Multiple Locks

```json
{ 
    "macs": [
        "XX:XX:XX:XX:XX", 
        "XX:XX:XX:XX:XY"
	]
}
```
##### All Locks

```json
{ 
	//empty body
}
```

#### EXAMPLE RESPONSE

```json
{
    "result": "success",
    "message": "Command successfully completed",
    "error_code": 0,
    "data": {
        "locks": [
            {
                "mac": "XX:XX:XX:XX:XX:XX",
                "created_by": "J Doe",
                "created_date": "2018-04-04T16:00:00Z",
                "setup_count": 0,
                "hw_version": "3p",
                "fw_version": "2.10",
                "serial": "XXX-XXX-XXXX",
                "internal_battery": 0,
                "external_battery": 0,
                "status": "ready",
                "flags": "none"
            }
        ]
    },
    "request": "lock",
    "api_version": { ... }
}
```

|Parameter|Description|
|--|--|
|```result``` | String value representing the result of the call. Either ```success``` or ```failure```|
|```message``` | Readable description of the error|
|```error_code``` | Int value of the error thrown|
|```locks``` | Array of locks contain information about all the requested locks|
|```>mac``` | Mac address of the lock|
|```>created_by``` | Name of the user who registered the lock to the company|
|```>created_date``` | Date that the lock was registered to the company|
|```>setup_count``` | Number of times the lock has been set up. Used for tracking logs|
|```>hw_version``` | Hardware version of the lock|
|```>fw_version``` | Current firmware version of the lock|
|```>serial``` | Serial number of the lock.  This can also be found laser-etched onto the lock|
|```>internal_battery``` | Value of the internal battery of the lock in millivolts|
|```>external_battery``` | Value of the external battery of the lock in millivolts|
|```>status``` | Status of the lock. Indicates if the lock is ready to be used or awaiting setup|
|```>flags``` | An array of flags that indicate if any data needs to be synced to the lock (ie: quick-clicks, keys, etc). Syncing is handled automatically on the unlock endpoint|
|```request``` | Name of the request|
|```api_version``` | Current information about API versions ([see section on API versions](#api-versions))|

[back to top](#overview)
<br/>
<br/>

---
### ```POST /unlock/```

Used to unlock a lock. Requires a session string from the lock (*see [Nokē Mobile library documentation](https://github.com/noke-inc/noke-mobile-library-ios#nok%C4%93-mobile-library-for-ios)*), a mac address, and can optionally take a tracking key to associate to lock activity. 

#### HEADERS

|  Key | Value  |
|--|--|
|Content-Type | application/json  |
|Authorization | Bearer *api_key* |

#### BODY

|Parameter|Description|
|--|--|
|```mac``` | The mac address of the lock|
|```session```| Unique session generated by the lock and read by the phone when connecting. (*see [Nokē Mobile library documentation](https://github.com/noke-inc/noke-mobile-library-ios#nok%C4%93-mobile-library-for-ios)*)|
|```tracking_key``` | An optional string used to associate to lock activity|

```json
{
    "mac": "XX:XX:XX:XX:XX:XX",
    "session": "0000040ba7b0d91f7f45ad25698a18375c7e4f16",
    "tracking_key": "bob@gmail.com"
}
```
#### EXAMPLE RESPONSE

```json
{
    "result": "success",
    "message": "Command successfully completed",
    "error_code": 0,
    "data": {
        "commands": "COMMAND_STRING_HERE"
    },
    "token": "TOKEN_HERE",
    "request": "unlock",
    "api_version": { ... }
}
```

|Parameter|Description|
|--|--|
|```result``` | String value representing the result of the call. Either ```success``` or ```failure```|
|```message``` | Readable description of the error|
|```error_code``` | Int value of the error thrown|
|```commands``` | A string of commands sent to the lock by the [Nokē Mobile library](https://github.com/noke-inc/noke-mobile-library-ios)|
|```request``` | Name of the request|
|```api_version``` | Current information about API versions ([see section on API versions](#api-versions))|

[back to top](#overview)
<br/>
<br/>

---
### ```POST /unshackle/```

Used to remove the shackle from an HD padlock. Operates identically to the ```/unlock/``` endpoint.  Please see [above](#post-unlock) for more details. 

[back to top](#overview)
<br/>
<br/>

---
### ```POST /keys/```

Used to request offline keys for a lock or locks, invalidate any existing keys, or view the status of any offline keys.  These offline keys can be used by the mobile libraries to unlock the lock without an active network connection (*see [Nokē Mobile library documentation](https://github.com/noke-inc/noke-mobile-library-ios#nok%C4%93-mobile-library-for-ios)*).

#### Issue Keys

#### HEADERS

|  Key | Value  |
|--|--|
|Content-Type | application/json  |
|Authorization | Bearer *api_key* |

#### BODY

|Parameter|Description|
|--|--|
|```macs``` | The mac address(es) of the lock(s)|
|```tracking_keys```| Identifying string used to track activity and revoke keys|

```json
{
    "issue": [
    	{
            "macs": ["XX:XX","YY:YY","ZZ:ZZ"],
            "tracking_keys": ["bob","linda"]
    	}
    ]
}
```
#### EXAMPLE RESPONSE

```json
{
    "data": {
        "locks": [
            {
                "mac": "XX:XX",
                "keys": [
                	{
                        "tracking_key": "bob",
                        "offline_key": "abc",
                        "unlock_command": "123"
                	},
                	{
                        "tracking_key": "linda",
                        "offline_key": "def",
                        "unlock_command": "456"
                	}
                ]
            },
           {
                "mac": "YY:YY",
                "keys": [
                	{
                        "tracking_key": "bob",
                        "offline_key": "ghi",
                        "unlock_command": "789"
                	},
                	{
                        "tracking_key": "linda",
                        "offline_key": "jkl",
                        "unlock_command": "A00"
                	}
                ]
            },
             {
                "mac": "ZZ:ZZ",
                "keys": [
                	{
                        "tracking_key": "bob",
                        "offline_key": "ghi",
                        "unlock_command": "789"
                	},
                	{
                        "tracking_key": "linda",
                        "offline_key": "jkl",
                        "unlock_command": "A00"
                	}
                ]
            }
        ]
    },
    "api_version": { ... }
}
```

|Parameter|Description|
|--|--|
|```mac``` | Mac address of the lock.|
|```tracking_key``` | String used to track activity associated with the key|
|```offline_key``` | Key used by the mobile library to encrypt the unlock command|
|```unlock_command```| Command sent by the mobile library to the lock to unlock the lock|
|```error_code``` | Int value of the error thrown|
|```request``` | Name of the request|
|```api_version``` | Current information about API versions ([see section on API versions](#api-versions))|

#### View All Issued Keys

##### HEADERS

|  Key | Value  |
|--|--|
|Content-Type | application/json  |
|Authorization | Bearer *api_key* |

#### BODY

|Parameter|Description|
|--|--|
|```macs``` | The mac address(es) of the lock(s)|
|```tracking_keys```| Identifying string used to track activity and revoke keys|

Input (empty)

```json
{}
```

Input (limit by mac)

```json
{
    "display": {
        "macs": ["XX:XX"]
    }
}
```

Input (limit by tracking key)

```json
{
    "display": {
        "tracking_keys": ["bob"]
    }
}
```


#### EXAMPLE RESPONSE

```json
{
    "data": {
        "locks": [
            {
                "mac": "XX:XX",
                "keys": [
                	{
                        "tracking_key": "bob",
                        "offline_key": "abc",
                        "unlock_command": "123"
                	},
                	{
                        "tracking_key": "linda",
                        "offline_key": "def",
                        "unlock_command": "456"
                	}
                ]
            },
           {
                "mac": "YY:YY",
                "keys": [
                	{
                        "tracking_key": "bob",
                        "offline_key": "ghi",
                        "unlock_command": "789"
                	},
                	{
                        "tracking_key": "linda",
                        "offline_key": "jkl",
                        "unlock_command": "A00"
                	}
                ]
            },
             {
                "mac": "ZZ:ZZ",
                "keys": [
                	{
                        "tracking_key": "bob",
                        "offline_key": "ghi",
                        "unlock_command": "789"
                	},
                	{
                        "tracking_key": "linda",
                        "offline_key": "jkl",
                        "unlock_command": "A00"
                	}
                ]
            }
        ]
    },
    "api_version": { ... }
}
```

|Parameter|Description|
|--|--|
|```mac``` | Mac address of the lock.|
|```tracking_key``` | String used to track activity associated with the key|
|```offline_key``` | Key used by the mobile library to encrypt the unlock command|
|```unlock_command```| Command sent by the mobile library to the lock to unlock the lock|
|```error_code``` | Int value of the error thrown|
|```request``` | Name of the request|
|```api_version``` | Current information about API versions ([see section on API versions](#api-versions))|

#### Revoke Keys

##### HEADERS

|  Key | Value  |
|--|--|
|Content-Type | application/json  |
|Authorization | Bearer *api_key* |

#### BODY

|Parameter|Description|
|--|--|
|```macs``` | The mac address(es) of the lock(s)|
|```tracking_keys```| Identifying string used to track activity and revoke keys|

Input (revoke specific)

```json
{
    "revoke": [
        {"tracking_keys": ["bob"], "macs": ["XX:XX"]}
    ]
}
```

Input (revoke all keys for mac/macs)

```json
{
    "revoke": [
        {
            "macs": ["YY:YY"]
        }
    ]
}
```

Input (revoke all keys for tracking key/keys)

```json
{
    "revoke": [
    	{
        	"tracking_keys": ["bob"]
    	}
    ]
}
```


#### EXAMPLE RESPONSE

```json
{
    "data": {
        "locks": [
            {
                "mac": "XX:XX",
                "keys": [
                	{
                        "tracking_key": "linda",
                        "offline_key": "def",
                        "unlock_command": "456"
                	}
                ]
            },
           {
                "mac": "YY:YY",
                "keys": [
                	{
                        "tracking_key": "steve",
                        "offline_key": "jkl",
                        "unlock_command": "A00"
                	}
                ]
            }
        ]
    },
    "api_version": { ... }
}
```

|Parameter|Description|
|--|--|
|```mac``` | Mac address of the lock.|
|```tracking_key``` | String used to track activity associated with the key|
|```offline_key``` | Key used by the mobile library to encrypt the unlock command|
|```unlock_command```| Command sent by the mobile library to the lock to unlock the lock|
|```error_code``` | Int value of the error thrown|
|```request``` | Name of the request|
|```api_version``` | Current information about API versions ([see section on API versions](#api-versions))|

[back to top](#overview)
<br/>
<br/>

---
### ```Quick Clicks```

Used to issue quick clicks for a lock or locks, revoke existing quick clicks, or view the status of any active quick clicks.  Quick clicks can be used to unlock the lock without additional hardware. 

#### Overview

A quick click is a code that allows entry to a lock via a series of long and short presses. This allows access to a lock without the use of additional software / hardware outside the lock once a quick click has been synced with a lock.

Quick click codes are represented as a string of 1's and 0's representing long and short presses respectively. The exact mechanism of how the quick click is used is dependent on the lock. For example, the standard padlocks use the shackle to enter quick clicks, while the HD padlocks and the U-Lock use a push button at the base of the lock. Typically, when entering a quick click, an LED on the lock will light up for a short press, and the LED will change color on a long press.

Quick clicks may be unlimited use, or may have a limited use count which is rendered unusable once the quick click is used a given number of times.

Quick clicks are created and removed via the API and are synced with a lock when an online unlock is requested. 

**IMPORTANT**:

* Each quick click MUST be a unique code. Quick click codes can only be reused once they have been removed from a lock.
* Quick clicks are only added to / removed from a lock via an *online* unlock.
* Each lock can hold up to 100 quick click codes.
* Quick click codes must be between 3 to 24 characters
* Quick click's can have a limited use count between 1-254 uses OR may be unlimited use represented by a use count of 255

#### ```POST /qc/issue/```

Used to issue quick clicks for a lock. This can either be to issue a new code or to update the number of times an existing code can be used.

##### HEADERS

| Key           | Value            |
| ------------- | ---------------- |
| Content-Type  | application/json |
| Authorization | Bearer *api_key* |

##### BODY

|Parameter|Description|
|--|--|
|```mac``` | A mac address from a lock|
|```quick_clicks``` | An array of quick clicks to create or update|
|```>code``` | A series of 1's and 0's representing the quick click code. Must be at least 3 characters long and no more than 24.|
|```>uses``` | The number of times the code can be used. Any number 1-255 where 255 is a special case and represents unlimited uses.|

###### EXAMPLE REQUEST

```json
{
    "mac": "XX:XX:XX:XX:XX",
    "quick_clicks":[
        {	
            "code": "10100",
            "uses": 255
        },
        {
            "code": "010",
            "uses": 10
        }
    ]
}
```
##### EXAMPLE RESPONSES

Success
```json
{
   "result": "success",
   "message": "Command successfully completed",
   "error_code": 0,
   "error_details": null,
   "data": [
      {
         "mac": "E7:29:0E:6F:79:BB",
         "quick_clicks": [
            {
               "code": "00000",
               "pending": true,
               "uses": 1,
               "start_uses": 1
            }
         ]
      }
   ],
   "request": "qc/issue",
   "api_version": { ... }
}
```
Mixed Success and Failure (version >= 1.0.0)
```json
{
   "result": "success",
   "message": "Command successfully completed",
   "error_code": 0,
   "error_details": [
      {
         "mac": "E7:29:0E:6F:79:BB",
         "code": "10001",
         "msg": "use count can't be greater than 255",
         "action": "issue"
      }
   ],
   "data": [
      {
         "mac": "E7:29:0E:6F:79:BB",
         "quick_clicks": [
            {
               "code": "00000",
               "pending": true,
               "uses": 30,
               "start_uses": 30
            }
         ]
      }
   ],
   "request": "qc/issue",
   "api_version": { ... }
}
```

|Parameter|Description|
|--|--|
|```result``` | String value representing the result of the call. Either ```success``` or ```failure```|
|```message``` | Readable description of the error|
|```error_code``` | Int value of the error thrown|
|```error_details``` | A list of errors encountered. This is particularly useful if multiple quick clicks are issued and some fail while others succeed. ```Field added in version 0.1.0, but not used till 1.0.0```|
|```>mac```| The mac address of the lock with an error|
|```>code```| The quick click code that failed|
|```>msg```| The failure message|
|```>action```| The attempted action that resulted in the error (issue, revoke, etc)|
|```data``` | A list of successful quick click creations/updates. ```Added in version 0.1.0```|
|```>mac```| The mac address of the lock with a success|
|```>quick clicks```| A List of successful quick clicks|
|```>>code```| The quick click code|
|```>>pending```| The code's pending state (if applicable)|
|```>>revoked```| The code's revocation state (if applicable)|
|```>>uses```| Number of times remaining that the code may be used|
|```>>start_uses```| Total number of times the code was set to be used|
|```request``` | Name of the request|
|```api_version``` | Current information about API versions ([see section on API versions](#api-versions))|

[back to top](#overview)

<br/>
<br/>

---
#### ```POST /qc/revoke/```

Used to revoke quick clicks from a lock. This is request is very similar to an issue request, just the uses field is not required. The response is identical.

##### HEADERS

| Key           | Value            |
| ------------- | ---------------- |
| Content-Type  | application/json |
| Authorization | Bearer *api_key* |

##### BODY

|Parameter|Description|
|--|--|
|```mac``` | A mac address from a lock|
|```quick_clicks``` | An array of quick clicks to revoke (identified by the code)|
|```>code``` | A series of 1's and 0's representing the quick click code|

[back to top](#overview)

<br/>
<br/>

---
#### ```POST /qc/display/```

Used to view information about the quick clicks for the given locks. If no macs are included in the request ALL quick clicks for ALL locks will be returned.

##### HEADERS

| Key           | Value            |
| ------------- | ---------------- |
| Content-Type  | application/json |
| Authorization | Bearer *api_key* |

##### BODY

|Parameter|Description|
|--|--|
|```macs``` | An array of mac addresses from the lock(s) to display|

```json
{
	"macs": [
        "YY:YY:YY:YY:YY:YY",
        "XX:XX:XX:XX:XX:XX"
	]
}
```

##### EXAMPLE RESPONSE

```json
{
    "result": "success",
    "message": "Command successfully completed",
    "error_code": 0,
    "error_details": null,
    "data":  [
        {
            "mac": "YY:YY:YY:YY:YY",
            "quick_clicks": [
                {
                    "code": "101",
                    "pending": true,
                    "revoked": false,
                    "uses": 255,
                    "start_uses": 255
                },
                {
                    "code": "010",
                    "pending": true,
                    "revoked": false,
                    "uses": 10,
                    "start_uses": 15
                }
            ]
        },
        {
            "mac": "XX:XX:XX:XX:XX",
            "quick_clicks": [
                {
                    "code": "101",
                    "pending": false,
                    "revoked": true,
                    "uses": 255,
                    "start_uses": 255
                }
            ]
        }
    ],
    "request": "qc/display",
    "api_version": { ... }
}
```

The response parameters are the same as for [/qc/issue/](#post-qc-issue), but the error_details field will likely always be null for a display request since it either succeeds or fails all together.

[back to top](#overview)

<br/>
<br/>

---
#### ```POST /qc/```

Used to request all three quick click action types in one call. This request can contain issues and revokes, but will always be treated as a display call. This is because if no macs are included in the request for display, it will default to displaying ALL quick clicks for ALL locks. This has been the case in the past, but may have been a source of confusion. Also as a result of this, it is impossible to return information about successful issues and revokes in this combined call, but error-details will still contain details about failures.

It is likely that this call will eventually be deprecated and removed at some future time. Before that happens there will need to be some changes to the other calls such as allowing issuing quick clicks for multiple locks instead of just one at a time.

##### HEADERS

| Key           | Value            |
| ------------- | ---------------- |
| Content-Type  | application/json |
| Authorization | Bearer *api_key* |

##### BODY

The request parameters are the same as for the requests above except that for issue and revoke multiple locks can be specified.

|Parameter|Description|
|--|--|
|```issue```| An array of quick clicks to issue|
|```>mac``` | A mac address from a lock|
|```>quick_clicks``` | An array of quick clicks to create or update|
|```>>code``` | A series of 1's and 0's representing the quick click code|
|```>>uses``` | The number of times the code can be used|
|```revoke```| An array of quick clicks to revoke|
|```>mac``` | A mac address from a lock|
|```>quick_clicks``` | An array of quick clicks to revoke (identified by the code)|
|```>>code``` | A series of 1's and 0's representing the quick click code|
|```display```| The locks to display|
|```>macs``` | An array of mac addresses from the lock(s) to display|


```json
{
    "issue": [
        {
            "mac": "AA:AA:AA:AA:AA",
			"quick_clicks":[
				{	
					"code": "101",
					"uses": 255
				},
				{
					"code": "010",
					"uses": 10
				}
			]
        }
	],
    "revoke": [
        {
            "mac": "BB:BB:BB:BB:BB",
			"quick_clicks":[
				{	
					"code": "101"
				}
            ]
        },
        {
            "mac": "CC:CC:CC:CC:CC",
			"quick_clicks":[
				{	
					"code": "101"
				}
            ]
        }
    ],
    "display": {
        "macs": ["YY:YY:YY:YY:YY", "XX:XX:XX:XX:XX"]
    }
}
```
[back to top](#overview)
<br/>
<br/>

## Activity

---
### ```POST /upload/```

Used to upload lock activity logs from a phone app using the  *Nokē Mobile library*. Requires a session string from the lock (*see [Nokē Mobile library documentation](https://github.com/noke-inc/noke-mobile-library-ios#nok%C4%93-mobile-library-for-ios)*), a mac address, and can optionally take a tracking key to associate to lock activity. 


**NOTE:** this route uses a mobile api key rather than the server api key.


#### HEADERS


|  Key | Value  |
|--|--|
|Content-Type | application/json  |
|Authorization | Bearer *mobile_api_key* |


#### BODY

```json
{
    "logs": [
        {
            "session": "8675309j3nNy1V3G0ty0uRnUMB3r",
            "mac": "XX:XX:XX:XX:XX:XX",
            "received_time": 1518141482,
            "responses": ["8675f1v3tHr3309867", "530931gHTs1x75309"]
        }
	]
}
```

[back to top](#overview)
<br/>
<br/>

---
### ```POST /activity/```

Used to view human-readable activity logs. Can find specific activity logs via filters or can be used to find the most recent activity for a company. Use as many or as few filters to narrow down specific activity data.

#### HEADERS

|  Key | Value  |
|--|--|
|Content-Type | application/json  |
|Authorization | Bearer *api_key* |

#### BODY

|Filter Parameter|Description|
|--|--|
|```lock_macs``` | An array of MAC addresses for locks to return activity for|
|```actions``` | An array of activity actions to return: ```alarm_triggered```, ```button_touched```, ```failed_unlock```, ```locked```, ```proximity_stats```, ```setup_unlocked```, ```start_up```, ```unknown```, ```unlock_via_access_code```, ```unlocked```, ```unlocked_via_quick_click```, ```wake_stats```. By default button_touched, proximity_stats, and wake_stats activity is excluded.|
|```tracking_keys``` | An array of tracking ids to return activity for|
|```before``` | A datetime (YYYY-MM-DDThh:mm:ssZ) specifying the latest activity to return.|
|```after``` | A datetime specifying the earliest activity to return.|

##### All activity (except excluded activity actions)

```json
{
	//empty body
}
```

##### Filtered

```json
{
    "lock_macs": [
        "XX:XX:XX:XX:XX:XX", 
        "XX:XX:XX:XX:XX:XY"
    ],
    "actions": [
        "unlock", 
        "alarm_triggered"
    ],
    "tracking_keys": [
        "bill@gmail.com", 
        "ted@gmail.com"
    ],    
    "before": "2018-05-23T20:08:34Z",
    "after": "2018-05-21T23:22:16Z"
}
```

#### EXAMPLE RESPONSE

```json
{
    "result": "success",
    "message": "Command successfully completed",
    "error_code": 0,
    "data": {
        "activity": [
            {
                "lock_mac": "XX:XX:XX:XX:XX:XX",
                "action": "unlocked",
                "recorded_time": "2018-03-15T17:26:41Z",
                "range_date_start": "2018-03-15T17:20:43Z",
                "range_date_end": "2018-03-15T17:20:43Z",
                "tracking_key": ""
            },
            {
                "lock_mac": "XX:XX:XX:XX:XX:XX",
                "action": "locked",
                "recorded_time": "2018-03-16T13:36:37Z",
                "range_date_start": "2018-02-14T17:21:09Z",
                "range_date_end": "2018-03-16T13:36:27Z",
                "tracking_key": ""
            }
        ]
    },
    "request": "activity",
    "api_version": { ... }
}
```

|Parameter|Description|
|--|--|
|```result``` | String value representing the result of the call. Either ```success``` or ```failure```|
|```message``` | Readable description of the error|
|```error_code``` | Int value of the error thrown|
|```activity``` | Array of activity logs|
|```lock_mac``` | Mac address of the lock|
|```action``` | Action type of the log (ie: unlocked, locked)|
|```recorded_time``` | Date that the log was received by the mobile device|
|```range_date_start``` | If the exact time is not known (offline unlock or quick-click) this is the known earliest date the event could have occurred|
|```range_date_end``` | If the exact time is not known (offline unlock or quick-click) this is the known latest date the event could have occurred|
|```tracking_key``` | A string sent up on the unlock endpoint that can be used to associate activity|
|```request``` | Name of the request|
|```api_version``` | Current information about API versions ([see section on API versions](#api-versions))|

[back to top](#overview)
<br/>
<br/>




## API Versions

While we try to make most changes to the API backwards-compatible this is not always the case. In order to keep from breaking client code we have implemented semantic versions to identify the features/changes made to the API.

### API Version Response

This information is returned with all of the responses shown above. It identifies the version of the response, the available versions, and the default version used if no particular version is requested.

#### EXAMPLE RESPONSE

```json
    "api_version": {
        "response": "0.1.0",
        "available": {
            "0.0.0": {
                "changes": "",
                "status": "deprecated"
            },
            "0.1.0": {
                "changes": "Added ErrorDetails field in response which can hold details about multiple errors and is particularly useful when a request has multiple parts that can succeed or fail independently. Added API versioning. For qc/issue and qc/revoke added successes to Data field.",
                "status": "default"
            },
            "1.0.0": {
                "changes": "error codes/messages from quick click calls changing to be more descriptive and more standardized",
                "status": "active"
            }
        },
        "default": "0.1.0"
   }
```
|Parameter|Description|
|--|--|
|```api_version``` | Current information about API versions|
|```>response``` | The version of this response|
|```>available``` | A list of known API versions keyed by version number (versions are semantic in nature)|
|```>>changes``` | A summary of changes made in this version|
|```>>status``` | The current status of this version i.e. deprecated, default, active|
|```>default``` | This is the version returned if a specific API version is not requested|

### API Version Request

A client can specify which version of the API they wish to use with the following parameter added to any* of the above requests. If the parameter is not included in the request then the current default version will be used. 

As a reminder on semantic versions: 
* A change of the first number, major, indicates a potentially breaking change (feature or otherwise).
* A change of the second number, minor, indicates a new non-breaking feature.
* A change of the last number, patch, indicates a bug fix. 

Patch changes are informational only and requesting one specifically will not likely have any affect. In other words if you request 1.2.1 and a bug was fixed in 1.2.2 you'll still get the bug fix.

The same is likely true of minor changes. Since these changes are non-breaking (may add functionality but not modify or remove existing functionality) you'd may find these features' effects even if you request a lower version.

Major changes are those that remove a feature or change it in such a way that it might break something on the client side. This could be as simple as a change in error messages (in case some code depends on a specific string) or as complicated as replacing an endpoint with a new one. These changes are the ones that may require you to request an older version.

*Note that this is currently only enabled on quick click requests. It will be enabled on other requests as time and requirements permit.

#### BODY

|Parameter|Description|
|--|--|
|```api_version``` | The API version to execute for this request|

Input 

```json
    "api_version": "1.0.0"
```

[back to top](#overview)
<br/>
<br/>



## Firmware Update

Updating the firmware on a Noke device is a two-step process:

1. Put the device into firmware update mode
2. Send the firmware file to the device

### Firmware Update Mode

To place a Noke device into firmware update mode, make a request to the ```fwupdate``` endpoint. This endpoint functions similarly to the ```unlock/``` endpoint, but instead of unlocking the lock, returns a command that places the lock into firmware update mode.

### ```POST /fwupdate/```


Used to place a Noke device into firmware update mode. Requires a session string from the lock (*see [Nokē Mobile library documentation](https://github.com/noke-inc/noke-mobile-library-ios#nok%C4%93-mobile-library-for-ios)*), and a mac address


#### HEADERS


|  Key | Value  |
|--|--|
|Content-Type | application/json  |
|Authorization | Bearer *api_key* |


#### BODY

|Parameter|Description|
|--|--|
|```mac``` | The mac address of the lock|
|```session```| Unique session generated by the lock and read by the phone when connecting. (*see [Nokē Mobile library documentation](https://github.com/noke-inc/noke-mobile-library-ios#nok%C4%93-mobile-library-for-ios)*)|

```json
{
    "mac": "XX:XX:XX:XX:XX:XX",
    "session": "8675309j3nNy1V3G0ty0uRnUMB3r",
}
```

#### EXAMPLE RESPONSE

```json
{
    "result": "success",
    "message": "Command successfully completed",
    "error_code": 0,
    "data": {
        "commands": "COMMAND_STRING_HERE"
    },
    "token": "TOKEN_HERE",
    "request": "unlock"
}
```

|Parameter|Description|
|--|--|
|```result``` | String value representing the result of the call. Either ```success``` or ```failure```|
|```message``` | Readable description of the error|
|```error_code``` | Int value of the error thrown|
|```commands``` | A string of commands sent to the lock by the [Nokē Mobile library](https://github.com/noke-inc/noke-mobile-library-ios)|
|```request``` | Name of the request|


### Sending the Firmware file to the Noke Device

Once the Noke device is in firmware update mode, the firmware can now be updated by sending it a new firmware file. Noke devices use a chip provided by **Nordic Semiconductor**, therefore we've found that the best way to send a firmware file is to use the DFU library provided by Nordic.

Please follow the instructions provided by Nordic to implement their library into your mobile application. The DFU library is available for [iOS](https://github.com/NordicSemiconductor/IOS-Pods-DFU-Library) and [Android](https://github.com/NordicSemiconductor/Android-DFU-Library)

**Note**: The firmware files needed are currently not available to the public. To get the firmware file needed to update a Noke device, please contact a member of the Noke development team.

[back to top](#overview)