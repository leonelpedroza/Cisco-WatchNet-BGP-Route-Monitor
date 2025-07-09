# WatchNet - BGP Route Monitor for Cisco IOS

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![IOS: 15.0+](https://img.shields.io/badge/IOS-15.0%2B-blue.svg)](https://www.cisco.com)
[![TCL: 8.3+](https://img.shields.io/badge/TCL-8.3%2B-green.svg)](https://www.tcl.tk/)

A TCL-based monitoring script for Cisco IOS routers that tracks specific BGP routes and generates real-time alerts when routes become unstable or disappear from the routing table.

## 🎯 Overview

WatchNet provides automated monitoring for critical BGP routes in your network infrastructure. It detects:
- **Route Flapping**: When a route appears/disappears repeatedly (age < 60 seconds)
- **Missing Routes**: When a monitored route is not present in the routing table
- **Route Recovery**: When a previously problematic route becomes stable again

The script integrates with your existing monitoring infrastructure through SNMP traps and syslog messages, ensuring your NOC team is immediately notified of routing issues.

## ✨ Features

- **Real-time BGP route monitoring** with configurable check intervals
- **SNMP v2c trap generation** for integration with network management systems
- **Syslog message generation** with severity levels
- **State persistence** to avoid duplicate alerts
- **Route flapping detection** based on route age
- **Embedded Event Manager (EEM) integration** for automated execution
- **Debug mode** for troubleshooting
- **Configurable thresholds and parameters**

## 📋 Requirements

- Cisco IOS 15.0 or later
- TCL support enabled on the router
- SNMP configuration (for trap generation)
- Syslog configuration (for logging)
- Flash storage space for script files

## 🚀 Basic Setup

### 1. Enable TCL on your Cisco Router

```cisco
configure terminal
scripting tcl low-encryption
exit
```

### 2. Configure SNMP (if not already configured)

```cisco
configure terminal
snmp-server community public RO
snmp-server community private RW
snmp-server host 172.168.1.1 version 2c SnMpSeRvErPaSsWoRd
snmp-server enable traps bgp
exit
```

### 3. Configure Syslog (if not already configured)

```cisco
configure terminal
logging host 172.168.1.1
logging trap debugging
logging facility local7
exit
```

### 4. Copy the Script to Router Flash

```cisco
copy tftp://your-tftp-server/watchnet-bgp-monitor.tcl flash:
```

Or via USB:
```cisco
copy usbflash0:watchnet-bgp-monitor.tcl flash:
```

### 5. Verify the Script

```cisco
dir flash: | include watchnet
more flash:watchnet-bgp-monitor.tcl
```

## 📖 How to Use

### Method 1: Manual Execution (Testing)

Run the script manually to test functionality:

```cisco
enable
tclsh flash:watchnet-bgp-monitor.tcl
```

### Method 2: Automated Execution with EEM (Production)

Configure Embedded Event Manager to run the script automatically every 60 seconds:

```cisco
configure terminal
event manager applet BGP_ROUTE_MONITOR
 event timer watchdog time 60
 action 1.0 cli command "enable"
 action 2.0 cli command "tclsh flash:watchnet-bgp-monitor.tcl"
exit
```

To verify EEM configuration:
```cisco
show event manager policy registered
```

### Method 3: Cron-based Execution with Kron (Alternative)

```cisco
configure terminal
kron policy-list BGP_MONITOR_POLICY
 cli tclsh flash:watchnet-bgp-monitor.tcl
 
kron occurrence BGP_MONITOR_SCHEDULE in 1 recurring
 policy-list BGP_MONITOR_POLICY
exit
```

## ⚙️ Configuration

Edit the configuration section in the script to match your environment:

```tcl
array set config {
    watched_route   "10.1.1.1/32"          # Route to monitor
    syslog_server   "172.168.1.1"          # Syslog server IP
    next_hop        "192.168.2.1"          # Expected next-hop IP
    snmp_community  "SnMpSeRvErPaSsWoRd"   # SNMP community string
    snmp_timeout    15                      # SNMP timeout in seconds
    snmp_retry      2                       # SNMP retry count
    flap_threshold  60                      # Seconds before route is considered stable
    debug_mode      0                       # Set to 1 for verbose output
}
```

## 📊 Alert Types

### SNMP Traps Generated

| Trap Type | OID | Description | Severity |
|-----------|-----|-------------|----------|
| NO_BGP_MAIN_ROUTE | 1.3.6.1.4.1.9.9.187.2.0.1 | Route missing from table | Critical |
| BGP_MAIN_ROUTE_FLAPPING | 1.3.6.1.4.1.9.9.187.2.0.1 | Route age < threshold | Warning |
| BGP_MAIN_ROUTE_RECOVERED | 1.3.6.1.4.1.9.9.187.2.0.1 | Route became stable | Info |

### Syslog Messages

```
%WATCHNET-2-CRITICAL: Main route 10.1.1.1/32 via 192.168.2.1 NOT IN TABLE
%WATCHNET-3-WARNING: Main route 10.1.1.1/32 via 192.168.2.1 FLAPPING (age: 15s)
%WATCHNET-5-NOTICE: Main route 10.1.1.1/32 via 192.168.2.1 RECOVERED
```

## 🔍 Troubleshooting

### Enable Debug Mode

Edit the script and set `debug_mode` to 1:
```tcl
array set config {
    ...
    debug_mode      1
    ...
}
```

### Check EEM Policy Status

```cisco
show event manager policy registered
show event manager history events
```

### View Script Logs

```cisco
show logging | include WATCHNET
```

### Manual Route Check

```cisco
show ip route 10.1.1.1
show ip bgp 10.1.1.1/32
show ip bgp summary
```

### Common Issues

1. **Script not executing**: Verify TCL is enabled and script permissions
2. **No SNMP traps**: Check SNMP configuration and community string
3. **No syslog messages**: Verify syslog configuration and connectivity
4. **False positives**: Adjust `flap_threshold` value

## 📁 File Structure

```
watchnet-bgp-monitor/
├── README.md                          # This file
├── watchnet-bgp-monitor.tcl          # Basic monitoring script
├── watchnet-bgp-monitor-enhanced.tcl # Enhanced version with more features
└── examples/
    ├── eem-config.txt                # EEM configuration examples
    └── snmp-config.txt               # SNMP configuration examples
```

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request. For major changes, please open an issue first to discuss what you would like to change.

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- Cisco TCL Scripting Documentation
- Cisco EEM Configuration Guide
- Network Engineering Community

## 📞 Support

For issues, questions, or contributions, please create an issue in the GitHub repository.

---

**Note**: Always test scripts in a lab environment before deploying to production routers. Ensure you have proper change management procedures in place.
