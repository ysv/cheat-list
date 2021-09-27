* ### List pods for a specific node
```bash
kubectl get pods --all-namespaces -o wide --field-selector spec.nodeName=<node>
```

