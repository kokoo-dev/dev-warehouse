## References

---

- github: https://github.com/localstack/localstack
- docs: https://docs.localstack.cloud/overview/
  - configuration: https://docs.localstack.cloud/references/configuration/
- docker hub: https://hub.docker.com/r/localstack/localstack
- aws cli: https://awscli.amazonaws.com/v2/documentation/api/2.0.34/index.html


## Example

---

### SQS

~~~sh
# create
aws --endpoint-url http://localhost:4566 sqs create-queue --queue-name queue.fifo --attributes FifoQueue=true

# list
aws --endpoint-url http://localhost:4566 sqs list-queues

# get
aws --endpoint-url http://localhost:4566 sqs get-queue-url --queue-name queue.fifo

# delete
aws --endpoint-url http://localhost:4566 sqs delete-queue --queue-url http://localhost.localstack.cloud:4566/queue/ap-northeast-2/000000000000/queue.fifo

# send
aws --endpoint-url http://localhost:4566 sqs send-message --queue-url http://localhost.localstack.cloud:4566/queue/ap-northeast-2/000000000000/queue.fifo --message-body "Sample Messaage" --message-group-id group1 --message-deduplication-id deduplication1

# receive
aws --endpoint-url http://localhost:4566 sqs receive-message --queue-url http://localhost.localstack.cloud:4566/queue/ap-northeast-2/000000000000/queue.fifo
~~~