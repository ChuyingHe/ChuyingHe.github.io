
# Triggers

Trigger includes the following resources:
- EventListener: 
    This object will automatically generated **Route**. If want to use `https` instead of `http`, in the auto-generated **Route**, add this to  `.spec`:
    
    ```yaml
    tls:
        termination: edge
        insecureEdgeTerminationPolicy: Redirect
    ```
- TriggerTemplates
- TriggerBindings or ClusterTriggerBindings