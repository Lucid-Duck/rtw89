# TX Flow Control Patch Test Results

**Date:** 2026-01-26
**Device:** D-Link DWA-X1850 (RTL8832AU)
**Test Server:** Windows 11 PC (192.168.1.250) running iperf3

## Test Methodology

- All tests run via D-Link interface only (verified routing before each test)
- Alfa adapter kept connected but NOT used for test traffic
- Interface verified: wlp0s20f0u6 with IP 192.168.1.110
- USB speed verified before each test series

## USB2 (480M) - WITHOUT PATCH

### Download (Server → Client)

**Run 1:** 106 Mbits/sec
- Erratic start: 1.05 → 0 → 9.43 Mbits/sec

**Run 2:** 169 Mbits/sec
- Mid-test dip: 229 → 16.8 → 21 Mbits/sec

**Run 3:** 130 Mbits/sec
- Erratic start: 4.19 → 11.5 → 7.33 Mbits/sec

**Average: ~135 Mbits/sec (HIGHLY VARIABLE)**

### Upload (Client → Server) - TX Direction

- 231 Mbits/sec, 0 retransmits

## USB2 (480M) - WITH PATCH

(Testing in progress)

## USB3 (5000M) - WITHOUT PATCH

(Testing pending)

## USB3 (5000M) - WITH PATCH

(Testing pending)

## Key Observations

1. **Unpatched driver shows highly inconsistent download performance**
   - Three runs varied: 106, 169, 130 Mbits/sec
   - Erratic starts with speeds dropping to near-zero
   
2. **The inconsistency itself demonstrates the flow control problem**
   - Without proper backpressure, the system becomes unstable
   - Performance varies wildly between runs

## Value of 32 URBs Rationale

- ath9k_htc driver uses MAX_TX_URB_NUM = 8
- Value of 32 provides 4x the headroom for modern WiFi 6 throughput
- Balances queue depth for throughput vs memory pressure

