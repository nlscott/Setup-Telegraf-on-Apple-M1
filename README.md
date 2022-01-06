# Setup-Telegraf-on-Apple-M1

I'm setting up a couple of Apple M1 Build servers and part of the setup is pushing metrics into Grafana with Telegraf.  Here's my basic install and setup.



 ## Download and Install Telegraf

Down the installer from: https://portal.influxdata.com/downloads/

Because this is a M1 Mac mini, I selected the `macOS ARM64 (via singed .dmg)`

Expand dmg and copy Telegraf.app to the  `/Applications` folder

Open the `Telegraf.app` and allow permissions to teminal



## Set config File

Copy from a previous install or create `/Applications/Telegraf.app/Contents/Resources/etc/telegraf/telegraf.conf` . I used the defaults below. Theres nothing fancy going on here, I'm using the defaults from the example conf file.

```bash
# Telegraf Configuration

# Global tags can be specified here in key="value" format.
[global_tags]

# Configuration for telegraf agent
[agent]
  interval = "60s"
  round_interval = true
  metric_batch_size = 1000
  metric_buffer_limit = 10000
  collection_jitter = "0s"
  flush_interval = "10s"
  flush_jitter = "0s"
  precision = "s"
  hostname = ""
  omit_hostname = false

###############################################################################
#                            OUTPUT PLUGINS                                   #
###############################################################################
[[outputs.influxdb]]
  urls = ["http://servername:8086"]
  database = "telegraf"
  timeout = "5s"

###############################################################################
#                            INPUT PLUGINS                                    #
###############################################################################
# Read metrics about cpu usage
# [[inputs.cpu]]

# Read metrics about disk usage by mount point
[[inputs.disk]]
  ## Ignore mount points by filesystem type.
  ignore_fs = ["tmpfs", "devtmpfs", "devfs", "iso9660", "overlay", "aufs", "squashfs"]

# Get kernel statistics from /proc/stat
[[inputs.kernel]]
  # no configuration

# Read metrics about memory usage
[[inputs.mem]]
  # no configuration

# Get the number of processes and group them by status
[[inputs.processes]]
  # no configuration

# Read metrics about swap memory usage
[[inputs.swap]]
  # no configuration

# Read metrics about system load & uptime
[[inputs.system]]
  ## Uncomment to remove deprecated metrics.
  # fielddrop = ["uptime_format"]
```



## Launch at Startup

Then I create a  Launchagent for the user running telegraf. I copy the below and put it at `~/Library/Launchagents/com.telegraf.plist`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>EnvironmentVariables</key>
	<dict>
		<key>PATH</key>
		<string>/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/Library/Apple/usr/bin:/usr/local/sbin</string>
	</dict>
	<key>KeepAlive</key>
	<dict>
		<key>SuccessfulExit</key>
		<true/>
	</dict>
	<key>Label</key>
	<string>com.telegraf</string>
	<key>ProgramArguments</key>
	<array>
		<string>/Applications/Telegraf.app/Contents/Resources/usr/bin/telegraf</string>
		<string>--config</string>
		<string>/Applications/Telegraf.app/Contents/Resources/etc/telegraf/telegraf.conf</string>
	</array>
	<key>RunAtLoad</key>
	<true/>
</dict>
</plist>

```



Then I reboot the machine and log into my Grafana server. Duplicate an existing Dashboard from another build server and then wait a couple minutes and refresh until I see the new hostname pop up.