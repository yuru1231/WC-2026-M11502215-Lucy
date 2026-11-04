# Lab 2: gNB TDD Traffic Analysis

## 1. Goal and experimental setup

This lab compares three 5-second UDP iperf3 tests to examine the relationship between the broadcast common TDD configuration and measured traffic performance. Configuration captures are separate reference recordings, supplied by the instructor, and are not recordings of the original iperf3 runs. The JSON files report performance but do not specify SIB1 TDD settings.

| Scenario | Reference TDD setting | iperf3 traffic direction | Performance JSON | Scenario capture |
|---|---|---|---|---|
| A | DL-heavy | Downlink | `scenario-a-downlink.json` | `scenario-a-dl-heavy-downlink.pcapng` |
| B | DL-heavy | Uplink | `scenario-b-uplink.json` | `scenario-b-dl-heavy-uplink.pcapng` |
| C | Balanced | Uplink | `scenario-c-uplink.json` | `scenario-c-balanced-uplink.pcapng` |

## Task 1 — Confirm the traffic direction

The field `start → test_start → reverse` determines the iperf3 traffic direction in this lab. `reverse = 1` means downlink and `reverse = 0` means uplink under the supplied client/server mapping.

| Scenario | Reverse value | Direction |
|---|---:|---|
| A | 1 | Downlink |
| B | 0 | Uplink |
| C | 0 | Uplink |

**Question — Does the traffic direction shown in the JSON match the scenario description? Show proof.**

Yes. Scenario A has `reverse = 1`, which corresponds to downlink traffic in this lab. Scenarios B and C both have `reverse = 0`, corresponding to uplink traffic. The values agree with the three supplied scenario descriptions.

## Task 2 — Compare throughput

Use `end → sum → bits_per_second`, not the individual interval fields. Convert bit/s to Mbps by dividing by 1,000,000.

\[
T_{\mathrm{Mbps}}=\frac{\mathrm{bits\_per\_second}}{10^6}.
\]

| Scenario | Direction | `end.sum.bits_per_second` (bit/s) | Throughput (Mbps) |
|---|---|---:|---:|
| A | Downlink | 90,654,195.8358403 | **90.65** |
| B | Uplink | 109,117,249.69037457 | **109.12** |
| C | Uplink | 124,416,749.13023271 | **124.42** |

**Throughput improvement, B → C:**

\[
\begin{aligned}
\mathrm{Improvement}&=\frac{T_C-T_B}{T_B}\times100\%\\
&=\frac{124.41674913-109.11724969}{109.11724969}\times100\%\\
&\approx\boxed{14.02\%}.
\end{aligned}
\]

The downlink throughput in A is **90.65 Mbps**, and the uplink throughput in B is **109.12 Mbps**. C has the higher uplink throughput (**124.42 Mbps**), representing an increase of approximately **15.30 Mbps** or **14.02%** over B.

**JSON evidence:**

**Scenario A**  
<img width="230" alt="Task 2 throughput A" src="https://github.com/user-attachments/assets/d1517882-62ca-44ff-8e3a-022ed8a35e94" />

**Scenario B**  
<img width="264" alt="Task 2 throughput B" src="https://github.com/user-attachments/assets/906758ec-4de1-4a92-8de1-25db1118e244" />

**Scenario C**  
<img width="270" alt="Task 2 throughput C" src="https://github.com/user-attachments/assets/269506b3-6317-4131-ae9e-e59eb0fe0447" />

## Task 3 — Check the trade-off

Packet loss and jitter were obtained from `end → sum → lost_percent` and `end → sum → jitter_ms`.

| Scenario | Throughput (Mbps) | Packet loss (%) | Jitter (ms) |
|---|---:|---:|---:|
| B | 109.12 | **0.236** | **0.116** |
| C | 124.42 | **2.134** | **0.162** |

**Questions and answers.** Scenario C has higher uplink throughput than Scenario B, but it does **not** have lower packet loss. Compared with B, C has **higher throughput, higher packet loss, and higher jitter**. The observed trade-off is improved uplink throughput alongside worse measured loss and jitter; these observations alone do not demonstrate that the TDD setting was the only cause.

**JSON evidence:**

**Scenario A (supplementary)**  
<img width="214" alt="Task 3 metrics A" src="https://github.com/user-attachments/assets/5d5970a1-c2b4-4c8d-8360-e3ade9c57d7e" />

