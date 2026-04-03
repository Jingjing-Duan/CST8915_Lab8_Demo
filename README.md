# CST8915_Lab8_Demo

## Demo Link
https://youtu.be/5n8huhbm4sw

## Task 2 – Improving and Extending the Deployment

In the original deployment, MongoDB and RabbitMQ were running inside Kubernetes without persistent storage. This means that if a pod was deleted or restarted, all data such as orders and messages would be lost. In this task, I updated the deployment to make both services more reliable and persistent.

### MongoDB: replication and persistence

I changed MongoDB from a single instance to a StatefulSet with 3 replicas. This allows MongoDB to run multiple instances, which improves availability. If one pod fails, the others can still continue working.

I also updated the MongoDB service to a headless service by setting:

```yaml
clusterIP: None
```

This gives each pod a stable DNS name like:

* mongodb-0.mongodb
* mongodb-1.mongodb
* mongodb-2.mongodb

These names are important for communication between replicas.

To make the data persistent, I added a volumeClaimTemplates section. This creates a separate PersistentVolumeClaim (PVC) for each MongoDB pod. Each pod stores its data in:

```yaml
/data/db
```

Because of this, even if a pod is deleted and recreated, the data is still there.

### RabbitMQ: persistent messages

For RabbitMQ, I added persistent storage so that messages are not lost when the pod restarts.

I mounted a volume at:

```yaml
/var/lib/rabbitmq
```

This is where RabbitMQ stores its data. I also used volumeClaimTemplates to create a PVC for it.

The ConfigMap for RabbitMQ plugins is still mounted, so the required plugins continue to work.

### Why persistence is important

Without persistent storage, any restart would delete all data. By using PVCs:

* MongoDB keeps all order data
* RabbitMQ keeps all messages and queues

This makes the system much more stable.

### How I tested it

To test persistence, I deleted the MongoDB pod:

```bash
kubectl delete pod mongodb-0
```

After the pod was recreated, I checked the database again and confirmed that the order data was still there. This shows that the storage is working correctly.

### Azure managed services

Instead of running MongoDB and RabbitMQ inside Kubernetes, Azure provides managed services.

**MongoDB alternative: Azure Cosmos DB (MongoDB API)**
This is a managed database service that supports MongoDB. It provides automatic scaling, backups, and high availability, so we don’t need to manage database pods manually.

**RabbitMQ alternative: Azure Service Bus**
This is a managed messaging service. It handles queues, reliability, and scaling automatically. It is easier to use in production compared to running RabbitMQ inside Kubernetes.

### Summary

In this task, I improved the system by adding persistence and basic high availability. MongoDB now runs with 3 replicas and persistent storage, and RabbitMQ keeps its messages even after restarts. These changes make the system more reliable and closer to a real-world deployment.
