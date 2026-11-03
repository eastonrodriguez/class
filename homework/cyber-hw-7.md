# IT HW 8 - Data Center Risk Assessment & Remediation Roadmap
**Client:** Meridian Financial Services 
**By:** Weston Romero, IT Infrastructure Consultant | **Date:** November 3, 2026
**Based on:** The Meridian Data Center Site Survey Findings 

---

## Part 1 - Executive Summary

Meridian's data center runs core banking, trading, and customer data, but it isn't well protected against everyday problems. The generator supplies 200 kW, but the building uses 306 kW. The battery backups are rated 160 kVA combined, but the servers draw 180 kW. All internet traffic goes through one provider and one firewall, and nobody is on site for a reasonable amount of hours. The server management cards still use factory passwords and sit on the same network as production, so one hacked server could give an attacker control of the hardware. An audit has already flagged availability and physical security. Putting all this into account it's important to remember downtime is expensive. A 2024 ITIC survey found that an hour of downtime costs over $300,000 for more than 90% of mid-size and large companies, and $1M-$5M+ for 41% of them (not to mention fines or lawsuits that could also stack up from this). That means that one 4 hour power outage can cost somewhere from $1M-$4M depending on the situation. Because of all of these issues my recommendations are first to fix the power system so either power path can carry the full load (about $400K-$1.0M), second add a second internet provider ($10K-$100K), and third make sure to finish the free security fixes in 30 days to be as cost effect as possible.

## References and Assumptions

1. ITIC, *2024 Hourly Cost of Downtime Survey*: https://calyptix.com/wp-content/uploads/ITIC-2024-Hourly-Cost-of-Downtime-Survey-Results-Part-2.pdf
2. Riello UPS, MTBF/MTTR/availability: https://www.riello-ups.com/questions/52-how-is-ups-resilience-measured-mtbf-mttr-availability
3. Eaton, "Maximizing UPS Availability": https://www.voltimum.co.uk/pdf-catalogues/eaton-maximizing-ups-availability?page=2
4. NREL, *Emergency Diesel Generator Reliability and Installation Energy Security*, NREL/TP-5C00-76553 (2020): https://www.nrel.gov/docs/fy20osti/76553.pdf
5. Data Center Knowledge, "Generators Are Key to Improving Reliability": https://www.datacenterknowledge.com/node/60676
6. ASHRAE TC 9.9 recommended inlet range: 18-27°C (64.4-80.6°F)
7. Uptime Institute tier availability targets from the assignment tip
8. Meridian Financial Services Data Center Site Survey Findings


