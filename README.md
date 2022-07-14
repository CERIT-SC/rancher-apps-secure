# rancher-apps-secure

## Implementation Notes
- use `{{ .Release.Name }}` with `{{ .Release.Namespace }}` in all object names to allow multiple instances in the same namespace. 