**Scenario B**  
<img width="228" alt="Task 3 metrics B" src="https://github.com/user-attachments/assets/82cb7687-6cc5-4182-afad-5685f980aea3" />

**Scenario C**  
<img width="257" alt="Task 3 metrics C" src="https://github.com/user-attachments/assets/60922887-84e1-465b-8711-38306cf750f4" />

## Task 4 — Read SIB1 and draw the TDD configuration

### 4.1 SIB1 evidence and parameters

The SIB1 reference captures were decoded in Wireshark under `NR RRC → BCCH-DL-SCH-Message → systemInformationBlockType1 → servingCellConfigCommon → tdd-UL-DL-ConfigurationCommon → pattern1`.

| Item | DL-heavy (A/B reference) | Balanced (C reference) |
|---|---|---|
| Configuration capture | `tdd-dl-heavy-sib1.pcapng` | `tdd-balanced-sib1.pcapng` |
| SIB1 packet number | **2** | **2** |
| `referenceSubcarrierSpacing` | **30 kHz** | **30 kHz** |
| `dl-UL-TransmissionPeriodicity` | **5 ms** | **2.5 ms** |
| `nrofDownlinkSlots` | **7** | **2** |
| `nrofDownlinkSymbols` | **6** | **6** |
| `nrofUplinkSlots` | **2** | **2** |
| `nrofUplinkSymbols` | **4** | **4** |
| `pattern2` present? | **No** | **No** |

**Balanced SIB1 screenshot:**  
<img width="737" alt="Balanced SIB1 reference configuration" src="https://github.com/user-attachments/assets/9b2f57d7-220d-479a-b206-466e0e5dc9a0" />

**DL-heavy SIB1 screenshot:**  
<img width="736" alt="DL-heavy SIB1 reference configuration" src="https://github.com/user-attachments/assets/a60b431a-4d90-438b-8c7c-ead29a8c14b3" />

### 4.2 Reference slot duration and allocation rules

For the 30 kHz reference subcarrier spacing and normal cyclic prefix used in this exercise, one reference slot lasts **0.5 ms** and contains **14 OFDM symbols**. The common pattern places full DL slots first and full UL slots last. The remaining mixed slot consists of **6 D symbols, 4 F symbols and 4 U symbols**. Flexible symbols are not assumed to be uplink or guard time.

**Mixed-slot detail (symbol indices 0–13):**
<img width="878" height="188" alt="image" src="https://github.com/user-attachments/assets/4ba63891-eaef-4a09-a606-6fc43f752bdd" />



### 4.3 TDD time-allocation diagrams (common 0–5 ms axis)

**DL-heavy:** The repeating period is 5 ms, corresponding to ten 0.5 ms slots: `D D D D D D D M U U`.


<img width="733" height="202" alt="image" src="https://github.com/user-attachments/assets/adc8c048-6b9d-4f8c-b897-366ea09f6d4a" />


**Balanced:** The repeating period is 2.5 ms, corresponding to five slots: `D D M U U`; the pattern repeats twice in the common 5 ms interval.
<img width="959" height="266" alt="image" src="https://github.com/user-attachments/assets/7c676178-080b-4e57-83a3-cd047b3d1a48" />


`D` = DL, `U` = UL, `F` = flexible, `M` = mixed slot (6D/4F/4U). Dashed vertical lines mark repeating-period boundaries. Slot indices shown in both diagrams are global indices from 0 to 9.

### 4.4 Configured symbol counts and UL share

For each complete repeating sequence:

\[
N_{DL}=14N_{DL\,slots}+N_{DL\,symbols},\qquad
N_{UL}=14N_{UL\,slots}+N_{UL\,symbols}.
\]
\[
N_F=14N_{slots}-N_{DL}-N_{UL},\qquad
UL\text{ symbol share}=\frac{N_{UL}}{14N_{slots}}\times100\%.
\]

For **DL-heavy** (5 ms / 0.5 ms = 10 slots):

\[
N_{DL}=7(14)+6=104,\quad N_{UL}=2(14)+4=32,\quad N_F=140-104-32=4.
\]
\[
UL\text{ share}=32/140\times100\%=22.86\%.
\]

For **Balanced** (2.5 ms / 0.5 ms = 5 slots):

\[
N_{DL}=2(14)+6=34,\quad N_{UL}=2(14)+4=32,\quad N_F=70-34-32=4.
\]
\[
UL\text{ share}=32/70\times100\%=45.71\%.
\]

