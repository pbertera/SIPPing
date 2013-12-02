# SIPPing Python SIP Packet forging tool

SIPPing is a simple SIP packet forging tool written in pure Python.

With SIPPing you can create SIP Requests based on simple text templates. In the command line you can define variables that will be substituted in template.

### Features:

* inline parsable templates for SIP messaging
* "ping style" behavior: SIPPing sends out a SIP Request and wait for the response
* aggressive mode: do not wait for any response and send out the request
* you can use Python code for dynamic variable generation
* you can dynamically load any python module used by your python variables
* *Content-Length* and *CSeq* headers can be automatically generated (if not present in template file)
* print on stdout received response with regex substitution applied

### Usage note:

This software is released for didactical and debugging purposes. You're free to use it at your own risk.
You can modify and redistribute this program under the [LGPLv3](http://www.gnu.org/licenses/lgpl-3.0.txt) license terms.

### Templates:

SIPPing uses a simple templates mechanism based on Python dictionary-based string formatting.

For example using this template saved on file *test-template.txt*:

    OPTIONS sip:%(user)s@%(destination)s:%(port)s;line=kutixubf SIP/2.0
    Via: SIP/2.0/UDP 192.168.10.1:5060;branch=z9hG4bK001b84f6;rport
    Max-Forwards: 70
    From: "fake" <sip:fake@192.168.10.1>;tag=as2e95fad1
    To: <sip:%(user)s@%(destination)s:%(port)s;line=kutixubf>
    Contact: <sip:fake@192.168.10.1:5061>
    Call-ID: 7066d2f12e6f22ec1dc1231f4cade6be@172.16.18.40:5060
    User-Agent: SIPPing
    Date: Wed, 24 Apr 2013 20:35:23 GMT
    Allow: INVITE, ACK, CANCEL, OPTIONS, BYE, REFER, SUBSCRIBE, NOTIFY, INFO, PUBLISH
    Supported: replaces, timer

You need to define three variables: **user**, **destination** and **port**, for example using this command line:

    sipping.py -r test-template.txt -d 172.16.18.35 -p 5060 -S 172.16.18.90  -P 5061 -c 3 -vuser:120 -v destination:192.168.20.1 -v port:5060
    
* **-r test-template.txt** instruct the command to use the template file *test-template.txt*
* **-d 172.16.18.35** define the destination IP for this SIP request
* **-p 5060** define the destination port for this SIP request
* **-S 172.16.18.90** define the local source address used to send this SIP request
* **-P 5061** define the local source port
* **-c 3** define the number of requests to send out
* **-v user:120** define the template substitution variable **user** with value **120**
* **-v destination:192.168.20.1** define the template substitution variable **destination** with value **192.168.20.1**
* **-v port:5060** define the template substitution variable **port** with value **5060**

This command execution will sends out 3 SIP request to 172.16.18.35 using 5060 as a source address and 5061 as a source port following the full SIP request:

    OPTIONS sip:120@192.168.20.1:5060;line=kutixubf SIP/2.0
    Content-Length: 0
    Via: SIP/2.0/UDP 192.168.10.1:5060;branch=z9hG4bK001b84f6;rport
    From: "fake" <sip:fake@192.168.10.1>;tag=as2e95fad1
    Supported: replaces, timer
    User-Agent: SIPPing
    To: <sip:120@192.168.20.1:5060;line=kutixubf>
    Contact: <sip:fake@192.168.10.1:5061>
    CSeq: 0 OPTIONS
    Allow: INVITE, ACK, CANCEL, OPTIONS, BYE, REFER, SUBSCRIBE, NOTIFY, INFO, PUBLISH
    Call-ID: 7066d2f12e6f22ec1dc1231f4cade6be@172.16.18.40:5060
    Date: Wed, 24 Apr 2013 20:35:23 GMT
    Max-Forwards: 70
    
    OPTIONS sip:760.snomlabo.local:5060 SIP/2.0

As you can see **user**, **destination** and **port** format strings are substituted with command line defined variables. You can notice also that **Content-Length** and **CSeq** header are automatically generated, you can avoid the automatic generation of these headers defining its in template file.

Here the result of the command execution:

    sent Request OPTIONS to 172.16.18.35:5060 cseq=0
    received Response 200 OK from 172.16.18.35:5060 cseq=0
    sent Request OPTIONS to 172.16.18.35:5060 cseq=1
    received Response 200 OK from 172.16.18.35:5060 cseq=1
    sent Request OPTIONS to 172.16.18.35:5060 cseq=2
    received Response 200 OK from 172.16.18.35:5060 cseq=2
    
    --- statistics ---
    3 packets transmitted, 3 packets received, 0.0% packet loss

#### Runtime variables

SIPPing already defines some variables compiled at runtime using command line parameters:

* **dest_ip** destination IP address (extracted from the **-d** option )
* **dest_port** destination port (extracted from **-p** option)
* **source_ip** source ip address (extracted from **-S** option)
* **source_port** source port address (extracted from **-P** option)
* **seq** seq number (auto generated if CSeq header is missing from the template)

You don't need to define these variables with **-v** switch: values are already extracted from command line options.

#### Python dynamic variables

SIPPing supports also Python dynamic variables evaluated at runtime as a Python code.
You need to define these variables with a name starting with a **dot** (**.**) in command line switch.

Take a look on this example template that uses the **date** variable:

**Template file** *test-date.txt*

    OPTIONS sip:%(dest_ip)s:%(dest_port)s SIP/2.0
    Via: SIP/2.0/UDP %(source_ip)s:%(source_port)s
    Max-Forwards: 70
    From: "fake" <sip:fake@%(source_ip)s>
    To: <sip:%(dest_ip)s:%(dest_port)s>
    Contact: <sip:fake@%(source_ip)s:%(source_port)s>
    Call-ID: fake-id@%(source_ip)s
    User-Agent: SIPPing
    Date: %(date)s
    Allow: INVITE, ACK, CANCEL, OPTIONS, BYE, REFER, SUBSCRIBE, NOTIFY, INFO, PUBLISH
    Supported: replaces, timer 

As you can see the header **Date** is defined trough the variable **date**.

Using this command line:

    ./sipping.py -d 172.16.18.35 -S 172.16.18.90 -v '.date:time.strftime("%a, %d %b %Y %H:%M:%S GMT", time.gmtime())' -V -r test-date.txt

**Note that the date name starts with a dot (.).**

The **date** variable will be substitute with the Python code:
    
    time.strftime("%a, %d %b %Y %H:%M:%S +0000", time.gmtime())

The previous command will sends out this SIP request:

    OPTIONS sip:172.16.18.35:5060 SIP/2.0
    Content-Length: 0
    Via: SIP/2.0/UDP 172.16.18.90:5060
    From: "fake" <sip:fake@172.16.18.90>
    Supported: replaces, timer
    User-Agent: SIPPing
    To: <sip:172.16.18.35:5060>
    Contact: <sip:fake@172.16.18.90:5060>
    CSeq: 1 OPTIONS
    Allow: INVITE, ACK, CANCEL, OPTIONS, BYE, REFER, SUBSCRIBE, NOTIFY, INFO, PUBLISH
    Call-ID: fake-id@172.16.18.90
    Date: Thu, 25 Apr 2013 003024 +0000
    Max-Forwards: 70

As you can see the **Date** header has beer substituted with the generated date.
If in your python code you need an additional module you can dynamically include with the **-m** switch.

Here the list of available options:

    Options:
        -h, --help           show this help message and exit
        -c COUNT             Total number of queries to send
        -i WAIT              Specify packet send interval time in seconds
        -T TIMEOUT           Specify receiving timeout in seconds
        -v VAR               add a template variable in format varname:value
        -V                   be verbose dumping full requests / responses
        -q                   be quiet and never print any report
        -a                   aggressive mode: ignore any response
        -S SOURCE_IP         Specify ip address to bind for sending and receiving
                             UDP datagrams
        -P SOURCE_PORT       Specify the port number to use as a source port in UDP
                             datagrams
        -d DEST_IP           *mandatory* Specify the destination ip address
        -p DEST_PORT         *mandatory* Specify the destination port number
        -r REQUEST_TEMPLATE  Specify the request template file
        -t                   print the default request template
        -m MODULES           load python modules used in python interpreted template
                             variables
        -O OUT_REGEX         regex to apply to response received, (default '(.* )*')
        -R OUT_REPLACE       print this replace string applied to the response

## Examples

Here a list of examples and some SIP templates contained in the examples/ directory.
All following examples uses a snom phone as a target device and require these parameters configured on the phone:

* [user_sipusername_as_line](http://wiki.snom.com/wiki/index.php/Settings/ user_sipusername_as_line) (aka "Support for broken registrar") to "on"
* [filter_registrar](http://wiki.snom.com/wiki/index.php/Settings/filter_registrar) to "off"
* [network_id_port](http://wiki.snom.com/Settings/network_id_port): 5060

In my example command line I used these parameters:
    
* **151** is a valid sip account
* **172.16.18.35** is the phone IP
* **5060** is the phone [network_id_port](http://wiki.snom.com/Settings/network_id_port)
* **172.16.18.90** is the PC ip address
* **5061** is the PC source port

### re-register a snom phone

With this command you can force a re-registration of a snom phone.
Be aware that you need to reconfigure these parameters on the phone:

    sipping.py -r examples/snom-check-sync-register.txt -v user:151  -d 172.16.18.35 -p 5060 -S 172.16.18.90  -P 5061 -c1


### trigger up a check-sync on a snom phone (no reboot)

This command force a phone to synchronize its settings with the provisioning server

    sipping.py -r examples/snom-check-sync.txt -v user:151  -d 172.16.18.35 -p 5060 -S 172.16.18.90  -P 5060 -c1
    
### trigger up a check-sync on a snom phone (with reboot)

This command force a phone to reboot and synchronize its settings with the provisioning server

    sipping.py -r examples/snom-check-sync-reboot.txt -v user:151  -d 172.16.18.35 -p 5060 -S 172.16.18.90  -P 5060 -c1

### send a snom PnP multicast request

This command sends out a SUBSCRIBE for PnP provisioning

    sipping.py -d sip.mcast.net -p 5060 -S 172.16.18.91 -P 5060 -r examples/snom-pnp.txt -v model:snom720 -v mac:3C0754399E3D

### fire-up a minibrowser application on a snom phone

This command sends to the phone a minibrowser XML application.

This command requires the [xml_notify](http://wiki.snom.com/Settings/xml_notify) setting enabled on the phone.

    sipping.py -r examples/snom-notify-minibrowser.txt -d 172.16.18.35 -p 5060 -S 172.16.18.90  -P 5060 -c1 -a
    
**Note:** this example uses also the *%(seq)d* formatter

### snom phones led controls

With this example template you can turn on a phone button.
This command uses a Python variable (**callid**) in order to generate a random Call-ID SIP header.
You need to configure a function key as a button with number **1** (see **key** variable).

    ./sipping.py -a -q -d 172.16.18.35 -r examples/snom-led-on.txt -v user:202 -v key:1 -v .callid:"''.join(random.choice(string.ascii_lowercase + string.digits) for x in range(6))" -vcolor:$color -m string -m random -c1

With this command you can turn off the phone led:

    ./sipping.py -d 172.16.18.35 -r examples/snom-led-off.txt -v user:202 -v key:$key -v .callid:"''.join(random.choice(string.ascii_lowercase + string.digits) for x in range(6))" -m string -m random -c1 -a -q

More details about this protocol [here](http://wiki.snom.com/Category:HowTo:LED_Remote_Control)

## include an external file

This command uses a python evaluated variable to include an external file:

**file:** *examples/snom-notify-minibrowser-external.txt*

    NOTIFY sip:%(dest_ip)s:%(dest_port)s SIP/2.0
    Via: SIP/2.0/UDP %(source_ip)s:%(source_port)s
    From: <sip:fake@%(source_ip)s>;tag=2502
    To: <sip:fake@%(dest_ip)s>;tag=2502
    Call-ID: blablub@snom320xxx
    Max-Forwards: 70
    Event: xml
    Subscription-State: active;expires=30000
    Content-Type: application/snomxml
    
    %(appfile)s

**file:** *examples/app.xml*

    <?xml version="1.0" encoding="UTF-8"?>
    <SnomIPPhoneText>
     <Title>Loaded app</Title>
     <Prompt>Prompt Text</Prompt>
     <Text>
     This is a test application loaded from an external file
     </Text>
    </SnomIPPhoneText>

**Command executed:**

    ./sipping.py -d 172.16.18.35 -S 172.16.18.90 -v '.appfile:open("examples/app.xml", "r").read()'  -r examples/snom-notify-minibrowser-external.txt -c 1

**Request sent:**

    NOTIFY sip:172.16.18.35:5060 SIP/2.0
    Content-Length: 208
    Via: SIP/2.0/UDP 172.16.18.90:5060
    From: <sip:fake@172.16.18.90>;tag=2502
    Subscription-State: active;expires=30000
    To: <sip:fake@172.16.18.35>;tag=2502
    CSeq: 0 NOTIFY
    Max-Forwards: 70
    Call-ID: blablub@snom320xxx
    Content-Type: application/snomxml
    Event: xml
    
    <?xml version="1.0" encoding="UTF-8"?>
    <SnomIPPhoneText>
     <Title>Loaded app</Title>
     <Prompt>Prompt Text</Prompt>
     <Text>
     This is a test application loaded from an external file
     </Text>
    </SnomIPPhoneText>
    
## send contiuous REGISTER with incremental CSeq and random Call-ID headers

This example shows how to send out REGISTER requests every 3 seconds (-i 3) with incremental CSeq header and random generated Call-ID.

**file:** *examples/register.txt*

    REGISTER sip:%(dest_ip)s SIP/2.0
    Via: SIP/2.0/UDP %(source_ip)s:%(source_port)s;branch=z9hG4bK-p985iy;rport
    From: "fake" <sip:fake@%(source_ip)s>;tag=as2e95fad1
    To: <sip:%(user)s@%(dest_ip)s:%(dest_port)s;line=kutixubf>
    Contact: <sip:fake@%(source_ip)s:%(source_port)s>
    Call-ID: %(callid)s@fake
    CSeq: %(seq)d REGISTER
    Max-Forwards: 70
    Supported: path, outbound, gruu
    User-Agent: SIPPing fake UA
    Expires: 3600
    Content-Length: 0

**Command executed:**

    ./sipping.py -r examples/register.txt -d 172.16.18.35 -vuser:testuser -v .callid:"''.join(random.choice(string.ascii_lowercase + string.digits) for x in range(6))" -m string -m random -i 3 -S  172.16.18.90

**First request sent:**

    REGISTER sip:172.16.18.35 SIP/2.0
    Content-Length: 0
    Via: SIP/2.0/UDP 172.16.18.90:5060;branch=z9hG4bK-p985iy;rport
    From: "fake" <sip:fake@172.16.18.90>;tag=as2e95fad1
    Supported: path, outbound, gruu
    Expires: 3600
    User-Agent: SIPPing fake UA
    To: <sip:Snom@172.16.18.35:5060;line=kutixubf>
    Contact: <sip:fake@172.16.18.35:5060>
    CSeq: 1 REGISTER
    Call-ID: j8c5vl@fake
    Max-Forwards: 70
    
**Second request sent:**

    REGISTER sip:172.16.18.35 SIP/2.0
    Content-Length: 0
    Via: SIP/2.0/UDP 172.16.18.90:5060;branch=z9hG4bK-p985iy;rport
    From: "fake" <sip:fake@172.16.18.90>;tag=as2e95fad1
    Supported: path, outbound, gruu
    Expires: 3600
    User-Agent: SIPPing fake UA
    To: <sip:Snom@172.16.18.35:5060;line=kutixubf>
    Contact: <sip:fake@172.16.18.35:5060>
    CSeq: 2 REGISTER
    Call-ID: w6yj5g@fake
    Max-Forwards: 70

and so on ...

## Installation
You need to download the sipping.py script and run it:

    python sipping.py [options]
or

    chmod +x sipping.py
    ./sipping.py [options]

You can download SIPPing from the git master branch on [github](https://github.com/snom-it/SIPPing/archive/master.zip).

#### Requirements:
There is no particular requirements, this script runs on standard Python 2.X (tested with >= v2.4), no additional modules is required. Runs on GNU/Linux, MacOSX and Windows (untested at the moment).

## FAQ

* **Q:** when I run sipping.py I receive this error:

    **ERROR: error in template processing. unsupported format character 'X' (0xYZ) at index K**
    
* **R:** this means that you used a wrong format string: all format string must be wrote in this format: **%(NAME)s** where **NAME** is the variable name (pleas note the starting *%* char and the ending *s*). The only exception is for **%(seq)d** variable. You can read more information about dictionary based string formatting [here](http://www.diveintopython.net/html_processing/dictionary_based_string_formatting.html) and [here](http://docs.python.org/2/library/stdtypes.html#string-formatting).

* **Q:** when I run sipping.py I receive this error:

    **ERROR: missing template variable. 'var_name'**

* **A:** this means that a variable with name **var_name** is missing, you can declare via command line using the *-v var_name:value* switch.

[![Bitdeli Badge](https://d2weczhvl823v0.cloudfront.net/pbertera/SIPPing/trend.png)](https://bitdeli.com/free "Bitdeli Badge")
