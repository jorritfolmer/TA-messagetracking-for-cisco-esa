# Cisco ESA Ironport messagetracking technical add-on for Splunk

This TA parses the unwieldly Cisco Ironport logging into tidy single events containing OSI L3, message and threat metadata.

## Workings

1. It consumes cisco:esa:textmail sourcetype logging through a scheduled search 
2. Once per hour
3. And outputs cisco:esa:messagetracking

## Setup

0. Install Splunk Add-on for Cisco ESA, but disable the tags
1. Change the index from which the saved search reads, if needed
2. Change the index to which the saved search writes, if needed
3. Do not change the schedule of the saved search unless you know what you're doing

## Coverage

The saved search significatly outperforms previous versions in terms of internal_message_id coverage.
Based on a corpus of 1 hour log this TA reaches 64% coverage:

| count of internal_message_id in cisco:esa:textmail | count of internal_message_id in cisco:esa:messagetracking | coverage
|-----|------|------|
| 5700|  3700|  65% |

This looks horrible, but in reality the ones missing from cisco:esa:messagetracking are:

- double bounces due to expired delivery 
- delayed messages due to 4xx temporary error codes that last longer than 1 hour

The Splunk search you can use to determine the coverage for your own mail per hour:

```
(index=main sourcetype="cisco:esa:textmail") OR (index=summary sourcetype="cisco:esa:messagetracking" ) 
| fields internal_message_id,sourcetype 
| bin _time span=1h
| stats count by internal_message_id,sourcetype,_time
| stats count(eval(sourcetype=="cisco:esa:textmail")) as textmail count(eval(sourcetype=="cisco:esa:messagetracking")) as messagetracking by internal_message_id,_time
| stats sum(textmail) as textmail sum(messagetracking) as messagetracking by _time
| eval coverage=round((messagetracking/textmail),2)

```

## CIM

This add-on maps cisco:esa:messagetracking to the following datamodels, filling a number of default Splunk Enterprise Security panels:
- email
- intrusion detection
- malware

## Contributers

This add-on is maintained by Jorrit Folmer. 

## Support

This is an open source project without warranty of any kind. No support is provided. However, a public repository and issue tracker are available at https://github.com/jorritfolmer/TA-messagetracking-for-cisco-esa-ironport
