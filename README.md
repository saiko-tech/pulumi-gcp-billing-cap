# gcp-billing-cap (Pulumi)

Prevent excessive cloud costs via GCP Billing Alerts, Pub/Sub & Cloud Functions.
Essentially a pulumified version of https://cloud.google.com/billing/docs/how-to/notify.

But don't let me explain it - here's a picture instead!

<p align="center">
  <img src="https://user-images.githubusercontent.com/5138316/140429689-164dfc5d-5226-4dba-bf55-351f0d594b4d.png" />
</p>

This is a very crude/simplified implementation at this stage - feature requests and contributions welcome.

## Installation

Add the following to your Pulumi project's `requirements.txt`:

```
git+ssh://git@github.com/saiko-tech/gcp-billing-cap@539a748ad2bbd3dc6aa98fbc69474bf7df9075d5#egg=gcp-billing-cap
```

**NOTE: make sure to use an explicit Git SHA like in the above example isstead of `@master` etc. - always do this when linking against dependencies via Git or you will become the victim of a supply chain attack!**

## Usage

```py
cloudresourcemanager_enable = projects.Service(
    'cloudresourcemanager-api',
    service='cloudresourcemanager.googleapis.com')

billingbudgets_enable = projects.Service(
    'billingbudgets-api',
    service='billingbudgets.googleapis.com')

# '/path/to/gcp-billing-cap/capper' needs to point to the `/capper` directory of this repo
shutil.make_archive('/tmp/capper', 'zip', '/path/to/gcp-billing-cap/capper')

billing_project = gcp.organizations.get_project().name
billing_account = organizations.get_billing_account(billing_account=config.require('gcpBillingAccount'))

capper.GCPBillingCap(
    'gcp-billing-cap',
    args=capper.GCPBillingCapArgs(
        billing_account=billing_account.id,
        billing_project=billing_project,
        currency_code='GBP', # must match the currency used in your GCP billing account
        max_spend='100', # £100 per month, must be a string unfortunately
        capper_zip_path='/tmp/capper.zip'),
    opts=ResourceOptions(depends_on=[billingbudgets_enable, cloudresourcemanager_enable]))
```

## Aknowledgements

- https://cloud.google.com/billing/docs/how-to/notify
- https://github.com/greenpeace/gp-gcp-disable-billing-cap-cost
