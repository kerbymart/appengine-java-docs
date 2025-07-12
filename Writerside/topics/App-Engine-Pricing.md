# App Engine Pricing

App Engine applications run as instances within the standard environment, available in General Availablity (GA), or the [flexible environment](https://web.archive.org/web/20160424224456/https://cloud.google.com/appengine/docs/flexible/), available in Beta.

Instances within the the standard environment have access to a daily limit of resource usage that is provided at no charge defined by a set of [quotas](https://web.archive.org/web/20160424224456/https://cloud.google.com/appengine/docs/quotas). Beyond that level, applications will incur charges as outlined below.

Instances within the flexible environment are charged the [cost of the underlying Google Compute Engine Virtual Machines](https://web.archive.org/web/20160424224456/https://cloud.google.com/compute/pricing). This is Beta-only pricing and will increase once the flexible environment is GA. All other services and APIs are priced as described below.

To control your application costs, you can set a [spending limit](#spending_limit).

When charging in local currency, Google will convert the prices listed into applicable local currency pursuant to the conversion rates published by leading financial institutions.

1.  [Resource billing rates](#Billable_Resource_Unit_Costs)
    -   [Costs for Datastore Calls](#costs-for-datastore-calls)
    -   [Costs for Search](#search_pricing)
2.  [Managing the billing settings](#managing)
    -   [Billing settings](#billing_settings)
    -   [Spending limits](#spending_limit)
3.  [Understanding billing](#understanding_billing)
    -   [Daily and monthly charges](#cost_cycles)
    -   [Taxes](#cost_taxes)
    -   [Grace periods](#status)

## Resource billing rates

<table>
<thead>
<tr class="header">
<th>Resource</th>
<th>Unit</th>
<th>Unit cost (in US $)</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Standard Runtime Instances<sup>*</sup></td>
<td>Instance hours</td>
<td>$0.05</td>
</tr>
<tr class="even">
<td>Outgoing Network Traffic</td>
<td>Gigabytes</td>
<td>$0.12</td>
</tr>
<tr class="odd">
<td>Incoming Network Traffic</td>
<td>Gigabytes</td>
<td>Free</td>
</tr>
<tr class="even">
<td>Datastore Storage</td>
<td>Gigabytes per month</td>
<td>$0.18</td>
</tr>
<tr class="odd">
<td>Blobstore, Logs, and Task Queue Stored Data</td>
<td>Gigabytes per month</td>
<td>$0.026</td>
</tr>
<tr class="even">
<td>Dedicated Memcache</td>
<td>Gigabytes per hour</td>
<td>$0.06</td>
</tr>
<tr class="odd">
<td>Logs API</td>
<td>Gigabytes</td>
<td>$0.12</td>
</tr>
<tr class="even">
<td>SSL Virtual IPs<sup>**</sup> (VIPs)</td>
<td>Virtual IP per month</td>
<td>$39.00</td>
</tr>
<tr class="odd">
<td colspan="2">Sending Email, Shared Memcache, Cron, APIs (URLFetch, Task Queues, Image, Sockets, Files, Users, and Channel)</td>
<td>No Additional Charge</td>
</tr>
</tbody>
</table>

<sup>\*</sup> Frontend and backend instances cost the same. Price may vary based on the [performance settings associated with a module](https://web.archive.org/web/20160424224456/https://cloud.google.com/appengine/docs/python/modules/#instance_classes).  
<sup>\*\*</sup> SNI SSL certificate slots are offered for no additional charge.

**Note:** If your free app exceeds the free quota for some resource needed to initiate a request, such as the bandwidth quota or instance hours quota, users of the app will get a server error, such as a 503 error. For all other services, exceeding quota from a free app will generate a quota exception that your app can handle gracefully by showing a message to the users. See [When a Resource is Depleted](https://web.archive.org/web/20160424224456/https://cloud.google.com/appengine/docs/quotas#When_a_Resource_is_Depleted) for more details.

**Note:** Please note that there is an initial start up cost of 15 minute instance time for each instance start up.

### Costs for Google Cloud Datastore Calls

Cloud Datastore operations are billed as follows:

<table>
<thead>
<tr class="header">
<th>Operation</th>
<th>Cost</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Read / Write</td>
<td>$0.06 per 100,000 operations</td>
</tr>
<tr class="even">
<td>Small</td>
<td>Free</td>
</tr>
</tbody>
</table>

Calls to the Datastore API result in the following billable operations. Small datastore operations include calls to allocate datastore ids or keys-only queries. These operations are free. This table shows how calls map to Datastore operations:

<table>
<thead>
<tr class="header">
<th>API Call</th>
<th>Cloud Datastore Operations</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Entity Get (per entity)</td>
<td>1 read</td>
</tr>
<tr class="even">
<td>New Entity Put (per entity, regardless of entity size)</td>
<td>2 writes + 2 writes per indexed property value + 1 write per composite index value</td>
</tr>
<tr class="odd">
<td>Existing Entity Put (per entity)</td>
<td>1 write + 4 writes per modified indexed property value + 2 writes per modified composite index value</td>
</tr>
<tr class="even">
<td>Entity Delete (per entity)</td>
<td>2 writes + 2 writes per indexed property value + 1 write per composite index value</td>
</tr>
<tr class="odd">
<td>Query<sup>*</sup></td>
<td>1 read + 1 read per entity retrieved</td>
</tr>
<tr class="even">
<td>Projection Query</td>
<td>1 read</td>
</tr>
<tr class="odd">
<td>Projection Query with "Distinct" option</td>
<td>1 read + 1 read per entity retrieved</td>
</tr>
</tbody>
</table>

<sup>\*</sup> Queries that specify an offset are charged for the number of results that are skipped in addition to those that are returned.

#### New Cloud Datastore Pricing Starting July 1st, 2016

On July 1, 2016, Google Cloud Datastore pricing will change from charging per operation to charging per entity. This much simpler pricing means it will cost significantly less to use the full power of Google Cloud Datastore.

For example, in the current pricing, writing a new entity with 1 indexed property would cost 4 write operations. In the new pricing, it would cost only 1 entity write. Similarly, deleting this entity in the current pricing would cost 4 write operations, but in the new pricing it would cost only 1 entity delete.

<table>
<thead>
<tr class="header">
<th>Resource</th>
<th>Free Quota per day</th>
<th>Pricing beyond the free quota</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Stored data</td>
<td>1 GB</td>
<td>$0.18 / GB / month</td>
</tr>
<tr class="even">
<td>Entity Reads</td>
<td>50,000</td>
<td>$0.06 per 100,000 entities</td>
</tr>
<tr class="odd">
<td>Entity Writes</td>
<td>20,000</td>
<td>$0.18 per 100,000 entities</td>
</tr>
<tr class="even">
<td>Entity Deletes</td>
<td>20,000</td>
<td>$0.02 per 100,000 entities</td>
</tr>
<tr class="odd">
<td>Small Operations</td>
<td>Unlimited. Includes calls to allocate Datastore IDs or keys-only queries.</td>
<td>Not applicable.</td>
</tr>
</tbody>
</table>

Effective with this price change, [standard network costs](https://web.archive.org/web/20160424224456/https://cloud.google.com/compute/pricing#internet_egress) will apply.

### Costs for Search

Fees for use of the Search API are listed in the table below. Refer to the [Java](https://web.archive.org/web/20160424224456/https://cloud.google.com/appengine/docs/java/search/#Java_Search_API_quotas) and [Python](https://web.archive.org/web/20160424224456/https://cloud.google.com/appengine/docs/python/search/#Python_Search_API_quotas) documentation for a detailed description of each type of Search call.

<table>
<thead>
<tr class="header">
<th>Resource</th>
<th>Cost</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>Total Storage (Documents and Indexes)</td>
<td>$0.18 per GB per month<sup>*</sup></td>
</tr>
<tr class="even">
<td>Queries</td>
<td>$0.50 per 10K queries</td>
</tr>
<tr class="odd">
<td>Indexing Searchable Documents</td>
<td>$2.00 per GB</td>
</tr>
</tbody>
</table>

<sup>\*</sup> Storage is at Datastore rates.

## Managing billing

You must be a project owner to be a billing administrator and complete basic billing tasks. For more information, see the support help page on how to [Manage billing administrators](https://web.archive.org/web/20160424224456/https://support.google.com/cloud/answer/4430947).

### Billing settings

To use computing resources above the free quotas, add a billing account and enable billing for your project in the Google Cloud Platform Console. For further instructions, see [Enabling billing](https://web.archive.org/web/20160424224456/https://cloud.google.com/appengine/docs/developers-console/#billing).

If you already have a billing account when you create a project, billing is enabled for that project by default. If you have more than one billing account when you create a project, you must select an account to associate with your project. If you don't have a billing account when you create a project, then you must add a billing account and enable billing for that project to use resources beyond the free quotas.

If you disable an application, you should also [disable billing for that application](https://web.archive.org/web/20160424224456/https://support.google.com/cloud/answer/6293499#disable-billing), as the app can still be charged for fixed billing costs, like datastore storage.

After you enable billing, the spending limit is set to unlimited by default. You may want to [set a spending limit](https://web.archive.org/web/20160424224456/https://cloud.google.com/appengine/docs/developers-console/#billing) so there is a limit to the amount you can be charged per day.

### Spending limits

The spending limit is the maximum resource cost you want to pay in a day for a project for [App Engine resources](#Billable_Resource_Unit_Costs). You may still be charged for usage of other Google Cloud Platform resources beyond the spending limit. If you have multiple projects, you may want to set a spending limit for each project.

The spending limit should be large enough to be able to handle spikes in resource usage. When an application exceeds its daily spending limit, any operation whose free quota has been exhausted fails.

By default, the spending limit for a project is unlimited. To limit application costs, [set a spending limit](https://web.archive.org/web/20160424224456/https://cloud.google.com/appengine/docs/developers-console/#billing).

## Understanding billing

To view your app's charges, in the Google Cloud Platform Console, go to [Billing](https://web.archive.org/web/20160424224456/https://console.cloud.google.com/billing). Select the billing account, then go to the **History** page.

Only billing administrators can view the transaction history. No paper invoices are sent to the billing contact.

The transaction history shows all account activity related to resource charges and payments. The report uses the US Pacific timezone.

**Note:** Daily usage costs are rounded to the penny when they are displayed in the App Engine dashboard, but the transaction history accumulates the actual full costs. Therefore the sum of the daily charges may not be identical to the amount reported (and billed) in the transaction history. Daily usage is posted on dashboard the next day but it can take longer to update the transaction history, so the transaction history may not include the most recent usage history.

### Daily and monthly charges

Charges are posted daily and monthly:

-   **Daily**: Every day you are charged for the resources you actually use. Usage up to the free quota limits is included in the usage total, but not in the billable amount. Usage above the free quota is charged at the regular rates.
-   **Monthly**: At the beginning of each month all daily charges for the previous month are summed, applicable taxes are computed, and the total charges are debited from the payment method that is linked to the app.

### Taxes

Some countries tax App Engine fees. If taxes apply in your country of residence, your bill will include any applicable taxes. Note that the spending limit does not include taxes. Taxes are added to your charges after daily spend has been calculated, so the final charge to your account may be larger than the spending limit amount. To see any taxes in your bill, in the Google Cloud Platform Console, go to [Billing](https://web.archive.org/web/20160424224456/https://console.cloud.google.com/billing). Select the billing account, then go to the **History** page to view the transaction history.

### Grace periods

You can view your app's current billing status in the [App Engine dashboard](https://web.archive.org/web/20160424224456/https://console.cloud.google.com/project/_/appengine) in the Cloud Platform Console. If a payment fails, the app's account is delinquent and enters a grace period status. You have until the end of the grace period to pay the outstanding balance. During the grace period, the app will continue to run with its budget constraints. If payment is not received, your quotas may be reverted to the [default levels](https://web.archive.org/web/20160424224456/https://cloud.google.com/appengine/docs/quotas#Resources).

To clear the outstanding charges you can go to the **Transaction History** page where you can click **Make a Payment**. You may need to go to the **Billing Settings** page first, where you can add another payment source or correct a problem with an existing account, such as an expired card. If payment succeeds, the billing status will change to **Billing Enabled**.

## Help

-   See the [Cloud Billing](https://web.archive.org/web/20160424224456/https://support.google.com/cloud/topic/3340598) guide for more information.

-   Report billing problems to Cloud Services by filling out this <a href="https://web.archive.org/web/20160424224456/https://support.google.com/code/go/cloud_billing" target="_blank">online form</a>.
