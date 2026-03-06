# ADR-002: Container Strategy — Docker + Apptainer Dual Runtime

## Status
Accepted

## Date
2026-03-06

## Author
Srilakshmi with Gemini at reviewer capacity

## Context
This project targets three execution environments:
- Local development on Mac (Docker via Docker Desktop)
- On-prem HPC clusters with Slurm (Apptainer required — rootless, no daemon)
- Cloud Kubernetes (Docker via containerd)

Slurm clusters in pharma HPC (including Eli Lilly, Owkin) use Apptainer (formerly Singularity)
because it runs rootless, requires no daemon, and produces immutable SIF images.
Kubernetes uses Docker/containerd natively. Slurm is developed and commercially supported by SchedMD.

We need a container strategy that works across all three without maintaining
two completely separate container definitions.

## Decision
- **Docker is the build format.** All containers are defined as Dockerfiles.
- **Apptainer SIF is derived from Docker images** using `Bootstrap: docker` in .def files.
- **Dockerfiles are designed with Apptainer constraints:**
  - Flat layers (no complex multi-stage that breaks conversion)
  - No ENTRYPOINT tricks (Apptainer uses %runscript)
  - Standard filesystem layout
  - GPU access via `--gpus all` (Docker) and `--nv` (Apptainer)
- **CI builds both:** GitHub Actions produces Docker image (GHCR) + Apptainer SIF in parallel.

## Consequences
- Single Dockerfile is source of truth for dependencies
- Apptainer .def file is thin wrapper (~20 lines)
- GPU passthrough must be tested in both runtimes (different mechanisms)
- Mac users cannot run Apptainer natively — requires Linux VM (OrbStack/Lima)

## Alternatives Considered
- **Apptainer-first:** K8s support immature, Docker Hub ecosystem loss. Rejected.
- **Podman:** Doesn't solve HPC gap. Rejected.
- **Enroot (NVIDIA):** Too NVIDIA-specific, small community. Noted for future GPU-only Slurm jobs.
