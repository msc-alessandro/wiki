
## Configuração do NGINX

```
{
  "kind": "Service",
  "apiVersion": "v1",
  "metadata": {
    "name": "nginx",
    "namespace": "default",
    "selfLink": "/api/v1/namespaces/default/services/nginx",
    "uid": "b859603f-973d-11e9-9f54-2c4d54e2a2ee",
    "resourceVersion": "177223",
    "creationTimestamp": "2019-06-25T11:38:15Z",
    "labels": {
      "app": "nginx"
    }
  },
  "spec": {
    "ports": [
      {
        "name": "80-80",
        "protocol": "TCP",
        "port": 80,
        "targetPort": 80,
        "nodePort": 30822
      }
    ],
    "selector": {
      "app": "nginx"
    },
    "clusterIP": "10.98.20.9",
    "type": "NodePort",
    "sessionAffinity": "None",
    "externalTrafficPolicy": "Cluster"
  },
  "status": {
    "loadBalancer": {}
  }
}

```