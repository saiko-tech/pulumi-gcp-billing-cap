# gcp-billing-cap (Pulumi)

Prevent excessive cloud costs via GCP Billing Alerts, Pub/Sub & Cloud Functions.<br>
Essentially a pulumified version of https://cloud.google.com/billing/docs/how-to/notify.

But don't let me explain it - here's a picture instead!

<p align="center">
  <img src="https://user-images.githubusercontent.com/5138316/140429689-164dfc5d-5226-4dba-bf55-351f0d594b4d.png" />
</p>

This is a very crude/simplified implementation at this stage - feature requests and contributions welcome.

## Installation

1. Add the following to your Pulumi project's `requirements.txt`:

```
git+https://github.com/saiko-tech/pulumi-gcp-billing-cap@260924ed0a7fab60ed3605a57a59cd67f814c5b3#egg=pulumi-gcp-billing-cap
```

**NOTE: make sure to use an explicit Git SHA like in the above example isstead of `@master` etc. - always do this when linking against dependencies via Git or you may become the victim of a supply chain attack!**

2. Clone this repo, you will need the `capper` folder

```
git clone https://github.com/saiko-tech/pulumi-gcp-billing-cap.git
```

## Usage

```terminal
$ pulumi config set gcpBillingAccount 010101-F0F0F0-F0F0F0
```

```py
import shutil
import pulumi_gcp as gcp
import pulumi_gcp_billing_cap as capper

cloudresourcemanager_enable = gcp.projects.Service('cloudresourcemanager-api', service='cloudresourcemanager.googleapis.com')

cloudbilling_enable = gcp.projects.Service('cloudbilling-api', service='cloudbilling.googleapis.com')

billingbudgets_enable = gcp.projects.Service('billingbudgets-api', service='billingbudgets.googleapis.com')

cloudfunctions_enable = projects.Service("cloudfunctions-api", service="cloudfunctions.googleapis.com")

# '/path/to/gcp-billing-cap/capper' needs to point to the `/capper` directory of this repo
shutil.make_archive('/tmp/capper', 'zip', '/path/to/gcp-billing-cap/capper')

billing_project = gcp.organizations.get_project()
billing_account = gcp.organizations.get_billing_account(billing_account=config.require('gcpBillingAccount'))

capper.GCPBillingCap(
    'gcp-billing-cap',
    args=capper.GCPBillingCapArgs(
        billing_account=billing_account.id,
        billing_project_name=billing_project.name,
        billing_project_number=billing_project.number,
        currency_code='GBP',  # must match the currency used in your GCP billing account
        max_spend='100',  # £100 per month, must be a string unfortunately
        location='europe-west1',  # where the capper source code is stored in GCS
        capper_zip_path='/tmp/capper.zip'),
    opts=ResourceOptions(depends_on=[billingbudgets_enable, cloudresourcemanager_enable, cloudbilling_enable, cloudfunctions_enable]))
```

## Aknowledgements

- https://cloud.google.com/billing/docs/how-to/notify
- https://github.com/greenpeace/gp-gcp-disable-billing-cap-cost
