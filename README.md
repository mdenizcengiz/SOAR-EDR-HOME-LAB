# SOAR-EDR-HOME-LAB


### Lab Architecture and Overview
This hands-on security automation lab simulates a real-world incident response workflow using LimaCharlie EDR, Tines automation platform, Slack for team alerts, and email for escalation. I deployed a Windows VM in Azure, enabled RDP access via firewall rules, and installed the LimaCharlie agent to monitor host activity. I then created and tested custom detection rules for credential dumping tools like LaZagne.The detection events were forwarded in real-time to Tines via webhooks, where I built a dynamic automation story to parse payloads, alert through Slack and email, and prompt for analyst input. Based on user decisions, the automation either posted to Slack or triggered a live host isolation via LimaCharlie’s API. This lab demonstrates end-to-end SOC capabilities, including threat detection, alerting, incident triage, and automated containment.

### Project Summary
-Deployed a Windows 10 virtual machine in Azure and created a rule to enable secure RDP access for testing.

-Installed the LimaCharlie agent via PowerShell to enable endpoint telemetry and real-time visibility.

-Disabled Windows Defender’s real-time protection to allow execution of offensive security tools for testing purposes.

-Wrote and tested a custom detection rule to identify LaZagne.exe activity using LimaCharlie’s D&R engine.

-Validated detection performance through rule testing and observed alerts triggered by actual executions.

-Set up a Slack channel for team notifications and built a webhook listener in Tines to receive detections.

-Connected LimaCharlie to Tines using a real-time output stream, triggering workflow automation from incoming alerts.

-Parsed and inspected detection payloads in Tines, confirming field extraction such as file path, hash, command line, and user.

-Configured Slack and email actions in Tines to distribute alerts with enriched context to the appropriate teams.

-Added a manual decision prompt in Tines asking whether to isolate the host, simulating human desicion response control.

-Built a conditional flow where No sends an alert to Slack and Yes triggers an HTTP POST to LimaCharlie’s API to isolate the sensor.

-Successfully executed host isolation via API call and confirmed status change in the LimaCharlie dashboard.

-Implemented logic to fetch and report the final isolation result back into Slack for visibility.

-Built and published the complete automation pipeline, enabling real-time detection, triage, alerting, and remediation from end to end.

-Gained practical experience in EDR tuning, automation workflows, user-driven response, and multi-channel alert delivery.


### Steps
### 1. Create Firewall Rule for RDP Access
I created a new inbound NSG rule named SOAR-EDR-Firewall in Azure to allow RDP access on TCP port 3389. This lets me securely connect to the virtual machine from an authorized IP address using Remote Desktop.