| TDD setting | Repeating duration (ms) | Total reference slots | DL symbols | UL symbols | Flexible symbols | Configured UL share (%) |
|---|---:|---:|---:|---:|---:|---:|
| DL-heavy | 5 | 10 | 104 | 32 | 4 | **22.86** |
| Balanced | 2.5 | 5 | 34 | 32 | 4 | **45.71** |

**Common 5 ms comparison:**

| TDD setting | Repeating duration (ms) | Repetitions in 5 ms | DL symbols in 5 ms | UL symbols in 5 ms | Flexible symbols in 5 ms |
|---|---:|---:|---:|---:|---:|
| DL-heavy | 5 | 1 | 104 | **32** | 4 |
| Balanced | 2.5 | 2 | 68 | **64** | 8 |

**Question — Can two patterns have the same UL symbols per period but different UL symbols per unit time?**

Yes. Both configurations allocate 32 UL symbols in one repeating sequence. However, the DL-heavy period lasts 5 ms and the balanced period lasts only 2.5 ms. Over the same 5 ms interval, Balanced offers **64** configured UL symbols compared with **32** for DL-heavy. The symbol shares are configuration counts, not measurements of actual scheduled airtime.

### 4.5 Connect SIB1 to performance results

| Scenario | Reference TDD setting | JSON traffic direction | Configured UL share (%) | Throughput (Mbps) | Packet loss (%) |
|---|---|---|---:|---:|---:|
| A | DL-heavy | Downlink | 22.86 | 90.65 | 0.443 |
| B | DL-heavy | Uplink | 22.86 | 109.12 | 0.236 |
| C | Balanced | Uplink | 45.71 | 124.42 | 2.134 |

**Which decoded parameters differ?** The periodicity changes from 5 ms (DL-heavy) to 2.5 ms (Balanced), and `nrofDownlinkSlots` changes from 7 to 2. The SCS, partial DL/UL symbol counts and number of full UL slots remain the same.

**Which configuration provides more UL symbols per unit time?** Balanced. It provides 64 configured UL symbols per 5 ms, versus 32 in DL-heavy, and its configured UL share is 45.71% rather than 22.86%.

**How does this help explain B/C throughput?** C has more configured UL opportunities and a higher measured throughput (124.42 versus 109.12 Mbps, +14.02%). The throughput gain is smaller than the increase in UL symbol share, and packet loss is also higher in C. Scheduling, radio conditions, offered load and other system settings affect performance, so this is not proof of a single causal factor.

**Why can A and B share SIB1 despite opposite traffic directions?** A and B use the same reference broadcast TDD setting, which allocates common DL, UL and flexible resources. The iperf3 traffic direction is independently selected by the application test, not by the `reverse` field of the SIB1 message. The separately recorded configuration captures support the instructor-provided scenario mapping; they do not independently prove what was broadcast during each original performance run.

## Task 5 — Is the TDD setting suitable?

| Scenario | TDD setting | Traffic direction | Is the setting suitable for this direction? Why? |
|---|---|---|---|
| A | DL-heavy | Downlink | **Yes.** The configuration dedicates 104 of 140 symbols per 5 ms to DL, offering more DL opportunities. |
| B | DL-heavy | Uplink | **Less suitable for heavy UL traffic.** It dedicates only 32 of 140 symbols per 5 ms (22.86%) to UL. |
| C | Balanced | Uplink | **More suitable for UL than B.** It dedicates 64 of 140 symbols per 5 ms (45.71%) to UL. |

**Why might C achieve better uplink throughput than B?** The Balanced reference configuration provides twice as many configured UL symbols over 5 ms as DL-heavy. The measured uplink throughput is 124.42 Mbps in C and 109.12 Mbps in B, an increase of 14.02%. Nonetheless, C has more packet loss, and actual throughput depends on scheduling and other radio conditions.

## Task 6 — Find GTP-U information

The selected frame contains an outer IPv4/UDP header, a GTP-U header, and an inner IPv4 packet. The selected inner packet is TCP, which is acceptable for this task because the requirement is to identify two IPv4 headers in a GTP-U packet.

| Item | Value |
|---|---|
| Scenario capture filename | `scenario-a-dl-heavy-downlink.pcapng` |
| Packet number | **1832** |
| Outer source IP | `192.168.70.129` |
| Outer destination IP | `192.168.70.134` |
| Inner source IP | `10.0.0.6` |
| Inner destination IP | `192.168.70.135` |
| TEID | **`0x00000011` (17)** |

