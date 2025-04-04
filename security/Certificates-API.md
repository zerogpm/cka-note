# Kubernetes Certificate Signing Request (CSR) Guide

## Creating a Certificate Signing Request

Create a CertificateSigningRequest object with the name `akshay` with the contents of the akshay.csr file.

As of kubernetes 1.19, the API to use for CSR is `certificates.k8s.io/v1`.

Please note that an additional field called `signerName` should also be added when creating CSR. For client authentication to the API server we will use the built-in signer `kubernetes.io/kube-apiserver-client`.

### First, encode the certificate request in base64:

```bash
cat akshay.csr | base64 -w 0
```

### Then create a YAML object:

```yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: akshay
spec:
  request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1ZqQ0NBVDRDQVFBd0VURVBNQTBHQTFVRUF3d0dZV3R6YUdGNU1JSUJJakFOQmdrcWhraUc5dzBCQVFFRgpBQU9DQVE4QU1JSUJDZ0tDQVFFQXN1MnlyS2R4SlhwYmRJWjJDVzA0OTVCWkxKYjU5WVI0enBUcThtVWhETkQ3CjJNN3pyRnlTUUdFWXR6VjY4bWVVRUcxQ2NWZkc0RkxCT215bmEva1dFQXM3ZFgvTmNCM2FsaFlHQ29YTlV2TXgKNVc0d1ZNb2lBT2w2YW5zSlVMUHVXOXhHYXVPeFlrYVFHazhZMG04elJwTFduNmE5dEJhWDVNRjhJdytaenFaNgpDUnhIZk9lWFlDVlg5MlVFNmkvOWtCOTRwRFZleU02c0Z2aFFQZ0ZOS0RQQ25idE1nemt4dndTeDVBQnZ1OUJ0ClRjZys2R2grNjRvNE5yUUw0ZkRaTkRNSGkrdWhWMS9zOGZOKzk3RFRRQ1hNRW1lQ0lqWVRoRWRvZWZ3WEd4aGYKNnEyb2cvV3Z5VmMyclRMcXc4M3FrRnJ0ckhEeDhvZHc0WVVscGd1d3h3SURBUUFCb0FBd0RRWUpLb1pJaHZjTgpBUUVMQlFBRGdnRUJBRXgrYWpneGV5YU5mK05CL2M5czlmUCtyTFpGWHRkZEdyajM2VlhhMlRzdkx3WTNmaWd4CldpRnAwUDVQSi9YMmtybVdZOVk1dzNXN0JnY24xalhOcHMvbzlXSWVoN3hQRm5KTFVxVE1jci9DWi9EZGJhUDIKY2ZLQTVoM3ZId29uRnVpSEViU1VBQ0NQVFJjZlhMcmJmUWE3cFB0SjQ0NUhkRFdueGZTTE82UmdRTEYyaDJ3SwpZSm5XNTRqZUgyWXBtMm5zZC8xSGlQSVZNUTdiRjVnN3VMWHovUkxuQ2dYTWdSUEhzZUZvaUZXbDhNOTloTXJUCkFlNUFKWlhnL3M4eG10Y2M2WG95OHhEZExKMjZtYUJSbGtnQ2Z6RGNtcHBvd2FqZHpXeGxweGpHczd3WURuU2IKUnVvQk5hVmhsbEdISTF6a0lMQmd5djF6TnBFU2ZEeVczWmc9Ci0tLS0tRU5EIENFUlRJRklDQVRFIFJFUVVFU1QtLS0tLQo=
  signerName: kubernetes.io/kube-apiserver-client
  usages:
    - client auth
```

## Checking CSR Status

What is the Condition of the newly created Certificate Signing Request object?

You can check using:

```bash
k get csr
```

**Output:**

```
NAME    AGE   SIGNERNAME                          REQUESTOR          REQUESTEDDURATION   CONDITION
akshay  115s  kubernetes.io/kube-apiserver-client kubernetes-admin   <none>              Pending
```

