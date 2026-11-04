# IT HW 8 - Data Center Risk Assessment & Remediation Roadmap
**Client:** Meridian Financial Services 
**By:** Weston Romero, IT Infrastructure Consultant | **Date:** November 3, 2026
**Based on:** The Meridian Data Center Site Survey Findings 

---

## Part 1 - Executive Summary

Meridian's data center runs core banking, trading, and customer data, but it isn't well protected against everyday problems. The generator supplies 200 kW, but the building uses 306 kW. The battery backups are rated 160 kVA combined, but the servers draw 180 kW. All internet traffic goes through one provider and one firewall, and nobody is on site for a reasonable amount of hours. The server management cards still use factory passwords and sit on the same network as production, so one hacked server could give an attacker control of the hardware. An audit has already flagged availability and physical security. Putting all this into account it's important to remember downtime is expensive. A 2024 ITIC survey found that an hour of downtime costs over $300,000 for more than 90% of mid-size and large companies, and $1M-$5M+ for 41% of them (not to mention fines or lawsuits that could also stack up from this). That means that one 4 hour power outage can cost somewhere from $1M-$4M depending on the situation. Because of all of these issues my recommendations are first to fix the power system so either power path can carry the full load (about $400K-$1.0M), second add a second internet provider ($10K-$100K), and third make sure to finish the free security fixes in 30 days to be as cost effect as possible.

---

## Part 2 - Risk Register

**Scoring:** Likelihood (1-5) × Impact (1-5). Critical = 15+, High = 10-14, Medium = 5-9, Low = 1-4. **Cost:** Low (<$10K), Medium ($10K-$100K), High (>$100K). Costs are rough estimates.

| ID | Category | Description | L | I | Score | Current Controls | Recommended Control | Cost |
|---|---|---|---|---|---|---|---|---|
| R-01 | Power | Generator is 200 kW but the facility uses 306 kW | 4 | 5 | 20 (Critical) | One generator, 72 h fuel | Replace with N+1 generators sized for 306 kW at 80% load or less | High |
| R-02 | Power | Two 80 kVA UPS units (160 kVA) can't carry the 180 kW IT load | 4 | 5 | 20 (Critical) | Two UPS units, A/B feeds | Resize so either side carries the full load | High |
| R-03 | Power | Only one utility feed | 3 | 5 | 15 (Critical) | Generator | Add a second feed or a backup site | High |
| R-04 | Power | Servers are single-corded, so A/B power gives no real backup. UPS runtime isn't documented | 3 | 4 | 12 (High) | Two PDUs per row | Rack-level transfer switches now, dual power supplies later | Medium |
| R-05 | Operational | No spare parts on site (UPS parts take a day, CRAC compressor parts take 48-72 h) | 3 | 4 | 12 (High) | Vendor contract | Stock a spare UPS module and compressor kit | Medium |
| R-06 | Cooling | Rows 5-8 have no containment; the 75°F average inlet is only 5.6°F under the 80.6°F limit and can hide hot spots | 4 | 3 | 12 (High) | Containment on rows 1-4 | Extend containment, add blanking panels and sensors | Medium |
| R-07 | Network | One internet provider, no BGP | 3 | 5 | 15 (Critical) | None | Second provider on a separate path with BGP | Medium |
| R-08 | Network | One firewall, no HA | 2 | 5 | 10 (High) | None | Add a second firewall in HA | Medium |
| R-09 | Network | iDRAC cards use the default `root`/`calvin` login plus a weak `svc-monitor` password | 5 | 5 | 25 (Critical) | None | Change both passwords now | Low |
| R-10 | Network | iDRAC is on the production network with IPMI over LAN and SNMP v1 ("public") enabled | 4 | 5 | 20 (Critical) | None | Own management network behind a firewall; turn off IPMI over LAN; use SNMPv3 | Low |
| R-11 | Network | iDRAC firmware is 4 years old (2.10.10.10; current is 6.10.30.20) | 4 | 4 | 16 (Critical) | None | Upgrade firmware and review it quarterly | Low |
| R-12 | Physical | No cameras in the server room, no rack doors, any badged employee can escort visitors, and no media destruction log | 3 | 4 | 12 (High) | Badge + PIN, mantrap, entrance cameras | Cameras, lockable racks, trained escorts only, destruction log | Medium |
| R-13 | Operational | Staff on site only 7 AM-7 PM weekdays (60 of 168 hours); after hours depends on a 4-hour vendor response | 4 | 4 | 16 (Critical) | Vendor paged by alarms | 24/7 NOC or remote hands plus an on-call engineer | High |

