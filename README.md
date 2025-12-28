# Decentralized Secure File Transfer System (P2P)

[![repo stars](https://img.shields.io/github/stars/harish795r/Decentralized-Secure-File-Transfer-System-Using-P2P-Architecture?style=social)](https://github.com/harish795r/Decentralized-Secure-File-Transfer-System-Using-P2P-Architecture/stargazers) [![python](https://img.shields.io/badge/python-3.8%2B-blue.svg)](https://www.python.org/) [![license](https://img.shields.io/badge/license-See%20repo-lightgrey)](https://github.com/harish795r/Decentralized-Secure-File-Transfer-System-Using-P2P-Architecture)

## Table of Contents
- [Project Description](#project-description)
- [Key Features](#key-features)
- [Requirements / Prerequisites](#requirements--prerequisites)
- [Architecture & Design Overview](#architecture--design-overview)

## Project Description
This repository implements a decentralized peer-to-peer (P2P) file transfer application in Python. It enables secure, encrypted, serverless file sharing between peers on the same LAN. The project provides: a dark-themed Tkinter GUI for user interaction, UDP-based peer discovery, direct TCP file streaming, optional password-based encryption, and real-time transfer progress and logging.

The application enforces a simple startup license verification (license.json / license.sig / rootCA.pem) before launching.

## Key Features
- LAN peer discovery using UDP broadcast (PeerDiscovery)
- Direct peer-to-peer file transfers over TCP (Sender / Receiver)
- Optional AES-256 encryption with PBKDF2-HMAC-SHA256 key derivation
- Real-time progress bar, transfer speed calculation, and pause/resume control
- Dark-themed Tkinter GUI with transfer logs (Logger)
- Resilient port handling: attempts alternate ports when requested ports are busy
- Startup license verification using RSA signature (startup_auth.py)

## Requirements / Prerequisites
- Supported OS: Linux, macOS, Windows (Tkinter required)
- Python: 3.8+ recommended
- Python packages:
  - cryptography
- Standard library modules used: socket, threading, tkinter, base64, os, time, argparse, sys

Network ports used (defaults):
- UDP discovery: 60000 (DISCOVERY_PORT)
- Request port: 60001 (REQUEST_PORT)
- Sender TCP port: starts at 5000 (Sender will try requested port and increment if busy)

Install dependencies (example):

pip install cryptography

Notes:
- Tkinter is usually bundled with Python on most platforms. If missing, install it via your OS package manager.
- There is no explicit requirements.txt in the repository; adding one is recommended for reproducible installs.

## Architecture & Design Overview
This section describes the core components and how they interact.

Core components (files -> responsibilities):
- main.py
  - Top-level GUI application. Initializes Logger, PeerDiscovery, Sender, and Receiver; manages UI screens and user actions.
- discovery.py (PeerDiscovery)
  - UDP-based discovery: broadcasts presence and listens for peers. Maintains a thread-safe peers map.
- sender.py (Sender)
  - TCP server that streams file data in CHUNK_SIZE (4096 bytes) blocks. Supports optional encryption: derives an AES key via PBKDF2 (utils.derive_key), sends header containing filename/filesize and base64-encoded salt/IV for encrypted transfers. Implements pause/resume, speed calculation, and robust port binding (tries alternative ports if busy).
- receiver.py (Receiver)
  - Connects to Sender to fetch header and stream the file to disk. Parses headers to determine encryption metadata and decrypts on-the-fly when needed. Updates progress UI and speed.
- utils.py
  - Utility helpers: local_ip() detection and derive_key(password, salt) implementation using PBKDF2-HMAC-SHA256.
- logger.py
  - Thread-safe logger that enqueues messages and updates a Tkinter Text widget.
- startup_auth.py
  - Verifies license.json using license.sig and rootCA.pem (RSA/SHA256). The app exits if verification fails or files are missing.

Data & network flow (high level):
1. Discovery: peers broadcast presence via UDP. Other peers add discovered IPs to their peers list.
2. Header exchange: Receiver connects to Sender's TCP port and reads a short header (filename|filesize or filename|filesize|salt_b64|iv_b64).
3. File transfer: Sender streams (encrypted) bytes in chunks over TCP; Receiver writes to disk (and decrypts if necessary).
4. UI: main.py provides controls for sending/receiving, progress bars, speed readouts, pause/resume, and logging.

Security considerations (brief):
- Encryption uses PBKDF2-HMAC-SHA256 (100,000 iterations) to derive a 32-byte AES key from a password + random salt. IV and salt are transmitted in the header when encryption is enabled.
- The startup license verification provides a basic application-level attestation but is not a substitute for per-peer authentication.
- Discovery uses UDP broadcasts and is intended for LAN use only.

---

This README was generated with the GitHub Copilot Chat Assistant.