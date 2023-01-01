---
date: 2023-01-02 00:00:00 +1300
title: Create Azure IP Groups based on the IP address ranges of countries
author: Luke
categories:
- Azure
toc: false
header:
  teaser: ''

---
An [IP Group](https://learn.microsoft.com/azure/firewall/ip-groups?WT.mc_id=AZ-MVP-5004796 "IP Groups in Azure Firewall") in Microsoft Azure is a logical container of IP address ranges for private and public addresses.

> IP Groups allow you to group and manage IP addresses for Azure Firewall rules in the following ways:
>
> * As a source address in DNAT rules
> * As a source or destination address in network rules
> * As a source address in application rules
>
> An IP Group can have a single IP address, multiple IP addresses, one or more IP address ranges or addresses and ranges in combination.

The IP Group allows you to define an IP address that can be used in conjunction with Azure Firewall, to allow or deny internal or external traffic from a perspective set of IP addresses.

> The following IPv4 address format examples are valid to use in IP Groups:
>
> * Single address: 10.0.0.0
> * CIDR notation: 10.1.0.0/32
> * Address range: 10.2.0.0-10.2.0.31

By default, the Azure Firewall blocks outbound and inbound traffic; however, you may want to enable _(or block)_ traffic to and from specific countries - there is no built-in geo-filtering with Azure Firewall, as you can use other services, such as the Web Application Gateway and with the [Application Gateway ](https://learn.microsoft.com/azure/application-gateway/overview?WT.mc_id=AZ-MVP-5004796 "What is Azure Application Gateway?")and [Azure Front Door](https://learn.microsoft.com/azure/web-application-firewall/afds/waf-front-door-geo-filtering?WT.mc_id=AZ-MVP-5004796 "What is geo-filtering on a domain for Azure Front Door Service?") to block and allow access, and other third party services such as Cloudflare.

However, you may want to control access to and from specific countries _(or other services)_ with Azure Firewall - this is where the IP Groups can be effective, and because we won't be editing the Firewall directly - we won't run into issues with delays without having to wait for the Azure Firewall policies to be updated.

To solve the issue of creating the IP groups and finding and keeping the IP groups up-to-date with various countries' IP ranges - I have created a PowerShell function to retrieve supported [countries](https://www.ipdeny.com/ipblocks/data/aggregated/ "IP Deny aggregated list")' IP CIDR ranges and create the relevant IP groups.

![Azure IP Group - Country IP ranges](/uploads/azureipgroupscript.png "Azure IP Group - Country IP ranges")

With IP Groups, there are a few things to keep in mind:

* You can have 200 IP Groups per firewall with a maximum of **5000 individual IP addresses or IP prefixes per each IP Group**.

For a country like New Zealand, the 5000 limit for the address ranges is acceptable - but for other countries, like the United States or United Kingdom, this can be an issue, where the total IP ranges can grow to over 20k - to deal with this, the script will create multiple IP Groups, and append a number to the end. 

Suppose IPs are manually added to the groups. In that case, they won't be added - the script will add in any different or new IP ranges, ignoring any current IP ranges (_this means it won't delete any IP ranges that are removed from the source IP list from IPDeny)_; however, I recommend that anything added outside of this script is kept in a separate IP group.

**_As with any script, I recommend this is tested in a test environment first._**

So - lets take a look at it: