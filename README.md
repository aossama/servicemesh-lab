# Istio Lab

As more organizations adopt service mesh technology, it's becoming crucial to adopt a multi- service mesh topology within OCP clusters. This repository contains various use cases for securing istio with simulation to a practical approach for demonstrating multi Service Mesh capabilities within OpenShift 4 clusters.

## Scenario

The scenarios in this reposiroty assumes a soft multi-tenant environment where two different business units, (north) and (south) are develping their applications. Each business unit delivers it's applications on a separata service mesh data plane (north-mesh) and (south-mesh).

## Before you begin

* This scenario was developed in an OpenShift environment deployed on AWS.
* Knowledge with OpenShift Service Mesh is assumed.
