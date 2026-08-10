---
title:       "Diagnosing Distroless .NET Applications on Kubernetes"
date:        2026-08-10
tags:        ["dotnet", "kubernetes", "diagnostics", "containers"]
categories:  ["dotnet", "Kubernetes"]
---

# Diagnosing Distroless .NET Applications on Kubernetes

Minimal container images are a good production default. Distroless and chiseled
.NET images reduce image size and attack surface by leaving out package managers,
shells, and troubleshooting tools.

That becomes a challenge when a running application has high CPU usage, increasing
memory consumption, or unexplained latency. Installing tools in the application
container is not an option, and rebuilding the image changes the environment that
needs to be investigated.

[dotnet-k8s-debug-containers](https://github.com/koepalex/dotnet-k8s-debug-containers)
provides a separate diagnostics image for this scenario. It adds the standard
.NET diagnostic tools to a running Pod through a Kubernetes ephemeral container,
without adding them to the application image.

## Why Use a Separate Diagnostics Container?

The `diag` image is based on Azure Linux 3 and contains:

* `dotnet-counters`
* `dotnet-trace`
* `dotnet-dump`
* `dotnet-gcdump`
* `dotnet-stack`

The application container remains unchanged. It does not need a shell, the .NET
SDK, or a `/diag` volume mount.

The diagnostics container joins the target container's process namespace. A
PowerShell helper locates the .NET runtime's diagnostic socket through `/proc`
and exposes it under `/diag`, where the standard `dotnet-*` tools can discover
the process.

The same `/diag` directory is used for collected traces, dumps, and GC dumps.
It is backed by a Pod-scoped `emptyDir` that is mounted only into the ephemeral
container.

## Prepare the Pod

The required Pod settings must be present before an incident occurs. Kubernetes
cannot add a shared process namespace or a new Pod volume through an ephemeral
container.

The following sample shows the important parts:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app
spec:
  shareProcessNamespace: true
  securityContext:
    fsGroup: 1654
    fsGroupChangePolicy: OnRootMismatch
  containers:
    - name: app
      image: ghcr.io/example/my-app:latest
      securityContext:
        runAsUser: 1654
        runAsNonRoot: true
        allowPrivilegeEscalation: false
        capabilities:
          drop:
            - ALL
  volumes:
    - name: diagnostics
      emptyDir: {}
```

For a Deployment, add the settings below `.spec.template.spec` instead. The
important requirements are:

* `shareProcessNamespace: true` makes the application process visible to the
  diagnostics container.
* The application runs as UID `1654`, matching the default user in the dotnet 
  and `diag` images.
* Pod `fsGroup` `1654` makes the diagnostics volume writable.
* A writable `emptyDir` named `diagnostics` is available for the ephemeral
  container.
* .NET diagnostics remain enabled. This is the runtime default.

The complete sample is maintained in
[`examples/kubernetes/pod-with-diag-volume.yaml`](https://github.com/koepalex/dotnet-k8s-debug-containers/blob/main/examples/kubernetes/pod-with-diag-volume.yaml).

The operator running the helper also needs permission to:

* read Pods;
* update `pods/ephemeralcontainers`;
* create `pods/exec` requests.

Creating `pods/attach` requests is required only when directly attaching to the
ephemeral container.

## Start a Reusable Diagnostics Session

Clone the repository on a machine that has PowerShell and `kubectl` configured
for the target cluster:

```powershell
git clone https://github.com/koepalex/dotnet-k8s-debug-containers.git
Set-Location .\dotnet-k8s-debug-containers
```

Start the diagnostics container without directly attaching its primary shell:

```powershell
.\scripts\Start-DotnetDiagSession.ps1 `
  -Pod my-app `
  -TargetContainer app `
  -Namespace default `
  -NoAttach
```

The helper validates the target container and the `diagnostics` volume, creates
the ephemeral container, waits for it to start, and prepares .NET diagnostic
socket discovery.

It generates a unique name such as `dotnet-diag-a1b2c` and prints commands for
entering the container and listing .NET processes. Use the printed command
rather than constructing it manually:

```text
kubectl exec -it --namespace default pod/my-app --container dotnet-diag-a1b2c -- /bin/sh
```

`-NoAttach` is important for a reusable session. The container's primary shell
continues running when a later `kubectl exec` session ends, so artifacts can
still be copied.

Ephemeral containers cannot be restarted or replaced. Omitting
`-ContainerName` lets the helper generate a new unique name for every session.

Use `-WhatIf` before an incident if you want to inspect the generated Pod
payload without creating an ephemeral container:

```powershell
.\scripts\Start-DotnetDiagSession.ps1 `
  -Pod my-app `
  -TargetContainer app `
  -Namespace default `
  -NoAttach `
  -WhatIf
```

## Collect Runtime Evidence

After entering the diagnostics container, list the automatically discoverable
.NET processes:

```sh
dotnet-trace ps
```

Use the returned process ID for the following commands.

Start with runtime counters when you need a low-overhead view of CPU, allocation,
garbage collection, thread pool, and exception activity:

```sh
dotnet-counters monitor --process-id <pid> System.Runtime
```

Collect a trace for offline performance analysis:

```sh
dotnet-trace collect \
  --process-id <pid> \
  --output /diag/app.nettrace
```

Collect a memory dump when a complete process snapshot is required:

```sh
dotnet-dump collect \
  --process-id <pid> \
  --output /diag/app.dmp
```

Collect a GC dump for managed heap analysis with a smaller artifact than a full
process dump:

```sh
dotnet-gcdump collect \
  --process-id <pid> \
  --output /diag/app.gcdump
```

The files are written to the ephemeral container's `/diag` volume. The
application container does not need access to that directory.

## Copy the Artifacts

Exit the `kubectl exec` shell after collection. Because the diagnostics session
was created with `-NoAttach`, the ephemeral container remains available.

Use the generated container name printed by the helper:

```sh
kubectl cp \
  --namespace default \
  my-app:/diag/app.nettrace \
  ./app.nettrace \
  --container dotnet-diag-a1b2c
```

The diagnostics image includes `tar`, which `kubectl cp` uses for the transfer.
Repeat the command for `.dmp` or `.gcdump` artifacts.

The `emptyDir` and its files disappear when the Pod is removed. Copy required
artifacts before restarting, replacing, or deleting the Pod.

If you omit `-NoAttach`, the helper attaches directly to the container's primary
shell. In that mode, copy artifacts from another terminal before exiting the
attached shell. Exiting it terminates the ephemeral container, and Kubernetes
cannot restart it.

## Production Safety and Troubleshooting

> **Production guidance:** Prefer continuous metrics, distributed traces, and
> structured logs as the first line of investigation, ideally collected through
> OpenTelemetry and exported outside the Pod. This observability data is safer
> and cheaper to collect continuously than an on-demand process dump. However,
> it cannot explain every failure. A dump or targeted runtime trace may still be
> required for issues such as unexplained memory retention, deadlocks, thread
> pool starvation, native crashes, or application state that was not captured by
> existing instrumentation. Use this diagnostics workflow when normal
> observability narrows the problem but does not provide enough evidence to find
> the root cause.

Diagnostic collection affects the process being observed. Start with counters
and collect only the evidence needed for the investigation. Traces add CPU and
I/O overhead, while full memory dumps can briefly pause the process and require
significant memory, disk space, and transfer time.

Dump files can contain credentials, personal data, request payloads, and other
process memory. Store and transfer them as sensitive production data, restrict
access, and delete them according to the applicable retention policy.

Common failure cases include:

* **The target process is not visible:** confirm that
  `shareProcessNamespace: true` was set in the Pod template before the Pod was
  created.
* **No diagnostic socket is found:** confirm that .NET diagnostics are enabled
  and that the application process uses the same UID as the diagnostics image.
* **The diagnostics volume is rejected:** confirm that the Pod declares an
  `emptyDir` named `diagnostics`. The helper cannot add the Pod-level volume.
* **The Kubernetes API returns `Forbidden`:** check access to Pods,
  `pods/ephemeralcontainers`, and `pods/exec`.
* **The ephemeral container is blocked:** review Pod Security, seccomp, and
  AppArmor policies for the namespace.
* **A previous container name already exists:** start another session without
  specifying `-ContainerName` so the helper generates a unique name.

The helper fails early when the target container or diagnostics volume is
missing, and it verifies socket discovery before presenting the session as
ready.

## Conclusion

Minimal application images and production diagnostics do not have to be
conflicting goals. A prepared Pod template and an ephemeral diagnostics
container keep troubleshooting tools out of the application image while still
providing access to counters, traces, memory dumps, and GC dumps when they are
needed.

The source, container definitions, helper scripts, and current usage examples
are available in the
[dotnet-k8s-debug-containers repository](https://github.com/koepalex/dotnet-k8s-debug-containers).
