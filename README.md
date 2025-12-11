# Vulnerabilities in a Chinese White-Label Smart Camera
This repository documents security vulnerabilities discovered in a Chinese white-label (OEM/ODM) smart camera, sold under the brand “Zhuji,” and built on the Chenyun IoT platform. 

## Device information
- **Brand:** zhuji(white-label OEM)
- **Control APP:**: X-IoT
- **Firmware version:** HQLS_IOT66DP241129-10001
- **Platform:** Chenyun OTA  
- **Purchase link:** https://www.aliexpress.com/item/1005008580317191.html

## Summary of Vulnerabilities
- **Default/weak password:**
Hard-coded password allows unauthorized LAN access and full device control. This password appears in the instruction and app.
- **OTA Information Disclosure:**
When the mobile application performs an OTA update check, the request is transmitted over HTTP in plaintext. The OTA server responds with internal backend URLs and the device ID, exposing sensitive information due to lack of authentication and encryption.
- **Weak OTA Authentication Controls:**
The OTA server does not enforce authentication or rate limiting, allowing unlimited unauthenticated queries and exposing the system to enumeration and probing attacks.
- **Sensitive Information Exposure:**
The device ID functions as a critical authentication identifier for LAN control of the camera. This identifier is exposed in multiple ways, including plaintext transmission during OTA update requests and broadcast as the Wi-Fi SSID during AP pairing mode. Because the device ID is required for authentication and can be intercepted or observed easily, its exposure enables attackers to escalate to unauthorized device access and takeover.

## Attack Reproduction
### 1. Review the product instruction.
![ccc51cd4c2720f0d32f913d89b1904a](https://github.com/user-attachments/assets/152a10c3-da73-4a92-83e4-6d8cddca767c)
Observe that the device uses a fixed default password and the device ID as the login identifier. This indicates the presence of a default/weak credential authentication mechanism.

### 2. OTA Traffic Inspection via Packet Capture
1. Trigger a firmware update check through the official mobile application.
2. Capture network traffic using Wireshark.
<img width="2431" height="919" alt="image" src="https://github.com/user-attachments/assets/bf3af1af-ced1-4239-90b6-5fcabfb6ea6f" />
3. Observe that the OTA communication is transmitted over unencrypted HTTP.
4. The OTA response includes sensitive metadata such as the OTA backend server address and the device ID in plaintext.
<img width="1057" height="727" alt="5cee78733317e6f76225134fcb14ca5" src="https://github.com/user-attachments/assets/0f0018e1-b144-422e-8ed6-99af39f315e3" />

### 3. Analysis of OTA Backend Login Behavior
1. Locate the OTA backend login page associated with the device’s update infrastructure.
<img width="2879" height="1461" alt="0b6fc660afd089db599faec13736cd9" src="https://github.com/user-attachments/assets/3404d3d8-0aad-4213-b215-315b7579c71b" />
2. Perform repeated login attempts to evaluate authentication behavior (800 times).
3. The interface accepts unlimited failed attempts without rate limiting, lockout, or CAPTCHA.
4. This demonstrates a lack of brute-force protections in the OTA management interface.

### 4. Examination of Device AP Mode Behavior
1. Power on the device in AP (Access Point) provisioning mode.
2. Scan nearby Wi-Fi networks.
3. The device’s hotspot SSID reveals the device ID directly, exposing a critical identifier used for LAN authentication.
<img width="713" height="801" alt="15fee273a65874bc91f1a703835f333" src="https://github.com/user-attachments/assets/277f3dc6-2751-4f2a-909f-fc59488366ec" />
