# Kubernetes — Local Setup & Command Reference

![Kubernetes](https://img.shields.io/badge/Kubernetes-Local%20Setup-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)
![Docker Desktop](https://img.shields.io/badge/Docker-Desktop-1D63ED?style=for-the-badge&logo=docker&logoColor=white)
![MERN Stack](https://img.shields.io/badge/Examples-MERN%20Stack-1A6B4A?style=for-the-badge)
![Commands](https://img.shields.io/badge/Reference-kubectl-C84B1F?style=for-the-badge)

> A GitHub-ready collection of the uploaded Kubernetes learning notes and command cheat sheet. The content is organized into a local setup guide followed by a practical `kubectl` reference.

## Original color palette

The first guide uses a warm paper background, dark ink, and these accent colors. GitHub controls the page theme and sanitizes many custom CSS styles, so the palette is represented here with badges and labels rather than relying on unsupported page-wide CSS.

![Accent orange](https://img.shields.io/badge/Accent- C84B1F-C84B1F?style=flat-square)
![Accent green](https://img.shields.io/badge/Accent-1A6B4A-1A6B4A?style=flat-square)
![Accent blue](https://img.shields.io/badge/Accent-1B4587-1B4587?style=flat-square)
![Accent gold](https://img.shields.io/badge/Accent-B8860B-B8860B?style=flat-square)

## Contents

1. [Kubernetes Local Setup — Complete Notes](#part-1-kubernetes-local-setup--complete-notes)
2. [Kubernetes Commands Cheat Sheet](#part-2-kubernetes-commands-cheat-sheet)

---

# Part 1: Kubernetes Local Setup — Complete Notes

::: layout
::: nav-brand
::: logo
K8s.Local
:::

::: sub
Sheryians Coding School
:::
:::

::: nav-group-label
Overview
:::

[[0]{.nav-num} What is Kubernetes](#what-is-k8s) [[1]{.nav-num}
Architecture](#architecture)

::: nav-group-label
Core Components
:::

[[2]{.nav-num} Pod](#pod) [[3]{.nav-num} Deployment](#deployment)
[[4]{.nav-num} ReplicaSet](#replicaset) [[5]{.nav-num}
Service](#service) [[6]{.nav-num} Ingress](#ingress) [[7]{.nav-num}
Ingress Controller](#ingress-controller)

::: nav-group-label
Setup
:::

[[8]{.nav-num} Enable Kubernetes](#enable-k8s) [[9]{.nav-num} Writing
YAML Files](#yaml-files) [[10]{.nav-num} Labels & Selectors](#labels)

::: nav-group-label
Run & Debug
:::

[[11]{.nav-num} Deploy Your App](#deploy) [[12]{.nav-num} Debug
Commands](#debug) [[13]{.nav-num} Autoscaling (HPA)](#hpa)
[[14]{.nav-num} Secrets (Local)](#secrets)

::: {role="main"}
::: hero
::: hero-eyebrow
Local Setup · Docker Desktop
:::

# Kubernetes on Your *Local Machine*

A complete guide to understanding and running Kubernetes locally using
Docker Desktop. Covers every core component with examples from a real
MERN stack project.

::: hero-meta
[Docker Desktop]{.hero-tag} [MERN Stack]{.hero-tag} [nginx
Ingress]{.hero-tag} [Node.js · Express]{.hero-tag}
:::
:::

::: {#what-is-k8s .section .section}
::: section-header
::: section-num-big
00
:::

::: section-title-block
::: section-title
What is Kubernetes?
:::

::: section-subtitle
The problem it solves and why you need it
:::
:::
:::

::: concept
::: concept-label
Core Concept
:::

**Kubernetes (K8s)** is a container orchestration system. It manages how
your Docker containers run --- making sure the right number are running,
restarting them when they crash, routing traffic between them, and
scaling them up when load increases. You describe what you want in YAML
files and Kubernetes makes it happen.
:::

### Without vs With Kubernetes

  Situation            Without Kubernetes                     With Kubernetes
  -------------------- -------------------------------------- -------------------------------------------------
  Container crashes    App goes down, manual restart needed   Kubernetes restarts it automatically
  Traffic spike        App slows down or crashes              HPA adds more containers automatically
  Multiple services    Manual ports, complex networking       Internal DNS --- services talk by name
  Deploy new version   Downtime during update                 Rolling update --- zero downtime
  Multiple replicas    Manage each container manually         Set replicas: 3 and Kubernetes handles the rest

### Key Terms at a glance

  Term                            Simple meaning
  ------------------------------- ----------------------------------------------------------------------------------
  [Pod]{.f-name}                  Smallest unit. One or more containers running together.
  [Deployment]{.f-name}           Manages pods --- keeps them running, handles updates and rollbacks.
  [Service]{.f-name}              Stable network endpoint that finds pods using labels and routes traffic to them.
  [Ingress]{.f-name}              HTTP routing rules --- which domain/path goes to which service.
  [Ingress Controller]{.f-name}   The actual pod (nginx) that reads Ingress rules and routes real traffic.
  [Namespace]{.f-name}            Virtual cluster inside a cluster. Used to separate environments.
  [ConfigMap]{.f-name}            Store non-secret config (env variables) separately from your image.
  [Secret]{.f-name}               Store sensitive config (passwords, tokens) encoded in the cluster.
:::

::: {#architecture .section .section}
::: section-header
::: section-num-big
01
:::

::: section-title-block
::: section-title
Architecture
:::

::: section-subtitle
How all components fit together
:::
:::
:::

::: concept
::: concept-label
Key Concept
:::

A Kubernetes cluster has two parts: the **Control Plane** (the brain ---
decides what runs where) and **Worker Nodes** (the muscle --- where your
pods actually run). On Docker Desktop both live on your local machine.
:::

### Full traffic flow

::: flow
::: flow-step
::: flow-line
::: flow-dot
:::

::: flow-connector
:::
:::

::: flow-content
::: flow-title
Browser / Client
:::

::: flow-detail
Makes HTTP request to express.local or localhost
:::
:::
:::

::: flow-step
::: flow-line
::: flow-dot
:::

::: flow-connector
:::
:::

::: flow-content
::: flow-title
Ingress Controller (nginx pod)
:::

::: flow-detail
Receives traffic. Reads Ingress rules to decide where to send it.
:::
:::
:::

::: flow-step
::: flow-line
::: flow-dot
:::

::: flow-connector
:::
:::

::: flow-content
::: flow-title
Ingress Resource (your ingress.yaml rules)
:::

::: flow-detail
Defines which host/path maps to which Service
:::
:::
:::

::: flow-step
::: flow-line
::: flow-dot
:::

::: flow-connector
:::
:::

::: flow-content
::: flow-title
Service (ClusterIP)
:::

::: flow-detail
Finds pods using label selector. Load balances across all matching pods.
:::
:::
:::

::: flow-step
::: flow-line
::: {.flow-dot style="background:var(--accent2)"}
:::
:::

::: flow-content
::: flow-title
Pod (your Express container)
:::

::: flow-detail
Handles the request and sends back a response
:::
:::
:::
:::

### Cluster Architecture Diagram

::: {style="background:var(--paper2);border:1px solid var(--rule);border-radius:12px;padding:24px 16px;margin:16px 0;overflow-x:auto;"}
![](data:image/svg+xml;base64,PHN2ZyBzdHlsZT0id2lkdGg6MTAwJTttYXgtd2lkdGg6ODYwcHg7ZGlzcGxheTpibG9jazttYXJnaW46MCBhdXRvO2ZvbnQtZmFtaWx5OiYjMzk7SUJNIFBsZXggTW9ubyYjMzk7LG1vbm9zcGFjZTsiIHZpZXdib3g9IjAgMCA4NjAgNTIwIiB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciPgo8IS0tIE91dGVyIENsdXN0ZXIgYm9yZGVyIC0tPgo8cmVjdCBmaWxsPSJub25lIiBoZWlnaHQ9IjUwMCIgcng9IjE2IiBzdHJva2U9IiNENENFQkMiIHN0cm9rZS1kYXNoYXJyYXk9IjYsNCIgc3Ryb2tlLXdpZHRoPSIyIiB3aWR0aD0iODQwIiB4PSIxMCIgeT0iMTAiIC8+Cjx0ZXh0IGZpbGw9IiM2QjY0NTUiIGZvbnQtc2l6ZT0iMTEiIGZvbnQtd2VpZ2h0PSI2MDAiIGxldHRlci1zcGFjaW5nPSIwLjFlbSIgeD0iMzAiIHk9IjMyIj5LVUJFUk5FVEVTIENMVVNURVIgKERvY2tlciBEZXNrdG9wKTwvdGV4dD4KPCEtLSDilIDilIAgTUFTVEVSIE5PREUg4pSA4pSAIC0tPgo8cmVjdCBmaWxsPSJyZ2JhKDI3LDY5LDEzNSwwLjA1KSIgaGVpZ2h0PSIzMTAiIHJ4PSIxMiIgc3Ryb2tlPSIjMUI0NTg3IiBzdHJva2Utd2lkdGg9IjEuNSIgd2lkdGg9IjM0MCIgeD0iMzAiIHk9IjQ0IiAvPgo8cmVjdCBmaWxsPSIjMUI0NTg3IiBoZWlnaHQ9IjI4IiByeD0iMTIiIHdpZHRoPSIzNDAiIHg9IjMwIiB5PSI0NCIgLz4KPHJlY3QgZmlsbD0iIzFCNDU4NyIgaGVpZ2h0PSIxNCIgd2lkdGg9IjM0MCIgeD0iMzAiIHk9IjU4IiAvPgo8dGV4dCBmaWxsPSIjZmZmIiBmb250LXNpemU9IjExIiBmb250LXdlaWdodD0iNzAwIiBsZXR0ZXItc3BhY2luZz0iMC4wOGVtIiB0ZXh0LWFuY2hvcj0ibWlkZGxlIiB4PSIyMDAiIHk9IjYzIj5NQVNURVIgTk9ERSAgwrcgIENvbnRyb2wgUGxhbmU8L3RleHQ+CjwhLS0gQVBJIFNlcnZlciAtLT4KPHJlY3QgZmlsbD0icmdiYSgyNyw2OSwxMzUsMC4xKSIgaGVpZ2h0PSI1MiIgcng9IjgiIHN0cm9rZT0iIzFCNDU4NyIgc3Ryb2tlLXdpZHRoPSIxLjIiIHdpZHRoPSIxNDAiIHg9IjUwIiB5PSI4NiIgLz4KPHRleHQgZmlsbD0iIzFCNDU4NyIgZm9udC1zaXplPSIxMCIgZm9udC13ZWlnaHQ9IjcwMCIgdGV4dC1hbmNob3I9Im1pZGRsZSIgeD0iMTIwIiB5PSIxMDgiPkFQSSBTZXJ2ZXI8L3RleHQ+Cjx0ZXh0IGZpbGw9IiM2QjY0NTUiIGZvbnQtc2l6ZT0iOSIgdGV4dC1hbmNob3I9Im1pZGRsZSIgeD0iMTIwIiB5PSIxMjQiPkVudHJ5IHBvaW50IGZvciBhbGw8L3RleHQ+Cjx0ZXh0IGZpbGw9IiM2QjY0NTUiIGZvbnQtc2l6ZT0iOSIgdGV4dC1hbmNob3I9Im1pZGRsZSIgeD0iMTIwIiB5PSIxMzUiPmt1YmVjdGwgY29tbWFuZHM8L3RleHQ+CjwhLS0gU2NoZWR1bGVyIC0tPgo8cmVjdCBmaWxsPSJyZ2JhKDI3LDY5LDEzNSwwLjEpIiBoZWlnaHQ9IjUyIiByeD0iOCIgc3Ryb2tlPSIjMUI0NTg3IiBzdHJva2Utd2lkdGg9IjEuMiIgd2lkdGg9IjE0MCIgeD0iMjEwIiB5PSI4NiIgLz4KPHRleHQgZmlsbD0iIzFCNDU4NyIgZm9udC1zaXplPSIxMCIgZm9udC13ZWlnaHQ9IjcwMCIgdGV4dC1hbmNob3I9Im1pZGRsZSIgeD0iMjgwIiB5PSIxMDgiPlNjaGVkdWxlcjwvdGV4dD4KPHRleHQgZmlsbD0iIzZCNjQ1NSIgZm9udC1zaXplPSI5IiB0ZXh0LWFuY2hvcj0ibWlkZGxlIiB4PSIyODAiIHk9IjEyNCI+QXNzaWducyBwb2RzIHRvPC90ZXh0Pgo8dGV4dCBmaWxsPSIjNkI2NDU1IiBmb250LXNpemU9IjkiIHRleHQtYW5jaG9yPSJtaWRkbGUiIHg9IjI4MCIgeT0iMTM1Ij53b3JrZXIgbm9kZXM8L3RleHQ+CjwhLS0gQ29udHJvbGxlciBNYW5hZ2VyIC0tPgo8cmVjdCBmaWxsPSJyZ2JhKDI3LDY5LDEzNSwwLjEpIiBoZWlnaHQ9IjUyIiByeD0iOCIgc3Ryb2tlPSIjMUI0NTg3IiBzdHJva2Utd2lkdGg9IjEuMiIgd2lkdGg9IjE0MCIgeD0iNTAiIHk9IjE1NCIgLz4KPHRleHQgZmlsbD0iIzFCNDU4NyIgZm9udC1zaXplPSIxMCIgZm9udC13ZWlnaHQ9IjcwMCIgdGV4dC1hbmNob3I9Im1pZGRsZSIgeD0iMTIwIiB5PSIxNzYiPkNvbnRyb2xsZXIgTWdyPC90ZXh0Pgo8dGV4dCBmaWxsPSIjNkI2NDU1IiBmb250LXNpemU9IjkiIHRleHQtYW5jaG9yPSJtaWRkbGUiIHg9IjEyMCIgeT0iMTkyIj5SdW5zIFJlcGxpY2FTZXQsPC90ZXh0Pgo8dGV4dCBmaWxsPSIjNkI2NDU1IiBmb250LXNpemU9IjkiIHRleHQtYW5jaG9yPSJtaWRkbGUiIHg9IjEyMCIgeT0iMjAzIj5EZXBsb3ltZW50IGxvb3BzPC90ZXh0Pgo8IS0tIGV0Y2QgLS0+CjxyZWN0IGZpbGw9InJnYmEoMTg0LDEzNCwxMSwwLjA4KSIgaGVpZ2h0PSI1MiIgcng9IjgiIHN0cm9rZT0iI0I4ODYwQiIgc3Ryb2tlLXdpZHRoPSIxLjIiIHdpZHRoPSIxNDAiIHg9IjIxMCIgeT0iMTU0IiAvPgo8dGV4dCBmaWxsPSIjQjg4NjBCIiBmb250LXNpemU9IjEwIiBmb250LXdlaWdodD0iNzAwIiB0ZXh0LWFuY2hvcj0ibWlkZGxlIiB4PSIyODAiIHk9IjE3NiI+ZXRjZDwvdGV4dD4KPHRleHQgZmlsbD0iIzZCNjQ1NSIgZm9udC1zaXplPSI5IiB0ZXh0LWFuY2hvcj0ibWlkZGxlIiB4PSIyODAiIHk9IjE5MiI+Q2x1c3RlciBzdGF0ZSBzdG9yZTwvdGV4dD4KPHRleHQgZmlsbD0iIzZCNjQ1NSIgZm9udC1zaXplPSI5IiB0ZXh0LWFuY2hvcj0ibWlkZGxlIiB4PSIyODAiIHk9IjIwMyI+KGtleS12YWx1ZSBEQik8L3RleHQ+CjwhLS0gUG9kcyBydW5uaW5nIG9uIG1hc3RlciBub3RlIC0tPgo8cmVjdCBmaWxsPSJyZ2JhKDIwMCw3NSwzMSwwLjA3KSIgaGVpZ2h0PSI0OCIgcng9IjYiIHN0cm9rZT0iI0M4NEIxRiIgc3Ryb2tlLWRhc2hhcnJheT0iNCwzIiBzdHJva2Utd2lkdGg9IjEiIHdpZHRoPSIzMDAiIHg9IjUwIiB5PSIyMjIiIC8+Cjx0ZXh0IGZpbGw9IiNDODRCMUYiIGZvbnQtc2l6ZT0iOSIgZm9udC13ZWlnaHQ9IjcwMCIgdGV4dC1hbmNob3I9Im1pZGRsZSIgeD0iMjAwIiB5PSIyNDEiPuKamSBTeXN0ZW0gUG9kcyAgKHJ1bm5pbmcgb24gTWFzdGVyKTwvdGV4dD4KPHRleHQgZmlsbD0iIzZCNjQ1NSIgZm9udC1zaXplPSI5IiB0ZXh0LWFuY2hvcj0ibWlkZGxlIiB4PSIyMDAiIHk9IjI1NiI+a3ViZS1kbnMgIMK3ICBtZXRyaWNzLXNlcnZlciAgwrcgIGluZ3Jlc3Mtbmdpbng8L3RleHQ+Cjx0ZXh0IGZpbGw9IiM2QjY0NTUiIGZvbnQtc2l6ZT0iOSIgdGV4dC1hbmNob3I9Im1pZGRsZSIgeD0iMjAwIiB5PSIyNjgiPmNvcmVkbnMgIMK3ICBrdWJlLXByb3h5ICDCtyAgZXRjZCBwb2Q8L3RleHQ+CjwhLS0gTWV0cmljcyBTZXJ2ZXIgY2FsbG91dCAtLT4KPHJlY3QgZmlsbD0icmdiYSgyNiwxMDcsNzQsMC4wNykiIGhlaWdodD0iMzIiIHJ4PSI2IiBzdHJva2U9IiMxQTZCNEEiIHN0cm9rZS13aWR0aD0iMSIgd2lkdGg9IjMwMCIgeD0iNTAiIHk9IjI4MiIgLz4KPHRleHQgZmlsbD0iIzFBNkI0QSIgZm9udC1zaXplPSI5IiBmb250LXdlaWdodD0iNzAwIiB0ZXh0LWFuY2hvcj0ibWlkZGxlIiB4PSIyMDAiIHk9IjI5OCI+8J+TiiBtZXRyaWNzLXNlcnZlciBwb2Qg4oaSIGZlZWRzIENQVSBkYXRhIHRvIEhQQTwvdGV4dD4KPHRleHQgZmlsbD0iIzZCNjQ1NSIgZm9udC1zaXplPSI5IiB0ZXh0LWFuY2hvcj0ibWlkZGxlIiB4PSIyMDAiIHk9IjMxMCI+TXVzdCBiZSBpbnN0YWxsZWQgZm9yIGt1YmVjdGwgdG9wIC8gSFBBIHRvIHdvcms8L3RleHQ+CjwhLS0g4pSA4pSAIFdPUktFUiBOT0RFIOKUgOKUgCAtLT4KPHJlY3QgZmlsbD0icmdiYSgyNiwxMDcsNzQsMC4wNCkiIGhlaWdodD0iMzEwIiByeD0iMTIiIHN0cm9rZT0iIzFBNkI0QSIgc3Ryb2tlLXdpZHRoPSIxLjUiIHdpZHRoPSI0NDAiIHg9IjQwMCIgeT0iNDQiIC8+CjxyZWN0IGZpbGw9IiMxQTZCNEEiIGhlaWdodD0iMjgiIHJ4PSIxMiIgd2lkdGg9IjQ0MCIgeD0iNDAwIiB5PSI0NCIgLz4KPHJlY3QgZmlsbD0iIzFBNkI0QSIgaGVpZ2h0PSIxNCIgd2lkdGg9IjQ0MCIgeD0iNDAwIiB5PSI1OCIgLz4KPHRleHQgZmlsbD0iI2ZmZiIgZm9udC1zaXplPSIxMSIgZm9udC13ZWlnaHQ9IjcwMCIgbGV0dGVyLXNwYWNpbmc9IjAuMDhlbSIgdGV4dC1hbmNob3I9Im1pZGRsZSIgeD0iNjIwIiB5PSI2MyI+V09SS0VSIE5PREUgIMK3ICBXaGVyZSBQb2RzIFJ1bjwvdGV4dD4KPCEtLSBrdWJlbGV0IC0tPgo8cmVjdCBmaWxsPSJyZ2JhKDI2LDEwNyw3NCwwLjEpIiBoZWlnaHQ9IjUyIiByeD0iOCIgc3Ryb2tlPSIjMUE2QjRBIiBzdHJva2Utd2lkdGg9IjEuMiIgd2lkdGg9IjEyMCIgeD0iNDE2IiB5PSI4NiIgLz4KPHRleHQgZmlsbD0iIzFBNkI0QSIgZm9udC1zaXplPSIxMCIgZm9udC13ZWlnaHQ9IjcwMCIgdGV4dC1hbmNob3I9Im1pZGRsZSIgeD0iNDc2IiB5PSIxMDgiPmt1YmVsZXQ8L3RleHQ+Cjx0ZXh0IGZpbGw9IiM2QjY0NTUiIGZvbnQtc2l6ZT0iOSIgdGV4dC1hbmNob3I9Im1pZGRsZSIgeD0iNDc2IiB5PSIxMjQiPlJ1bnMgcG9kcyBvbjwvdGV4dD4KPHRleHQgZmlsbD0iIzZCNjQ1NSIgZm9udC1zaXplPSI5IiB0ZXh0LWFuY2hvcj0ibWlkZGxlIiB4PSI0NzYiIHk9IjEzNSI+dGhpcyBub2RlPC90ZXh0Pgo8IS0tIGt1YmUtcHJveHkgLS0+CjxyZWN0IGZpbGw9InJnYmEoMjYsMTA3LDc0LDAuMSkiIGhlaWdodD0iNTIiIHJ4PSI4IiBzdHJva2U9IiMxQTZCNEEiIHN0cm9rZS13aWR0aD0iMS4yIiB3aWR0aD0iMTIwIiB4PSI1NTQiIHk9Ijg2IiAvPgo8dGV4dCBmaWxsPSIjMUE2QjRBIiBmb250LXNpemU9IjEwIiBmb250LXdlaWdodD0iNzAwIiB0ZXh0LWFuY2hvcj0ibWlkZGxlIiB4PSI2MTQiIHk9IjEwOCI+a3ViZS1wcm94eTwvdGV4dD4KPHRleHQgZmlsbD0iIzZCNjQ1NSIgZm9udC1zaXplPSI5IiB0ZXh0LWFuY2hvcj0ibWlkZGxlIiB4PSI2MTQiIHk9IjEyNCI+SGFuZGxlcyBuZXR3b3JraW5nPC90ZXh0Pgo8dGV4dCBmaWxsPSIjNkI2NDU1IiBmb250LXNpemU9IjkiIHRleHQtYW5jaG9yPSJtaWRkbGUiIHg9IjYxNCIgeT0iMTM1Ij5ydWxlcyBmb3Igc2VydmljZXM8L3RleHQ+CjwhLS0gQ29udGFpbmVyIFJ1bnRpbWUgLS0+CjxyZWN0IGZpbGw9InJnYmEoMjYsMTA3LDc0LDAuMSkiIGhlaWdodD0iNTIiIHJ4PSI4IiBzdHJva2U9IiMxQTZCNEEiIHN0cm9rZS13aWR0aD0iMS4yIiB3aWR0aD0iMTMwIiB4PSI2OTIiIHk9Ijg2IiAvPgo8dGV4dCBmaWxsPSIjMUE2QjRBIiBmb250LXNpemU9IjEwIiBmb250LXdlaWdodD0iNzAwIiB0ZXh0LWFuY2hvcj0ibWlkZGxlIiB4PSI3NTciIHk9IjEwOCI+Q29udGFpbmVyPC90ZXh0Pgo8dGV4dCBmaWxsPSIjMUE2QjRBIiBmb250LXNpemU9IjEwIiBmb250LXdlaWdodD0iNzAwIiB0ZXh0LWFuY2hvcj0ibWlkZGxlIiB4PSI3NTciIHk9IjEyMCI+UnVudGltZTwvdGV4dD4KPHRleHQgZmlsbD0iIzZCNjQ1NSIgZm9udC1zaXplPSI5IiB0ZXh0LWFuY2hvcj0ibWlkZGxlIiB4PSI3NTciIHk9IjEzMyI+RG9ja2VyIC8gY29udGFpbmVyZDwvdGV4dD4KPCEtLSBEZXBsb3ltZW50IGJveCAtLT4KPHJlY3QgZmlsbD0icmdiYSgyMDAsNzUsMzEsMC4wNCkiIGhlaWdodD0iMTgwIiByeD0iOCIgc3Ryb2tlPSIjQzg0QjFGIiBzdHJva2UtZGFzaGFycmF5PSI1LDMiIHN0cm9rZS13aWR0aD0iMS4yIiB3aWR0aD0iNDAwIiB4PSI0MTYiIHk9IjE1NCIgLz4KPHRleHQgZmlsbD0iI0M4NEIxRiIgZm9udC1zaXplPSI5IiBmb250LXdlaWdodD0iNzAwIiBsZXR0ZXItc3BhY2luZz0iMC4wOGVtIiB4PSI0OTAiIHk9IjE3MiI+REVQTE9ZTUVOVCAgKG1hbmFnZXMgUmVwbGljYVNldCk8L3RleHQ+CjwhLS0gUmVwbGljYVNldCBib3ggLS0+CjxyZWN0IGZpbGw9InJnYmEoMjAwLDc1LDMxLDAuMDQpIiBoZWlnaHQ9IjE0NCIgcng9IjYiIHN0cm9rZT0iI0M4NEIxRiIgc3Ryb2tlLWRhc2hhcnJheT0iMywzIiBzdHJva2Utd2lkdGg9IjEiIHdpZHRoPSIzNzYiIHg9IjQyOCIgeT0iMTc4IiAvPgo8dGV4dCBmaWxsPSIjQzg0QjFGIiBmb250LXNpemU9IjkiIGZvbnQtd2VpZ2h0PSI2MDAiIGxldHRlci1zcGFjaW5nPSIwLjA2ZW0iIHg9IjUwMiIgeT0iMTk0Ij5SRVBMSUNBU0VUICAoZW5zdXJlcyBkZXNpcmVkIHBvZCBjb3VudCk8L3RleHQ+CjwhLS0gUG9kIDEgLS0+CjxyZWN0IGZpbGw9InJnYmEoMjcsNjksMTM1LDAuMDgpIiBoZWlnaHQ9IjEwOCIgcng9IjgiIHN0cm9rZT0iIzFCNDU4NyIgc3Ryb2tlLXdpZHRoPSIxLjIiIHdpZHRoPSIxMDgiIHg9IjQ0MCIgeT0iMjAwIiAvPgo8dGV4dCBmaWxsPSIjMUI0NTg3IiBmb250LXNpemU9IjkiIGZvbnQtd2VpZ2h0PSI3MDAiIHRleHQtYW5jaG9yPSJtaWRkbGUiIHg9IjQ5NCIgeT0iMjIwIj5QT0QgMTwvdGV4dD4KPHJlY3QgZmlsbD0icmdiYSgyNyw2OSwxMzUsMC4xMikiIGhlaWdodD0iMjIiIHJ4PSI0IiB3aWR0aD0iODQiIHg9IjQ1MiIgeT0iMjI2IiAvPgo8dGV4dCBmaWxsPSIjMUI0NTg3IiBmb250LXNpemU9IjguNSIgdGV4dC1hbmNob3I9Im1pZGRsZSIgeD0iNDk0IiB5PSIyNDEiPmV4cHJlc3Mg8J+fojwvdGV4dD4KPHJlY3QgZmlsbD0icmdiYSgyNiwxMDcsNzQsMC4xKSIgaGVpZ2h0PSIxOCIgcng9IjMiIHdpZHRoPSI4NCIgeD0iNDUyIiB5PSIyNTIiIC8+Cjx0ZXh0IGZpbGw9IiMxQTZCNEEiIGZvbnQtc2l6ZT0iOCIgdGV4dC1hbmNob3I9Im1pZGRsZSIgeD0iNDk0IiB5PSIyNjUiPkNsdXN0ZXJJUDwvdGV4dD4KPHJlY3QgZmlsbD0icmdiYSgxODQsMTM0LDExLDAuMDgpIiBoZWlnaHQ9IjE0IiByeD0iMyIgd2lkdGg9Ijg0IiB4PSI0NTIiIHk9IjI3NCIgLz4KPHRleHQgZmlsbD0iI0I4ODYwQiIgZm9udC1zaXplPSI4IiB0ZXh0LWFuY2hvcj0ibWlkZGxlIiB4PSI0OTQiIHk9IjI4NCI+U2VjcmV0IOKavzwvdGV4dD4KPHRleHQgZmlsbD0iIzZCNjQ1NSIgZm9udC1zaXplPSI4IiB0ZXh0LWFuY2hvcj0ibWlkZGxlIiB4PSI0OTQiIHk9IjMwMCI+YXBwOiBleHByZXNzPC90ZXh0Pgo8IS0tIFBvZCAyIC0tPgo8cmVjdCBmaWxsPSJyZ2JhKDI3LDY5LDEzNSwwLjA4KSIgaGVpZ2h0PSIxMDgiIHJ4PSI4IiBzdHJva2U9IiMxQjQ1ODciIHN0cm9rZS13aWR0aD0iMS4yIiB3aWR0aD0iMTA4IiB4PSI1NjIiIHk9IjIwMCIgLz4KPHRleHQgZmlsbD0iIzFCNDU4NyIgZm9udC1zaXplPSI5IiBmb250LXdlaWdodD0iNzAwIiB0ZXh0LWFuY2hvcj0ibWlkZGxlIiB4PSI2MTYiIHk9IjIyMCI+UE9EIDI8L3RleHQ+CjxyZWN0IGZpbGw9InJnYmEoMjcsNjksMTM1LDAuMTIpIiBoZWlnaHQ9IjIyIiByeD0iNCIgd2lkdGg9Ijg0IiB4PSI1NzQiIHk9IjIyNiIgLz4KPHRleHQgZmlsbD0iIzFCNDU4NyIgZm9udC1zaXplPSI4LjUiIHRleHQtYW5jaG9yPSJtaWRkbGUiIHg9IjYxNiIgeT0iMjQxIj5leHByZXNzIPCfn6I8L3RleHQ+CjxyZWN0IGZpbGw9InJnYmEoMjYsMTA3LDc0LDAuMSkiIGhlaWdodD0iMTgiIHJ4PSIzIiB3aWR0aD0iODQiIHg9IjU3NCIgeT0iMjUyIiAvPgo8dGV4dCBmaWxsPSIjMUE2QjRBIiBmb250LXNpemU9IjgiIHRleHQtYW5jaG9yPSJtaWRkbGUiIHg9IjYxNiIgeT0iMjY1Ij5DbHVzdGVySVA8L3RleHQ+CjxyZWN0IGZpbGw9InJnYmEoMTg0LDEzNCwxMSwwLjA4KSIgaGVpZ2h0PSIxNCIgcng9IjMiIHdpZHRoPSI4NCIgeD0iNTc0IiB5PSIyNzQiIC8+Cjx0ZXh0IGZpbGw9IiNCODg2MEIiIGZvbnQtc2l6ZT0iOCIgdGV4dC1hbmNob3I9Im1pZGRsZSIgeD0iNjE2IiB5PSIyODQiPlNlY3JldCDimr88L3RleHQ+Cjx0ZXh0IGZpbGw9IiM2QjY0NTUiIGZvbnQtc2l6ZT0iOCIgdGV4dC1hbmNob3I9Im1pZGRsZSIgeD0iNjE2IiB5PSIzMDAiPmFwcDogZXhwcmVzczwvdGV4dD4KPCEtLSBQb2QgMyAoSFBBIHNjYWxlZCkgLS0+CjxyZWN0IGZpbGw9InJnYmEoMjYsMTA3LDc0LDAuMDgpIiBoZWlnaHQ9IjEwOCIgcng9IjgiIHN0cm9rZT0iIzFBNkI0QSIgc3Ryb2tlLWRhc2hhcnJheT0iNCwyIiBzdHJva2Utd2lkdGg9IjEuMiIgd2lkdGg9IjEwOCIgeD0iNjg0IiB5PSIyMDAiIC8+Cjx0ZXh0IGZpbGw9IiMxQTZCNEEiIGZvbnQtc2l6ZT0iOSIgZm9udC13ZWlnaHQ9IjcwMCIgdGV4dC1hbmNob3I9Im1pZGRsZSIgeD0iNzM4IiB5PSIyMjAiPlBPRCAzPC90ZXh0Pgo8dGV4dCBmaWxsPSIjMUE2QjRBIiBmb250LXNpemU9IjgiIHRleHQtYW5jaG9yPSJtaWRkbGUiIHg9IjczOCIgeT0iMjM0Ij4oSFBBIHNjYWxlZCk8L3RleHQ+CjxyZWN0IGZpbGw9InJnYmEoMjYsMTA3LDc0LDAuMTIpIiBoZWlnaHQ9IjIyIiByeD0iNCIgd2lkdGg9Ijg0IiB4PSI2OTYiIHk9IjI0MCIgLz4KPHRleHQgZmlsbD0iIzFBNkI0QSIgZm9udC1zaXplPSI4LjUiIHRleHQtYW5jaG9yPSJtaWRkbGUiIHg9IjczOCIgeT0iMjU1Ij5leHByZXNzIPCfn6E8L3RleHQ+CjxyZWN0IGZpbGw9InJnYmEoMTg0LDEzNCwxMSwwLjA4KSIgaGVpZ2h0PSIxNCIgcng9IjMiIHdpZHRoPSI4NCIgeD0iNjk2IiB5PSIyNjYiIC8+Cjx0ZXh0IGZpbGw9IiNCODg2MEIiIGZvbnQtc2l6ZT0iOCIgdGV4dC1hbmNob3I9Im1pZGRsZSIgeD0iNzM4IiB5PSIyNzYiPlNlY3JldCDimr88L3RleHQ+Cjx0ZXh0IGZpbGw9IiM2QjY0NTUiIGZvbnQtc2l6ZT0iOCIgdGV4dC1hbmNob3I9Im1pZGRsZSIgeD0iNzM4IiB5PSIzMDAiPmFwcDogZXhwcmVzczwvdGV4dD4KPCEtLSDilIDilIAgQk9UVE9NIFJPVzogU2VydmljZSwgSW5ncmVzcywgSFBBIOKUgOKUgCAtLT4KPHJlY3QgZmlsbD0icmdiYSgyNiwxMDcsNzQsMC4wNykiIGhlaWdodD0iNjAiIHJ4PSI4IiBzdHJva2U9IiMxQTZCNEEiIHN0cm9rZS13aWR0aD0iMS4zIiB3aWR0aD0iMjAwIiB4PSIzMCIgeT0iMzc0IiAvPgo8dGV4dCBmaWxsPSIjMUE2QjRBIiBmb250LXNpemU9IjEwIiBmb250LXdlaWdodD0iNzAwIiB0ZXh0LWFuY2hvcj0ibWlkZGxlIiB4PSIxMzAiIHk9IjM5NiI+U2VydmljZSAoQ2x1c3RlcklQKTwvdGV4dD4KPHRleHQgZmlsbD0iIzZCNjQ1NSIgZm9udC1zaXplPSI5IiB0ZXh0LWFuY2hvcj0ibWlkZGxlIiB4PSIxMzAiIHk9IjQxMiI+RmluZHMgcG9kcyBieSBsYWJlbDwvdGV4dD4KPHRleHQgZmlsbD0iIzZCNjQ1NSIgZm9udC1zaXplPSI5IiB0ZXh0LWFuY2hvcj0ibWlkZGxlIiB4PSIxMzAiIHk9IjQyNCI+YXBwOiBleHByZXNzPC90ZXh0Pgo8cmVjdCBmaWxsPSJyZ2JhKDIwMCw3NSwzMSwwLjA3KSIgaGVpZ2h0PSI2MCIgcng9IjgiIHN0cm9rZT0iI0M4NEIxRiIgc3Ryb2tlLXdpZHRoPSIxLjMiIHdpZHRoPSIxNjAiIHg9IjI1MCIgeT0iMzc0IiAvPgo8dGV4dCBmaWxsPSIjQzg0QjFGIiBmb250LXNpemU9IjEwIiBmb250LXdlaWdodD0iNzAwIiB0ZXh0LWFuY2hvcj0ibWlkZGxlIiB4PSIzMzAiIHk9IjM5NiI+SW5ncmVzczwvdGV4dD4KPHRleHQgZmlsbD0iIzZCNjQ1NSIgZm9udC1zaXplPSI5IiB0ZXh0LWFuY2hvcj0ibWlkZGxlIiB4PSIzMzAiIHk9IjQxMiI+Um91dGVzIGV4dGVybmFsPC90ZXh0Pgo8dGV4dCBmaWxsPSIjNkI2NDU1IiBmb250LXNpemU9IjkiIHRleHQtYW5jaG9yPSJtaWRkbGUiIHg9IjMzMCIgeT0iNDI0Ij5IVFRQIHRyYWZmaWM8L3RleHQ+CjxyZWN0IGZpbGw9InJnYmEoMTg0LDEzNCwxMSwwLjA3KSIgaGVpZ2h0PSI2MCIgcng9IjgiIHN0cm9rZT0iI0I4ODYwQiIgc3Ryb2tlLXdpZHRoPSIxLjMiIHdpZHRoPSIxNjAiIHg9IjQzMCIgeT0iMzc0IiAvPgo8dGV4dCBmaWxsPSIjQjg4NjBCIiBmb250LXNpemU9IjEwIiBmb250LXdlaWdodD0iNzAwIiB0ZXh0LWFuY2hvcj0ibWlkZGxlIiB4PSI1MTAiIHk9IjM5NiI+SFBBPC90ZXh0Pgo8dGV4dCBmaWxsPSIjNkI2NDU1IiBmb250LXNpemU9IjkiIHRleHQtYW5jaG9yPSJtaWRkbGUiIHg9IjUxMCIgeT0iNDEyIj5TY2FsZXMgcG9kcyB2aWE8L3RleHQ+Cjx0ZXh0IGZpbGw9IiM2QjY0NTUiIGZvbnQtc2l6ZT0iOSIgdGV4dC1hbmNob3I9Im1pZGRsZSIgeD0iNTEwIiB5PSI0MjQiPm1ldHJpY3Mtc2VydmVyIENQVTwvdGV4dD4KPHJlY3QgZmlsbD0icmdiYSgyNyw2OSwxMzUsMC4wNikiIGhlaWdodD0iNjAiIHJ4PSI4IiBzdHJva2U9IiMxQjQ1ODciIHN0cm9rZS13aWR0aD0iMS4zIiB3aWR0aD0iMjIwIiB4PSI2MTAiIHk9IjM3NCIgLz4KPHRleHQgZmlsbD0iIzFCNDU4NyIgZm9udC1zaXplPSIxMCIgZm9udC13ZWlnaHQ9IjcwMCIgdGV4dC1hbmNob3I9Im1pZGRsZSIgeD0iNzIwIiB5PSIzOTYiPlNlY3JldHMgLyBDb25maWdNYXA8L3RleHQ+Cjx0ZXh0IGZpbGw9IiM2QjY0NTUiIGZvbnQtc2l6ZT0iOSIgdGV4dC1hbmNob3I9Im1pZGRsZSIgeD0iNzIwIiB5PSI0MTIiPkVudiB2YXJzIGluamVjdGVkIGludG88L3RleHQ+Cjx0ZXh0IGZpbGw9IiM2QjY0NTUiIGZvbnQtc2l6ZT0iOSIgdGV4dC1hbmNob3I9Im1pZGRsZSIgeD0iNzIwIiB5PSI0MjQiPnBvZHMgYXQgcnVudGltZTwvdGV4dD4KPCEtLSDilIDilIAgRVhURVJOQUwgVFJBRkZJQyDilIDilIAgLS0+CjxyZWN0IGZpbGw9InJnYmEoMTQsMTUsMjAsMC4wNCkiIGhlaWdodD0iNDQiIHJ4PSI4IiBzdHJva2U9IiNENENFQkMiIHN0cm9rZS13aWR0aD0iMS4yIiB3aWR0aD0iODEwIiB4PSIzMCIgeT0iNDU0IiAvPgo8dGV4dCBmaWxsPSIjNkI2NDU1IiBmb250LXNpemU9IjEwIiBmb250LXdlaWdodD0iNjAwIiB0ZXh0LWFuY2hvcj0ibWlkZGxlIiB4PSI0MzUiIHk9IjQ3MyI+8J+MkCAgQnJvd3NlciAvIGN1cmwgIOKGkiAgbG9jYWxob3N0ICDihpIgIEluZ3Jlc3MgQ29udHJvbGxlciAobmdpbnggcG9kIG9uIE1hc3RlcikgIOKGkiAgU2VydmljZSAg4oaSICBQb2Q8L3RleHQ+Cjx0ZXh0IGZpbGw9IiM2QjY0NTUiIGZvbnQtc2l6ZT0iOSIgdGV4dC1hbmNob3I9Im1pZGRsZSIgeD0iNDM1IiB5PSI0OTAiPmt1YmVjdGwgIOKGkiAgQVBJIFNlcnZlciAg4oaSICBTY2hlZHVsZXIgLyBDb250cm9sbGVyICDihpIgIGt1YmVsZXQgb24gV29ya2VyICDihpIgIFBvZCBjcmVhdGVkPC90ZXh0Pgo8IS0tIEFycm93OiBtYXN0ZXIg4oaUIHdvcmtlciAtLT4KPGxpbmUgbWFya2VyLWVuZD0idXJsKCNhcnIpIiBzdHJva2U9IiNENENFQkMiIHN0cm9rZS13aWR0aD0iMiIgeDE9IjM3MCIgeDI9IjQwMCIgeTE9IjIwMCIgeTI9IjIwMCI+PC9saW5lPgo8ZGVmcz4KPG1hcmtlciBpZD0iYXJyIiBtYXJrZXJoZWlnaHQ9IjciIG1hcmtlcndpZHRoPSI3IiBvcmllbnQ9ImF1dG8iIHJlZng9IjYiIHJlZnk9IjMuNSI+Cjxwb2x5Z29uIGZpbGw9IiNENENFQkMiIHBvaW50cz0iMCAwLCA3IDMuNSwgMCA3Ij48L3BvbHlnb24+CjwvbWFya2VyPgo8L2RlZnM+CjwhLS0gQXJyb3c6IHNlcnZpY2UgLT4gcG9kcyAtLT4KPGxpbmUgbWFya2VyLWVuZD0idXJsKCNhcnIyKSIgc3Ryb2tlPSIjMUE2QjRBIiBzdHJva2UtZGFzaGFycmF5PSI0LDIiIHN0cm9rZS13aWR0aD0iMS4yIiB4MT0iMTMwIiB4Mj0iNDk0IiB5MT0iMzc0IiB5Mj0iMzA4Ij48L2xpbmU+CjxkZWZzPgo8bWFya2VyIGlkPSJhcnIyIiBtYXJrZXJoZWlnaHQ9IjYiIG1hcmtlcndpZHRoPSI2IiBvcmllbnQ9ImF1dG8iIHJlZng9IjUiIHJlZnk9IjMiPgo8cG9seWdvbiBmaWxsPSIjMUE2QjRBIiBwb2ludHM9IjAgMCwgNiAzLCAwIDYiPjwvcG9seWdvbj4KPC9tYXJrZXI+CjwvZGVmcz4KPCEtLSBBcnJvdzogSFBBIC0+IHJlcGxpY2FzZXQgLS0+CjxsaW5lIG1hcmtlci1lbmQ9InVybCgjYXJyMykiIHN0cm9rZT0iI0I4ODYwQiIgc3Ryb2tlLWRhc2hhcnJheT0iNCwyIiBzdHJva2Utd2lkdGg9IjEuMiIgeDE9IjUxMCIgeDI9IjYxNiIgeTE9IjM3NCIgeTI9IjMzNCI+PC9saW5lPgo8ZGVmcz4KPG1hcmtlciBpZD0iYXJyMyIgbWFya2VyaGVpZ2h0PSI2IiBtYXJrZXJ3aWR0aD0iNiIgb3JpZW50PSJhdXRvIiByZWZ4PSI1IiByZWZ5PSIzIj4KPHBvbHlnb24gZmlsbD0iI0I4ODYwQiIgcG9pbnRzPSIwIDAsIDYgMywgMCA2Ij48L3BvbHlnb24+CjwvbWFya2VyPgo8L2RlZnM+Cjwvc3ZnPg==)
:::

### Project file structure

::: code-wrap
::: code-header
[Project Layout]{.code-label}
:::

    your-project/
    ├── Backend/
    │   ├── server.js
    │   ├── package.json
    │   └── dockerfile
    └── k8s/               # all Kubernetes YAML files here
        ├── deployment.yaml    # how to run the app
        ├── service.yaml       # how to reach the app
        └── ingress.yaml       # how external traffic enters
:::
:::

::: {#pod .section .section}
::: section-header
::: section-num-big
02
:::

::: section-title-block
::: section-title
Pod
:::

::: section-subtitle
The smallest deployable unit in Kubernetes
:::
:::
:::

::: concept
::: concept-label
Key Concept
:::

A **Pod** is a wrapper around one or more containers. All containers in
a pod share the same network (same IP address) and storage. In practice,
most pods contain a single container --- your app. You rarely create
pods directly --- a Deployment creates and manages them for you.
:::

### What a pod contains

  What                      Description
  ------------------------- -----------------------------------------------------------------------------------------------------------------------------------------------------------
  [Container(s)]{.f-name}   One or more Docker containers. They share the same localhost inside the pod.
  [IP Address]{.f-name}     Each pod gets its own IP inside the cluster. This IP changes every time a pod restarts --- which is why you use a Service instead of the pod IP directly.
  [Labels]{.f-name}         Key-value tags like app: express. Services and Deployments use these to find the pod.
  [Resources]{.f-name}      CPU and memory limits and requests defined in the pod spec.

### Pod lifecycle

::: scale-diagram
[Pending]{.hi} → pod is scheduled, image is being pulled\
    ↓\
[Running]{.hi} → container started successfully\
    ↓\
[Ready]{.green} → readiness probe passed, pod receives traffic ✅\
    ↓\
[CrashLoopBackOff]{.red} → container keeps crashing, Kubernetes retries\
    ↓\
[ImagePullBackOff]{.red} → cannot pull the Docker image
:::

::: note
::: note-icon
💡
:::

You **never create pods directly** in production. Always use a
Deployment. If a pod is created directly and crashes, nothing restarts
it. A Deployment automatically replaces crashed pods.
:::

### Check pod status

::: code-wrap
::: code-header
[Terminal]{.code-label}

copy
:::

    kubectl get pods                       # list all pods
    kubectl describe pod <pod-name>         # full details + events
    kubectl logs <pod-name>                # app console output
    kubectl exec -it <pod-name> -- sh     # shell inside pod
:::
:::

::: {#deployment .section .section}
::: section-header
::: section-num-big
03
:::

::: section-title-block
::: section-title
Deployment
:::

::: section-subtitle
Manages pods --- runs, scales, updates and rolls back
:::
:::
:::

::: concept
::: concept-label
Key Concept
:::

A **Deployment** is a manager for your pods. You tell it what image to
run, how many replicas you want, and what resources each pod needs. It
ensures that exact state is always maintained --- if a pod crashes,
Deployment creates a new one. If you update the image, it does a rolling
update with zero downtime.
:::

### What Deployment is responsible for

  Responsibility                  What it means
  ------------------------------- ------------------------------------------------------------------------------------------------------------------------------------
  [Replica management]{.f-name}   Always keeps the exact number of pods running. If one crashes, a new one is created immediately.
  [Rolling update]{.f-name}       When you push a new image, Kubernetes replaces pods one by one. Old pods serve traffic until new ones are ready --- zero downtime.
  [Rollback]{.f-name}             If a new version has a bug, one command goes back to the previous working version.
  [Scaling]{.f-name}              Change replica count manually or let HPA do it automatically based on CPU.

### Full deployment.yaml with explanation

::: code-wrap
::: code-header
[deployment.yaml]{.code-label}
:::

    apiVersion: apps/v1          # Deployment lives in apps group
    kind: Deployment
    metadata:
      name: express-deployment
    spec:
      replicas: 2              # always keep 2 pods running
      selector:
        matchLabels:
          app: express          # (1) manage pods that have this label
      template:              # blueprint for each pod
        metadata:
          labels:
            app: express        # (2) stick this label on every pod created
            owner: ankur       # extra label — deployment ignores it
        spec:
          containers:
            - name: express
              image: express-k8s:latest
              imagePullPolicy: IfNotPresent  # use local image if it exists
              ports:
                - containerPort: 3000         # port your app listens on
              resources:
                limits:
                  memory: "128Mi"            # max memory pod can use
                  cpu: "500m"               # max CPU (500m = 0.5 core)
                requests:
                  memory: "64Mi"             # memory reserved for this pod
                  cpu: "250m"               # CPU reserved (required for HPA)
:::

### Key fields explained

  Field                                 Value                    What it does
  ------------------------------------- ------------------------ ----------------------------------------------------------------------------------------------------------------
  [apiVersion]{.f-name}                 [apps/v1]{.f-val}        Deployment belongs to the apps API group. Not the core group (v1) --- that\'s for Pods and Services.
  [replicas]{.f-name}                   [2]{.f-val}              Always maintain 2 running pods. If one crashes, a new one is created immediately.
  [selector.matchLabels]{.f-name}       [app: express]{.f-val}   The Deployment manages pods that have ALL these labels. Must match template.metadata.labels.
  [template.metadata.labels]{.f-name}   [app: express]{.f-val}   Labels stamped on every pod this Deployment creates. Service uses these to find pods.
  [imagePullPolicy]{.f-name}            [IfNotPresent]{.f-val}   Use local image if available. On EKS change this to Always so it pulls from ECR on every restart.
  [resources.requests]{.f-name}         [cpu: 250m]{.f-val}      CPU/memory reserved for this pod. HPA requires requests to be set --- it uses this to calculate utilization %.
  [resources.limits]{.f-name}           [cpu: 500m]{.f-val}      Maximum CPU/memory the pod can use. Prevents one pod from starving others.

### CPU units explained

  Value              Meaning
  ------------------ -----------------------
  [1000m]{.f-name}   1 full CPU core
  [500m]{.f-name}    Half a CPU core
  [250m]{.f-name}    Quarter of a CPU core

### Useful Deployment commands

::: code-wrap
::: code-header
[Terminal]{.code-label}

copy
:::

    kubectl get deployments
    kubectl rollout status deployment/express-deployment
    kubectl rollout undo deployment/express-deployment    # rollback
    kubectl scale deployment express-deployment --replicas=5
    kubectl rollout restart deployment/express-deployment # re-pull image
:::
:::

::: {#replicaset .section .section}
::: section-header
::: section-num-big
04
:::

::: section-title-block
::: section-title
ReplicaSet
:::

::: section-subtitle
Ensures a fixed number of identical pods are always running
:::
:::
:::

::: concept
::: concept-label
Key Concept
:::

A **ReplicaSet** is the component that guarantees *N* copies of your pod
are always running. If a pod crashes, ReplicaSet creates a replacement.
If you have too many pods, it deletes extras. You almost never create a
ReplicaSet directly --- a **Deployment creates and manages a ReplicaSet
for you**, and also handles rolling updates and rollbacks on top of it.
:::

### Deployment → ReplicaSet → Pods relationship

::: scale-diagram
[Deployment]{.hi} → owns and manages a ReplicaSet\
    ↓\
[ReplicaSet]{.hi} → ensures *replicas: 2* pods are always alive\
    ↓\
[Pod 1 (running ✅) Pod 2 (running ✅)]{.green}\
    ↓ Pod 1 crashes\
[ReplicaSet detects: 1 pod running, desired is 2]{.red}\
    ↓\
[ReplicaSet creates Pod 3 automatically ✅]{.green}
:::

### Why you don\'t write ReplicaSet YAML directly

  If you use                       What you get
  -------------------------------- ------------------------------------------------------------------------------------------
  [Deployment]{.f-name}            ReplicaSet management + rolling updates + rollback history. Always use this in practice.
  [ReplicaSet directly]{.f-name}   Pod count maintenance only. No rolling updates, no rollback. Not recommended.

::: note
::: note-icon
💡
:::

When you run `kubectl get replicasets` you\'ll see an auto-generated
ReplicaSet like `express-deployment-7d9f8b6c4` --- the hash suffix is a
fingerprint of the pod template. When you update the image, a **new
ReplicaSet** is created and the old one is scaled down to zero (but kept
for rollback).
:::

### Viewing ReplicaSets

::: code-wrap
::: code-header
[Terminal]{.code-label}

copy
:::

    kubectl get replicasets                          # list all ReplicaSets
    kubectl describe replicaset <rs-name>           # details + events

    # After a rollout you will see TWO replicasets:
    # express-deployment-7d9f8b6c4   2         2         2       (new — active)
    # express-deployment-5c8a3d1b2   0         0         0       (old — kept for rollback)
:::

### How ReplicaSet selects pods

ReplicaSet uses **label selectors** --- it looks for pods with matching
labels and counts them. If you manually create a pod with the same
labels, the ReplicaSet will count it and may delete one of its own pods
to maintain the desired count. This is why labels must be unique per
Deployment.

::: code-wrap
::: code-header
[replicaset.yaml --- for reference only, use Deployment in
practice]{.code-label}
:::

    apiVersion: apps/v1
    kind: ReplicaSet
    metadata:
      name: express-rs
    spec:
      replicas: 2                  # desired pod count
      selector:
        matchLabels:
          app: express            # manage pods with this label
      template:
        metadata:
          labels:
            app: express
        spec:
          containers:
            - name: express
              image: express-k8s:latest
              imagePullPolicy: IfNotPresent
              ports:
                - containerPort: 3000
:::
:::

::: {#service .section .section}
::: section-header
::: section-num-big
04
:::

::: section-title-block
::: section-title
Service
:::

::: section-subtitle
Stable network endpoint that routes traffic to pods
:::
:::
:::

::: concept
::: concept-label
Key Concept
:::

Pods are temporary --- they crash and restart with new IP addresses. A
**Service** gives you a stable endpoint that never changes. It finds
pods using **label selectors** and load balances traffic across all
matching pods automatically. When a pod is added or removed, the Service
updates its list instantly.
:::

### How Service finds pods (Labels & Selectors)

::: code-wrap
::: code-header
[service.yaml]{.code-label}
:::

    apiVersion: v1              # Service is a core resource
    kind: Service
    metadata:
      name: express-service
    spec:
      selector:
        app: express            # find ALL pods with label app=express
      ports:
        - protocol: TCP
          port: 80              # port the service listens on (inside cluster)
          targetPort: 3000      # port your app listens on inside the container
      type: ClusterIP          # only accessible inside the cluster
:::

### Three Service types

  Type           Accessible from            Use case
  -------------- -------------------------- ---------------------------------------------------------------------
  ClusterIP      Inside cluster only        Pod-to-pod communication. Default type. Used with Ingress.
  NodePort       External via port number   Quick local testing. Access via localhost:30001. No Ingress needed.
  LoadBalancer   External via cloud LB      Production on AWS/GCP. Creates real Load Balancer. Costs money.

### port vs targetPort

  Field                   Value            Meaning
  ----------------------- ---------------- ----------------------------------------------------------------------------------------------------------
  [port]{.f-name}         [80]{.f-val}     The port the Service exposes inside the cluster. Other pods call this service on port 80.
  [targetPort]{.f-name}   [3000]{.f-val}   The port your app is actually listening on inside the container. Must match containerPort in deployment.

### NodePort for local testing

::: code-wrap
::: code-header
[service.yaml (NodePort)]{.code-label}
:::

    type: NodePort
    ports:
      - port: 80
        targetPort: 3000
        nodePort: 30001    # access via localhost:30001 in browser
:::

### How Service load balances

::: scale-diagram
[Service receives request (selector: app=express)]{.hi}\
    ↓\
[Looks up Endpoints list (live pod IPs)]{.hi}\
    ↓\
[Request 1 → Pod 1 (192.168.1.10)]{.green}\
[Request 2 → Pod 2 (192.168.1.11)]{.green}\
[Request 3 → Pod 3 (192.168.1.12)]{.green}\
[Request 4 → Pod 1 (round robin) ✅]{.green}
:::

::: code-wrap
::: code-header
[Terminal]{.code-label}

copy
:::

    kubectl get services
    kubectl get endpoints express-service   # see which pod IPs are in the pool
:::
:::

::: {#ingress .section .section}
::: section-header
::: section-num-big
05
:::

::: section-title-block
::: section-title
Ingress
:::

::: section-subtitle
HTTP routing rules --- host and path based traffic routing
:::
:::
:::

::: concept
::: concept-label
Key Concept
:::

An **Ingress** resource is a set of HTTP routing rules written in YAML.
It says things like: \"traffic for `express.local/` should go to
`express-service`\" or \"traffic for `express.local/auth` should go to
`auth-service`\". The Ingress itself does nothing --- it only works when
an **Ingress Controller** is installed to read and act on these rules.
:::

### Full ingress.yaml with explanation

::: code-wrap
::: code-header
[ingress.yaml]{.code-label}
:::

    apiVersion: networking.k8s.io/v1   # Ingress is in the networking group
    kind: Ingress
    metadata:
      name: express-ingress
    spec:
      ingressClassName: nginx         # which controller handles this rule
      rules:
        - host: express.local          # domain to match
          http:
            paths:
              - path: /
                pathType: Prefix         # match / and everything after
                backend:
                  service:
                    name: express-service
                    port:
                      number: 80
:::

### Path based routing --- multiple services on one domain

::: code-wrap
::: code-header
[ingress.yaml --- multi service]{.code-label}
:::

    rules:
      - host: express.local
        http:
          paths:
            - path: /
              pathType: Prefix
              backend:
                service:
                  name: main-service       # express.local/ → main app
                  port: { number: 80 }
            - path: /auth
              pathType: Prefix
              backend:
                service:
                  name: auth-service       # express.local/auth → auth service
                  port: { number: 80 }
:::

### Key fields explained

  Field                         Value                     What it does
  ----------------------------- ------------------------- --------------------------------------------------------------------------------------------------------------------------------------------------------------
  [ingressClassName]{.f-name}   [nginx]{.f-val}           Tells Kubernetes which Ingress Controller should handle this rule. If you have multiple controllers (nginx, traefik), each only picks up rules meant for it.
  [host]{.f-name}               [express.local]{.f-val}   Only traffic with this Host header matches this rule. Add it to /etc/hosts for local development.
  [pathType: Prefix]{.f-name}   [Prefix]{.f-val}          Matches the path and everything after it. /api matches /api/users, /api/data etc.
  [pathType: Exact]{.f-name}    [Exact]{.f-val}           Only matches that exact path. /api does NOT match /api/users.

### For local development --- skip the host

::: code-wrap
::: code-header
[ingress.yaml --- no host (easiest)]{.code-label}
:::

    rules:
      - http:                     # no host: field — matches all traffic
          paths:
            - path: /
              pathType: Prefix
              backend:
                service:
                  name: express-service
                  port:
                    number: 80
:::

Access via `http://localhost` directly --- no /etc/hosts edit needed.
:::

::: {#ingress-controller .section .section}
::: section-header
::: section-num-big
06
:::

::: section-title-block
::: section-title
Ingress Controller
:::

::: section-subtitle
The actual nginx pod that handles real traffic
:::
:::
:::

::: concept
::: concept-label
Key Concept
:::

The **Ingress Controller** is a pod running nginx inside your cluster.
It watches all Ingress resources and configures itself to route traffic
according to those rules. Without the controller, your ingress.yaml is
just a config file sitting there doing nothing. The controller is what
makes it actually work.
:::

### Ingress vs Ingress Controller

  What                     Ingress Resource                        Ingress Controller
  ------------------------ --------------------------------------- --------------------------------------------------
  What it is               A YAML file with routing rules          An actual nginx pod running in your cluster
  Created by               You --- kubectl apply -f ingress.yaml   You install once --- kubectl apply -f nginx-url
  Does it route traffic?   No --- it only defines rules            Yes --- reads rules and routes real HTTP traffic
  Namespace                default (your app namespace)            ingress-nginx (its own namespace)

::: step-label
Install nginx Ingress Controller
:::

::: code-wrap
::: code-header
[Terminal]{.code-label}

copy
:::

    kubectl apply -f \
      https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.12.1/deploy/static/provider/cloud/deploy.yaml
:::

::: {.step-label .green}
Verify controller is running
:::

::: code-wrap
::: code-header
[Terminal]{.code-label}

copy
:::

    kubectl get pods -n ingress-nginx

    # Expected output:
    NAME                                        READY   STATUS
    ingress-nginx-controller-xxxxxxxxx-xxxxx    1/1     Running ✅
:::

### Add express.local to /etc/hosts (Mac)

::: code-wrap
::: code-header
[Terminal]{.code-label}

copy
:::

    sudo nano /etc/hosts

    # Add this line at the bottom:
    127.0.0.1  express.local
:::

Now `http://express.local` in your browser will route to your nginx
Ingress Controller on localhost.

::: note
::: note-icon
💡
:::

**ingressClassName: nginx** in your ingress.yaml is how the controller
knows which rules are meant for it. If you had traefik installed too, it
only picks up rules with `ingressClassName: traefik`.
:::
:::

::: {#enable-k8s .section .section}
::: section-header
::: section-num-big
07
:::

::: section-title-block
::: section-title
Enable Kubernetes on Docker Desktop
:::

::: section-subtitle
One-click setup --- no extra tools needed
:::
:::
:::

::: step-label
Step 1 --- Enable in Docker Desktop settings
:::

Open Docker Desktop → click the **gear icon** (Settings) → click
**Kubernetes** in the left sidebar → check **Enable Kubernetes** → click
**Apply & Restart**.

Wait 2--3 minutes. When the bottom status bar shows a green Kubernetes
icon, it is ready.

::: {.step-label .green}
Step 2 --- Verify kubectl is working
:::

::: code-wrap
::: code-header
[Terminal]{.code-label}

copy
:::

    kubectl version
    kubectl get nodes

    # Expected output:
    NAME             STATUS   ROLES
    docker-desktop   Ready    control-plane ✅
:::

::: {.step-label .blue}
Step 3 --- Check current context
:::

::: code-wrap
::: code-header
[Terminal]{.code-label}

copy
:::

    kubectl config current-context
    # Should return: docker-desktop

    kubectl config get-contexts
    # Lists all clusters kubectl knows about
:::

::: note
::: note-icon
💡
:::

The `~/.kube/config` file stores all cluster connection info. When you
later create an EKS cluster with eksctl, it adds a new entry here and
switches the current context to EKS automatically. Switch back with
`kubectl config use-context docker-desktop`.
:::
:::

::: {#yaml-files .section .section}
::: section-header
::: section-num-big
08
:::

::: section-title-block
::: section-title
Writing YAML Files
:::

::: section-subtitle
Structure, apiVersion, and how to combine files
:::
:::
:::

::: concept
::: concept-label
Key Concept
:::

Every Kubernetes resource is defined in a YAML file. Each file must have
**apiVersion**, **kind**, **metadata**, and **spec**. The `apiVersion`
tells Kubernetes which API group the resource belongs to. Different
resources live in different groups.
:::

### apiVersion --- which API group

  Resource                                        apiVersion                       Why
  ----------------------------------------------- -------------------------------- -------------------------------------------------------------------------------------------
  [Pod, Service, ConfigMap, Secret]{.f-name}      [v1]{.f-val}                     Core group --- the oldest, most fundamental Kubernetes resources. No group prefix needed.
  [Deployment, StatefulSet, DaemonSet]{.f-name}   [apps/v1]{.f-val}                Apps group --- higher-level workload resources that manage pods.
  [Ingress, NetworkPolicy]{.f-name}               [networking.k8s.io/v1]{.f-val}   Networking group --- resources that manage cluster networking.
  [HorizontalPodAutoscaler]{.f-name}              [autoscaling/v2]{.f-val}         Autoscaling group --- resources that control scaling behavior.

### Putting multiple resources in one file

::: code-wrap
::: code-header
[combined.yaml]{.code-label}
:::

    apiVersion: apps/v1
    kind: Deployment
    # ... deployment config

    ---                    # separator — new resource starts here

    apiVersion: v1
    kind: Service
    # ... service config
:::

::: note
::: note-icon
💡
:::

Keep files **separate** as your project grows. One file per resource
makes it easier to find and edit things. Use `kubectl apply -f k8s/` to
apply the entire folder at once.
:::
:::

::: {#labels .section .section}
::: section-header
::: section-num-big
09
:::

::: section-title-block
::: section-title
Labels & Selectors
:::

::: section-subtitle
How Deployments manage pods and Services find them
:::
:::
:::

::: concept
::: concept-label
Key Concept
:::

**Labels** are key-value tags on pods. **Selectors** are filters used by
Deployments and Services to find pods with specific labels. This is how
all three resources connect --- the Deployment owns pods via selector,
the Service routes to pods via selector. No hardcoded pod names or IPs
anywhere.
:::

### The three-way connection

::: code-wrap
::: code-header
[How they connect]{.code-label}
:::

    # deployment.yaml
    selector:
      matchLabels:
        app: express      # (1) Deployment manages pods with this label
    template:
      metadata:
        labels:
          app: express    # (2) Pod gets stamped with this label

    ---
    # service.yaml
    selector:
      app: express        # (3) Service routes to pods with this label
:::

### Selector matching rules

  Pod Labels                             Selector: app=express                         Result
  -------------------------------------- --------------------------------------------- ----------------------------------
  [app: express]{.f-val}                 Matches                                       [Selected ✅]{.pill .pill-green}
  [app: express, env: prod]{.f-val}      Matches (extra label ignored)                 [Selected ✅]{.pill .pill-green}
  [app: backend]{.f-val}                 Does not match                                [Ignored ❌]{.pill .pill-red}
  [app: express, env: staging]{.f-val}   Partially matches selector app=express only   [Selected ✅]{.pill .pill-green}

### Multiple deployments using labels

::: code-wrap
::: code-header
[Using env label to separate deployments]{.code-label}
:::

    # staging-deployment.yaml
    selector:
      matchLabels:
        app: express
        env: staging     # only manages staging pods

    # production-deployment.yaml
    selector:
      matchLabels:
        app: express
        env: production  # only manages production pods
:::

Both deployments exist in the same cluster but never interfere with each
other because their selectors are different.
:::

::: {#deploy .section .section}
::: section-header
::: section-num-big
10
:::

::: section-title-block
::: section-title
Deploy Your App
:::

::: section-subtitle
Build image, apply YAML, access in browser
:::
:::
:::

::: step-label
Step 1 --- Build Docker image
:::

::: code-wrap
::: code-header
[Terminal]{.code-label}

copy
:::

    cd Backend/
    docker build -t express-k8s:latest .
:::

::: step-label
Step 2 --- Apply all YAML files
:::

::: code-wrap
::: code-header
[Terminal]{.code-label}

copy
:::

    kubectl apply -f k8s/    # applies all files in the k8s folder

    # Or apply individually:
    kubectl apply -f k8s/deployment.yaml
    kubectl apply -f k8s/service.yaml
    kubectl apply -f k8s/ingress.yaml
:::

::: {.step-label .green}
Step 3 --- Verify everything is running
:::

::: code-wrap
::: code-header
[Terminal]{.code-label}

copy
:::

    kubectl get pods           # STATUS: Running, READY: 1/1
    kubectl get services       # service exists with correct ports
    kubectl get ingress        # ingress has an ADDRESS
:::

::: {.step-label .blue}
Step 4 --- Access in browser
:::

  Setup                                              URL
  -------------------------------------------------- ----------------------------------
  [No host in ingress]{.f-name}                      [http://localhost]{.f-val}
  [host: express.local (with /etc/hosts)]{.f-name}   [http://express.local]{.f-val}
  [NodePort service]{.f-name}                        [http://localhost:30001]{.f-val}

### Update your app

::: code-wrap
::: code-header
[Terminal]{.code-label}

copy
:::

    # 1. Rebuild image after code changes
    docker build -t express-k8s:latest .

    # 2. Restart deployment to use new image
    kubectl rollout restart deployment/express-deployment
:::
:::

::: {#debug .section .section}
::: section-header
::: section-num-big
11
:::

::: section-title-block
::: section-title
Debug Commands
:::

::: section-subtitle
Finding and fixing problems in your cluster
:::
:::
:::

### Common errors and what to do

  Error                                                            Cause                                                                    Debug command
  ---------------------------------------------------------------- ------------------------------------------------------------------------ ---------------------------------------------------------------------
  [ImagePullBackOff]{.f-name style="color:var(--accent)"}          Cannot pull Docker image. Usually local image not built or wrong name.   `kubectl describe pod <name>` → check Events section
  [CrashLoopBackOff]{.f-name style="color:var(--accent)"}          Container keeps crashing. App has a startup error.                       `kubectl logs <pod-name>` → see app error output
  [503 Service Unavailable]{.f-name style="color:var(--accent)"}   Ingress controller running but no pods found. Label mismatch.            `kubectl get endpoints <service>` → if \<none\> labels don\'t match
  [404 Not Found]{.f-name style="color:var(--accent)"}             nginx running but no matching ingress rule for that host/path.           `kubectl describe ingress <name>` → check rules
  [Pending (pod)]{.f-name style="color:var(--accent)"}             No node has enough CPU/memory to schedule the pod.                       `kubectl describe pod <name>` → check Events
  [ECONNREFUSED]{.f-name style="color:var(--accent)"}              Pod calling another service using localhost instead of service name.     Use `http://service-name` not `http://localhost:PORT`

### Essential debug commands

::: code-wrap
::: code-header
[Terminal]{.code-label}

copy
:::

    # Check what is running
    kubectl get pods
    kubectl get services
    kubectl get ingress
    kubectl get all                              # everything at once

    # Deep inspect
    kubectl describe pod <pod-name>              # events + full config
    kubectl describe ingress <ingress-name>      # routing rules + backend IPs
    kubectl describe service <service-name>      # selector + endpoints

    # Logs
    kubectl logs <pod-name>                      # app output
    kubectl logs <pod-name> -f                  # follow live
    kubectl logs <pod-name> --previous          # logs from crashed container

    # Network debug
    kubectl get endpoints <service-name>        # pod IPs in service pool
    kubectl exec -it <pod-name> -- sh           # shell inside pod
    # Inside pod: test another service
    wget -qO- http://some-service/api/data
:::
:::

::: {#hpa .section .section}
::: section-header
::: section-num-big
12
:::

::: section-title-block
::: section-title
Autoscaling (HPA)
:::

::: section-subtitle
Automatically add and remove pods based on CPU usage
:::
:::
:::

::: concept
::: concept-label
Key Concept
:::

The **HorizontalPodAutoscaler (HPA)** watches CPU usage across your pods
and automatically scales the replica count up or down. When CPU exceeds
your threshold it adds pods. When traffic drops it removes them. It
requires **metrics-server** to read CPU data --- on Docker Desktop this
is *not* pre-installed and must be added manually.
:::

### Step 0 --- Install Metrics Server (required for HPA)

::: warn
::: note-icon
⚠️
:::

**HPA will not work without metrics-server.** Running `kubectl top pods`
will return *\"Metrics API not available\"* and HPA will show
`<unknown>/50%` for CPU until metrics-server is installed and running.
:::

::: code-wrap
::: code-header
[Terminal --- install metrics-server on Docker Desktop]{.code-label}

copy
:::

    # Option A: Official manifest with kubelet insecure TLS patch (recommended for Docker Desktop)
    kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

    # Docker Desktop uses a self-signed cert — patch metrics-server to skip TLS verify:
    kubectl patch deployment metrics-server -n kube-system \
      --type=json \
      -p='[{"op":"add","path":"/spec/template/spec/containers/0/args/-","value":"--kubelet-insecure-tls"}]'

    # Wait for it to be ready (~30 seconds)
    kubectl rollout status deployment/metrics-server -n kube-system

    # Verify it works — you should see CPU and Memory columns
    kubectl top nodes
    kubectl top pods
:::

::: code-wrap
::: code-header
[Terminal --- Option B: Helm install (if you have Helm)]{.code-label}

copy
:::

    helm repo add metrics-server https://kubernetes-sigs.github.io/metrics-server/
    helm repo update
    helm upgrade --install metrics-server metrics-server/metrics-server \
      --namespace kube-system \
      --set args={--kubelet-insecure-tls}
:::

::: note
::: note-icon
💡
:::

**Where does metrics-server run?** It runs as a pod in the `kube-system`
namespace on the master/control-plane node. Verify with:
`kubectl get pods -n kube-system | grep metrics`. You should see
`metrics-server-xxxx 1/1 Running`.
:::

### Create HPA via command

::: code-wrap
::: code-header
[Terminal]{.code-label}

copy
:::

    kubectl autoscale deployment express-deployment \
      --min=1 \
      --max=5 \
      --cpu-percent=50
:::

### Or as a YAML file (recommended)

::: code-wrap
::: code-header
[hpa.yaml]{.code-label}
:::

    apiVersion: autoscaling/v2
    kind: HorizontalPodAutoscaler
    metadata:
      name: express-hpa
    spec:
      scaleTargetRef:
        apiVersion: apps/v1
        kind: Deployment
        name: express-deployment    # must match deployment name
      minReplicas: 1
      maxReplicas: 5
      metrics:
        - type: Resource
          resource:
            name: cpu
            target:
              type: Utilization
              averageUtilization: 50   # scale when CPU > 50%
:::

### HPA scaling flow

::: scale-diagram
[Traffic increases → CPU per pod goes above 50%]{.hi}\
    ↓\
[HPA adds new pod (up to maxReplicas: 5)]{.green}\
    ↓\
[Requests spread across more pods]{.hi}\
    ↓\
[CPU per pod drops --- each pod handles less ✅]{.green}\
    ↓\
[Traffic drops → CPU goes low]{.hi}\
    ↓\
[HPA removes extra pods (down to minReplicas: 1) ✅]{.green}
:::

::: warn
::: note-icon
⚠️
:::

**resources.requests must be set** in your deployment for HPA to work.
HPA calculates utilization as *current CPU / requested CPU × 100*.
Without requests defined, HPA has no baseline to calculate from and will
not scale.
:::

::: code-wrap
::: code-header
[Terminal]{.code-label}

copy
:::

    kubectl get hpa              # see current CPU%, desired vs actual replicas
    kubectl top pods             # real-time CPU and memory per pod
:::
:::

::: {#secrets .section .section}
::: section-header
::: section-num-big
14
:::

::: section-title-block
::: section-title
Managing Secrets Locally
:::

::: section-subtitle
Store sensitive config (DB passwords, API keys, tokens) safely in your
local cluster
:::
:::
:::

::: concept
::: concept-label
Key Concept
:::

A **Secret** stores sensitive data like database passwords, API keys,
and JWT secrets inside the Kubernetes cluster --- encoded in base64.
Pods then access these values as environment variables or file mounts.
Unlike a ConfigMap (which is for non-sensitive config), Secrets signal
to Kubernetes that this data should be handled with care (not logged,
RBAC-restricted). In a local Docker Desktop setup secrets are fine
as-is; in production you would use external vaults.
:::

::: warn
::: note-icon
⚠️
:::

**base64 is NOT encryption.** It is just encoding. Anyone with
`kubectl get secret` access can decode it. Never commit secret YAML
files to git. Use `.gitignore` to exclude them, or use a tool like
*sealed-secrets* or *external-secrets* for production.
:::

### Step 1 --- Create base64 values

::: code-wrap
::: code-header
[Terminal --- encode your values]{.code-label}

copy
:::

    # Encode any string to base64
    echo -n 'mypassword' | base64
    # → bXlwYXNzd29yZA==

    echo -n 'mongodb://localhost:27017/mydb' | base64
    # → bW9uZ29kYjovL2xvY2FsaG9zdDoyNzAxNy9teWRi

    # Decode to verify
    echo 'bXlwYXNzd29yZA==' | base64 --decode
:::

### Step 2 --- Write secret.yaml

::: code-wrap
::: code-header
[k8s/secret.yaml]{.code-label}
:::

    apiVersion: v1
    kind: Secret
    metadata:
      name: express-secret
    type: Opaque                # Opaque = generic key-value secret
    data:
      MONGO_URI: bW9uZ29kYjovL2xvY2FsaG9zdDoyNzAxNy9teWRi  # base64 encoded
      JWT_SECRET: bXlzdXBlcnNlY3JldGtleQ==
      DB_PASSWORD: bXlwYXNzd29yZA==
:::

::: note
::: note-icon
💡
:::

**Tip --- use stringData instead of data** to write plain text directly
(Kubernetes encodes it for you). This is easier for local development
but the result is identical in the cluster.
:::

::: code-wrap
::: code-header
[k8s/secret.yaml --- using stringData (plaintext, easier
locally)]{.code-label}
:::

    apiVersion: v1
    kind: Secret
    metadata:
      name: express-secret
    type: Opaque
    stringData:                  # plain text — k8s base64-encodes automatically
      MONGO_URI: "mongodb://localhost:27017/mydb"
      JWT_SECRET: "mysupersecretkey"
      DB_PASSWORD: "mypassword"
:::

### Step 3 --- Apply the Secret

::: code-wrap
::: code-header
[Terminal]{.code-label}

copy
:::

    kubectl apply -f k8s/secret.yaml

    # Verify it was created
    kubectl get secrets
    kubectl describe secret express-secret   # shows keys but NOT values

    # Decode a specific value to verify
    kubectl get secret express-secret -o jsonpath='{.data.MONGO_URI}' | base64 --decode
:::

### Step 4 --- Inject Secret into your Deployment as env vars

::: code-wrap
::: code-header
[deployment.yaml --- env section]{.code-label}
:::

    spec:
      containers:
        - name: express
          image: express-k8s:latest
          env:
            - name: MONGO_URI          # name of env var inside the container
              valueFrom:
                secretKeyRef:
                  name: express-secret # secret name from metadata.name
                  key: MONGO_URI         # key inside the secret
            - name: JWT_SECRET
              valueFrom:
                secretKeyRef:
                  name: express-secret
                  key: JWT_SECRET
            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: express-secret
                  key: DB_PASSWORD
:::

### Access in your Node.js/Express code

::: code-wrap
::: code-header
[server.js]{.code-label}
:::

    // Kubernetes injects secrets as environment variables
    const mongoUri    = process.env.MONGO_URI;
    const jwtSecret   = process.env.JWT_SECRET;
    const dbPassword  = process.env.DB_PASSWORD;

    // Same code works locally with .env and in Kubernetes with Secrets
:::

### Secret vs ConfigMap --- when to use which

  Use case                    ConfigMap               Secret
  --------------------------- ----------------------- ------------
  Database password           ❌ Never                ✅ Yes
  API base URL                ✅ Yes                  Not needed
  JWT secret key              ❌ Never                ✅ Yes
  NODE_ENV = production       ✅ Yes                  Not needed
  MongoDB connection string   ❌ if it has password   ✅ Yes
  Port number                 ✅ Yes                  Not needed

### Add secret.yaml to your project structure

::: code-wrap
::: code-header
[Project Layout (updated)]{.code-label}
:::

    your-project/
    ├── Backend/
    │   ├── server.js
    │   ├── package.json
    │   └── dockerfile
    └── k8s/
        ├── deployment.yaml
        ├── service.yaml
        ├── ingress.yaml
        ├── hpa.yaml
        └── secret.yaml       # ← add to .gitignore !
:::

::: warn
::: note-icon
⚠️
:::

**Always add secret.yaml to .gitignore.** Committing secrets to git ---
even encoded base64 --- is a serious security risk. Add
`k8s/secret.yaml` to your `.gitignore` file. Share secrets with
teammates via a secure channel (1Password, Bitwarden, private Slack DM),
never via git.
:::

### Apply order matters

::: code-wrap
::: code-header
[Terminal --- always apply secret BEFORE deployment]{.code-label}

copy
:::

    # 1. Secret must exist before pods that reference it start up
    kubectl apply -f k8s/secret.yaml

    # 2. Then apply deployment (pods can now find the secret)
    kubectl apply -f k8s/deployment.yaml
    kubectl apply -f k8s/service.yaml
    kubectl apply -f k8s/ingress.yaml

    # Verify env vars were injected into the pod
    kubectl exec -it <pod-name> -- sh
    # Inside pod:
    echo $MONGO_URI
    echo $JWT_SECRET
:::
:::

Sheryians Coding School · Kubernetes Local Setup Notes Docker Desktop ·
MERN Stack · nginx Ingress
:::
:::

---

# Part 2: Kubernetes Commands Cheat Sheet

::: {.crossnote .markdown-preview}
# Kubernetes Commands Cheat Sheet

## Basic Cluster & Context {#basic-cluster--context}

  Command                               What it does                    Important Flags   Flag Description
  ------------------------------------- ------------------------------- ----------------- ------------------
  `kubectl version`                     Shows client & server version   `--short`         Short output
  `kubectl cluster-info`                Displays cluster info           ---               ---
  `kubectl config get-contexts`         Lists all contexts              ---               ---
  `kubectl config use-context <name>`   Switch context                  ---               ---
  `kubectl config current-context`      Shows current context           ---               ---

------------------------------------------------------------------------

## Pods

  Command                            What it does             Important Flags              Flag Description
  ---------------------------------- ------------------------ ---------------------------- -------------------------
  `kubectl get pods`                 List pods                `-o wide`                    More details (IP, node)
                                                              `-A`                         All namespaces
  `kubectl describe pod <pod>`       Detailed pod info        ---                          ---
  `kubectl logs <pod>`               Show logs                `-f`                         Follow logs
                                                              `--tail=50`                  Last 50 lines
  `kubectl exec -it <pod> -- bash`   Access container shell   `-c <container>`             Specify container
  `kubectl delete pod <pod>`         Delete pod               `--grace-period=0 --force`   Force delete

------------------------------------------------------------------------

## Deployments

  Command                                            What it does          Important Flags   Flag Description
  -------------------------------------------------- --------------------- ----------------- ------------------
  `kubectl get deployments`                          List deployments      `-A`              All namespaces
  `kubectl create deployment <name> --image=<img>`   Create deployment     `--replicas=3`    Set replicas
  `kubectl scale deployment <name>`                  Scale deployment      `--replicas=5`    Number of pods
  `kubectl rollout status deployment <name>`         Check rollout         ---               ---
  `kubectl rollout undo deployment <name>`           Rollback deployment   ---               ---

------------------------------------------------------------------------

## Services

  Command                              What it does      Important Flags        Flag Description
  ------------------------------------ ----------------- ---------------------- ------------------
  `kubectl get svc`                    List services     ---                    ---
  `kubectl expose deployment <name>`   Create service    `--type=NodePort`      Service type
                                                         `--port=80`            Service port
                                                         `--target-port=8080`   Container port
  `kubectl describe svc <name>`        Service details   ---                    ---

------------------------------------------------------------------------

## Namespaces

  Command                      What it does        Important Flags   Flag Description
  ---------------------------- ------------------- ----------------- ------------------
  `kubectl get ns`             List namespaces     ---               ---
  `kubectl create ns <name>`   Create namespace    ---               ---
  `kubectl delete ns <name>`   Delete namespace    ---               ---
  `kubectl get pods -n <ns>`   Pods in namespace   ---               ---

------------------------------------------------------------------------

## Nodes

  Command                          What it does              Important Flags         Flag Description
  -------------------------------- ------------------------- ----------------------- ------------------
  `kubectl get nodes`              List nodes                `-o wide`               Extra details
  `kubectl describe node <node>`   Node info                 ---                     ---
  `kubectl cordon <node>`          Mark node unschedulable   ---                     ---
  `kubectl uncordon <node>`        Enable scheduling         ---                     ---
  `kubectl drain <node>`           Evacuate pods             `--ignore-daemonsets`   Skip daemonsets

------------------------------------------------------------------------

## Config & Secrets {#config--secrets}

  Command                                  What it does       Important Flags              Flag Description
  ---------------------------------------- ------------------ ---------------------------- ------------------
  `kubectl create configmap <name>`        Create configmap   `--from-literal=key=value`   Inline data
  `kubectl get configmap`                  List configmaps    ---                          ---
  `kubectl create secret generic <name>`   Create secret      `--from-literal=key=value`   Inline secret
  `kubectl get secrets`                    List secrets       ---                          ---

------------------------------------------------------------------------

## Apply & Files {#apply--files}

  Command                          What it does       Important Flags   Flag Description
  -------------------------------- ------------------ ----------------- ------------------
  `kubectl apply -f file.yaml`     Apply config       `-R`              Recursive
  `kubectl delete -f file.yaml`    Delete resources   ---               ---
  `kubectl diff -f file.yaml`      Show changes       ---               ---
  `kubectl replace -f file.yaml`   Replace resource   ---               ---

------------------------------------------------------------------------

## Debugging & Monitoring {#debugging--monitoring}

  Command                What it does            Important Flags                           Flag Description
  ---------------------- ----------------------- ----------------------------------------- -------------------------
  `kubectl top pods`     Resource usage (pods)   ---                                       Metrics server required
  `kubectl top nodes`    Node usage              ---                                       ---
  `kubectl get events`   Cluster events          `--sort-by=.metadata.creationTimestamp`   Sort events
  `kubectl describe`     Debug any resource      ---                                       Works for all

------------------------------------------------------------------------

## Labels & Selectors {#labels--selectors}

  Command                                What it does      Important Flags   Flag Description
  -------------------------------------- ----------------- ----------------- ------------------
  `kubectl label pods <pod> key=value`   Add label         `--overwrite`     Replace existing
  `kubectl get pods -l key=value`        Filter by label   ---               ---

------------------------------------------------------------------------

## Port Forwarding

  Command                                    What it does      Important Flags   Flag Description
  ------------------------------------------ ----------------- ----------------- ------------------
  `kubectl port-forward pod/<pod> 8080:80`   Forward port      ---               Local:Remote
  `kubectl port-forward svc/<svc> 8080:80`   Forward service   ---               ---

------------------------------------------------------------------------

## Quick Power Moves ⚡ {#quick-power-moves-}

  Command                        What it does
  ------------------------------ -------------------------
  `kubectl get all`              Everything in namespace
  `kubectl delete all --all`     Delete everything
  `kubectl api-resources`        List resource types
  `kubectl explain <resource>`   Docs inside terminal

------------------------------------------------------------------------

## Pro Tips (Real Dev Stuff)

-   Use aliases:

    ``` {.language-bash .bash info="bash" data-role="codeBlock"}
    alias k=kubectl
    ```

-   Auto-completion:

    ``` {.language-bash .bash info="bash" data-role="codeBlock"}
    source <(kubectl completion bash)
    ```

-   Dry run before applying:

    ``` {.language-bash .bash info="bash" data-role="codeBlock"}
    kubectl apply -f file.yaml --dry-run=client
    ```
:::

