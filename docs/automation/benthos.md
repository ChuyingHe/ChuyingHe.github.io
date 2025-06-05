
- [What is Benthos](https://www.youtube.com/watch?v=88DSzCFV4Ng&t=1s&ab_channel=Jeffail)
- [Official documentation](https://docs.redpanda.com/redpanda-connect/get-started/about/)
- [Git repo](https://github.com/redpanda-data/connect)

**Benthos** has been acquired by RedPanda. Now its known as **Redpanda Connect**.

A streaming ETL service for **Data Engineering**:

- no runtime dependencies
- declarative config
- stateless
- payload agnostic: suitable for all format of data
- logs, metrics, tracing

<img src="../benthos_imgs/example.png" />


# Start
```bash
# install
brew install benthos

# make a config
benthos create nats/protobuf/aws-sqs > ./config.yaml

# run
benthos -c ./config.yaml
```

One can also use `benthos` as a Docker container.


# Architecture

<img src="../benthos_imgs/architecture.png" />

- Inputs: wrap message in a so called **Transaction**
- Processors: one/multi **Threads**
- Outputs: to destinations

!!! warning "Acknowledgement"
    Only when the ALL tje Outputs successfully reached their goal, there will be a **Acknowledgement** send back to the data source - for instance Kafka

!!! note
    By default, nothing get dropped - if sth happens, the message will be send over and over again until its acknowledged.



# Features

- Unit Test: execute by `benthos test .`
- Bloblang: Benthos' native mapping language
    - example
        <img src="../benthos_imgs/bloblang.png" width=500 />
    - subcommand: 
        ```bash
        benthos blobl 'this.(article | comment | share).content'
        ```
    - benthos blobl server: to test bloblang
