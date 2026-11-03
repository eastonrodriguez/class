# IT HW 8 - Data Center Risk Assessment & Remediation Roadmap
**Client:** Meridian Financial Services 
**By:** Weston Romero, IT Infrastructure Consultant | **Date:** November 3, 2026
**Based on:** The Meridian Data Center Site Survey Findings 

---

## Part 1 - Executive Summary

Meridian's data center runs core banking, trading, and customer data, but it isn't well protected against everyday problems. The generator supplies 200 kW, but the building uses 306 kW. The battery backups (UPS) are rated 160 kVA combined, but the servers draw 180 kW. All internet traffic goes through one provider and one firewall, and nobody is on site 64% of the week (108 of 168 hours). The server management cards still use factory passwords and sit on the same network as production, so one hacked server could give an attacker control of the hardware. An audit has already flagged availability and physical security.

Downtime is expensive. The 2024 ITIC survey found that an hour of downtime costs over $300,000 for more than 90% of mid-size and large companies, and $1M-$5M+ for 41% (not counting fines or lawsuits). Using $300K/hour (low) and $1M/hour (high), one 4-hour power outage costs **$1.2M-$4.0M**. **My top recommendation:** fix the power system so either power path can carry the full load (about $400K-$1.0M), add a second internet provider ($10K-$100K), and finish the free security fixes in 30 days. Avoiding one 4-hour outage pays for the power fix. The roadmap cuts the total risk score from 217 to 65 (down 70%) and removes every Critical risk.
