#containers
## Recover from Expired Certificates

Certificates managed and used by k3s are valid for 1 year. During k3s startup, the remaining time of these certificates are checked. If the remaining time is less than 90 days, k3s will rotate these certificates so they are valid for another year. However if you find yourself in a situation where the certificates are not rotated and they are expired, you can recover from it using the following steps.

- Verify the current expiration date of the server certificates

```bash
$ ls /mnt/data/kubernetes/k3s/server/tls/*.crt | xargs -I {} sh -c "echo {};  openssl x509 -in {} -noout -text | grep Validity -A 2 ; echo ''"

/mnt/data/kubernetes/k3s/server/tls/client-admin.crt
        Validity
            Not Before: Apr  4 00:10:21 2022 GMT
            Not After : Apr  4 00:10:21 2023 GMT

/mnt/data/kubernetes/k3s/server/tls/client-auth-proxy.crt
        Validity
            Not Before: Apr  4 00:10:21 2022 GMT
            Not After : Apr  4 00:10:21 2023 GMT

...
```

- Verify the current expiration date of the agent certificates

```bash
$ ls /mnt/data/kubernetes/k3s/agent/*.crt | xargs -I {} sh -c "echo {};  openssl x509 -in {} -noout -text | grep Validity -A 2 ; echo ''"

/mnt/data/kubernetes/k3s/agent/client-ca.crt
        Validity
            Not Before: Apr  4 00:10:21 2022 GMT
            Not After : Apr  4 00:10:21 2032 GMT

/mnt/data/kubernetes/k3s/agent/client-k3s-controller.crt
        Validity
            Not Before: Apr  4 00:10:21 2022 GMT
            Not After : Apr  4 00:10:21 2023 GMT

...
```

- Manually set the date to when the certificates were still valid for LESS than 90 days

```bash
$ date -s "3 APR 2023 18:00:00"

Mon Apr  3 18:00:00 CEST 2023
```

- Restart k3s, this should trigger the rotation of all the certificates

```bash
$ systemctl restart k3s
```

- Verify that `kubectl` works again and that pods are active

```bash
$ kubectl get pods -n company
```

- Restore the date

```bash
$ hwclock --hctosys
```
