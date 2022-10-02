# Container

- [Specification](#specification)

## Specification

- [Kubernetes Container Runtime Interface (CRI)](https://github.com/kubernetes/cri-api)
- [Open Container Initiative (OCI)](https://opencontainers.org)
    - [OCI Image Specification](https://github.com/opencontainers/image-spec)
    - [OCI Runtime Specification](https://github.com/opencontainers/runtime-spec)

```
           +--------------------------+
container  |       CLI / Daemon       |
management |  (dockerd, crictl, ...)  |  -----+
           +--------------------------+       |
                        |                     |
                     CRI spec           non-CRI spec
                        |                     |
                        v                     |
           +--------------------------+       |
           |           CRI            |       |
           +--------------------------+       |
high-level |  CRI-compatible runtime  |  <----+
 runtime   | (containerd, cri-o, ...) |
           +--------------------------+
                        |
                     OCI spec
                        |
                        v
           +--------------------------+
           |           OCI            |
           +--------------------------+
low-level  |  OCI-compatible runtime  |
 runtime   |     (runc, crun, ...)    |
           +--------------------------+
```
