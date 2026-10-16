# SOC Lab: Security Monitoring with Splunk

## Project overview

In this lab, I installed Splunk Enterprise and Splunk Universal Forwarder on Ubuntu, configured a receiving port, added a Linux authentication-log input, and used Splunk Search & Reporting to explore ingested events.

The goal was to understand how security logs become searchable in a SIEM. This is a foundational SOC project: collecting authentication activity provides the data analysts need before building detections or investigating incidents.

## Environment and tools

| Component | Purpose |
| --- | --- |
| Ubuntu virtual machines | Splunk server and forwarder environments described in the lab |
| Splunk Enterprise | Receive, index, and search events |
| Splunk Universal Forwarder | Monitor the authentication log and forward events |
| `/var/log/auth.log` | Linux authentication-event source |
| TCP 9997 | Configured Splunk receiving port |
| TCP 8000 | Splunk web interface |
| Linux terminal and `netstat` | Installation, configuration, and listener verification |

## Log flow

The intended two-VM design was: Ubuntu authentication log → Universal Forwarder → Splunk receiver on TCP 9997 → indexed events → Search & Reporting.

The evidence shows searchable authentication events. However, the forward-server screenshot records `127.0.0.1:9997`, a loopback destination. That screenshot alone does not establish forwarding to a different VM. The lab instructions' `192.168.80.130` address is an example, not a verified address for my Splunk server.

## Step-by-step implementation

Commands below explain the documented workflow. Package filenames and the receiver address are placeholders where the report does not consistently establish an exact value. They are not ready-to-run scripts.

### 1. Install Splunk Enterprise

I downloaded a Debian installation package on the Ubuntu server and installed it from the terminal. The server screenshot shows a Splunk 9.4.3 package; the worksheet also contains older 8.2.4 examples.

```text
cd ~/Downloads
sudo apt install ./<splunk-enterprise-package>.deb
cd /opt/splunk/bin
sudo ./splunk start --accept-license
```

Starting Splunk initialized the instance and prompted for an administrator account. Password entry was interactive; no password is included in this walkthrough. The command spelling above corrects a typo in the worksheet.

### 2. Access the Splunk web interface

After startup, I opened Splunk Web on port 8000 and accessed its administration interface. The lab also included enabling startup at boot:

```text
sudo /opt/splunk/bin/splunk enable boot-start
```

The web interface is used to configure the instance and search data; it is separate from the receiving port used by forwarders.

### 3. Configure the receiving port

In Splunk Web, I navigated to **Settings → Forwarding and receiving → Configure receiving**, added TCP port **9997**, and checked the listener from the terminal:

```bash
netstat -auntp | grep 9997
```

The screenshot shows that `netstat` was initially unavailable, so I installed `net-tools` before repeating the check. The resulting output showed `splunkd` listening on port 9997.

### 4. Install and initialize Universal Forwarder

On the forwarder Ubuntu environment, I installed the Universal Forwarder package and started it:

```text
sudo apt install ./<splunk-universal-forwarder-package>.deb
cd /opt/splunkforwarder/bin
sudo ./splunk start --accept-license
```

The forwarder collected local data for forwarding. Its configuration was managed through the command line rather than a Splunk Web interface.

### 5. Configure the forwarding destination

The lab procedure used this command format:

```text
sudo /opt/splunkforwarder/bin/splunk add forward-server <SPLUNK_SERVER_VM_IP>:9997
```

The actual screenshot shows `127.0.0.1:9997` and an “already present” response after authentication. This means that destination was already configured. Because it is loopback, it refers to the forwarder's own VM. The report does not show a later correction to a remote server address.

### 6. Monitor Linux authentication logs

I added `/var/log/auth.log` as a monitored input and assigned the sourcetype `linux_secure`:

```bash
sudo /opt/splunkforwarder/bin/splunk add monitor /var/log/auth.log -sourcetype linux_secure
```

The screenshot confirms that the monitor was added. The file path identifies what the forwarder reads, while the sourcetype labels the events for searching and processing.

The lab procedure also included starting the forwarder and enabling boot startup:

```bash
sudo /opt/splunkforwarder/bin/splunk enable boot-start
sudo /opt/splunkforwarder/bin/splunk start
```

### 7. Confirm that authentication events were searchable

In Search & Reporting, I used Data Summary to inspect the available sources and sourcetypes. The screenshots show `/var/log/auth.log` and `linux_secure`.

I explored events using:

```spl
index=*
```

Then narrowed the search to the authentication sourcetype:

```spl
index=* sourcetype=linux_secure
```

The wildcard searches matching indexes accessible to the user; it does not mean every internal index is automatically included. The selected time range also determines which events appear.

### 8. Read the event metadata

I examined the index, source, sourcetype, and Splunk server fields to understand where the events were stored and how they were labelled. The report records the `main` index and shows an authentication-filtered search with **24 events** for its selected time range.

Other screenshots and answers contain different event counts. These are snapshots from different searches or moments, so they are not presented as one consistent total.

## Skills demonstrated

- Installing and administering Splunk on Ubuntu
- Configuring a receiving port and checking a listening service
- Installing and configuring Universal Forwarder
- Adding a file-monitor input and assigning a sourcetype
- Searching authentication events with basic SPL
- Understanding index, source, sourcetype, time range, and server metadata
- Distinguishing demonstrated log ingestion from an unverified network path

This lab covers log collection and search. It does not claim completed alert rules, dashboards, incident investigations, or production deployment.

## Lab evidence

Screenshot publication is pending approval. The walkthrough above summarizes the completed lab evidence.

