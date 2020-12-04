# Ketu

A Clojure Apache Kafka client with core.async api

```clojure
[com.appsflyer/ketu "0.6.0"]
```

## Features

* **Channels API**: Take kafka data from a channel and send data to kafka through a channel.
* **Consumer Source**: Polls records from kafka and puts them on a channel.
* **Producer Sink**: Takes records from a channel and sends them to kafka.
* **Shapes**: Transform the original objects of the java client to clojure data and back.
* **Simple Configuration**: Friendly, validated configuration.

## Configuration reference

Anything that is not documented is not supported and might change.

Note: `int` is used for brevity but can also mean `long`. Don't worry about it.

#### Common options (both source and sink accept these)
| Key | Type | Req? | Notes |
|-----|------|------|-------|
| :brokers | string | required | Comma separated `host:port` values e.g "broker1:9092,broker2:9092" |
| :topic | string | required |  |
| :name | string | required | Simple human-readable identifier, used in logs and thread names |
| :key-type | `:string`,`:byte-array` | optional | Default `:byte-array`, used in configuring key serializer/deserializer |
| :value-type | `:string`,`:byte-array` | optional | Default `:byte-array`, used in configuring value serializer/deserializer |
| :internal-config | map | optional | A map of the underlying java client properties, for any extra lower level config |

#### Consumer-source options
| Key | Type | Req? | Notes |
|-----|------|------|-------|
| :group-id | string | required |  |
| :shape | `:value:`, `[:vector <fields>]`,`[:map <fields>]`, or an arity-1 function of `ConsumerRecord` | optional | If unspecified, channel will contain ConsumerRecord objects. [Examples](#data-shapes) |

#### Producer-sink options
| Key | Type | Req? | Notes |
|-----|------|------|-------|
| :shape | `:value`, `[:vector <fields>]`,`[:map <fields>]`, or an arity-1 function of the input returning `ProducerRecord` | optional | If unspecified, you must put ProducerRecord objects on the channel. [Examples](#data-shapes) |
| :compression-type | `"none"` `"gzip"` `"snappy"` `"lz4"` `"zstd"` | optional | Default `"none"`, values are same as "compression.type" of the java producer |
| :workers | int | optional | Default `1`, number of threads that take from the channel and invoke the internal producer |

## Data shapes

You don't have to deal with ConsumerRecord or ProducerRecord objects.<br>
To get a clojure data structure with any of the ConsumerRecord fields, configure the consumer shape:
```clojure
; Value only:
{:topic "names"
 :key-type :string
 :value-type :string
 :shape :value}
(<!! consumer-chan)
;=> "v"

; Vector:
{:shape [:vector :key :value :topic]}
(<!! consumer-chan)
;=> ["k" "v" "names"]

; Map
{:shape [:map :key :value :topic]}
(<!! consumer-chan)
;=> {:key "k", :value "v", :topic "names"}
```
Similarly, to put a clojure data structure on the producer channel:
```clojure
; Value only:
{:key-type :string
 :value-type :string
 :shape :value}
(>!! producer-chan "v")

; Vector:
{:shape [:vector :key :value]}
(>!! producer-chan ["k" "v"])

; Vector with topic in each message:
{:shape [:vector :key :value :topic]}
(>!! producer-chan ["k1" "v1" "names"])
(>!! producer-chan ["k2" "v2" "events"])
```

## Development & Contribution

We welcome feedback and would love to hear about use-cases other than ours. You can open issues, send pull requests,
or contact us at clojurians slack.