## Approving a CSR

```bash
k certificate approve akshay
```

**Output:**

```
certificatesigningrequest.certificates.k8s.io/akshay approved
```

## Investigating Suspicious CSRs

You are not aware of a request coming in. What groups is this CSR requesting access to?

Check the details about the request. Preferably in YAML.

```bash
kubectl get csr agent-smith -o yaml
```

**Output:**

```yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  creationTimestamp: "2025-04-04T03:38:29Z"
  name: agent-smith
  resourceVersion: "3932"
  uid: 990c181a-5cb5-4619-9a26-544a34c8104c
spec:
  extra:
    authentication.kubernetes.io/credential-id:
      - X509SHA256=2bb72baf41347cb5d798de9a84d17b94e908b9b17457973d63d8efebc4e9a3d0
  groups:
    - system:masters
    - system:authenticated
  request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1dEQ0NBVUFDQVFBd0V6RVJNQThHQTFVRUF3d0libVYzTFhWelpYSXdnZ0VpTUEwR0NTcUdTSWIzRFFFQgpBUVVBQTRJQkR3QXdnZ0VLQW9JQkFRRE8wV0pXK0RYc0FKU0lyanBObzV2UklCcGxuemcrNnhjOStVVndrS2kwCkxmQzI3dCsxZUVuT041TXVxOTlOZXZtTUVPbnJEVU8vdGh5VnFQMncyWE5JRFJYall5RjQwRmJtRCs1eld5Q0sKeTNCaWhoQjkzTUo3T3FsM1VUdlo4VEVMcXlhRGtuUmwvanYvU3hnWGtvazBBQlVUcFdNeDRCcFNpS2IwVSt0RQpJRjVueEF0dE1Wa0RQUTdOYmVaUkc0M2IrUVdsVkdSL3o2RFdPZkpuYmZlek90YUF5ZEdMVFpGQy93VHB6NTJrCkVjQ1hBd3FDaGpCTGt6MkJIUFI0Sjg5RDZYYjhrMzlwdTZqcHluZ1Y2dVAwdEliT3pwcU52MFkwcWRFWnB3bXcKajJxRUwraFpFV2trRno4MGxOTnR5VDVMeE1xRU5EQ25JZ3dDNEdaaVJHYnJBZ01CQUFHZ0FEQU5CZ2txaGtpRwo5dzBCQVFzRkFBT0NBUUVBUzlpUzZDMXV4VHVmNUJCWVNVN1FGUUhVemFsTnhBZFlzYU9SUlFOd0had0hxR2k0CmhPSzRhMnp5TnlpNDRPT2lqeWFENnRVVzhEU3hrcjhCTEs4S2czc3JSRXRKcWw1ckxaeTlMUlZyc0pnaEQ0Z1kKUDlOTCthRFJTeFJPVlNxQmFCMm5XZVlwTTVjSjVURjUzbGVzTlNOTUxRMisrUk1uakRRSjdqdVBFaWM4L2RoawpXcjJFVU02VWF3enlrcmRISW13VHYybWxNWTBSK0ROdFYxWWllKzBIOS9ZRWx0K0ZTR2poNUw1WVV2STFEcWl5CjRsM0UveTNxTDcxV2ZBY3VIM09zVnBVVW5RSVNNZFFzMHFXQ3NiRTU2Q0M1RGhQR1pJcFVibktVcEF3a2ErOEUKdndRMDdqRytocGtueG11RkFlWHhnVXdvZEFMYUo3anUvVERJY3c9PQotLS0tLUVORCBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0K
  signerName: kubernetes.io/kube-apiserver-client
  usages:
    - digital signature
    - key encipherment
    - server auth
  username: agent-x
status: {}
```

That doesn't look very right. Reject that request.

```bash
kubectl certificate deny agent-smith
```

## Deleting a CSR

```bash
kubectl delete csr agent-smith
```
