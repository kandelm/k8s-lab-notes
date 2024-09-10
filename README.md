# k8s-lab-notes

In Kubernetes, **requests** and **limits** are used to manage and allocate computing resources (such as CPU and memory) for containers running in a pod. They help ensure that resources are allocated efficiently and prevent one container from using more than its share, which could negatively affect other containers.

### 1. **Requests**
- **Definition**: The amount of CPU and memory **guaranteed** to a container.
- **Behavior**: 
  - Kubernetes uses the requested resources to determine on which node to schedule the pod. 
  - A container **will always get at least the resources it requested**.
  - If the resources are available on the node, the container can use more than what it requested, but there is no guarantee beyond the request.
- **Usage Example**: 
  - If a pod requests 500m CPU and 256Mi memory, Kubernetes schedules the pod on a node that has at least these resources available.
  
### 2. **Limits**
- **Definition**: The maximum amount of CPU and memory a container can use.
- **Behavior**: 
  - A container cannot use more resources than its limits. 
  - If a container tries to use more CPU than its limit, Kubernetes will throttle the container.
  - If a container tries to use more memory than its limit, Kubernetes will kill the container.
  
### Key Differences:

| Feature       | **Request**                                        | **Limit**                                         |
|---------------|----------------------------------------------------|---------------------------------------------------|
| **Purpose**   | Minimum resources guaranteed to a container        | Maximum resources a container is allowed to use   |
| **Impact on Scheduling** | Kubernetes uses requests to schedule pods to nodes | Limits do not affect scheduling |
| **Usage**     | If available, a container can use more than its request | The container is restricted to its limit |
| **Enforcement**| No enforcement until resources exceed the available node capacity | Enforced by Kubernetes; exceeding memory limit can result in OOM (Out Of Memory) killing |

### Example of a Pod with Requests and Limits:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  containers:
  - name: example-container
    image: nginx
    resources:
      requests:
        memory: "128Mi"
        cpu: "500m"
      limits:
        memory: "256Mi"
        cpu: "1"
```

--------------------------------------------------------------------------------------


In Kubernetes, both **ReplicaSets** and **Deployments** are used for managing the desired state of applications, but they serve different purposes and offer different features.

### **ReplicaSet**

1. **Purpose**:
   - Ensures that a specified number of pod replicas are running at any given time.
   - Manages the scaling and replication of pods.

2. **Functionality**:
   - A ReplicaSet maintains a stable set of replica pods running at any given time.
   - It helps ensure that the desired number of pod replicas are maintained. If a pod fails or is deleted, the ReplicaSet will create a new one to replace it.
   - It does not handle updates or rollouts of new versions of applications.

3. **Use Case**:
   - Primarily used to maintain a stable number of pods without providing advanced management features.
   - Typically used indirectly through Deployments, but can be used directly in certain scenarios.

4. **Configuration**:
   - Specifies a selector to identify the pods it manages.
   - Defines the desired number of replicas and the pod template.

5. **Example YAML**:

   ```yaml
   apiVersion: apps/v1
   kind: ReplicaSet
   metadata:
     name: example-replicaset
   spec:
     replicas: 3
     selector:
       matchLabels:
         app: example
     template:
       metadata:
         labels:
           app: example
       spec:
         containers:
         - name: example-container
           image: nginx
   ```

### **Deployment**

1. **Purpose**:
   - Manages ReplicaSets and provides declarative updates to applications.
   - Simplifies the process of updating and rolling back applications.

2. **Functionality**:
   - A Deployment automatically creates and manages ReplicaSets to ensure the desired state of your application.
   - Handles rolling updates, allowing you to update the application version seamlessly.
   - Provides features for rolling back to previous versions if necessary.

3. **Use Case**:
   - Used for deploying and managing applications with lifecycle management features.
   - Suitable for continuous deployment, updates, and rollbacks.

4. **Configuration**:
   - Specifies the desired state of the application, including the ReplicaSet's configuration.
   - Manages the rollout process, including scaling and updating the pods.

5. **Example YAML**:

   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: example-deployment
   spec:
     replicas: 3
     selector:
       matchLabels:
         app: example
     template:
       metadata:
         labels:
           app: example
       spec:
         containers:
         - name: example-container
           image: nginx
   ```

### Key Differences:

| Feature              | **ReplicaSet**                                         | **Deployment**                                      |
|----------------------|--------------------------------------------------------|-----------------------------------------------------|
| **Primary Function** | Ensures a specified number of replicas are running    | Manages ReplicaSets and handles updates/rollouts    |
| **Updates**          | Does not handle application updates                    | Manages rolling updates and rollbacks               |
| **Use Case**         | For maintaining a stable number of pods                | For deploying and managing application updates      |
| **Direct Management**| Can be managed directly                                | Typically manages ReplicaSets indirectly             |
| **Rollback**         | Not supported directly                                 | Supports rollbacks to previous versions             |

In summary, while a ReplicaSet ensures a fixed number of pods, a Deployment offers a more comprehensive solution for deploying applications, including updates and rollbacks.