The outer IP addresses identify the GTP-U tunnel endpoints. The inner addresses belong to the encapsulated original IP packet. The TEID identifies a GTP-U tunnel at the receiving endpoint.

**Wireshark evidence:**  
<img width="926" alt="GTP-U outer IP, inner IP and TEID" src="https://github.com/user-attachments/assets/93c9bbd0-b3c6-43e0-8926-0afef1993c3b" />

## Task 7 — TCP retransmissions

The Wireshark display filter `tcp.analysis.retransmission` was used for each scenario capture. The table reports the number of displayed packets, not an estimate of lost UDP packets.

| Scenario | Displayed TCP retransmissions |
|---|---:|
| A | **10** |
| B | **8** |
| C | **8** |

**Screenshot A:**  
<img width="925" alt="Scenario A TCP retransmissions" src="https://github.com/user-attachments/assets/0aa3e829-344a-4abe-b5f2-33f48fd1a862" />

**Screenshot B:**  
<img width="926" alt="Scenario B TCP retransmissions" src="https://github.com/user-attachments/assets/b5c0d472-02cc-49e0-b270-bfb3d437348b" />

**Screenshot C:**  
<img width="926" alt="Scenario C TCP retransmissions" src="https://github.com/user-attachments/assets/d66468a5-1bb9-43a7-b352-4b2ec8671bc1" />

**Question — Do TCP retransmissions directly represent lost UDP performance packets?**

No. The measured performance tests use UDP, while iperf3 uses a separate TCP control connection. Thus, TCP retransmissions and UDP packet loss are distinct observations. For example, B and C each have eight displayed TCP retransmissions, but their UDP loss rates are 0.236% and 2.134%, respectively.

## Task 8 — RTT and packet interval

### 8A. TCP ACK RTT

A TCP control packet was selected in each capture using the display filter `tcp.analysis.ack_rtt`. The RTT to ACK the segment is a single TCP-control sample, not a UDP RTT and not the average RTT of the test.

| Scenario | Selected ACK packet number | ACKed packet number | TCP ACK RTT (s) | TCP ACK RTT (ms) |
|---|---:|---:|---:|---:|
| A | **1833** | 1831 | 0.000485 | **0.485** |
| B | **2068** | 2065 | 0.000331 | **0.331** |
| C | **1432** | 1430 | 0.000315 | **0.315** |

\[
RTT_{ms}=RTT_s\times1000.
\]

These selected samples give C < B < A, but three individual ACKs cannot establish an average RTT ranking or prove that TDD configuration caused the difference.

**Scenario A RTT screenshot:**  
<img width="736" alt="Scenario A TCP ACK RTT" src="https://github.com/user-attachments/assets/8ee24345-b24f-4612-b3d3-2a4f026bcf85" />

**Scenario B RTT screenshot:**  
<img width="740" alt="Scenario B TCP ACK RTT" src="https://github.com/user-attachments/assets/fa24feb9-2433-4730-bf06-2677343129f9" />

**Scenario C RTT screenshot:**  
<img width="741" alt="Scenario C TCP ACK RTT" src="https://github.com/user-attachments/assets/42bb037e-1ecb-4c30-afe7-4731eddfc4e3" />

### 8B. Packet interval — instructor-approved TCP substitution

**Method deviation from the original handout:** The original instructions request UDP packet intervals. Per the instructor's updated direction, TCP packets were used instead. The identical display filter `gtp && tcp.srcport == 5201` was applied to all three captures. Each result is the `Frame → Time delta from previous displayed frame` for the second of two **adjacent displayed TCP packets**. No TCP analysis flags were excluded.

| Scenario | Display filter | Consecutive displayed packet numbers | Time interval (s) | Time interval (ms) |
|---|---|---|---:|---:|
| A | `gtp && tcp.srcport == 5201` | **1834 → 1848** | 0.019316 | **19.316** |
| B | `gtp && tcp.srcport == 5201` | **2082 → 2083** | 0.000063 | **0.063** |
| C | `gtp && tcp.srcport == 5201` | **1432 → 1433** | 0.000009 | **0.009** |

\[
\Delta t_{ms}=1000\times\bigl(\text{Time delta from previous displayed frame in seconds}\bigr).
\]

