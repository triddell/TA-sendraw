# TA-sendraw

TA-sendraw provides a custom Splunk search command, `sendraw`, for sending raw events over TCP or UDP. The underlying command uses the Splunk Custom Search Command protocol, version 2.

## Supported Splunk Versions

The `sendraw` custom command will only work with 6.4.x or higher versions of Splunk. Version 2 of the Splunk Custom Search Command protocol was introduced as a technology preview with Splunk 6.4.

## System Requirements

The `sendraw` command is a binary application written in [Go](https://golang.org). The TA has binaries included for the following platforms:
* Darwin (OS X) 64-bit
* Linux 64-bit
* Windows 64-bit

## Installation

Use normal Splunk installation procedures to install the TA. This includes installation through the Splunk UI or "unzipping" and copying to `$SPLUNK_HOME/etc/apps`. The application is a Splunk .spl file, which can be renamed to .tgz and decompressed.

Released versions are available on GitHub: [`TA-sendraw`](https://github.com/triddell/TA-sendraw/releases)

## Configuration

Configuration of `sendraw` is made through a custom conf file, specifically `sendraw.conf`. This file *must* be available in `$SPLUNK_HOME/etc/apps/TA-sendraw/local`. The `local` directory will need to be created after installing the application.

In the `local` directory, create the `sendraw.conf` file.

Here is an example configuration file:

```
[general]
#logLevel = debug
#timeout = 5s

[rawtarget:tcp]
server = <your_server_ip>:10520
protocol = tcp

[rawtarget:udp]
server = <your_server_ip>:10521
protocol = udp
```

### General Stanza

The `[general]` stanza above is not required but it does include some commented configurations that can be used to modify the behavior of `sendraw`.

Valid values for `logLevel` include `debug`, `info`, `warn`, and `error`. The default is `warn`. Most log messages are either `debug` or `error`.

The default `timeout` is `10s` (ten seconds). Valid, but not necessarily good, values are available from the Go `time.ParseDuration` function [documentation](https://golang.org/pkg/time/#ParseDuration).

### "Target" Stanzas

The "target" stanzas can be named anything. The examples append `:tcp` and `:udp` but a stanza could just as easily be `[foobar]`.

Within a target stanza, it's pretty basic. Name the stanza what you want. Then add a `server` value which includes a hostname or IP address followed by a port number (separated by a colon). For `protocol`, choose either `tcp` or `udp`.

### Windows Event Support for QRadar

An additional configuration option is supported specifically for sending raw Windows events from Splunk to QRadar. Add the following to a target stanza for this support:

```
windowsHeader = true
```

### Usage

`sendraw` is a custom Splunk search command, so it's usually appended to a Splunk search. Here's a full example of a search command using `sendraw`:

```
index=_internal sourcetype=splunkd ERROR
| sendraw rawtarget:tcp
```

The base search, `index=_internal sourcetype=splunkd ERROR`, is simple and should work on nearly any Splunk system. If the search returns either zero records or many thousands, you may want to refine it to a managable number for your initial testing.

The final portion of the search, `| sendraw rawtarget:tcp`, is what calls the `sendraw` custom search command. A configured stanza name is the sole command argument.

Here is an example record that would get sent to the target host and port over either TCP or UDP:

```
06-05-2017 10:04:52.873 -0600 ERROR AuthenticationManagerSplunk - Login failed. Incorrect login for user: admin
```

This record is exactly like the `_raw` field value from a Splunk event.

`sendraw` can be used as an ad hoc search command but it will often be used to automate data integration with third-party servers using Splunk saved/scheduled searches.

### Testing

To faciliate testing using a local Splunk instance, a related TA has been developed. `TA-sendtest` provides test target indexes, TCP and UDP Splunk inputs, and a `sendraw.conf.sample` similar to what was used in this documentation.

Further information on this application is available on GitHub: [`TA-sendtest`](https://github.com/triddell/TA-sendtest)

### Troubleshooting

`sendraw` logs events at: `$SPLUNK_HOME/var/log/splunk/sendraw.log`

These events can be searched with Splunk using: `index=_internal source=*sendraw.log`

### Related Application

A similar application has been developed for sending CEF-formatted Splunk events to third-party servers. Further information on this application is available on GitHub: [`TA-sendcef`](https://github.com/triddell/TA-sendcef)