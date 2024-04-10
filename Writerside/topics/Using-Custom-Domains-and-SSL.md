# Using Custom Domains and SSL

  

[Python](https://web.archive.org/web/20160424231054/https://cloud.google.com/appengine/docs/python/console/using-custom-domains-and-ssl "View this page in the Python runtime") <span style="color: #ddd; padding: 0em .5em;">|</span><span style="color: black; font-weight:bold">Java</span> <span style="color: #ddd; padding: 0em .5em;">|</span>[PHP](https://web.archive.org/web/20160424231054/https://cloud.google.com/appengine/docs/php/console/using-custom-domains-and-ssl "View this page in the PHP runtime") <span style="color: #ddd; padding: 0em .5em;">|</span>[Go](https://web.archive.org/web/20160424231054/https://cloud.google.com/appengine/docs/go/console/using-custom-domains-and-ssl "View this page in the Go runtime")

App Engine allows applications to be served via a custom domain, such as `example.com`, instead of an `appspot.com` address. You can use a custom domain with or without SSL.

This page describes how to use a custom domain with an App Engine app and, optionally, how to set up SSL for the custom domain. It assumes that you already have an App Engine project created and deployed.

SSL support for your App Engine application goes above and beyond basic SSL by offering globally-distributed SSL endpoints and built-in load balancing to serve your app securely, reliably, and quickly to a worldwide audience.

## Contents

1.  [Adding a custom domain for your application](#adding_a_custom_domain_for_your_application)
    1.  [Using subdomains](#using_subdomains)
    2.  [Wildcard mappings](#wildcard_mappings)
    3.  [Using Strict-Transport-Security headers in a custom domain](#using_strict-transport-security_headers_in_a_custom_domain)
2.  [Adding SSL to your custom domain](#adding_ssl_to_your_custom_domain)
3.  [Obtaining a certificate](#obtaining_a_certificate)
    1.  [More about App Engine support for SSL certificates](#more_about_app_engine_support_for_ssl_certificates)
4.  [Purchasing a virtual IP (VIP)](#purchase_a_vip)
    1.  [VIP versus SNI](#vip_versus_sni)
    2.  [How to purchase](#how_to_purchase)

## Adding a custom domain for your application

To add a custom domain for your App Engine application:

1.  Purchase a new domain, unless you already have one you want to use. You can use any domain name registrar, including [Google Domains](https://web.archive.org/web/20160424231054/https://domains.google.com/).

2.  Add the domain to your app by clicking **Add a custom domain** in the [custom domains](https://web.archive.org/web/20160424231054/https://console.cloud.google.com/appengine/settings/domains) console page for your project: to display the *Add a new custom domain* form:

    ![Add a custom domain](https://web.archive.org/web/20160424231054im_/https://cloud.google.com/appengine/docs/images/domain_add2.png)

3.  In the *Add new custom domain* form, verify that you own the custom domain you want to use:

    1.  Supply the domain name ("example.com") and click **Verify**: this opens up a new tab titled *Webmaster Central*:
    2.  Follow the directions provided by the *Webmaster Central* tab, then click **Verify**.
    3.  Return to the *Add new custom domain* form in the Cloud Platform Console.

4.  Continue to the next step the *Add new custom domain* form, selecting the custom domain you want to point to your App Engine app:

    1.  Refresh the console domain page so it will list the domains properly.
    2.  If you want to use a subdomain, such as `www`, use the second option (`http://www.example.com`).
    3.  If you want to use a [naked domain](https://web.archive.org/web/20160424231054/http://www.pcmag.com/encyclopedia/term/62630/naked-domain), use the first option to specify a naked domain (such as `http://example.com`).
    4.  Click **Add** to create the desired mapping.

5.  In the final step in the *Add new custom domain* form, note the resource records listed along with their type (CNAME in this example) and the canonical name (`ghs.googlehosted.com` in this example). You'll need to add the resource records listed to the DNS configuration of your custom domain:

    1.  Log into your account at your domain registrar and open the DNS configuration page.
    2.  In the host records section, add each of the records listed in the *Add new custom domain* form. Typically you list the host name along with the canonical name as the address. In our example, we would add the host name `www` with the address `ghs.googlehosted.com`. (If you use a naked domain, you would use `@` as the host name and a corresponding value from the *Add new custom domain* form.)
    3.  Save your changes in your domain account. It may take awhile for these changes to take effect.

6.  Test for success by browsing to your app via its new custom domain URL: `www.example.com`.

### Using subdomains

If you set up a wildcard subdomain mapping for your custom domain, then your application serves requests for any subdomain that matches.

-   If the user browses a domain that matches an application version name or module name, the application serves that version.
-   If the user browses a domain that matches a module name, the application serves that module.

### Wildcard mappings

Note that wildcard mappings will work with App Engine modules.

You can use wildcards to map subdomains at any level, starting at third-level subdomains. For example, if your domain is `example.com` and you enter text in the web address field:

-   Entering `*` maps all subdomains of `example.com` to your app.
-   Entering `*.private` maps all subdomains of `private.example.com` to your app.
-   Entering `*.nichol.sharks.nhl` maps all subdomains of `nichol.sharks.nhl.example.com` to your app.
-   Entering `*.excogitate.system` maps all subdomains of `excogitate.system.example.com` to your app.

If you use Google Apps with other subdomains on your domain, such as `sites` and `mail`, those mappings have higher priority and are matched first, before any wildcard mapping takes place. In addition, if you have other App Engine apps mapped to other subdomains, those mappings also have higher priority than any wildcard mapping.

Note that some DNS providers might not work with wildcard subdomain mapping. In particular, a DNS provider must permit wildcards in CNAME host entries.

#### Wildcards and modules/versions

The above wildcard routing rules apply to URLs that contain components for modules, versions, and instances as well, following the module routing rules for the App Engine  
([Java](https://web.archive.org/web/20160424231054/https://cloud.google.com/appengine/docs/java/modules/routing#routing_via_url) | [Python](https://web.archive.org/web/20160424231054/https://cloud.google.com/appengine/docs/python/modules/routing#routing_via_url) | [Go](https://web.archive.org/web/20160424231054/https://cloud.google.com/appengine/docs/go/modules/routing#routing_via_url) | [PHP](https://web.archive.org/web/20160424231054/https://cloud.google.com/appengine/docs/php/modules/routing#routing_via_url)) SDK.

### Using Strict-Transport-Security headers in a custom domain

You cannot use `Strict-Transport-Security` headers unless your domain is whitelisted. To place your domain in the whitelist, contact `lgarron@chromium.org`.

## Adding SSL to your custom domain

To enable SSL for the custom domain used with your App Engine application:

1.  Make sure you have already [set up your custom domain](#adding_a_custom_domain_for_your_application) in your App Engine project.

2.  Get a certificate for your domain from the certificate authority of your choice. The exact procedure can vary depending on the authority: see [Obtaining a certificate](#obtaining_a_certificate) for details.

3.  Open your App Engine project in the [Google Cloud Platform Console](https://web.archive.org/web/20160424231054/https://console.cloud.google.com/).

4.  Ensure you have the right permissions. In order to upload an SSL certificate, you must have [verified ownership](#adding_a_custom_domain_for_your_application) (step 4) of all domains that it may serve, or their parent domains. For example:

    -   If the certificate is for `www.example.com` you may verify ownership of either `www.example.com` or `example.com`.
    -   If the certificate is for `www.example.com` and `sub.example.com` you can either verify ownership of both `www.example.com` and `sub.example.com`, or of `example.com`.
    -   If the certificate is for `*.example.com` you must verify ownership of `example.com`.

5.  Upload the SSL certificate to be used with your domain by clicking **Upload a new certificate** in the [SSL certificates](https://web.archive.org/web/20160424231054/https://console.cloud.google.com/appengine/settings/certificates) page.

    ![Upload a cert](https://web.archive.org/web/20160424231054im_/https://cloud.google.com/appengine/docs/images/upload_cert.png)

    1.  Follow the prompts to upload the public certificate and the unencrypted private key. (See the last step under [Obtaining a certificate](#obtaining_a_certificate) for more details on what these are and how to get them.)
    2.  Click **Upload**.

6.  Select the certificate you want to configure in the list displayed in the [SSL certificates](https://web.archive.org/web/20160424231054/https://console.cloud.google.com/appengine/settings/certificates) page for your App Engine project, then select the domain you are enabling for SSL and click **Save**.

7.  Test your changes by visiting your custom domain in your browser, using `https`, for example, `https://www.example.com`.

## Obtaining a certificate

**Important:** These instructions assume you have already [set up a custom domain](#adding_a_custom_domain_for_your_application) for your App Engine application. If you haven't done that yet, you'll need to do it before proceding.

The process for getting an SSL certificate will vary depending on the certificate authority you use. The instructions provided here might need to be adjusted slightly. Typically, each certificate authority provides instructions to lead you through the process.

To obtain a certificate for use with your App Engine app:

1.  Generate a certificate signing request (CSR):

    1.  Invoke the [openssl](https://web.archive.org/web/20160424231054/https://www.openssl.org/source/) tool:

        ```
        openssl req -nodes -newkey rsa:2048 -keyout myserver.key -out server.csr
        ```

    2.  When prompted, supply

        -   your 2 digit country code (e.g., US for United States),
        -   your city name
        -   your company name (or your own name if you don't have a company)
        -   your organizational unit or NA if you don't have this
        -   your domain name: www.example.com
        -   your email address

    You don't need to supply the other, optional values. The resulting CSR file is `server.csr`, which you'll use in a moment.

2.  Locate the certificate authority (CA) that you want to use, such as [Thawte](https://web.archive.org/web/20160424231054/https://www.thawte.com/), [Comodo](https://web.archive.org/web/20160424231054/https://www.comodo.com/), or other certificate authority, and purchase a certificate.

3.  The CA will request the contents of your CSR file: copy and paste the contents of `server.csr` as directed by your CA.

4.  The CA will request domain owner approval: follow the prompts to do this. You might find it easiest to use the email approval method, in which case you may need to set up an expected email address at your domain account so you can receive the approval request email from the CA and follow the instructions provided in that email. A typical expected address is `admin@example.com`.

5.  After you provide domain owner approval, the CA sends the certificate to you, typically as a `.zip` file: unzip this file.

6.  Convert the private key you created above (`myserver.key`) into the format expected by App Engine, unencrypted:

    ```
    openssl rsa -in myserver.key -out myserver.key.pem
    ```

7.  Concatenate all of the `.crt` files from your CA into one file, using this command:

    ```
    cat www_example_com.crt ASecureServerCA.crt ATrustCA.crt ATrustExternal.crt > concat.crt
    ```

    When you upload the private key and the concatenated certs using the Google Cloud Platform Console [SSL](https://web.archive.org/web/20160424231054/https://console.cloud.google.com/appengine/settings/certificates) page within the App Engine settings for your project, in the file selector for **private key**, supply the private key file you just converted into a pem file: `myserver.key.pem`. And in the file selector for **PEM encoded X.509 public key certificate**, supply the concatenated file containing all the certificates from your CA: `concat.crt`

To verify that the private key and certificate [match](https://web.archive.org/web/20160424231054/http://httpd.apache.org/docs/2.0/ssl/ssl_faq.html#verify), use the following commands:

```
    openssl x509 -noout -modulus -in concat.crt | openssl md5
    openssl rsa -noout -modulus -in myserver.key.pem | openssl md5
```

Both commands should return the same output.

To verify that a certificate and its CA chain is valid, the [openssl verify](https://web.archive.org/web/20160424231054/https://www.openssl.org/docs/manmaster/apps/verify.html) command may be useful:

```
    openssl verify -verbose -CAfile concat.crt  concat.crt
```

### More about App Engine support for SSL certificates

App Engine supports the following certificate types:

-   Single Domain/Hostname
-   Self-signed
-   Wildcard
-   Subject Alternative Name (SAN) / Multi Domain

It requires some things of your certificates and keys:

-   Private Key and Certificate should be uploaded in PEM format.
-   Private Keys must not be encrypted.
-   A certificate file can contain at most five certificates; this number includes chained and intermediate certificates.
-   All subject names on the host certificate should match or be subdomains of the user's verified domains.
-   Private keys must use RSA encryption.
-   **Maximum allowed key modulus: 2048 bits**

If the host certificate requires an intermediate or chained certificate (as many Certificate Authorities (CAs) issue), you will need to append the intermediate or chained certificates to the end of the public certificate file.

Some App Engine features use special subdomains. For example, an application can use subdomains to address application modules, or to address different versions of your application. To use these with SSL, it makes sense to set up a SAN or wildcard certificate. Wildcard certificates only support one level of subdomain.

## Purchasing a virtual IP (VIP)

**Important:** these instructions assume you have already [set up a custom domain](#adding_a_custom_domain_for_your_application) for your app and have uploaded your SSL certificate (see [Adding SSL to your custom domain](#adding_ssl_to_your_custom_domain)).

### VIP versus SNI

By default, App Engine SSL support uses server name indication (SNI): you don't need to set up or purchase anything to use it. Server Name Indication is a feature that extends SSL and [TLS](https://web.archive.org/web/20160424231054/https://wikipedia.org/wiki/Transport_Layer_Security). This extension allows multiple domains to share the same IP address while still allowing separate valid certificates for all the domains. Some older browsers and operating systems don't support SNI, most notably Internet Explorer and Safari on Windows XP and the default Android browser pre-Honeycomb. When a user visits an SNI site with a client that does not support SNI they will be unable to view the page when connecting via HTTPS. We recommend detecting browsers that do not support SNI and recommending a browser that supports it.

However, if you wish, you could alternatively use a virtual IP (VIP). A VIP is a dedicated IP address assigned for your application. This allows TLS to be used without the SNI extension and as such it will work on any browser or OS that supports SSL. Each VIP only supports one certificate. The Virtual IP address may change and therefore you should not use DNS A records; instead, use a CNAME record to avoid any issues caused by Virtual IP changes.

Be aware that SNI slots are free; VIPs are not. If you need to use a virtual IP (VIP) you will need to purchase it first: these are about $39 per month per VIP.

### How to purchase

To purchase a virtual IP address:

1.  Set up a [Google Apps account](https://web.archive.org/web/20160424231054/https://www.google.com/work/apps/business/) for your custom domain. When you sign up for this account, you must use your custom domain email as your login, for example, `admin@www.example.com`.

2.  Enable billing for your application in the Google Cloud Platform Console [billing](https://web.archive.org/web/20160424231054/https://console.cloud.google.com/settings) page.

3.  Set the daily [spending limit](https://web.archive.org/web/20160424231054/https://cloud.google.com/appengine/pricing/#spending_limit) high enough to pay for your VIPs, using the [application settings](https://web.archive.org/web/20160424231054/https://console.cloud.google.com/appengine/settings) page.

4.  Register the email address you used for your Google Apps account as a project owner in the [Permissions](https://web.archive.org/web/20160424231054/https://console.cloud.google.com/permissions/projectpermissions) page for your App Engine project.

5.  Log into the [Google Apps admin console](https://web.archive.org/web/20160424231054/https://admin.google.com/).

6.  Click the *Security* icon.

7.  Click **Show more** &gt; **SSL for App Engine Apps** to display the VIP form:

    ![VIP form](https://web.archive.org/web/20160424231054im_/https://cloud.google.com/appengine/docs/images/buy_vip.png)

8.  Click **Add a VIP** once for each VIP you wish to add. You will be charged monthly for each VIP. Note that if you add a VIP and then remove it immediately, you will still be billed for an entire day.

9.  Return to the [SSL certificates](https://web.archive.org/web/20160424231054/https://console.cloud.google.com/appengine/settings/certificates) page in the Google Cloud Platform Console, then click on the certificate you are using with this VIP.

10. In the dropdown menu under *Select SSL serving mode*, select **SNI+VIP**:

    ![serve as VIP](https://web.archive.org/web/20160424231054im_/https://cloud.google.com/appengine/docs/images/vip_cert.png)

11. Click **Save**.

12. Locate the CNAME that is displayed under the cert. You'll need to log into your domain account and open the DNS configuration page, then copy this value into the CNAME records for all domains using the certificate. Allow time for the DNS changes to propagate out.

Your SSL cert will now be served as a VIP.
