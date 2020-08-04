# States of a Pod

- Pending: The pod is accepted by the Kubernetes system but its container(s) is/are not created yet.

- Running: The pod is scheduled on a node and all its containers are created and at-least one container is in Running state.

- Succeeded: All container(s) in the Pod have exited with status 0 and will not be restarted.

- Failed: All container(s) of the Pod have exited and at least one container has returned a non-zero status.

- CrashLoopBackoff: The container fails to start and is tried again and again.

