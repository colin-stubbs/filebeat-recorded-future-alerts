# What is it?

Filebeat module to poll the Recorded Future Connect API to get alerts into an Elastic SIEM etc.

In reference to:
1. https://www.recordedfuture.com/
2. https://www.recordedfuture.com/solutions/threat-alerting/
3. https://api.recordedfuture.com/index.html

Elastic beats repo issue: https://github.com/elastic/beats/issues/27910

A full fork of beats is being edited here: https://github.com/colin-stubbs/beats

A pull request upstream will be issued once ready.

This is separate to the existing Recorded Future Threat Intelligence ingest sub-module described here: https://www.elastic.co/guide/en/beats/filebeat/7.x/filebeat-module-threatintel.html#recordedfuture

But this module pairs well with it, so you've got both useful threat intel as well as alerts out of Recorded Future in your SIEM.

We made the decision to create a separate module to avoid having to modify the threatintel.recordedfuture content for the time being.

# Why?

Recorded Future's "Enterprise" product offering is horrendously bad with respect to alerting options.

You can get alerts via email - yes, very 1995 I know - and via their mobile app which is equally useless in 2021.

A modern SOC needs a consistent and reliable alerting pipeline that alerts both humans and software components.

e.g. in 2021 you want OpsGenie, VictorOps, PagerDuty or similar.

All of those can and should simple receive webhooks from any product with an alerting capability.

The alerting pipeline also plumbs alerts into ticketing systems, e.g. JIRA etc, and SIEM's and similar event management solutions for visualisation, enrichment, as well as additional correlation based alerting.

Alerts in isolation, or in an email inbox in what is basically an unstructured format delivered via an unreliable delivery mechanism, are  beyond useless in 2021.

Given the feature hole in the Recorded Future product we're using an Elastic SIEM as alerting glue, and using filebeat to get the events into Elastic in order to trigger webhooks to other security infrastructure.

The module uses polls the Connect API /v2/alert/search API endpoint every 5 minutes, pulls alerts, throws them at filebeats output.

The module includes an Elasticsearch ingest pipeline to parse the event out to (mostly) ECS 1.10.0 compliant fields.

# Install,

Manual install at the moment,

Clone this repo somewhere and copy the module content to where it needs to go,

```
cp -Rpv /path/to/repo/module/recordedfuture /path/to/your/filebeat/files/module/recordedfuture
```

Typically that'll me to /usr/share/filebeat/module/recordedfuture or similar.

Run the following or similar to ensure the ingest pipeline is created within Elasticsearch, replacing variable values as below with those appropriate to your install.

```
filebeat setup -e --pipelines --modules "recordedfuture" -E "cloud.id=${CLOUD_ID}" -E "cloud.auth=${CLOUD_AUTH}"
```

That's it basically.

All of this assumes you're using a recent version of Elastic, probably with X-Pack features.

Tested on filebeat v7.14.1 to Elastic Cloud v7.14.1.

I have no idea if this will work on a free ELK stack and don't care. Pay for the software you use, particularly if you're a business who can afford to pay for Recorded Future.

If you're not, consider using https://cloud.elastic.co/ or the Google Cloud Platform official marketplace service https://console.cloud.google.com/marketplace/details/endpoints/elasticsearch-service.gcpmarketplace.elastic.co

# Config example,

Create /etc/filebeat/modules.d/recordedfuture.yml or put the below in your filebeat.yml under

```
# Module: recordedfuture
# Docs: TBC

- module: recordedfuture
  alerts:
    enabled: true

    # The input type, httpjson or file. Default: httpjson
    # input: httpjson

    # The interval to poll the API for updates. Default: 5m (5 minutes).
    # var.interval: 5m

    # How far back in time to start fetching alerts when run for the
    # first time. Value must be in hours. Default: 8760h (approx 1 year).
    # var.first_interval: 8760h

    # The URL used for Connect API alert search
    # var.url: "https://api.recordedfuture.com/v2/alert/search"

    # Set your API Token.
    var.api_token: "YOUR_API_TOKEN_HERE"
```

Depending on how you've configured filebeat you may need to restart filebeat to get it to recognise that.

Results,

Event data should look something like this,

![Event Data](https://raw.githubusercontent.com/colin-stubbs/filebeat-recorded-future-alerts/main/Screenshot%20from%202021-09-14%2019-32-55.png)

# Elastic Security Solution Alerts

A basic Elastic Security solution alerting rule is in this NDJSON file which you can import:

You should get alerts like this, which you can attach alerting actions too, e.g. create a JIRA ticket, or send the event as a webhook elsewhere, such as to OpsGenie.

![Security Alerts](https://github.com/colin-stubbs/filebeat-recorded-future-alerts/blob/main/Screenshot%20from%202021-09-15%2009-08-55.png)

# Elastic Dashboard

To visualise your Recorded Future alerts, there's visualisation and dashboard content in this file which you can import:

Here's what that looks like,

![Dashboard](https://raw.githubusercontent.com/colin-stubbs/filebeat-recorded-future-alerts/main/Screenshot%20from%202021-09-14%2020-00-36.png)

# Support

Contact Equate Technologies via contact_at_equatetechnologies-com_dot_au
