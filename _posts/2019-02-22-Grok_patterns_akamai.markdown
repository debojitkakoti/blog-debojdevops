---
layout: post
title:  "Learning grok patterns"
date:   2019-02-22 14:10:51 +0800
categories: Ops
tags: Ops
description: This posts contains how I learnt grok patterns by writing one for akamai log
---

The ability to query data being shipped to ELK stacks depends on the data being
readable. The unstructured data needs to be translated into structured messages.

[Grok](https://grok.nflabs.com/) is a way to match a line against a regular expression. 

We need to pass grok filters in logstash or data pipeline defination in ELK to
parse logs. Though you will find grok patterns for common logs such as apache or
nginx, sometimes we need to write our own grok patterns. Learning custom grok
pattern is very helpful. I will try to explain how I learnt writing custom grok
patterns.

Logstash already has some inbuilt patterns.

## Common grok patterns related to networking

[Common grok Patterns](https://github.com/elastic/logstash/blob/v1.4.2/patterns/grok-patterns)

```
# Networking
MAC (?:%{CISCOMAC}|%{WINDOWSMAC}|%{COMMONMAC})
CISCOMAC (?:(?:[A-Fa-f0-9]{4}\.){2}[A-Fa-f0-9]{4})
WINDOWSMAC (?:(?:[A-Fa-f0-9]{2}-){5}[A-Fa-f0-9]{2})
COMMONMAC (?:(?:[A-Fa-f0-9]{2}:){5}[A-Fa-f0-9]{2})
IPV6 ((([0-9A-Fa-f]{1,4}:){7}([0-9A-Fa-f]{1,4}|:))|(([0-9A-Fa-f]{1,4}:){6}(:[0-9A-Fa-f]{1,4}|((25[0-5]|2[0-4]\d|1\d\d|[1-9]?\d)(\.(25[0-5]|2[0-4]\d|1\d\d|[1-9]?\d)){3})|:))|(([0-9A-Fa-f]{1,4}:){5}(((:[0-9A-Fa-f]{1,4}){1,2})|:((25[0-5]|2[0-4]\d|1\d\d|[1-9]?\d)(\.(25[0-5]|2[0-4]\d|1\d\d|[1-9]?\d)){3})|:))|(([0-9A-Fa-f]{1,4}:){4}(((:[0-9A-Fa-f]{1,4}){1,3})|((:[0-9A-Fa-f]{1,4})?:((25[0-5]|2[0-4]\d|1\d\d|[1-9]?\d)(\.(25[0-5]|2[0-4]\d|1\d\d|[1-9]?\d)){3}))|:))|(([0-9A-Fa-f]{1,4}:){3}(((:[0-9A-Fa-f]{1,4}){1,4})|((:[0-9A-Fa-f]{1,4}){0,2}:((25[0-5]|2[0-4]\d|1\d\d|[1-9]?\d)(\.(25[0-5]|2[0-4]\d|1\d\d|[1-9]?\d)){3}))|:))|(([0-9A-Fa-f]{1,4}:){2}(((:[0-9A-Fa-f]{1,4}){1,5})|((:[0-9A-Fa-f]{1,4}){0,3}:((25[0-5]|2[0-4]\d|1\d\d|[1-9]?\d)(\.(25[0-5]|2[0-4]\d|1\d\d|[1-9]?\d)){3}))|:))|(([0-9A-Fa-f]{1,4}:){1}(((:[0-9A-Fa-f]{1,4}){1,6})|((:[0-9A-Fa-f]{1,4}){0,4}:((25[0-5]|2[0-4]\d|1\d\d|[1-9]?\d)(\.(25[0-5]|2[0-4]\d|1\d\d|[1-9]?\d)){3}))|:))|(:(((:[0-9A-Fa-f]{1,4}){1,7})|((:[0-9A-Fa-f]{1,4}){0,5}:((25[0-5]|2[0-4]\d|1\d\d|[1-9]?\d)(\.(25[0-5]|2[0-4]\d|1\d\d|[1-9]?\d)){3}))|:)))(%.+)?
IPV4 (?<![0-9])(?:(?:25[0-5]|2[0-4][0-9]|[0-1]?[0-9]{1,2})[.](?:25[0-5]|2[0-4][0-9]|[0-1]?[0-9]{1,2})[.](?:25[0-5]|2[0-4][0-9]|[0-1]?[0-9]{1,2})[.](?:25[0-5]|2[0-4][0-9]|[0-1]?[0-9]{1,2}))(?![0-9])
IP (?:%{IPV6}|%{IPV4})
HOSTNAME \b(?:[0-9A-Za-z][0-9A-Za-z-]{0,62})(?:\.(?:[0-9A-Za-z][0-9A-Za-z-]{0,62}))*(\.?|\b)
HOST %{HOSTNAME}
IPORHOST (?:%{HOSTNAME}|%{IP})
HOSTPORT %{IPORHOST}:%{POSINT}
```

## Procedure to write custom grok pattern.

[Grok debugger](https://grokdebug.herokuapp.com/) is very useful to write custom
patterns. Switch to debugger and paste your custom log.
As an example I will paste the following example akamai log.
```
2019-02-03	17:51:15	111.11.2.151	GET	/staticmobile.example.akamai.com/installations/example.txt	000	267575	162998	"-"	"Mozilla/5.0 (Linux; U; Android 8.0zh-cn; ONEPLUS A6000 Build/PKQ1.180716.001) AppleWebKit/537.36 (KHTML, like Gecko)Version/4.0 Chrome/57.0.2987.132 MQQBrowser/8.1 Mobile Safari/537.36"	"-"
``` 
Here first try to match the date and time part i.e. `2019-02-03 17:51:15` . 
For matching date and time there is already inbuilt pattern availabale that is
TIMESTAMP_ISO8601 , but in this case it is not matching the date time we have.
So we ll modify it according to our requirement.

Tick checkbox for "Add custom patterns" in the grok debugger, in that custom
pattern box we ll define TIMESTAMP_AKMAI pattern as follow

```
TIMESTAMP_AKMAI %{YEAR}-%{MONTHNUM}-%{MONTHDAY}%{SPACE}%{HOUR}:?%{MINUTE}(?::?%{SECOND})?%{ISO8601_TIMEZONE}?

```

Note that every grok pattern has two section %{pattern_here:keyword_here} .
First section contains pattern to match and second section contains the keyword
to hold the data.

In the pattern section you can write the pattern or substitute it with your own
variable. In our case TIMESTAMP_AKMAI is custom variable that is holding custom
grok pattern to match our date time.

After timestamp there is ip address. To match ipaddress or hostname you can use
inbuilt pattern called IPORHOST. So resulting pattern for matching timestamp and
ip will be.
```
%{TIMESTAMP_AKMAI:timestamp}%{SPACE}%{IPORHOST:clientip}

```
Same way after matching the ip, we have http methods to match. To match http
verbs we can use inbuilt "verb" pattern. The resulting pattern will be
```
%{TIMESTAMP_AKMAI:timestamp}%{SPACE}%{IPORHOST:clientip}%{SPACE}%{WORD:verb}

```

So the complete grok pattern for the above log will be as follows

```
%{TIMESTAMP_AKMAI:timestamp}%{SPACE}%{IPORHOST:clientip}%{SPACE}%{WORD:verb}%{SPACE}%{URIPATHPARAM:uri}%{SPACE}%{NUMBER:scs_tatus}%{SPACE}%{NUMBER:sc_bytes}%{SPACE}%{NUMBER:time_taken_ms}%{SPACE}\"-\"%{SPACE}%{QS:agent}%{SPACE}\"-\"


TIMESTAMP_AKMAI %{YEAR}-%{MONTHNUM}-%{MONTHDAY}%{SPACE}%{HOUR}:?%{MINUTE}(?::?%{SECOND})?%{ISO8601_TIMEZONE}?

```

