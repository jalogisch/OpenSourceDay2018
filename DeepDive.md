
name: inverse
layout: true
class: center, middle, inverse

#deep dive into processing pipelines

open dource day 2018, berlin

---

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

* system administrator / support engineer since since 2000
* work at graylog since 2016
* (@)jalogisch.red[*]
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
layout: true
class: center, middle, inverse


# what is processing pipelines

---

---

make a log message readable 

--
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
## ( overview )
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
rule 'foobar'
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
* MUST be uniq
* is the only identifier 
* SHOULD only changed very carefully
]
---
# rule name

* good names
  - 'extract_mac_from_cisco_message_field'
  - 'route_to_alert_stream'
  - 'ops_add_hw_location'
  - 'dev_extract_modul'

---
# rule name

* bad names
  - 'syslog'
  - 'test1'
  - 'foo'

---
# rule detail

* use comments in the rule
* write more small rules with one specific action (KISS)
* make them usefull for multiple pipelines

---
# rule creation

* construct rules with data
  - or know how your data will be transformed
* test rules 
  - have a test system 
  - know that adjustance need time
* do not expect valid data with first message


---
# rule creation

* access fields with $message.fiel_name
* fields need to be present
* field typ need to be set in rules

---
# rule creation

* to_string($message.ip_address)
* to_date($message.timestamp)

---
# rule creation

* use documentation as reference
* use tests (https://github.com/Graylog2/graylog2-server/tree/master/graylog2-server/src/test/resources/org/graylog/plugins/pipelineprocessor/functions) as reference
* contribute to the documentation of processing pipelines


---
# rule (common)

```
rule "anonymize-ip"
when
  has_field("ip_address")
then
  let ip_addr = to_string($message.ip_address);
  let hash = sha256(ip_addr);
  set_field("ip_address", hash);
end
```

---
# rule (common)




---
name: last-page
template: inverse

## That's all folks (for now)!