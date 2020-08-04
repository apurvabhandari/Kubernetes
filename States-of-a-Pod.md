# States of a Pod

- Pending: The pod is accepted by the Kubernetes system but its container(s) is/are not created yet. This includes time before being scheduled as well as time spent downloading images over the network, which could take a while.

- Running: The pod is scheduled on a node and all its containers are created and at-least one container is in Running state. 

- Succeeded: All container(s) in the Pod have exited with status 0 and will not be restarted.

- Failed: All container(s) of the Pod have exited and at least one container has returned a non-zero status.

- CrashLoopBackoff: The container fails to start and is tried again and again.

- Unknown: For some reason the state of the Pod could not be obtained, typically due to an error in communicating with the host of the Pod.
