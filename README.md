# helm-hcloud


## ops-tools

Houses all operations related items, including infra items like the 
- Hetzner Cloud Controller Manager
    - used to provision for example Cloud Load Balancers when asking for Service of type: LoadBalancer in kubernetes (k3s)
- Hetzner CSI Driver
    - used to be able to create Persitent volume claims with Hetzner Cloud Volumes as backing storage (which itself is replicated across [3 physical servers](https://docs.hetzner.com/cloud/volumes/faq/#how-does-hetzner-online-store-the-data-in-volumes))
- Ingress Nginx ... speaks for itself

```bash
helm dep update ./ops-tools
helm upgrade -n ops-tools --install -f ./ops-tools/values.yaml ops-tools ./ops-tools/ 
```