![image alt](https://github.com/mdenizcengiz/SOAR-EDR-HOME-LAB/blob/1687d062a9dda07f48701db3b1e316af2c93f8a3/Screenshots/1-Create%20firewall%20rule%20for%20conection.png)
### 2. Install LimaCharlie Agent via PowerShell
I installed the LimaCharlie agent on the Windows VM using a command-line executable in PowerShell. The install finished without issues, and the host immediately began communicating with the LimaCharlie backend.

![image alt](https://github.com/mdenizcengiz/SOAR-EDR-HOME-LAB/blob/1687d062a9dda07f48701db3b1e316af2c93f8a3/Screenshots/2-install%20lima%20charlie%20from%20powershell.png)

### 3. Confirm Connection in LimaCharlie Console
After installation, I checked the LimaCharlie console and saw that the sensor was showing as active. It correctly reported the internal and external IPs, timestamp of enrollment, and its status as online and not isolated.

![image alt](https://github.com/mdenizcengiz/SOAR-EDR-HOME-LAB/blob/1687d062a9dda07f48701db3b1e316af2c93f8a3/Screenshots/3-%20lima%20charlie%20connection.png)

### 4: Disable Real-Time Protection on Windows Defender
I turned off real-time protection in Windows Defender. This is typically done in testing or red team simulations to prevent Defender from blocking tools like LaZagne during execution.

![image alt](https://github.com/mdenizcengiz/SOAR-EDR-HOME-LAB/blob/1687d062a9dda07f48701db3b1e316af2c93f8a3/Screenshots/4-%20Turn%20off%20virus%20protection%20on%20server.png)

### 5. View Detection Rule
I reviewed a Sigma-based detection rule written to catch executions to use for my own execution detection.

![image alt](https://github.com/mdenizcengiz/SOAR-EDR-HOME-LAB/blob/1687d062a9dda07f48701db3b1e316af2c93f8a3/Screenshots/6-Respond%20rules.png)

### 6. Confirm LimaCharlie Service is Running
I confirmed that the LimaCharlie agent was running as a Windows service and set to start automatically. This told me the agent was installed correctly and was continuously monitoring the system.

![image alt](https://github.com/mdenizcengiz/SOAR-EDR-HOME-LAB/blob/1687d062a9dda07f48701db3b1e316af2c93f8a3/Screenshots/7.0-LimaCharlie%20installed%20on%20server%20host.png)

### 7. Draft Custom Detection Rule in LimaCharlie
I wrote a custom detection rule directly in LimaCharlie to catch DeviceCredentialDeployment.exe. The rule looks for new process creation events with a matching file path.

![image alt](https://github.com/mdenizcengiz/SOAR-EDR-HOME-LAB/blob/1687d062a9dda07f48701db3b1e316af2c93f8a3/Screenshots/7.1-Create%20a%20detection%20rule.png)

### 8. Import Rule into LimaCharlie Detection & Response (D&R)
I imported that rule into LimaCharlie’s Detection & Response section. From there, I was able to edit the rule, set it to trigger a "report" response, and include details like tags and author info.

![image alt](https://github.com/mdenizcengiz/SOAR-EDR-HOME-LAB/blob/1687d062a9dda07f48701db3b1e316af2c93f8a3/Screenshots/7.2-import%20into%20%20limacharlie.png)

### 9. Execute LaZagne in PowerShell
To simulate a credential theft scenario, I ran LaZagne.exe through PowerShell. The tool decrypted and displayed saved system credentials — representing a typical post-exploitation step.

![image alt](https://github.com/mdenizcengiz/SOAR-EDR-HOME-LAB/blob/1687d062a9dda07f48701db3b1e316af2c93f8a3/Screenshots/8.1-set%20lazagna%20in%20powershell.png)

### 10. Review LaZagne Execution Logs in LimaCharlie
I checked the LimaCharlie logs and confirmed that it captured full telemetry from the LaZagne execution. The logs contained everything — file path, process ID, hash, memory usage, and parent process details — proving that the detection rule worked perfectly.

![image alt](https://github.com/mdenizcengiz/SOAR-EDR-HOME-LAB/blob/1687d062a9dda07f48701db3b1e316af2c93f8a3/Screenshots/8.2-Lazagna%20logs%20in%20Limacharlie.png)

### 11. Test Detection Rule
I created a detection rule in LimaCharlie to detect when the LaZagne.exe tool runs. I tested the rule using sample event data to ensure it correctly matched malicious behavior on Windows.

![image alt](https://github.com/mdenizcengiz/SOAR-EDR-HOME-LAB/blob/1687d062a9dda07f48701db3b1e316af2c93f8a3/Screenshots/9-Test%20the%20event%20.png)

### 12. Verify Detections in LimaCharlie
Once the rule was active, I confirmed it triggered correctly. LimaCharlie showed detections for the LaZagne.exe process, verifying that my rule logic worked.

![image alt](https://github.com/mdenizcengiz/SOAR-EDR-HOME-LAB/blob/1687d062a9dda07f48701db3b1e316af2c93f8a3/Screenshots/10-Generated%20Detections.png)
 
### 13. Create Slack Alerts Channel
I created a dedicated #alerts channel in Slack where detection notifications would be posted.

![image alt](https://github.com/mdenizcengiz/SOAR-EDR-HOME-LAB/blob/1687d062a9dda07f48701db3b1e316af2c93f8a3/Screenshots/11-Setup%20Slack.png)

### 14. Build Webhook Listener in Tines
I set up a Webhook in Tines to receive LimaCharlie detection data. This webhook acts as the entry point for my automated workflow.

![image alt](https://github.com/mdenizcengiz/SOAR-EDR-HOME-LAB/blob/1687d062a9dda07f48701db3b1e316af2c93f8a3/Screenshots/12-%20Establish%20connection%20with%20webhook%20to%20limacharlies.png)

### 15. Connect LimaCharlie Output to Tines
I created a new output stream in LimaCharlie and connected it to the Tines webhook using the detect stream. This ensures detection events are pushed to Tines in real time.

![image alt](https://github.com/mdenizcengiz/SOAR-EDR-HOME-LAB/blob/1687d062a9dda07f48701db3b1e316af2c93f8a3/Screenshots/13-Connect%20it%20from%20output%20in%20limacharlie.png)

### 16. Verify Event Reception in Tines
After sending test events, I confirmed that Tines successfully received detection data from LimaCharlie.

![image alt](https://github.com/mdenizcengiz/SOAR-EDR-HOME-LAB/blob/1687d062a9dda07f48701db3b1e316af2c93f8a3/Screenshots/14.0-Detect%20the%20event.png)

### 17. Inspect Detection Payloads in Tines
I reviewed the event payload in Tines to verify it contained all expected data like COMMAND_LINE, HASH, FILE_PATH, and USER_NAME.

![image alt](https://github.com/mdenizcengiz/SOAR-EDR-HOME-LAB/blob/1687d062a9dda07f48701db3b1e316af2c93f8a3/Screenshots/14.1-Verify%20the%20event%20in%20Tines.png)

### 18. Connect Tines to Slack
I built a Tines workflow to send a Slack message to #alerts whenever a detection is received.

![image alt](https://github.com/mdenizcengiz/SOAR-EDR-HOME-LAB/blob/1687d062a9dda07f48701db3b1e316af2c93f8a3/Screenshots/15-Check%20connection%20between%20tines%20and%20slack.png)

### 19. Test Slack Alert Delivery
I tested the Slack step and verified that messages from Tines successfully appeared in the #alerts channel, confirming the connection.

![image alt](https://github.com/mdenizcengiz/SOAR-EDR-HOME-LAB/blob/1687d062a9dda07f48701db3b1e316af2c93f8a3/Screenshots/16-Verify%20connection%20on%20slack.png)

### 20. Configure Email Alerts
Finally, I added an email action to the same Tines story to send detection alerts to a disposable inbox for testing purposes.

![image alt](https://github.com/mdenizcengiz/SOAR-EDR-HOME-LAB/blob/1687d062a9dda07f48701db3b1e316af2c93f8a3/Screenshots/17-Connect%20email%20to%20square%20disposible%20inbox.png)

### 21. Fetch Detection Details into Slack Alerts
I created the Slack alert messages by adding key detection fields such as time, source IP, file path, command line, sensor ID, and the detection link.

![image alt](https://github.com/mdenizcengiz/SOAR-EDR-HOME-LAB/blob/1687d062a9dda07f48701db3b1e316af2c93f8a3/Screenshots/18-Fetch%20important%20fields%20into%20slack.png)

### 22. Format Email Alerts with Detection Data
I configured the email body to include the same detailed detection data . This ensures email recipients get alerts.

![image alt](https://github.com/mdenizcengiz/SOAR-EDR-HOME-LAB/blob/1687d062a9dda07f48701db3b1e316af2c93f8a3/Screenshots/19-Fetch%20important%20fields%20into%20Email.png)

### 23. Execute Host Isolation with LimaCharlie API
After enabling LimaCharlie credentials in Tines, I successfully made a POST request to isolate the compromised sensor, returning a 200 status response from the API.

![image alt](https://github.com/mdenizcengiz/SOAR-EDR-HOME-LAB/blob/1687d062a9dda07f48701db3b1e316af2c93f8a3/Screenshots/21-Get%20200%20for%20isolate%20request%20with%20limacharlie%20credentials.png)

### 24. Confirm Isolation in LimaCharlie Dashboard
I verified that the sensor’s network status changed to "Isolated" in the LimaCharlie dashboard, confirming that the API action was successful.

![image alt](https://github.com/mdenizcengiz/SOAR-EDR-HOME-LAB/blob/1687d062a9dda07f48701db3b1e316af2c93f8a3/Screenshots/22-Server's%20isolated%20connection.png)

### 25. Build Full Automation Pipeline
I finalized the complete automation story in Tines by receiving detections via webhook and prompted user for isolation.Based on user input either send slack alert or isolated the sensor.

![image alt](https://github.com/mdenizcengiz/SOAR-EDR-HOME-LAB/blob/1687d062a9dda07f48701db3b1e316af2c93f8a3/Screenshots/23-Entire%20Automation%20Flow.png)

### 26. Verify Isolation Status in Slack
I fetched the isolation status post-action and sent the result back to the #alerts Slack channel to finish the process.

![image alt](https://github.com/mdenizcengiz/SOAR-EDR-HOME-LAB/blob/1687d062a9dda07f48701db3b1e316af2c93f8a3/Screenshots/24-Final%20Isolation%20Status.png)




