# Day 2: The Microscope (Packet Inspection)

**Objective:** Prove that you look like Chrome by inspecting the network packets.
**Tool:** `tshark` (Wireshark CLI).

---

## 1. Why Wireshark?
Websites like `tls.peet.ws` are great, but sometimes you need to debug a local script or a specific target that doesn't echo your fingerprint back.
You need to see what is leaving your network card.

## 2. Installing TShark
*   **Windows:** Download Wireshark installer. Check the box "Install TShark". Add to PATH.
*   **Linux/Mac:** `apt install tshark` / `brew install tshark`.

## 3. The Command
We want to capture traffic on port 443 (HTTPS) and look for the "Client Hello" handshake.

```bash
tshark -i "Wi-Fi" -Y "ssl.handshake.type == 1" -T fields -e ssl.handshake.ciphersuit -e ssl.handshake.extensions_server_name
```
*   `-i "Wi-Fi"`: Listen on your interface (Check `ipconfig` or `ifconfig` for name).
*   `-Y "ssl..."`: Filter only Client Hello packets.
*   `-T fields`: Output clean text, not binary dump.

## 4. The Exercise
1.  Open Terminal. Run the TShark command.
2.  Run your `requests` script: `python my_script.py`.
3.  Watch TShark output. You will see a short list of ciphers (e.g., `c02b,c02f...`).
4.  Now open Chrome and visit a site.
5.  Watch TShark output. You will see a HUGE list of ciphers and extensions.

## 5. Analyzing the Difference
*   **GREASE:** Chrome sends random values like `0x0a0a` to test server resilience. Python does not.
*   **ALPN:** Chrome sends `h2,http/1.1`. Python might only send `http/1.1`.
*   **Extensions:** Chrome sends `session_ticket`, `extended_master_secret`, `renegotiation_info`...

This "Wire-Level" visibility is how you debug why a sophisticated anti-bot is blocking you. If your packet looks different, you lose.

## 6. Homework
1.  Get TShark running.
2.  Capture a Request from `curl` (CLI).
3.  Capture a Request from `Chrome`.
4.  Note 3 major differences in the handshake.
