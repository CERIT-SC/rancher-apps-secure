# Jupyter Notebook Standalone
The Jupyter Notebook is the original web application for creating and sharing computational documents.
It offers a simple, streamlined, document-centric experience.

## Images
You can choose to run either a custom image or one of the following:
- Minimal NB
- Datascience NB
- Scipy NB
- TensorFlow NB
- TensorFlow 2.7.0 with GPU and TensorBoard

If you use custom image, build on any image from existing Jupyter stack to avoid problems with missing dependencies for Jupyter itself. Also, the image must be functional under non-root user 1000.

## Notebook Authentication
When launching Jupyter Notebook app, you need to fill in token which is used as password for the notebook instance.

## Storage Authentication
It is possible to mount `storage-elixir1.priv.cerit-sc.cz` storage with UCO and secondary password as credentials.

## Web Access
Application is available through web page, URL is on domain `dyn.cloud.trusted.e-infra.cz`. The full name is composed from application name
and namespace and can be long. Therefore navigate to Ingress tab (right menu -> Service Discovery -> Ingresses) and choose.
