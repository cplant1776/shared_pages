Overview - Custom Metadata Types

# Overview

App settings that send both definition AND record to the Org, so they can be deployed with version control/packages.

## Quick Example

### Scenario:

You want records that store different support tiers you offer to your customers and a discount for each tier.

### Solution:

1. Create custom metadata type **Support Tier**
2. Add custom Number field **Minimum Spending**
3. Add custom Percent field **Default Discount**
4. Add records:

Label | Minimum Spending (Custom Field) | Default Discount (Custom Field)
--- | --- | ---
Bronze | 0 | 0%
Silver | 1,000 | 10%
Bronze | 5,000	 | 15%