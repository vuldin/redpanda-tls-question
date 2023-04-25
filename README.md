Testing HTTP proxy with TLS listener.

Delete any old certs in this directory (if you have ran through these steps previously:

```
./delete-certs.sh
```

Reset ownership, delete Redpanda data, restore config files in this directory:

```
./reset.sh
```

Save this sample `rpk` config to `/etc/redpanda/redpanda.yaml`:

```
rpk:
  kafka_api:
    brokers:
    - localhost:9092
    - localhost:9192
    - localhost:9292
    tls:
      cert_file: certs/node.crt
      key_file: certs/node.key
      truststore_file: certs/ca.crt
  admin_api:
    addresses:
    - localhost:9644
    - localhost:9744
    - localhost:9844
    tls:
      cert_file: certs/node.crt
      key_file: certs/node.key
      truststore_file: certs/ca.crt

```

Start the cluster:

```
docker compose up
```

`rpk` should now be able to connect to the external kafka and admin listeners via TLS:

```
rpk cluster info
rpk cluster health
```

Create a sample topic across all brokers:

```
rpk topic create continents -p 3
```

HTTP proxy has clients that connect to both the internal and external kafka listeners. But sending a request to the listener at port 8082 (that is connecting to the external kafka listener) doesn't make use of TLS:

```
curl -s \
  -X POST "http://localhost:8082/topics/continents" \
  -H "Content-Type: application/vnd.kafka.json.v2+json" \
  -d '{"records":[{"key": "from-proxy","value":"North America"}]}'
```

Consume the data created by the above command:

```
rpk topic consume continents
```

