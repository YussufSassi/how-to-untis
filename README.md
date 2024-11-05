
This quick guide outlines how you can bypass Untis' UI and get API access to the timetable software, without any need for an API key.

The reason this is not a library is partly because I am lazy, and because this is highly dependent on which school's timetable you want access to.

## What you will need
This is not an unauthorized way to access the platform, and not an "exploit" in any way - this is all expected behavior. 

In accordance with this, you will need:
- Access to a school's timetables (username, password and the school's Untis URL)

## Authentication
Untis uses both a `JSESSIONID` cookie and a Bearer-JWT Authorization token which will need to be refreshed. In my proof of concept (which won't be shared with this guide), I simply regenerated this authorization token with every query to the database, which also works perfectly fine.

To get the Bearer token though, you will need a session id. And to get the session ID, you will need to send a request to the schools login url: `https://<schools_subdomain>.webuntis.com/WebUntis/j_spring_security_check` with the following data:

```jsonc
{
"school": "asdf-school", // Your school's ID
"j_username": "user", // Your username with access to the timetables
"j_password": "pass123", // Password of the before mentioned user
"token": "" // This needs to be left blank, but still included in the request.
}
```

You can determine the school's ID from the URL. First, open https://webuntis.com. Then search for your school and click on it. The URL should now look like this: `https://niobe.webuntis.com/WebUntis/?school=<school_id>#/basic/login`. You can grab the ID from the ?school query string parameter.

After you have sent this post request, you can ignore the response body and just grab the `JSESSIONID` response cookie. It should look something like this: `F67FB51E162313ACD214C6D2EBCA7F4D`

Now you have the session ID, congrats! 

Next, we will get the Authorization token. To generate it, we will need to query the following URL with a GET request: `https:///<schools_subdomain>.webuntis.com/WebUntis/api/token/new`

You will need to send these headers for the request to work: 
```jsonc
{
 	"User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:130.0) Gecko/20100101 Firefox/130.0", // This is just a common user-agent as an example
	"Accept": "application/json", // Only works when you specify this header.
}
```
and, **most importantly** send your `JSESSIONID` cookie with the value of the session we got earlier.

This request will just return your authorization token in plaintext.

**We are now successfully authenticated and authorized!**

## Getting the timetable

Now that we have our session and token, we need to gather some more data about the school. The easiest way to do this, is by opening your timetable in webuntis and grabbing your current cookies. They should resemble this format:

```jsonc
{
	"schoolname": "_YmsganVzdCBraWRkaW5n=", // This is and underscore + the schools id we got from the URL in base64
	"Tenant-Id": "5037210", // This is a unique identifier of the school
	"JSESSIONID": session, // This is the session we got earlier
}
```


Then we will create a header object which looks like this:
```jsonc
{
"User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:130.0) Gecko/20100101 Firefox/130.0", // these help to mask our traffic as regular traffic
"Accept": "application/json, text/plain, */*", // ^^^^
"Authorization": "Bearer {token}", // "Bearer" + the Authorization token we got earlier

    }
```



And then finally we need to get URL params which look like this:
```jsonc
 {
"start": "2024-11-05T21:17:29Z", // Start date of data in ISO 8601
"end": "2024-10-05T21:17:29Z", // End date of data in ISO 8601
"format": "2", // Depends on school, check your network requests while fetching the timetable on XHR tab and looks for a requests to the URL "/entries" and grab it from there.
"resourceType": "STUDENT", // look above
"resources": "12143",// look above
"periodTypes": "NORMAL_TEACHING_PERIOD,ADDITIONAL_PERIOD,EVENT,STAND_BY_PERIOD,OFFICE_HOUR,EXAM,BREAK_SUPERVISION",// look above
 }
```


Then finally we can send a GET request to this URL  using the above cookies, headers and URL parameters: `https://<schools_subdomain>.webuntis.com/WebUntis/api/rest/view/v1/timetable/entries`

You will get a JSON schema similar to this (This is what it looks like for 3 days):

```jsonc
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "Timetable",
  "description": "JSON Schema for Untis timetable response object",
  "type": "object",
  "properties": {
    "days": {
      "type": "array",
      "items": {
        "type": "object",
        "properties": {
          "date": {
            "type": "string",
            "format": "date"
          },
          "resourceType": {
            "type": "string"
          },
          "resource": {
            "type": "object",
            "properties": {
              "id": {
                "type": "integer"
              },
              "shortName": {
                "type": "string"
              },
              "longName": {
                "type": "string"
              },
              "displayName": {
                "type": "string"
              }
            },
            "required": ["id", "shortName", "longName", "displayName"]
          },
          "status": {
            "type": "string"
          },
          "dayEntries": {
            "type": "array",
            "items": {}
          },
          "gridEntries": {
            "type": "array",
            "items": {
              "type": "object",
              "properties": {
                "ids": {
                  "type": "array",
                  "items": {
                    "type": "integer"
                  }
                },
                "duration": {
                  "type": "object",
                  "properties": {
                    "start": {
                      "type": "string",
                      "format": "date-time"
                    },
                    "end": {
                      "type": "string",
                      "format": "date-time"
                    }
                  },
                  "required": ["start", "end"]
                },
                "type": {
                  "type": "string"
                },
                "status": {
                  "type": "string"
                },
                "statusDetail": {
                  "type": "string"
                },
                "name": {
                  "type": "string"
                },
                "layoutStartPosition": {
                  "type": "integer"
                },
                "layoutWidth": {
                  "type": "integer"
                },
                "layoutGroup": {
                  "type": "integer"
                },
                "color": {
                  "type": "string"
                },
                "notesAll": {
                  "type": "string"
                },
                "icons": {
                  "type": "array",
                  "items": {}
                },
                "position1": {
                  "type": "array",
                  "items": {
                    "type": "object",
                    "properties": {
                      "current": {
                        "type": "object",
                        "properties": {
                          "type": {
                            "type": "string"
                          },
                          "status": {
                            "type": "string"
                          },
                          "shortName": {
                            "type": "string"
                          },
                          "longName": {
                            "type": "string"
                          }
                        },
                        "required": ["type", "status", "shortName", "longName"]
                      },
                      "removed": {
                        "type": "string"
                      }
                    }
                  }
                },
                "position2": {
                  "type": "array",
                  "items": {
                    "type": "object",
                    "properties": {
                      "current": {
                        "type": "object",
                        "properties": {
                          "type": {
                            "type": "string"
                          },
                          "status": {
                            "type": "string"
                          },
                          "shortName": {
                            "type": "string"
                          },
                          "longName": {
                            "type": "string"
                          }
                        },
                        "required": ["type", "status", "shortName", "longName"]
                      },
                      "removed": {
                        "type": "string"
                      }
                    }
                  }
                },
                "position3": {
                  "type": "array",
                  "items": {
                    "type": "object",
                    "properties": {
                      "current": {
                        "type": "object",
                        "properties": {
                          "type": {
                            "type": "string"
                          },
                          "status": {
                            "type": "string"
                          },
                          "shortName": {
                            "type": "string"
                          },
                          "longName": {
                            "type": "string"
                          }
                        },
                        "required": ["type", "status", "shortName", "longName"]
                      },
                      "removed": {
                        "type": "string"
                      }
                    }
                  }
                },
                "position4": {
                  "type": "array",
                  "items": {
                    "type": "object",
                    "properties": {
                      "current": {
                        "type": "object",
                        "properties": {
                          "type": {
                            "type": "string"
                          },
                          "status": {
                            "type": "string"
                          },
                          "shortName": {
                            "type": "string"
                          },
                          "longName": {
                            "type": "string"
                          }
                        },
                        "required": ["type", "status", "shortName", "longName"]
                      },
                      "removed": {
                        "type": "string"
                      }
                    }
                  }
                },
                "position5": {
                  "type": "array",
                  "items": {}
                },
                "lessonText": {
                  "type": "string"
                },
                "lessonInfo": {
                  "type": "string"
                },
                "substitutionText": {
                  "type": "string"
                },
                "userName": {
                  "type": "string"
                },
                "moved": {
                  "type": "string"
                },
                "durationTotal": {
                  "type": "string"
                },
                "link": {
                  "type": "string"
                }
              }
            }
          },
          "backEntries": {
            "type": "array",
            "items": {}
          }
        }
      }
    }
  }
}
```

The timetable is organized by days, and each day has grid entries representing teaching periods.
Each grid entry contains information about the class, teacher, subject, and room for that period.
The timetable also includes dates, resource types, and status information for each day.

This structure is quite complicated and unwieldy, so I used this little function to turn it into a more readable and usable format for applications that look at Untis from the student's side:

```python
def simplify_timetable_json(entries):
    return {
        "days": [
            {
                "date": day["date"],
                "classes": [
                    {
                        "startTime": entry["duration"]["start"],
                        "endTime": entry["duration"]["end"],
                        "subject": entry["position3"][0]["current"]["longName"],
                        "teacher": entry["position2"][0]["current"]["longName"],
                        "room": (
                            entry["position4"][0].get("current", {}).get("shortName")
                            if len(entry["position4"]) > 0
                            else ""
                        ),
                    }
                    for entry in day["gridEntries"]
                ],
            }
            for day in entries["days"]
        ]
    }
```


You have successfully fetched the current timetable from WebUntis, without any need for their proprietary API
