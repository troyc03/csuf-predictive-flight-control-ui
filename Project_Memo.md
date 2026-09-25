# Project Domain Summary

1. **Who is the customer/domain?**  
Our initial customer is Josh, co-owner of an aviation livestream channel. The broader domain is **small and mid-sized live broadcasters that cover real-world activity with useful live data**, such as aviation, motorsports, wildlife, and racing.

2. **What is the problem?**  
These broadcasters often have useful live data—speed, location, altitude, lap time, distance, etc.—but no simple way to automatically turn it into useful on-screen graphics during a livestream.

3. **Where is the money?**  
The stronger opportunity is **new advertising revenue**, not just cost savings. Live metrics could become sponsorable overlays, such as “Live telemetry presented by [Sponsor].” We still need to validate what broadcasters and sponsors would actually pay.

4. **How is it handled today?**  
Broadcasters often use static graphics, manual updates, custom scripts, APIs, and broadcast software like OBS or vMix. The process is fragmented and may require technical setup for each data source.

5. **What already exists?**  
Live graphics tools already exist, so our project is not just “dynamic graphics.” The gap we are investigating is a system that connects **different live data sources to automated graphics and sponsor placements** without requiring a custom solution each time.

6. **Why is this a real engineering project?**  
The project must ingest live data, normalize different formats, handle latency or missing data, trigger graphics automatically, and integrate reliably with live broadcast software. The challenge is the real-time data pipeline, not just the visual interface.

7. **What is still unknown?**  
We still need to determine whether broadcasters outside aviation have the same problem, whether sponsors will pay for metric-based overlays, how much those placements are worth, whether APIs are affordable and legally usable, and whether the system can maintain acceptable latency and reliability.
