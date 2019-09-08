# Replica Set Vs Replication Controller in K8s
Replica Set and Replication Controller do almost the same thing. Both of them ensure that a specified number of pod replicas are running at any given time.
The difference comes with the usage of selectors to replicate pods. <br>
Replica Set use Set-Based selectors while replication controllers use Equity-Based selectors.<br>
- Equity-Based Selectors: This type of selector allows filtering by label key and values. So, in layman terms, the equity-based selector will only look for the pods which will have the exact same phrase as that of the label.
Example: Suppose your label key says app=nginx, then, with this selector, you can only look for those pods with label app equal to nginx.
- Selector-Based Selectors: This type of selector allows filtering keys according to a set of values. So, in other words, the selector based selector will look for pods whose label has been mentioned in the set.
Example: Say your label key says app in (nginx, NPS, Apache). Then, with this selector, if your app is equal to any of nginx, NPS, or Apache, then the selector will take it as a true result.
