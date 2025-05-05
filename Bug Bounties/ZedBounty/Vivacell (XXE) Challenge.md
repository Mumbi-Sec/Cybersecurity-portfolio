## WhiteBox Penetration Test – Internal Web Application
This challenge demonstrates the XML external entity vulnerability that allows an attacker to interfere with an application's processing of XML data.

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

  ![Screenshot_2025-05-05_12_57_22](https://github.com/user-attachments/assets/536a72e0-9850-4853-a069-f9d369eae53b)



### Key Findings
**1. Access Control bypass**
    
- Access was denied to panel.php on live web server

![Screenshot_2025-05-05_15_56_47](https://github.com/user-attachments/assets/3be39ff4-0084-47eb-b2a5-064c451f87ef)
  
- Hardcoded credentials discovered via *panel.php* file.
  ![cred](https://github.com/user-attachments/assets/0e8ee05e-73ec-4911-9893-8ba698b11f81)

- Burpsuit for account takeover
  
    --> Intercept request with proxy
  
    --> Insert credentials and spoofed IP in HTTP Header.
  
    --> forward request
  
  ![Screenshot_2025-05-05_14_46_34](https://github.com/user-attachments/assets/878e17fb-3584-49c0-b823-fd9ab6d8b948)

  - Panel.php is now accessible
    
    ![Screenshot_2025-05-05_14_12_23](https://github.com/user-attachments/assets/992309f8-18b6-4d8f-baff-757cc84638d8)

**2. Vulnerability Discovery (XXE)**

Once the authentication barrier was bypassed, the application attempted to parse incoming XML payloads using the DOMDocument class with LIBXML_NOENT, which enables entity expansion — a classic vector for XXE attacks.

![XXE](https://github.com/user-attachments/assets/b945538f-799c-484e-be5f-ad3993572de9)





### Lessons Learned

