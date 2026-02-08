**Chapter 3 Summary**

* **Date:** Tuesday, September 2.
* **The Investigation:** Bill, Wes, and Patty visit Brent Geller's workspace to investigate the payroll failure. Brent explains that the SAN (Storage Area Network) crash happened during a firmware upgrade that went wrong the previous night.
* **The Smoking Gun:** They discover that the payroll failure wasn't caused by the SAN crash. Ann from Finance reports that the payroll data is corrupted with gibberish in the Social Security number field,.
* **The Cause:** Brent recalls a developer named Max asking about database table structures for the timekeeping application. It turns out Max deployed a tokenization security patch at the urgent request of John Pesche (CISO) to fix an audit finding regarding PII (Personally Identifiable Information) storage,,.
* **Change Management Failure:** John admits to bypassing the standard change process because of an upcoming audit deadline, claiming he had no choice. He also reveals the change wasn't tested because there was no test environment available,.
* **Resolution:** The team identifies the bad change. The payroll system is fixed by 7 p.m., and the SAN is restored by 11 p.m..
* **Public Fallout:** The chapter ends with Bill seeing a news report that Parts Unlimited failed to pay its workers, causing a PR nightmare and union outrage.
* **Reflection:** Bill realizes the importance of proper change management and the need for a structured process to prevent future incidents.
