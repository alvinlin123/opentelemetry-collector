# Kafka Exporter

Kafka exporter exports traces to Kafka. This exporter uses a synchronous producer
that blocks and does not batch messages, therefore it should be used with batch and queued retry
processors for higher throughput and resiliency. Message payload encoding is configurable.
 
The following settings are required:
- `protocol_version` (no default): Kafka protocol version e.g. 2.0.0

The following settings can be optionally configured:
- `brokers` (default = localhost:9092): The list of kafka brokers
- `topic` (default = otlp_spans): The name of the kafka topic to export to
- `encoding` (default = otlp_proto): The encoding of the payload sent to kafka. Available encodings:
  - `otlp_proto`: the payload is serialized to `ExportTraceServiceRequest`.
  - `jaeger_proto`: the payload is serialized to a single Jaeger proto `Span`.
  - `jaeger_json`: the payload is serialized to a single Jaeger JSON Span using `jsonpb`.
- `auth`
  - `plain_text`
    - `username`: The username to use.
    - `password`: The password to use
  - `tls`
    - `ca_file`: path to the CA cert. For a client this verifies the server certificate. Should
      only be used if `insecure` is set to true.
    - `cert_file`: path to the TLS cert to use for TLS required connections. Should
      only be used if `insecure` is set to true.
    - `key_file`: path to the TLS key to use for TLS required connections. Should
      only be used if `insecure` is set to true.
    - `insecure` (default = false): Disable verifying the server's certificate chain and host 
      name (`InsecureSkipVerify` in the tls config)
    - `server_name_override`: ServerName indicates the name of the server requested by the client
      in order to support virtual hosting.
  - `kerberos`
    - `service_name`: Kerberos service name
    - `realm`: Kerberos realm
    - `use_keytab`:  Use of keytab instead of password, if this is true, keytab file will be used instead of password
    - `username`: The Kerberos username used for authenticate with KDC
    - `password`: The Kerberos password used for authenticate with KDC
    - `config_file`: Path to Kerberos configuration. i.e /etc/krb5.conf
    - `keytab_file`: Path to keytab file. i.e /etc/security/kafka.keytab
- `metadata`
  - `full` (default = true): Whether to maintain a full set of metadata. 
                                    When disabled the client does not make the initial request to broker at the startup.
  - `retry`
    - `max` (default = 3): The number of retries to get metadata
    - `backoff` (default = 250ms): How long to wait between metadata retries
- `timeout` (default = 5s): Is the timeout for every attempt to send data to the backend.
- `retry_on_failure`
  - `enabled` (default = true)
  - `initial_interval` (default = 5s): Time to wait after the first failure before retrying; ignored if `enabled` is `false`
  - `max_interval` (default = 30s): Is the upper bound on backoff; ignored if `enabled` is `false`
  - `max_elapsed_time` (default = 120s): Is the maximum amount of time spent trying to send a batch; ignored if `enabled` is `false`
- `sending_queue`
  - `enabled` (default = false)
  - `num_consumers` (default = 10): Number of consumers that dequeue batches; ignored if `enabled` is `false`
  - `queue_size` (default = 5000): Maximum number of batches kept in memory before dropping data; ignored if `enabled` is `false`;
  User should calculate this as `num_seconds * requests_per_second` where:
    - `num_seconds` is the number of seconds to buffer in case of a backend outage
    - `requests_per_second` is the average number of requests per seconds.

Example configuration:

```yaml
exporters:
  kafka:
    brokers:
      - localhost:9092
    protocol_version: 2.0.0
```
