# 🩸 Threat Investigation & Wireshark Analysis Report: TryHackMe Carnage

This comprehensive walkthrough report details the methodology, logical progression, and exact Wireshark filters used to analyze network packet traffic (`carnage.pcap`) following a malicious malspam event at Bartell Ltd.

The incident began when Eric Fischer from the Purchasing Department opened an infected Microsoft Word document attachment and explicitly allowed malicious macro execution by clicking **"Enable Content."** This documentation provides an interactive step-by-step framework to map out the attack chain from the initial landing stage to data exfiltration and Command and Control (C2) beaconing.

---

## 🛠️ Phase-by-Phase Forensic Breakdown

### Step 1: Locating and Opening the Packet Capture
1. Log into the analyst environment (`THM-CARNAGE-VM`).
2. Navigate to `/home/ubuntu/Desktop/Analysis/`.
3. Right-click the `carnage.pcap` file (size: ~53.4 MiB) and select **Open With "Wireshark"**.
4. ![carnage](../images/carnage1.png)

---

### Step 2: Uncovering the Initial Infection Vector (HTTP Download)

#### **Question 1: What was the date and time for the first HTTP connection to the malicious IP?**
* **Methodology:** Filter the packet landscape exclusively for HTTP protocol streams to identify inbound artifact requests.
* **Filter:** `http`
* **Logic:** Locate the earliest `GET` request. Frame **1735** shows the victim host `10.9.23.102` connecting to destination IP `85.187.128.24`. Select this packet and inspect the **Frame Detail** section under *Arrival Time*.
* **Answer Format:** `yyyy-mm-dd hh:mm:ss`
* **Value:** `2021-09-24 16:44:38`
*  ![carnage](../images/carnage2.png)
*   

#### **Question 2: What is the name of the zip file that was downloaded?**
* **Filter:** `http.request.method == "GET"`
* **Logic:** Analyze the URI path within Frame **1735** under the *Hypertext Transfer Protocol* section. The target object is explicitly requested at the end of the directory structure.
* **Value:** `documents.zip`
*  ![carnage](../images/carnage3.png)

#### **Question 3: What was the domain hosting the malicious zip file?**
* **Filter:** `http`
* **Logic:** Inspect the HTTP headers inside packet **1735**. Look at the **Host:** line under the Hypertext Transfer Protocol metadata layer.
* **Value:** `attirenepal.com`
 ![carnage](../images/carnage3.png)
---

### Step 3: Decompressing the Payload Layer in the TCP Stream

#### **Question 4: Without downloading the file, what is the name of the file in the zip file?**
* **Methodology:** Follow the active layer-4 TCP connection stream to extract embedded ASCII or hex strings belonging to the compressed archive structure.
* **Logic:** Right-click packet **1735** (or frame **2173**), select **Follow** $\rightarrow$ **TCP Stream**. Wireshark compiles `tcp.stream eq 73`. Scroll down into the raw payload data right where the standard PK zip magic bytes (`PK\x03\x04`) begin.
* **Value:** `chart-1530076591.xls`
*  ![carnage](../images/carnage4.png)
*   ![carnage](../images/carnage5.png)

#### **Question 5: What is the name of the webserver of the malicious IP from which the zip file was downloaded?**
* **Logic:** Look at the inbound server response headers inside the same TCP stream window (`Stream 73`). Search for the **Server:** parameter sent by the remote host.
* **Value:** `LiteSpeed`
*  ![carnage](../images/carnage6.png)

#### **Question 6: What is the version of the webserver from the previous question?**
* **Logic:** Locate the adjacent metadata header **X-Powered-By:** inside the response sequence.
* **Value:** `PHP/7.2.34`
*  ![carnage](../images/carnage6.png)

---

### Step 4: Identifying Multi-Stage Malware Distribution Domains

#### **Question 7: Malicious files were downloaded to the victim host from multiple domains. What were the three domains involved with this activity?**
* **Methodology:** Correlate DNS resolution sequences right after the initial zip file retrieval window to observe the multi-staged distribution infrastructure.
* **Filter:** `dns and frame.time >= "Sep 24, 2021 16:45:11" && frame.time <= "Sep 24, 2021 16:45:30"`
* **Logic:** Inspect the *Standard query A* requests traversing outward from `10.9.23.102` to the local DNS server `10.9.23.5`.
* **Value:** `finejewels.com.au`, `thietbiagt.com`, `new.americold.com`
 ![carnage](../images/carnage7.png)
 ![carnage](../images/carnage8.png)
 ![carnage](../images/carnage9.png)

#### **Question 8: Which certificate authority issued the SSL certificate to the first domain from the previous question?**
* **Filter:** `tls.handshake.type == 11` or locate the initial TCP session bound to `finejewels.com.au`'s resolved IP addresses.
* **Logic:** Right-click the packet containing the TLS handshake traffic interaction for the target domain and select **Follow** $\rightarrow$ **TCP Stream**. Scroll to the certificate definition layer block to identify the *Issuer Organization (O)* property.
* **Value:** `GoDaddy`
  ![carnage](../images/carnage10.png)
 ![carnage](../images/carnage11.png)
---

### Step 5: Pinpointing Cobalt Strike Command and Control (C2) Infrastructures

