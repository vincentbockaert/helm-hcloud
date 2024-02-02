# ops-tools


## cert-manager with dns01 cloudflare

Create a secret containing a token with following permissions:

- Zone - DNS - Edit
- Zone - Zone - Read

Currently, created through Terraform :P, see [this stack](https://github.com/vincentbockaert/tf-cloudflare/tree/master/stacks/tokens)

More specifically the following [snippet](https://github.com/vincentbockaert/tf-cloudflare/blob/master/stacks/tokens/tokens.tf):

```terraform
# Token allowed to edit DNS entries
resource "cloudflare_api_token" "all_zones_dns_edit" {
  name = "all_zones_dns_edit"

  policy {
    permission_groups = [
      # https://developers.cloudflare.com/fundamentals/api/reference/permissions/#zone-permissions
      data.cloudflare_api_token_permission_groups.all.zone["DNS Read"],
      data.cloudflare_api_token_permission_groups.all.zone["DNS Write"],
    ]
    # all zones
    resources = {
      "com.cloudflare.api.account.zone.*" = "*"
    }
  }

  condition {
    request_ip {
      in = local.hcloud_k3s_nodes_addresses
    }
  }
}
```

I highly recommend that you set a condition on the token to only work when called from your node IP's.

Then to create the actual secret:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: cloudflare-api-token-secret
type: Opaque
stringData:
  api-token: BOGUS_HOCUS_POCUS
```

Applied with `kubectl -n ops-tools apply -f secret.yaml`. (rather manual apply than have it as part of my repo in the helm chart)

Once done, we just need to modify the *Issuer* manifest to use cloudflare as a solver with the token.

```yaml
solvers:
- dns01:
    cloudflare:
        apiTokenSecretRef:
        name: cloudflare-api-token-secret
        key: api-token
```