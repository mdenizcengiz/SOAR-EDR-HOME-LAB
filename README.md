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

### 2. Install LimaCharlie Agent via PowerShell
I installed the LimaCharlie agent on the Windows VM using a command-line executable in PowerShell. The install finished without issues, and the host immediately began communicating with the LimaCharlie backend.

### 3. Confirm Connection in LimaCharlie Console
After installation, I checked the LimaCharlie console and saw that the sensor was showing as active. It correctly reported the internal and external IPs, timestamp of enrollment, and its status as online and not isolated.

### 4: Disable Real-Time Protection on Windows Defender
I turned off real-time protection in Windows Defender. This is typically done in testing or red team simulations to prevent Defender from blocking tools like LaZagne during execution.

### 5. View Detection Rule for DeviceCredentialDeployment
I reviewed a Sigma-based detection rule written to catch executions to use for my own execution detection.

### 6. Confirm LimaCharlie Service is Running
I confirmed that the LimaCharlie agent was running as a Windows service and set to start automatically. This told me the agent was installed correctly and was continuously monitoring the system.

### 7. Draft Custom Detection Rule in LimaCharlie
I wrote a custom detection rule directly in LimaCharlie to catch DeviceCredentialDeployment.exe. The rule looks for new process creation events with a matching file path.

### 8. Import Rule into LimaCharlie Detection & Response (D&R)
I imported that rule into LimaCharlie’s Detection & Response section. From there, I was able to edit the rule, set it to trigger a "report" response, and include details like tags and author info.

### 9. Execute LaZagne in PowerShell
To simulate a credential theft scenario, I ran LaZagne.exe through PowerShell. The tool decrypted and displayed saved system credentials — representing a typical post-exploitation step.

### 10. Review LaZagne Execution Logs in LimaCharlie
I checked the LimaCharlie logs and confirmed that it captured full telemetry from the LaZagne execution. The logs contained everything — file path, process ID, hash, memory usage, and parent process details — proving that the detection rule worked perfectly.

### 11. Test Detection Rule
I created a detection rule in LimaCharlie to detect when the LaZagne.exe tool runs. I tested the rule using sample event data to ensure it correctly matched malicious behavior on Windows.

### 12. Verify Detections in LimaCharlie
Once the rule was active, I confirmed it triggered correctly. LimaCharlie showed detections for the LaZagne.exe process, verifying that my rule logic worked.
 
### 13. Create Slack Alerts Channel
I created a dedicated #alerts channel in Slack where detection notifications would be posted.

### 14. Build Webhook Listener in Tines
I set up a Webhook in Tines to receive LimaCharlie detection data. This webhook acts as the entry point for my automated workflow.

### 15. Connect LimaCharlie Output to Tines
I created a new output stream in LimaCharlie and connected it to the Tines webhook using the detect stream. This ensures detection events are pushed to Tines in real time.

### 16. Verify Event Reception in Tines
After sending test events, I confirmed that Tines successfully received detection data from LimaCharlie.

### 17. Inspect Detection Payloads in Tines
I reviewed the event payload in Tines to verify it contained all expected data like COMMAND_LINE, HASH, FILE_PATH, and USER_NAME.

### 18. Connect Tines to Slack
I built a Tines workflow to send a Slack message to #alerts whenever a detection is received.

### 19. Test Slack Alert Delivery
I tested the Slack step and verified that messages from Tines successfully appeared in the #alerts channel, confirming the connection.

### 20. Configure Email Alerts
Finally, I added an email action to the same Tines story to send detection alerts to a disposable inbox for testing purposes.

### 21. Fetch Detection Details into Slack Alerts
I created the Slack alert messages by adding key detection fields such as time, source IP, file path, command line, sensor ID, and the detection link.

### 22. Format Email Alerts with Detection Data
I configured the email body to include the same detailed detection data . This ensures email recipients get alerts.

### 23. Add a User Prompt for Manual Action
I added a user prompt step in the automation workflow to ask for isolation. This adds approval for remediation action. I created conditional flow to trigger a logic. If user selects no it sents slack notification. If yes machine is isolated using limacharlie api.

### 24. Execute Host Isolation with LimaCharlie API
After enabling LimaCharlie credentials in Tines, I successfully made a POST request to isolate the compromised sensor, returning a 200 status response from the API.

### 25. Confirm Isolation in LimaCharlie Dashboard
I verified that the sensor’s network status changed to "Isolated" in the LimaCharlie dashboard, confirming that the API action was successful.

### 26. Build Full Automation Pipeline
I finalized the complete automation story in Tines by receiving detections via webhook and prompted user for isolation.Based on user input either send slack alert or isolated the sensor.

### 27. Verify Isolation Status in Slack
I fetched the isolation status post-action and sent the result back to the #alerts Slack channel to finish the process.




