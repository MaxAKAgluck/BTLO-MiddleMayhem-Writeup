# BTLO-MiddleMayhem-Writeup
Middle Mayhem investigation BTLO Writeup

Description:

'''The security team at MiddleMayhem Inc. has detected unusual network traffic to their admin portal, but no security breaches have been confirmed. Your SOC team has been provided with SIEM logs from the incident. Analyze the attack pattern to determine how attackers bypassed authentication, gained remote code execution, and moved laterally through the network. '''

We are given a linux lab machine with access to splunk:

<img width="2551" height="892" alt="image" src="https://github.com/user-attachments/assets/8fa4454a-dab2-4427-a490-5610abc2ba07" />

1. Access the Website in the browser, presented in the bookmark, and identify the JavaScript framework and version used.

This is very easy - you don't have to peform any source code analysis  - just scroll to the bottom and along the copyright we see the JS framework (not the best security practice :( ).

2. Using Splunk, Find the attacker’s IP address

Next we dive into logs:

<img width="802" height="108" alt="image" src="https://github.com/user-attachments/assets/71d3da85-e7e1-483a-862d-0c07ab2655e7" />

Luckily where are only 2 sources, unlucky that one of them has 88000 entries...

The auth source type shows logs for ssh login attempts on the "dbserv" and the network logs are basically csv logs of network data.

Quickly scrolling through auth logs I noticed lots of requests from 172.217.164.174 which seem like someone was trying to brute force users on the server, this is potentially the answer.

If we filter for src_ip in network logs:

<img width="797" height="399" alt="image" src="https://github.com/user-attachments/assets/74e8b473-31e0-4262-a554-44053baa7a99" />

Maybe the first ip is the answer, but if look closely and try to filter for "*admin*" since the admin portal was mentioned in task:

<img width="796" height="126" alt="image" src="https://github.com/user-attachments/assets/3a251ddd-2e53-4a62-aee4-32b8a53b1d82" />

We find 2 destination IPs and the first one is actually the server!

The answer is src ip that made about 2100 requests and tried to enumerate the web server pages:

<img width="2186" height="401" alt="image" src="https://github.com/user-attachments/assets/a247ce17-17e2-4071-9fee-6eb3dbba4de7" />


3. Analyze the SIEM logs to determine how many unique URIs were accessed by the attacker

Use the stats command:

'''index=* sourcetype="network" ip_src="218.92.0.204" | stats count by http_request_uri'''

The answer is shown in statistics: 9930

4. Explore the site and identify two specific locations that could reveal internal structures or potential access points not meant for public eyes. Provide the two relative URLs

After looking at source code for comments or hints I visited robots.txt and here we go:

<img width="317" height="94" alt="image" src="https://github.com/user-attachments/assets/faa78b2f-3525-4111-a408-fe5323983b84" />

5. Based on the Framework and Version, what recent CVE could be used to bypass authorization? 

Googling "next js 15.0.0 cve auth bypass" we find the answer right away:

<img width="1203" height="615" alt="image" src="https://github.com/user-attachments/assets/7a2031da-a3bc-4366-a9ac-1f39f2a08a71" />

6. Find the relevant HTTP header in the SIEM logs that indicates CVE exploitation. Provide the header name.

Again, reading the first article:

<img width="1217" height="242" alt="image" src="https://github.com/user-attachments/assets/ad725e55-70e7-404d-ae2f-3a29d7628625" />

7. What interesting URI did the attacker access after exploiting the CVE

Filtering for events after the time he used that http middleware header:

<img width="2154" height="193" alt="image" src="https://github.com/user-attachments/assets/c31be092-342e-4c78-89ad-558cd32a7ce9" />

Attacker uploaded a shell.sh to /api-upload

8. The attacker tried uploading a reverse shell. Find out the IP and port to which the target would connect once the connection is established. 

Found in previous question.

9. After compromising the WebApp server, the attacker attempted lateral movement. Identify the technique used, as recorded in the SIEM logs

This is the ssh bruteforce I found earlier.

10. Identify the user account that achieved successful lateral movement to another server.

Looking at most recent entries:

<img width="1347" height="71" alt="image" src="https://github.com/user-attachments/assets/55c30a8c-854d-4c9e-a5d7-b706e2855e84" />

Overall, an easy investigation, about 30-60 minutes of time only, but an interesting CVE, I didn't know about that at all.

