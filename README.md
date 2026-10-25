# fleet-oai-gnb

Fleet bundle for the **OpenAirInterface 5G gNB** (OAI `2026.w43`) driving a **USRP B210** over USB, as an IntEdge
edge app in a user cluster. The chart is OAI's own (`charts/`, from the OAI Helm charts) with the fixes listed in
[NOTICE](NOTICE): as published, its B210 mode does not render.

## Topology
- A **user cluster** with a worker pool named **`radio`** whose node VM holds the B210 (USB passthrough: allow the
  radio on the site's Devices page and grant it to the tenant, then give the pool that device).
- The **5G core** is free5GC (`fleet-free5gc-control`), reached through the IntEdge **`amf` binding**
  (`amf.upstream.svc.cluster.local`), as in `fleet-ueransim`. PLMN 208/93, TAC 1, slice SST 1 / SD 010203.

## The image (air-gapped sites)
OAI's image downloads the B210 firmware from the internet at every start. `image/Dockerfile` bakes it in (UHD's own
downloader at build time, checked against UHD's manifest) on top of `oaisoftwarealliance/oai-gnb:2026.w43` pinned by
digest. Build it on a workstation or CI machine, never on a hub machine, and push it to the hub registry:

```bash
docker build -t registry.intedge/intedge/oai-gnb-b210:2026.w43 image/
docker push registry.intedge/intedge/oai-gnb-b210:2026.w43
```
Then set `nfimage.digest` in `values.yaml` to the pushed digest. Without the baked image, set
`radio.firmware: download` and `nfimage.repository: docker.io/oaisoftwarealliance/oai-gnb` (needs internet).

## Values (values.yaml)
| Key | Default | Meaning |
|---|---|---|
| `config.radio` | `b2xx` | the radio: `b2xx`, `x3xx`, `n3xx`, or `rfsim` (no hardware) |
| `radio.firmware` | `baked` | `baked`: the image carries the USRP firmware; `download`: fetched at every start |
| `radio.usbAccess` | `privileged` | the B210 reconnects on the USB bus when its firmware loads, under a new device node: only a privileged container with the bus mounted keeps it (b2xx only) |
| `config.useAdditionalOptions` | `-E --continuous-tx …` | `-E`: three-quarter sampling (less USB load); `--continuous-tx`: the USRP workaround OAI recommends for the B210 |
| `nodeSelector` | `intedge.io/pool: radio` | the pool that holds the radio |
| `resources` | 3 CPUs, 3 GiB, requests = limits | a Guaranteed pod: with the static CPU manager it gets cores of its own |

## Security notes
- The gNB container is **privileged** in b2xx mode (USB access, see above), in the tenant's own user cluster, on the
  radio pool's node only. A device plugin that hands the radio over without privileges is planned.
- Every image is pinned by digest except the gNB's tag until `nfimage.digest` is set.
- The OAI software carries patent clauses (OAI Public License v1.1 / CSSL v1.0): see NOTICE.

## Verify
```bash
kubectl -n oai-gnb get pods -o wide
kubectl -n oai-gnb logs deploy/oai-gnb -c gnb | grep -iE "usrp|uhd|B210|underflow|late"
```