[^1]: **R-01:** Power fails and the generator is asked to carry 153% of its rating. It trips, or operators have to shut down cooling (servers overheat) or IT (banking goes offline).
[^2]: **R-02:** One UPS fails and the other has to carry about 225% of its rating. It overloads, and the next power dip drops the servers.
[^3]: **R-03:** A storm knocks out the one utility feed, and the generator (R-01) can only carry about 65% of the building.
[^4]: **R-07:** A construction crew cuts the fiber, and online banking, wires, and trading links stay down until it's repaired.
[^5]: **R-09:** An intruder logs in with `root`/`calvin` and can power off or wipe core banking servers, leading to a data exposure and days of recovery.
[^6]: **R-10:** One hacked application server reaches the management cards and gets around every operating-system security tool.
[^7]: **R-11:** Old firmware is missing four years of fixes, so someone who can reach the iDRAC page can take over the server's management controller.
[^8]: **R-13:** A part fails at 1 AM Saturday. The vendor takes up to 4 hours to arrive and parts take 1-3 days, so the room runs without backup all weekend.

---

## Part 3 - Uptime Tier Analysis
**Current tier: Tier I**
| Tier | Availability | Allowed downtime/yr | Meridian meets it? |
|---|---|---|---|
| I | 99.671% | 28.8 h | **Yes** (basic UPS, generator, cooling, one power path) |
| II | 99.741% | 22.7 h | **No** (cooling is N+1, but the generator and UPS aren't) |
| III | 99.982% | 1.6 h | **No** (one feed, one generator, single-corded servers) |
| IV | 99.995% | 0.44 h | **No** |
**Three biggest gaps:**

| # | Gap | Now | Cost |
|---|---|---|---|
| 1 | Generator size and backup (R-01) | One 200 kW unit for a 306 kW facility | $250K-$600K |
| 2 | UPS size and backup (R-02) | 160 kVA total for a 180 kW load | $150K-$400K |
| 3 | Single power path and single-corded servers (R-03, R-04) | Can't be maintained without shutting down | $1.0M-$3.0M |

Gaps 1 and 2 block Tier II. Gap 3 is the main barrier to Tier III.

**Recommendation: do Tier II now and study Tier III within 18 months.** Gaps 1 and 2 cost about $400K-$1.0M and fix real undersizing. Tier I to II cuts allowed downtime by about 6.1 hours a year (worth $1.8M-$6.1M at $300K-$1M/hour), and one avoided 4-hour outage ($1.2M-$4.0M) covers the cost. Tier III costs more but cuts another 21.1 hours a year, so about one avoided outage covers that too. 

---

## Part 4 - Remediation Roadmap

| When | # | Action | Risks | Owner | Cost | Score | How we'll know it worked |
|---|---|---|---|---|---|---|---|
| **Now (0-30 days)** | I-1 | Change iDRAC passwords | R-09 | Security Engineer | Low | 25 → 5 | No iDRAC accepts the old passwords in a scripted test |
| | I-2 | Block iDRAC from production; turn off IPMI over LAN and SNMP v1 | R-10 | Network Engineer | Low | 20 → 10 | A scan from production shows iDRAC ports blocked |
| | I-3 | Start a media destruction log; limit escorts to trained staff | R-12 | Facilities Manager | Low | 12 → 9 | Every visit and destruction is logged and audited weekly |
| | I-4 | Write an after-hours plan; alarms page an internal on-call engineer too | R-13 | Operations Manager | Low | 16 → 12 | On-call answers every monthly test page within 15 min |
| **Soon (1-6 months)** | S-1 | Add a second internet provider with BGP | R-07 | Network Manager | Medium | 15 → 5 | Failover in 60 seconds or less |
| | S-2 | Add a second firewall in HA | R-08 | Network Manager | Medium | 10 → 5 | Failover interrupts traffic 10 seconds or less |
| | S-3 | Management network, firmware upgrade, SNMPv3 | R-10, R-11 | Security Engineer | Low | R-10: 10 → 5; R-11: 16 → 4 | All iDRAC on supported firmware, none reachable from production |
| | S-4 | Cameras and lockable rack doors | R-12 | Facilities Manager | Medium | 9 → 4 | All aisles covered; core banking racks lock |
| | S-5 | Extend containment to rows 5-8 | R-06 | Director of Facilities | Medium | 12 → 4 | All rack inlets 77°F or less for 30 days |
| | S-6 | Spare UPS module and compressor kit; rack transfer switches | R-05, R-04 | Operations Manager | Medium | R-05: 12 → 4; R-04: 12 → 6 | UPS module swap in 4 hours or less; pulling either feed keeps servers up |
| **Later (6-18 months)** | L-1 | N+1 generators sized for 306 kW at 80% load or less | R-01 | Facilities Director + CTO | High | 20 → 5 | Load-bank test carries the whole facility for 4 hours |
| | L-2 | Resize UPS so either side carries 180 kW; dual power supplies at refresh | R-02, R-04 | Facilities Director + CTO | High | R-02: 20 → 5; R-04: 6 → 4 | With either side off, the other carries the load with no server loss |
| | L-3 | Second utility feed or backup site | R-03 | CTO and Board | High | 15 → 5 | Independent feeds; automatic transfer test passes |
| | L-4 | 24/7 NOC or remote hands | R-13 | CTO / COO | High | 12 → 6 | First on-site action within 30 minutes on average |

**Overall:** total score 205 → 61 (down 70%); Critical risks 8 → 0.

---

## Part 5 - MTTR & Reliability Calculations

Availability = MTBF ÷ (MTBF + MTTR). MTTR = response time + wait for parts + install time, blended by on-site hours (35.7% of the week) and after hours (64.3%). I picked the two top power risks (R-01 generator, R-02 UPS) and the compressor part of R-05 for cooling (R-06 is an airflow problem with no MTBF).

- **UPS (R-02):** MTBF is 250,000 h (Riello UPS says that's typical for an online UPS). MTTR is 0.5 + 24 + 3 = 27.5 h on-site and 4 + 48 + 3 = 55 h after hours, which blends to **45.4 h** (install times are my assumption). Availability = **99.9818%**, about 1.59 h of downtime a year. That beats Tier I and II and is just under Tier III.
- **Generator (R-01):** MTBF is 10,000 h and MTTR is 48 h (**both my assumptions**, since the survey doesn't give them). Availability = **99.522%**, about 41.8 h a year. That's below Tier I. More important, even a healthy generator can't carry the load.
- **CRAC compressor (R-05):** MTBF is 60,000 h (**my assumption**). MTTR is 2.75 h response + 60 h parts + 6 h install = **68.8 h**. Availability = **99.8855%** per unit, about 10 h a year. That meets Tier I and II but not III. The cooling has plenty of spare capacity (about 150 tons vs. roughly 51 tons needed), so this risk is moderate.

| Component | MTBF (h) | MTTR (h) | Availability | Downtime/yr | vs. Tier I (99.671%) | vs. Tier III (99.982%) |
|---|---|---|---|---|---|---|
| UPS unit | 250,000 | 45.4 | 99.9818% | 1.59 h | Meets | Just below |
| Generator | 10,000 (assumed) | 48 | 99.522% | 41.8 h | Below | Below |
| CRAC compressor | 60,000 (assumed) | 68.8 | 99.8855% | 10.0 h | Meets | Below |

**Takeaway:** shorter repair times help more than better MTBF. Spares on site (S-6) would cut the UPS repair time from about 45 hours to about 4.

---

## References and Assumptions

1. ITIC, *2024 Hourly Cost of Downtime Survey*: https://calyptix.com/wp-content/uploads/ITIC-2024-Hourly-Cost-of-Downtime-Survey-Results-Part-2.pdf
2. Riello UPS, MTBF/MTTR/availability: https://www.riello-ups.com/questions/52-how-is-ups-resilience-measured-mtbf-mttr-availability
3. Eaton, "Maximizing UPS Availability": https://www.voltimum.co.uk/pdf-catalogues/eaton-maximizing-ups-availability?page=2
4. NREL, *Emergency Diesel Generator Reliability and Installation Energy Security*, NREL/TP-5C00-76553 (2020): https://www.nrel.gov/docs/fy20osti/76553.pdf
5. Data Center Knowledge, "Generators Are Key to Improving Reliability": https://www.datacenterknowledge.com/node/60676


