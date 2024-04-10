# Java DoS Protection Service Using YAML

  

[Python](https://web.archive.org/web/20160424230922/https://cloud.google.com/appengine/docs/python/config/dos "View this page in the Python runtime") <span style="color: #ddd; padding: 0em .5em;">|</span><span style="color: black; font-weight:bold">Java</span> <span style="color: #ddd; padding: 0em .5em;">|</span>[PHP](https://web.archive.org/web/20160424230922/https://cloud.google.com/appengine/docs/php/config/dos "View this page in the PHP runtime") <span style="color: #ddd; padding: 0em .5em;">|</span>[Go](https://web.archive.org/web/20160424230922/https://cloud.google.com/appengine/docs/go/config/dos "View this page in the Go runtime")

The App Engine Denial of Service (DoS) Protection Service enables you to protect your application from running out of quota when subjected to denial of service attacks or similar forms of abuse. You can blacklist IP addresses or subnets, and requests routed from those addresses or subnets will be dropped before your application code is called. No resource allocations, billed or otherwise, are consumed for these requests.

Do not use this service for security. It is designed for quantitative abuse prevention, such as preventing DoS attacks, only. Some requests from blacklisted users may still get through to your application.

1.  [About dos.yaml](#Java_app_yaml_About_dos_yaml)
2.  [Limits](#Java_app_yaml_Limits)
3.  [Uploading DoS configuration](#Java_app_yaml_Uploading_DoS_configuration)

## About dos.yaml

A `dos.yaml` file in the `WEB-INF` directory of your application configures DoS Protection Service blacklists for your application. The following is an example `dos.yaml` file:

```
blacklist:
- subnet: 1.2.3.4
  description: a single IP address
- subnet: 1.2.3.4/24
  description: an IPv4 subnet
- subnet: abcd::123:4567
  description: an IPv6 address
- subnet: abcd::123:4567/48
  description: an IPv6 subnet
```

The syntax of `dos.yaml` is the YAML format. For more information about this syntax, see [the YAML website](https://web.archive.org/web/20160424230922/http://www.yaml.org/).

A `dos.yaml` file consists of a number of blacklist entries. A blacklist entry has a `subnet`, and can optionally specify a `description`. The `subnet` is any valid IPv4 or IPv6 subnet in [CIDR](https://web.archive.org/web/20160424230922/https://wikipedia.org/wiki/CIDR_notation) notation.

## Limits

You may define a maximum of 100 blacklist entries in your configuration file. Uploading a configuration file with more than 100 entries will fail.

## Uploading DoS configuration

You can use `AppCfg` to upload DoS configs. When you upload your application to App Engine using `AppCfg update`, the DoS Protection Service is updated with the contents of `dos.yaml`.

You can update just the DoS configuration without uploading the rest of the application using the following command:

```
`AppCfg update_dos` <directory>
```

To delete all blacklist entries, change the `dos.yaml` file to just contain:

```
blacklist:
```
