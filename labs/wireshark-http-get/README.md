# Wireshark Lab - HTTP GET Request Analysis

## Overview

This lab demonstrates how to capture and analyze **HTTP traffic** using Wireshark.  
The goal is to observe how a browser communicates with a web server and inspect an **HTTP GET request**, its headers, and the **User-Agent** field.

This exercise helps understand how application-layer protocols appear on the network and why unencrypted HTTP traffic can be inspected by network observers.

---

## Lab Objectives

During this lab we:

- Captured live network traffic using **Wireshark**
- Generated HTTP traffic by visiting a website
- Identified an **HTTP GET request**
- Inspected **HTTP headers**
- Located the **User-Agent string**
- Analyzed the packet across multiple network layers

---

## Lab Environment

| Component | Value |
|----------|------|
| Tool | Wireshark |
| Browser | Firefox |
| Target Website | http://neverssl.com |
| Protocol | HTTP |
| Destination Port | 80 |

Example connection:

| Field | Value |
|------|------|
| Client IP | 192.168.50.186 |
| Server IP | 34.223.124.45 |

---

## Steps Performed

1. Started packet capture in **Wireshark**
2. Visited `http://neverssl.com` to generate HTTP traffic
3. Applied the display filter:
