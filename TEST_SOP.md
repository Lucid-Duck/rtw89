# RTW89 Driver Test Standard Operating Procedure

## Test Environment

| Component | Value |
|-----------|-------|
| Device Under Test | D-Link DWA-X1850 (RTL8832AU) |
| D-Link Interface | wlp0s20f0u6 |
| D-Link Driver | rtw89_8852au_git |
| D-Link IP | 192.168.1.110 |
| Alfa Interface | wlp0s13f0u2i3 (DO NOT USE FOR TESTS) |
| Alfa Driver | mt7921u |
| Alfa IP | 192.168.1.231 |
| iperf3 Server | 192.168.1.250 (Windows 11 PC) |
| USB Speed | 480M (USB2 via extension cable) |

## Pre-Test Verification Checklist

Before EVERY test, verify:

1. **Interface Check**: `ip -br addr show wlp0s20f0u6` - must show IP 192.168.1.110
2. **Driver Check**: Confirm rtw89_8852au_git is the driver
3. **Route Check**: `ip route get 192.168.1.250 from 192.168.1.110` - must show `dev wlp0s20f0u6`
4. **Band Check**: `iw dev wlp0s20f0u6 link` - verify frequency (2.4GHz = 24xx MHz, 5GHz = 5xxx MHz)

## CRITICAL: Driver Rebuild Procedure

**ALWAYS do a full clean rebuild when switching driver versions. No shortcuts.**

### Clean Rebuild Steps

```bash
# 1. CLEAN - Remove all compiled objects
make clean

# 2. BUILD - Full rebuild from scratch
make -j$(nproc)

# 3. VERIFY local build before installing
# Check for the flow control constant (32 = 0x20 in hex)
# PATCHED shows $0x20 in flow control logic
# UNPATCHED shows $0x2a (42) instead
objdump -d rtw89_usb_git.ko | grep -E 'mov.*\$0x20.*%eax|cmp.*\$0x20' | head -3

# 4. INSTALL
sudo make install

# 5. UNLOAD old modules
sudo rmmod rtw89_8852au_git rtw89_8852a_git rtw89_usb_git rtw89_core_git

# 6. RELOAD new modules
sudo modprobe rtw89_8852au_git

# 7. RECONNECT WiFi (driver reload disconnects)
nmcli connection up "8 Hertz WAN IP"

# 8. FIX ROUTING (always resets after reconnect)
sudo ip route replace 192.168.1.250 dev wlp0s20f0u6 src 192.168.1.110
```

### Verify Driver Version

The string "tx_inflight" does NOT appear in compiled binaries (it's a struct field name).
Instead, check for the flow control constant:

```bash
# PATCHED driver uses constant 32 (0x20)
# UNPATCHED driver uses constant 42 (0x2a)
objdump -d /lib/modules/$(uname -r)/extra/rtw89/rtw89_usb_git.ko | grep -E '\$0x20.*mov|cmp.*\$0x20' | head -2
```

If you see `$0x20` in the output, it's PATCHED.

## Test Sequence

### Phase 1: First Driver Version

#### 2.4GHz Band Tests
- 5 rounds of ping (10 pings each)
- 3 download tests (iperf3 -R)
- 3 upload tests (iperf3)

#### 5GHz Band Tests
- 5 rounds of ping (10 pings each)
- 3 download tests (iperf3 -R)
- 3 upload tests (iperf3)

### Phase 2: Driver Swap

Follow CRITICAL: Driver Rebuild Procedure above.

### Phase 3: Second Driver Version

Repeat all tests from Phase 1.

## Commands Reference

### Switch to 2.4GHz
```bash
nmcli connection modify "8 Hertz WAN IP" 802-11-wireless.band bg
nmcli connection down "8 Hertz WAN IP"
nmcli connection up "8 Hertz WAN IP"
# ALWAYS fix routing after reconnect!
sudo ip route replace 192.168.1.250 dev wlp0s20f0u6 src 192.168.1.110
```

### Switch to 5GHz
```bash
nmcli connection modify "8 Hertz WAN IP" 802-11-wireless.band a
nmcli connection down "8 Hertz WAN IP"
nmcli connection up "8 Hertz WAN IP"
# ALWAYS fix routing after reconnect!
sudo ip route replace 192.168.1.250 dev wlp0s20f0u6 src 192.168.1.110
```

### Fix Routing
```bash
# Route ALWAYS resets to Alfa after any WiFi reconnect
# Must run this EVERY TIME after reconnecting D-Link
sudo ip route replace 192.168.1.250 dev wlp0s20f0u6 src 192.168.1.110

# Verify:
ip route get 192.168.1.250 from 192.168.1.110
# Must show: dev wlp0s20f0u6
```

## Important Notes

1. **Route resets after EVERY reconnect** - Always fix routing after switching bands or reloading driver
2. **Clean rebuild required** - Never skip `make clean` when switching driver versions
3. **Verify before testing** - Check interface, band, and route before every test series
4. **D-Link only** - Never let test traffic go through Alfa (wlp0s13f0u2i3)
