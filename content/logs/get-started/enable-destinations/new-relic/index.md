---
title: Enable New Relic
pcx-content-type: how-to
weight: 64
layout: single
meta:
  title: Enable Logpush to New Relic
---

# Enable Logpush to New Relic

Cloudflare Logpush supports pushing logs directly to New Relic via the Cloudflare dashboard or via API.

## Manage via the Cloudflare dashboard

To enable a Logpush service to New Relic via the dashboard:

1.  Log in to the [Cloudflare dashboard](https://dash.cloudflare.com/login), and select the Enterprise domain you want to use with Logpush.

2.  Go to **Analytics** > **Logs**.

3.  Click **Connect a service** and a modal window will open.

4.  Select the dataset you want to push to a storage service.

5.  Select the data fields to include in your logs. You can add or remove fields later by modifying your settings in **Logs** > **Logpush**.

6.  Select **New Relic**.

7.  Enter the **New Relic Logs Endpoint**:

    US: `"https://log-api.newrelic.com/log/v1?Api-Key=<NR_LICENSE_KEY>&format=cloudflare"`

    EU: `"https://log-api.eu.newrelic.com/log/v1?Api-Key=<NR_LICENSE_KEY>&format=cloudflare"`

    Use the region that matches the one that has been set on your New Relic account. The `<NR_LICENSE_KEY>` field can be found on the New Relic dashboard. It can be retrieved by following [these steps](https://docs.newrelic.com/docs/apis/intro-apis/new-relic-api-keys/#manage-license-key).

8.  Click **Validate access**.

9.  Click **Save and Start Pushing** to finish enabling Logpush.

Once connected, Cloudflare lists New Relic as a connected service under **Logs** > **Logpush**. Edit or remove connected services from here.

## Manage via API

{{<render file="_enable-read-permissions.md">}}

### 1. Create a job

To create a job, make a `POST` request to the Logpush jobs endpoint with the following fields:

- **name** (optional) - Use your domain name as the job name.

- **logpull_options** (optional) - To configure fields, sample rate, and timestamp format, refer to [API configuration options](/logs/get-started/api-configuration/#options).

   {{<Aside type="note" header="Note">}}
   To query Cloudflare logs, New Relic requires fields to be sent as a Unix Timestamp.
   {{</Aside>}}

- **destination_conf** - A log destination consisting of an endpoint URL, a license key and a format in the string format below.

  - `<NR_ENDPOINT_URL>`: The New Relic HTTP logs intake endpoint, which is `https://log-api.newrelic.com/log/v1` for US or `https://log-api.eu.newrelic.com/log/v1` for the EU, depending on the region that has been set on your New Relic account.
  - `<NR_LICENSE_KEY>`: This key can be found on the New Relic dashboard and it can be retrieved by following [these steps](https://docs.newrelic.com/docs/apis/intro-apis/new-relic-api-keys/#manage-license-key).
  - `format`: The format is `cloudflare`.

    US: `"https://log-api.newrelic.com/log/v1?Api-Key=<NR_LICENSE_KEY>&format=cloudflare"`

    EU: `"https://log-api.eu.newrelic.com/log/v1?Api-Key=<NR_LICENSE_KEY>&format=cloudflare"`

- **max_upload_records** (optional) - The maximum number of log lines per batch. This must be at least 1,000 lines or more. Note that there is no way to specify a minimum number of log lines per batch. This means that log files may contain many fewer lines than specified.

- **max_upload_bytes** (optional) - The maximum uncompressed file size of a batch of logs. This must be at least 5 MB. Note that there is no way to set a minimum file size. This means that log files may be much smaller than this batch size. Nevertheless, it is recommended to set this parameter to 5,000,000.

- **dataset** - The category of logs you want to receive. Refer to [Log fields](/logs/reference/log-fields/) for the full list of supported datasets.

Example request using cURL:

```bash
curl -s https://api.cloudflare.com/client/v4/zones/<ZONE_ID>/logpush/jobs -X POST -d '
{
  "name": "<DOMAIN_NAME>",
  "logpull_options": "fields=ClientIP,ClientRequestHost,ClientRequestMethod,ClientRequestURI,EdgeEndTimestamp,EdgeResponseBytes,EdgeResponseStatus,EdgeStartTimestamp,RayID&timestamps=unix",
  "destination_conf": "https://log-api.newrelic.com/log/v1?Api-Key=<NR_LICENSE_KEY>&format=cloudflare",
  "max_upload_bytes": 5000000,
  "dataset": "http_requests",
  "enabled": true
}' \
-H "X-Auth-Email: <EMAIL>" \
-H "X-Auth-Key: <API_KEY>" | jq .
```

Response:

```bash
{
   "errors" : [],
   "messages" : [],
   "result" : {
      "dataset" : "http_requests",
      "destination_conf" : "https://log-api.newrelic.com/log/v1?Api-Key=<NR_LICENSE_KEY>&format=cloudflare",
      "enabled" : true,
      "error_message" : null,
      "frequency" : "high",
      "id" : 100,
      "kind" : "",
      "last_complete" : null,
      "last_error" : null,
      "logpull_options" : "fields=ClientIP,ClientRequestHost,ClientRequestMethod,ClientRequestURI,EdgeEndTimestamp,EdgeResponseBytes,EdgeResponseStatus,EdgeStartTimestamp,RayID&timestamps=unix",
      "logstream" : true,
      "max_upload_bytes" : 5000000,
      "name" : "<DOMAIN_NAME>"
   },
   "success" : true
}
```

### 2. Enable (update) a job

To enable a job, make a `PUT` request to the Logpush jobs endpoint. You will use the job ID returned from the previous step in the URL and send `{"enabled": true}` in the request body.

Example request using cURL:

```bash
curl -s -X PUT \
https://api.cloudflare.com/client/v4/zones/<ZONE_ID>/logpush/jobs/100 -d'{"enabled":true}' \
-H "X-Auth-Email: <EMAIL>" \
-H "X-Auth-Key: <API_KEY>" | jq .
```

Response:

```bash
{
   "errors" : [],
   "messages" : [],
   "result" : {
      "dataset" : "http_requests",
      "destination_conf" : "https://log-api.newrelic.com/log/v1?Api-Key=<NR_LICENSE_KEY>&format=cloudflare",
      "enabled" : true,
      "error_message" : null,
      "frequency" : "high",
      "id" : 100,
      "kind" : "",
      "last_complete" : "null",
      "last_error" : null,
      "logpull_options" : "fields=ClientIP,ClientRequestHost,ClientRequestMethod,ClientRequestURI,EdgeEndTimestamp,EdgeResponseBytes,EdgeResponseStatus,EdgeStartTimestamp,RayID&timestamps=unix",
      "logstream" : true,
      "max_upload_bytes" : 5000000,
      "name" : "<DOMAIN_NAME>"
   },
   "success" : true
}
```
## Install the Cloudflare quickstart

You can install the [cloudflare quickstart](https://newrelic.com/instant-observability/cloudflare-network-logs/fc2bb0ac-6622-43c6-8c1f-6a4c26ab5434/?utm_source=external_partners&utm_medium=referral&utm_campaign=global-ever-green-io-partner) in the New Relic Instant Observability page. The dashboards included contain the following information:

### Overview

Get a quick overview of the most important metrics from your websites and applications on the Cloudflare network.

![Cloudflare Network Logs install screen](/fundamentals/static/images/new-relic/dashboard/dash-1.png)

### Security

Get insights on threats to your websites and applications, including number of threats taken action on by the Web Application Firewall (WAF), threats over time, top threat countries, and more.

![Cloudflare Network security metrics screen](/fundamentals/static/images/new-relic/dashboard/dash-2.png)

### Performance

Identify and address performance issues and caching misconfigurations. Metrics include total requests, total versus cached requests, total versus origin requests.

![Cloudflare Network Logs performance metrics screen](/fundamentals/static/images/new-relic/dashboard/dash-3.png)

### Reliability

Get insights on the availability of your websites and Applications. Metrics include, edge response status over time, percentage of `3xx`/`4xx`/`5xx` errors over time, and more.

![Cloudflare Network Logs reliability metrics screen](/fundamentals/static/images/new-relic/dashboard/dash-4.png)