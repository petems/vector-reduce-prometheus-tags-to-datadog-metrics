# vector-reduce-prometheus-tags-to-datadog-metrics

## What is this?

A sandbox/lab repo for showing how Vector can remove tags and reduce tag cardinality from a Prometheus endpoint

## Why?

Promtheus metrics can have a very high tag cardinality. Custom metrics sent to Datadog are charged on complexity, and a high tag cardinality can bring additional costs.

## How?

In this demo, Vector is running a simple topology in which the `prometheus` source fetches a promtheus metircs from an app that generates random Prometheus data (https://github.com/little-angry-clouds/prometheus-data-generator)

We then run one of two transforms to reduce tag cardinality:

Removing a tag completely:

```toml
[transforms.remove_tag.hooks]
  process = """
function (event, emit)
\t-- Remove tag
\tevent.metric.tags.tag_to_remove = "remove_me"
\temit(event)
end"""
```

Setting a tag cardinality limit

```toml
[transforms.reduce_cardinality]
type = "tag_cardinality_limit"
inputs = [ "remove_tag" ]
mode = 'exact'

[transforms.reduce_cardinality.fields]
value_limit = 2
limit_exceeded_action = "drop_tag"
```

To run this scenario, make sure you have the `DD_API_KEY` environment variable set to your Datadog
API key and then run:

```bash
$ docker compose up
```