These are **individual filtered TCP observation intervals**, not mean TCP intervals and not the UDP data-packet intervals requested in the original handout. In B and C, the adjacent displayed packets include entries that Wireshark marks as duplicate ACK or out-of-order. Their short gaps may reflect duplicate capture observations rather than application pacing; nevertheless, they are reported as measured under the instructor-approved adjacent-displayed-TCP-packet method. They must not be used to infer the TDD slot pattern.

**Scenario A — adjacent TCP packet screenshots:**  
<img width="926" alt="Scenario A TCP packet interval first screenshot" src="https://github.com/user-attachments/assets/6984c14e-0f45-4d06-9002-de783f055144" />  
<img width="926" alt="Scenario A TCP packet interval second screenshot" src="https://github.com/user-attachments/assets/5ae52a47-0c5c-40c4-925e-3c6cd87a59c3" />

**Scenario B — adjacent TCP packet screenshots:**  
<img width="927" alt="Scenario B TCP packet interval first screenshot" src="https://github.com/user-attachments/assets/c5f8bd0c-0714-498c-8415-da235b3e4888" />  
<img width="928" alt="Scenario B TCP packet interval second screenshot" src="https://github.com/user-attachments/assets/6bbf3e4888-41d6-4af4-a7af-d5d525f73ea4" />

**Scenario C — adjacent TCP packet screenshots:**  
<img width="928" alt="Scenario C TCP packet interval first screenshot" src="https://github.com/user-attachments/assets/493614ae-a4aa-4d39-a0a3-1e1422d820a8" />  
<img width="926" alt="Scenario C TCP packet interval second screenshot" src="https://github.com/user-attachments/assets/016af505-f77e-4acd-8fd7-097b3356a8bd" />

## Final questions

**1. Does Scenario A use a TDD setting that matches its traffic direction?**  
Yes. Scenario A uses DL-heavy TDD for downlink traffic. Its reference pattern allocates 104 of 140 symbols per 5 ms (74.29%) to downlink.

**2. Why is Scenario B expected to have limited uplink resources? Cite a parameter or count.**  
Scenario B uses the DL-heavy reference pattern with `nrofUplinkSlots = 2` and `nrofUplinkSymbols = 4` over a 5 ms period. That corresponds to only 32 UL symbols out of 140, or 22.86%, compared with 64 UL symbols per 5 ms for Balanced.

**3. How much does uplink throughput change from B to C?**  
Uplink throughput increases from 109.12 Mbps in B to 124.42 Mbps in C. The increase is approximately 15.30 Mbps, or 14.02% relative to B.

**4. What negative result, if any, is visible in C?**  
Scenario C shows higher UDP packet loss (2.134% versus 0.236% in B). Its jitter is also higher (0.162 ms versus 0.116 ms).

**5. Why can increasing UL resources improve uplink throughput?**  
More configured UL symbols provide more potential uplink transmission opportunities within the same time window. If other conditions allow, this can reduce the time-resource constraint on uplink data.

**6. Is adding more UL resources always the best solution?**  
No. Scenario C achieves higher throughput but also has higher packet loss and jitter than B. UL resources must be balanced against DL needs, overhead, scheduling and actual traffic demand.

**7. Suggest a simple follow-up test that might improve throughput without too much packet loss.**  
Test one or more intermediate UL symbol shares while holding offered load and other radio conditions fixed. Compare throughput, loss and jitter across those configurations to identify an acceptable compromise.

**8. What do the configuration captures show that JSON does not, and why are they not direct proof of an earlier run's settings?**  
The separately captured SIB1 messages show reference TDD periodicity, SCS and DL/UL symbol allocation; iperf3 JSON reports direction and performance but not those TDD parameters. Because the SIB1 files were recorded separately from the performance runs, they document the instructor-supplied reference settings rather than independently verifying the exact broadcast configuration during each earlier test.

## Sources and measurement notes

- Supplied lab handout: *Lab 2: gNB TDD Traffic Analysis*.
- Provided iperf3 JSON results for Scenarios A–C.
- Supplied SIB1 reference capture screenshots and Wireshark scenario-capture screenshots, linked alongside each task above.
- 3GPP TS 38.331 (SIB1 and common TDD configuration) and TS 38.213 §11.1 (DL/UL assignment interpretation), as referenced by the lab handout.
- PRACH Configuration Index is an **optional** observation in the lab handout and is not required here.
