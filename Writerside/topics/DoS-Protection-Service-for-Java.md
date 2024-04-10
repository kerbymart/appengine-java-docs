# DoS Protection Service for Java

  

[Python](https://web.archive.org/web/20160424230240/https://cloud.google.com/appengine/docs/python/config/dos "View this page in the Python runtime") <span style="color: #ddd; padding: 0em .5em;">|</span><span style="color: black; font-weight:bold">Java</span> <span style="color: #ddd; padding: 0em .5em;">|</span>[PHP](https://web.archive.org/web/20160424230240/https://cloud.google.com/appengine/docs/php/config/dos "View this page in the PHP runtime") <span style="color: #ddd; padding: 0em .5em;">|</span>[Go](https://web.archive.org/web/20160424230240/https://cloud.google.com/appengine/docs/go/config/dos "View this page in the Go runtime")

The App Engine Denial of Service (DoS) Protection Service enables you to protect your application from running out of quota when subjected to denial of service attacks or similar forms of abuse. You can blacklist IP addresses or subnets, and requests routed from those addresses or subnets will be dropped before your application code is called. No resource allocations, billed or otherwise, are consumed for these requests.

Do not use this service for security. It is designed for quantitative abuse prevention, such as preventing DoS attacks, only. Some requests from blacklisted users may still get through to your application.

1.  [About dos.xml](#Java_appengine_web_xml_About_dos_xml)
2.  [Limits](#Java_appengine_web_xml_Limits)
3.  [Uploading DoS configuration](#Java_appengine_web_xml_Uploading_DoS_configuration)

By default, App Engine serves a generic error page to blacklisted addresses. You can configure your app to serve a custom response instead. For details, see [Custom Error Responses](https://web.archive.org/web/20160424230240/https://cloud.google.com/appengine/docs/java/config/appconfig#Java_appengine_web_xml_Custom_error_responses).

## About dos.xml

A `dos.xml` file in the `WEB-INF` directory of your application (alongside `appengine-web.xml`) configures DoS Protection Service blacklists for your application. The following is an example `dos.xml` file:

```
<?xml version="1.0" encoding="UTF-8"?>
<blacklistentries>
  <blacklist>
    <subnet>1.2.3.4</subnet>
    <description>a single IP address</description>
  </blacklist>
  <blacklist>
    <subnet>1.2.3.4/24</subnet>
    <description>an IPv4 subnet</description>
  </blacklist>
  <blacklist>
    <subnet>abcd::123:4567</subnet>
    <description>an IPv6 address</description>
  </blacklist>
  <blacklist>
    <subnet>abcd::123:4567/48</subnet>
    <description>an IPv6 subnet</description>
  </blacklist>
</blacklistentries>
```

For an XSD describing the format, check the file `docs/dos.xsd` in the SDK.

A `dos.xml` file consists of a number of blacklist entries. A blacklist entry has a `<subnet>`, and can optionally specify a `<description>`. The `<subnet>` is any valid IPv4 or IPv6 subnet in [CIDR](https://web.archive.org/web/20160424230240/https://wikipedia.org/wiki/CIDR_notation) notation.

## Limits

You may define a maximum of 100 blacklist entries in your configuration file. Uploading a configuration file with more than 100 entries will fail.

## Uploading DoS configuration

You can use `AppCfg` to upload DoS configs. When you upload your application to App Engine using `AppCfg update`, the DoS Protection Service is updated with the contents of `dos.xml`.

You can update just the DoS configuration without uploading the rest of the application using the following command:

```
`AppCfg update_dos` <directory>
```

To delete all blacklist entries, change the `dos.xml` file to just contain:

```
<?xml version="1.0" encoding="UTF-8"?>
<blacklistentries/>
```