#### **Question 9: What are the two IP addresses of the Cobalt Strike servers?**
* **Methodology:** Use global threat intelligence pivoting. Go to Wireshark, navigate to **Statistics** $\rightarrow$ **Conversations** $\rightarrow$ **TCP** tab. Sort connections by volume or destination port. Extract suspicious target destination IPs and perform OSINT verification via the **VirusTotal Community Tab**.
* **Value:** `185.106.96.158`, `185.125.204.174`
   ![carnage](../images/carnage12.png)
  ![carnage](../images/carnage13.png)
  ![carnage](../images/carnage14.png)
  ![carnage](../images/carnage15.png)
  ![carnage](../images/carnage16.png)


#### **Question 10: What is the Host header for the first Cobalt Strike IP address from the previous question?**
* **Filter:** `ip.addr == 185.106.96.158 and http`
* **Logic:** Isolate the target endpoint's interactive traffic stream. Select the first web query frame, right-click $\rightarrow$ **Follow** $\rightarrow$ **TCP Stream** to inspect request attributes.
* **Value:** `ocsp.verisign.com`
*   ![carnage](../images/carnage17.png)

#### **Question 11: What is the domain name for the first IP address of the Cobalt Strike server?**
* **Methodology:** Paste the IP address `185.106.96.158` directly into VirusTotal. Pivot over to the **Relations** tab and isolate high-confidence active mapping domain historical links.
* **Value:** `survmeter.live`
*  ![carnage](../images/carnage18.png)

#### **Question 12: What is the domain name of the second Cobalt Strike server IP?**
* **Methodology:** Submit `185.125.204.174` into VirusTotal. Inspect the **Relations** and **Community** tabs to read analyst comments defining structural C2 footprints.
* **Value:** `securitybusinpuff.com`
*  ![carnage](../images/carnage19.png)

---

### Step 6: Post-Infection Communication Profiles

#### **Question 13: What is the domain name of the post-infection traffic?**
* **Methodology:** Filter exclusively for outbound asynchronous command beacons utilizing structural application data submissions via modern HTTP verbs.
* **Filter:** `http.request.method == "POST"`
* **Logic:** Look inside the frame collection (e.g., Frame **3822** onwards). Select a packet pointing to the post-infection destination infrastructure (`208.91.128.6`) and parse the underlying domain header context.
* **Value:** `maldivehost.net`
*   ![carnage](../images/carnage20.png)

#### **Question 14: What are the first eleven characters that the victim host sends out to the malicious domain involved in the post-infection traffic?**
* **Filter:** `http.request.method == "POST"`
* **Logic:** Inspect the absolute URI string structure initiated by the host machine to transmit telemetry. In packet **3822**, look right after the root indicator segment `/` inside the requested URI.
* **Value:** `zLIisQRWZI9`
*   ![carnage](../images/carnage21.png)
    ![carnage](../images/carnage22.png)

#### **Question 15: What was the length for the first packet sent out to the C2 server?**
* **Logic:** Look up the exact structural frame mapping properties inside your filtered view for the earliest post-infection packet transmission sequence (Frame **3822**). Check the **Length** column in the Wireshark packet list view.
* **Value:** `281`
*   ![carnage](../images/carnage23.png)

#### **Question 16: What was the Server header for the malicious domain from the previous question?**
* **Logic:** Select packet **3822** (or the corresponding return frame **3851**), right-click $\rightarrow$ **Follow** $\rightarrow$ **TCP Stream**. Examine the remote hosting server information attributes exposed within the header.
* **Value:** `Apache/2.4.49 (cPanel) OpenSSL/1.1.1l mod_bwlimited/1.4`
*   ![carnage](../images/carnage24.png)
* ![carnage](../images/carnage25.png)

---

### Step 7: Profiling Victim Host IP-Check and Reconnaissance Activity

#### **Question 17: The malware used an API to check for the IP address of the victim’s machine. What was the date and time when the DNS query for the IP check domain occurred?**
* **Methodology:** Filter specifically for DNS records tracking outbound connection queries to lookup generic, public IP location identification lookup engines.
* **Filter:** `dns.qry.name contains "api"` or `dns`
* **Logic:** Select the target frame resolving the external query endpoint and read its corresponding packet capture timestamp value.
* **Answer Format:** `yyyy-mm-dd hh:mm:ss UTC`
* **Value:** `2021-09-24 17:00:04`
*   ![carnage](../images/carnage26.png)

#### **Question 18: What was the domain in the DNS query from the previous question?**
* **Logic:** Inspect the inner **Queries** flag component of the parsed transaction frame identified in Question 17.
* **Value:** `api.ipify.org`
  ![carnage](../images/carnage27.png)
---

### Step 8: Malicious Spam (Malspam) Campaign Verification

#### **Question 19: Looks like there was some malicious spam (malspam) activity going on. What was the first MAIL FROM address observed in the traffic?**
* **Methodology:** Filter network protocol layer flags specifically tracking standard transactional email operations.
* **Filter:** `smtp`
* **Logic:** Isolate the earliest outbound envelope interaction phase configuration packet data stream. Locate the `MAIL FROM:` parameter context block.
* **Value:** `farshin@mailfa.com`
*   ![carnage](../images/carnage28.png)

#### **Question 20: How many packets were observed for the SMTP traffic?**
* **Filter:** `smtp`
* **Logic:** Type the protocol keyword string directly into the filter bar and press Enter. Look down at the bottom-right status corner bar interface to observe the exact calculation for **Displayed:** packets out of the capture total.
* **Value:** `1439`
*   ![carnage](../images/carnage29.png)

---
