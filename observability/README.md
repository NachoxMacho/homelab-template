# Observability Options

There are several different routes to help monitor a cluster (and other infrastructure) depending on multiple factors.
Here I'll lay out a couple different options I've gotten working, and some of the benefits of each particular system.

## Grafana LGTM stack

This is the full grafana stack, with Loki, Grafana, Tempo, and Mimir for storage of the basic telemetry data points (Logs, Traces, and Metrics).
I personally use this in my homelab as the configurations are fairly straightforward, and similar across the main services.
It's also been fairly scalable, both up and down, as well as still being flexible with options like Alloy to ingest OpenTelemetery data and being compatible with other APIs like the prometheus one.

## Alternatives

### Jaeger for Tempo

Jaeger is the OG trace storage and visualization application.
I started initially using this, but for more persistent storage, I found that options like elasticsearch had higher minimums for resources.
This sucks for most of my homelabbing as I only have a handful of self created apps that need traces, so my trace load is fairly light, and I spend more resources for little benefit.
The UI is perfectly servicable, I had no problems with Jaeger itself, however as I started connecting prometheus and loki to grafana to visualize, it felt weird to go to a separate website entirely to view traces.
Then if I wasn't using the UI of Jaeger, there wasn't much attachment to sticking with it, so after experiencing Tempo scaling down better, I ended up sticking with Tempo.

Benefits of Jaeger over Tempo:

- Built in UI
- May scale better in the long term with lots of log ingestion
- More community support as the original

### OTel Collector for Alloy

### Prometheus for Alloy scraping

In the above configuration, I'm using grafana alloy to scrape metrics from the various servicemonitors and other metric endpoints.


### Thanos for Mimir

## Add-ons

Here are some other options for extending your observability.

### Pyroscope

### Parca

### Pyrra



