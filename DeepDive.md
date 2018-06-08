
name: inverse
layout: true
class: center, middle, inverse

---


---
#deep dive into processing pipelines

open dource day 2018, berlin

---

layout: false
.left-column[
	#about me
]
.right-column[
.pull-left[
	![avatoone](images/avatoone.jpg)
]
.pull-right[

* system administrator / support engineer since 2000
* @ graylog since 2016
* social media (@)jalogisch.red[*]
* [about.me/jandoberstein](https://about.me/jandoberstein)

.footnote[.red.bold[*] means 'yes, indeed!']
]
]

---
layout: false
.left-column[
# agenda
]
.right-column[

* why processing pipelines
* how it works
* processing
  * rules
  * pipelines
  * connections
* rules
  * best practice
  * how to construct
  * examples

]



---
.left-column[
## what do 
## processing pipelines

]

.right-column[

give the ability to ...

* grep information out of a string.red[*]
* add information to a string.red[*]
* modify information of a string.red[*]

.footnote[.red.bold[*] log messages are strings]
]


---
name: how
template: inverse

# why processing pipelines
---
template: inverse

###make a log message readable 

--
template: inverse

##create value for non specialist

---
layout: false
name: works

.left-column[
# how does it work
]

---
background-image: url(images/magic.gif)
.left-column[
# how does it work
]


---
.left-column[
# how does it work
## overview 
]

.right-column[


* write instructions
  * processing rules

* order the instructions
  * processing pipeline

* connect the message stream
 

]


---

.left-column[
# rules
]

.right-column[


.pull-left[
* name _inline_

* simple _when-then_

* no _else_ 
]

.pull-right[

```
rule "foobar"
when
	...
then
	...
end
```
]


]
---
.left-column[

#pipe

]

.right-column[

pipeline stages can be configured that **all** rules must match to be succesfull or that at **least one** of the rules must match to go to the next stage


.pull-left[

```
* stage 0 (all match)
  - rule foobar
  - rule barfoo
* stage 1 (one match)
  - rule foo
  - rule bar
* stage 2 (one match)
  - rule beer!
* stage 3 (one match)
  - rule whisky
```
]
]

---
.left-column[

#pipe

]

.right-column[

pipeline stages can be configured that **all** rules must match to be succesfull or that at **least one** of the rules must match to go to the next stage


.pull-left[
```
* stage 0 (all match)
  - rule foobar
  - rule barfoo
* stage 1 (one match)
  - rule foo
  - rule bar
* stage 2 (one match)
  - rule beer!
* stage 3 (one match)
  - rule whisky
``` 
]

.pull-right[

* stage 0 
   - _bar_ match
   - _bar_ match
* stage 1 
   - _bar_ **no-match**
   - _bar_ match
* stage 2
   - _bar_ **no-match**
     * `drop out`
* stage 3
   - **no-run**
]

]

---
name: rules
layout: true
class: center, middle, inverse

#rules
the backbone

---

---
layout: false
.left-column[
#rule 
##name
]

.right-column[
* **must** be uniq
* is the only identifier 
* **should** only changed very carefully
]
---

.left-column[
#rule 
##name

]
.right-column[
##bad names
  - 'syslog'
  - 'test1'
  - 'foo'
]
---

.left-column[
#rule 
##name
]
.right-column[
##good names
  - 'extract_mac_from_cisco_message_field'
  - 'route_to_alert_stream'
  - 'ops_add_hw_location'
  - 'dev_extract_modul'
]
---

.left-column[
#rule 
##detail

]
.right-column[
* use comments in the rule
* write more small rules with one specific action (**kiss**)
* make them usefull for multiple pipelines
]

---

.left-column[
#rule 
##creation
]
.right-column[
* construct rules with data
  - or know how your data will be transformed
* test rules 
  - have a test system 
  - know that adjustance need time
* do not expect valid data with first message
]

---

.left-column[
#rule
##creation
]

.right-column[
* access fields with `$message.field_name`
* field need to be present
* field typ need to be set in rules
]

---


.left-column[
#rule 
##creation
]

.right-column[
* field need to be present

```bash
rule "check hostname (error in server.log if missing)"
when
 to_string($message.hostname) == gw 
then
 ...
end
```

```bash
rule "check hostname (content check only if field is present)"
when
 has_field("hostname") AND to_string($message.hostname) == gw 
then
 ...
end
```

]


---
.left-column[
#rule 
##creation
]

.right-column[
* field typ need to be set in rules

```bash
rule "-4 hours"
when
 has_field("timestamp")
then
 set_field("timestamp", to_date($message.timestamp) - hours(4));
end
```
]



---

.left-column[
#rule
##creation
]

.right-column[
* use documentation as reference
* use tests [src/test/ressources](https://github.com/Graylog2/graylog2-server/tree/master/graylog2-server/src/test/resources/org/graylog/plugins/pipelineprocessor/functions) as reference
* contribute to the [documentation of processing pipelines](http://docs.graylog.org/en/stable/pages/pipelines.html)
]


---

.left-column[
#rule 
##common

]


.right-column[

```bash
rule "anonymize_ip"
when
  has_field("ip_address")
then
  let ip_addr = to_string($message.ip_address);
  let hash = sha256(ip_addr);
  set_field("ip_address", hash);
end

```
]

---

.left-column[
#rule
##common
]

.right-column[

```bash
rule "alert_on_sync_failures"
when
  has_field("sync_node") AND to_long($message.sync_node) != 0
then
  set_field("alert", "1");
end
```
]

---

.left-column[
#rule
##common
]

.right-column[

```bash
rule "auditd_kv_ex_prefix"
when
    has_field("is_auditd")
then
    // extract all key-value from "message" 
    // and prefix it with auditd_ 
    set_fields(
                fields: 
                        key_value(
                            value: to_string($message.message), 
                            trim_value_chars: "\""
                            ),
                prefix: "auditd_"
            );

end
```
]

---

.left-column[
#rule
##common
]

.right-column[

```bash
rule "mysql: extract slow query log"
when
  has_field("type") && 
  to_string($message.type) == "mysql-slow"
then
 let message_field = to_string($message.message);
 let action = grok(
 				pattern: "(?s) User@Host: (?:%{USERNAME:mysql_clientuser})(?:%{GREEDYDATA}) @ (?:%{DATA:mysql_clienthost}) \\[(?:%{DATA:mysql_clientip}\\]) %{GREEDYDATA} Query_time: %{NUMBER:mysql_querytime}(?:%{SPACE})Lock_time: %{NUMBER:mysql_locktime}(?:%{SPACE})Rows_sent: %{NUMBER:mysql_rowssent}(?:%{SPACE})Rows_examined: %{NUMBER:mysql_rowsexamined}(?:%{SPACE})(?:%{GREEDYDATA})SET timestamp=%{NUMBER:mysql_timestamp}\\;(?:%{GREEDYDATA:mysql_slow_query})\\;", 
 				value: message_field, 
 				only_named_captures: true);
 set_fields(action);
end
 
```

```
"(?s) User@Host: (?:%{USERNAME:mysql_clientuser})
(?:%{GREEDYDATA}) @ (?:%{DATA:mysql_clienthost}) 
\\[(?:%{DATA:mysql_clientip}\\]) %{GREEDYDATA} 
Query_time: %{NUMBER:mysql_querytime}(?:%{SPACE})
Lock_time: %{NUMBER:mysql_locktime}(?:%{SPACE})
Rows_sent: %{NUMBER:mysql_rowssent}(?:%{SPACE})
Rows_examined: %{NUMBER:mysql_rowsexamined}
(?:%{SPACE})(?:%{GREEDYDATA})
SET timestamp=%{NUMBER:mysql_timestamp}
\\;(?:%{GREEDYDATA:mysql_slow_query})\\;"
```

]


---

.left-column[
#rule
##common
]

.right-column[

```bash
rule "change_timezone_to_America/New_York"
when
 has_field("timestamp") AND
 // change only messages from a specific input 
 to_string($message.gl2_source_input) == "5aec2a970947040001c7e511" 
then
    // Without DST in mind 
    set_field("timestamp_minus", 
    			to_date($message.timestamp) - hours(4)
    			);
    

    // create new date object with correct timezone
    let ts_new = parse_date(
    	value: ts_orig, 
    	pattern: "yyyy-MM-dd'T'HH:mm:ss.SSS", 
    	timezone: "America/New_York");
    // set new timestamp with changed timezone 
    set_field("timestamp", ts_new);
end
```
]


---

.left-column[
#rule
##common
]

.right-column[

```bash
rule "check_timestamp"
when
    ( to_date($message.timestamp, "PST") - weeks(2) ) 
    	< ( now("PST") )
then
    // for testing just add a field
    set_field("old_input", "2_weeks_old");
    
    // after everything is working
    // drop the message - just uncomment the following
    // drop_message();
    
    // debug should be present for controll
    debug("TS 2 weeks in the past dropped from $message.source");
    
end
```
]



---

.left-column[
#rule
##common
]

.right-column[

```bash

rule "calc_processing_time"
when
  // REQUESTTIME Format hh:mi:ss.mmm
  has_field("REQUESTTIME") AND
  // RESPONSETIME Format hh:mi:ss.mmm
  has_field("RESPONSETIME")
then
    // the math of RESPONSETIME minus REQUESTTIME
    // translated to milliseconds
    set_field( "processing_time", parse_date(
    				value: to_string($message.RESPONSETIME), 
    				pattern: "HH:mm:ss.SSS", 
    				locale:"en" ).millis - parse_date(
    						value: to_string($message.REQUESTTIME),
    						pattern: "HH:mm:ss.SSS", 
    						locale:"en" ).millis );
end
```
]


---

.left-column[
#summary
## rules
]

.right-column[

* **when** should be very specific
 * try to sort away messages before heavy processing
 * actively choose what message get processed
* use _debug_ fields in the messages
 * e.g. what pipe and rule last touched the message
* use _debug_ function when deleting messages

]


---

.left-column[
#summary
## rules
## pipe
]

.right-column[


* **when** should be very specific
 * try to sort away messages before heavy processing
 * actively choose what message get processed
* use _debug_ fields in the messages
 * e.g. what pipe and rule last touched the message
* use _debug_ function when deleting messages


* prefer multiple **stages** over complicated rules
 * see [post working with cisco messages](https://jalogisch.de/2018/working-with-cisco-asa-nexus-on-graylog/).red[*]
* only run rules and pipelines you understand
* monitor the metrics (use [metric-reporter-plugin!](https://github.com/graylog-labs/graylog-plugin-metrics-reporter))
 * you will have messages that break your processing


.footnote[.red.bold[*] kudos to @wrf42] 
]



---

name: last-page
template: inverse

## That's all folks (for now)!
Slideshow created using [remark](http://github.com/gnab/remark).