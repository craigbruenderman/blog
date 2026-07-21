---
title: "Arista AVD - Part 1"
date: "2026-06-03"
draft: true
series: ["Arista AVD"]
series_order: 1
categories: ["Networking", "Automation"]
tags: ["Arista"]
---

## Overview

Arista AVD is an infrastructure as code tool developed by Arista to perform automated configuration of EOS devices, based on validated Arista network designs. It works by defining a set of parameters via YAML files, which are then parsed by an Ansible playbook to generate the intended config to be placed on EOS devices. Another playbook can then be invoked to push those intended configs via eAPI running on each target EOS device.

AVD seems very much to be a work in progress. Given that, and some of the typical issues with Ansible and Python dependencies, it can be tricky to understand how the AVD environment fits together and to get it working. In my case, ThreatLocker on top made it particularly miserable.

## Installation Options

There are multiple options to get a usable AVD environment:

* Run it locally on your computers native OS
* Run it locally in a container
* Ruyn it locally with VScode DevContainer
* Run it via Github CodeDeploy

## Components

### Python

Python is a dependency, and the particular verison of Python will determine which version of ansible-core to be used, according to [this matrix](https://docs.ansible.com/projects/ansible/latest/reference_appendices/release_and_maintenance.html#ansible-core-support-matrix). For instance, I'm working with Python 3.14.5, so I'll use ansible-core 2.21.

### ansible-core

### arista.avd Ansible Galaxy collection

```
pip install "pyavd[ansible]"
ansible-galaxy collection install arista.avd -p [collections dir]
```
