# Splunk Log Ingestion and VPN Log Analysis

**TryHackMe, Splunk: Basics (SOC Level 1 Path) | July 2026**

Taking a raw JSON log file, onboarding it into Splunk Enterprise properly, and then querying it with SPL to answer investigation questions.

**[Read the full report with all 14 screenshots (PDF)](Splunk-VPN-Log-Analysis.pdf)** for the original document exactly as written, with every figure.

---

## Project title

Ingesting and analysing VPN logs in Splunk Enterprise: data upload, custom index creation, and SPL investigation.

## Goal

The goal of this lab was to practise the full data onboarding process in Splunk and then use SPL (Search Processing Language) to answer investigation questions from the ingested data. The dataset was a JSON file of VPN connection logs containing 2,862 events with fields like username, source IP, source country, port, protocol, and action.

The lab was designed to test whether I could take a raw log file, get it into a SIEM correctly, and then query it to pull out specific facts an analyst would need during an investigation, such as how many events came from a particular user or IP address.

## Tools used

- Splunk Enterprise 8.2.6 (SIEM platform, web interface)
- SPL (Search Processing Language), including the `spath` and `stats` commands
- TryHackMe AttackBox (browser based Linux analyst workstation)
- Firefox, to access the Splunk web interface on the lab machine
- JSON formatted VPN log dataset (`VPNlogs.json`, 2,862 events)

## What I did

Started the lab environment and confirmed both machines were up: the AttackBox as my analyst workstation and the target lab machine running Splunk. I connected to the Splunk web interface through Firefox using the lab machine's IP.

Opened **Settings > Add Data** and chose the Upload method, since the VPN log file was stored locally on the AttackBox. Browsed to `/root/Rooms/SplunkBasic/` and selected `VPNlogs.json` as the data source.

Checked the Set Source Type stage. Splunk auto detected the file as `_json` and the event preview showed each log line breaking cleanly into one event with correct timestamps, so I accepted the detected source type.

On Input Settings, created a new custom index called `VPN_Logs` instead of sending the data to the default index. Splunk stores index names in lowercase, so it saved as `vpn_logs`. I selected it as the destination index.

Reviewed the full configuration (file name, source type, host, index) and submitted the upload. Splunk confirmed the file was uploaded successfully.

Opened Search and Reporting, set the time picker to **All time** (the logs were from January 2022, so the default 24 hour window would have returned nothing), and verified the ingestion with a stats count search.

Ran a series of SPL queries using `spath` to parse the JSON fields, then filtered and counted events by username, source IP, and source country to answer all five investigation questions.

## Investigation summary

The first thing I confirmed was that the ingestion itself was clean. Before answering any questions, I ran a simple count against the index:

```
index=vpn_logs
| stats count
```

This returned 2,862 events, which matched the size of the source file. **That check matters because if the event count is wrong at this stage, every answer after it would be wrong too.** Splunk had parsed the JSON properly, but I still added the `spath` command to my searches to make sure the JSON fields were extracted reliably before filtering on them.

From there, each question was a filtering exercise. To count events for the user Maleena, I piped the parsed events through a search filter and counted the results (60 events).

To identify who was behind the IP `107.14.182[.]38`, I flipped the logic. Instead of filtering by user, I filtered by the IP and asked Splunk to return the usernames seen with it, using `stats values(UserName)`. That returned a single user, Smith, across 26 events. That is a useful sanity check, because if multiple usernames had come back for one IP, that itself would be worth investigating in a real environment.

For the country based question I used the not equals operator (`Source_Country!="France"`) to count every event that did not originate from France, which returned 2,814. Subtracting from the total (2,862 minus 2,814 = 48) also told me exactly how many events did come from France, which is the kind of quick cross check I try to build into every search. The final query counted events for the IP `107.3.206[.]58` and returned 14.

The main decision points in this lab were: accepting the auto detected `_json` source type only after visually confirming the event breaks and timestamps in the preview, creating a dedicated index rather than polluting the default one, and setting the search window to All time before trusting any results. None of these are big dramatic decisions, but each one is a place where skipping the check produces wrong or empty results.

## Results and findings

- The VPN log file was ingested successfully into a dedicated custom index (`vpn_logs`) with all 2,862 events parsed and searchable
- Total events in the dataset: 2,862, verified against the source file
- The user Maleena generated 60 VPN log events
- The IP address `107.14.182[.]38` was associated with a single username, Smith, across 26 events
- 2,814 events originated from countries other than France, meaning 48 events came from France
- The IP address `107.3.206[.]58` appeared in 14 VPN events
- All five investigation questions were answered correctly on the first attempt, and the room was completed 100%

## Skills demonstrated

- SIEM data onboarding and log ingestion (Splunk Enterprise)
- Custom index creation and data source configuration
- Source type validation and JSON event parsing with `spath`
- SPL query writing: search filtering, `stats count`, `stats values`, negation operators
- Log analysis and event correlation, mapping users to IPs
- Data validation and quality checks before analysis
- Attention to detail: time range scoping, index naming behaviour
- Security investigation methodology and documentation

## What I learned

This lab taught me that getting data into a SIEM correctly is half the investigation. My queries only worked because the ingestion steps before them were done carefully: the right source type, a dedicated index, and confirmation that the event count matched the file.

I also learned two small but practical things about Splunk that I will not forget. Index names are forced to lowercase no matter how you type them, and the default 24 hour time window will silently hide older data. If I had not switched to All time, my first search would have returned zero events and I might have assumed the upload failed.

On the query side, I got more comfortable with `spath` for JSON data and with using `stats values()` to pivot a question around, asking which user is behind this IP instead of which IPs did this user come from. That pivot is something I expect to use constantly in real SOC work, especially during account compromise investigations.

Compared to my Splunk home lab, where I ingest live Windows Event Logs through a forwarder, this lab filled a gap: manually uploading a static file and controlling every stage of the onboarding wizard myself.

## Evidence: investigation screenshots

The screenshots below follow the same order as the steps described above, from environment setup through data onboarding to the final SPL queries.

