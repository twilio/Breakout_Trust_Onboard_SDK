# Contents
1. [Overview](#overview)
1. [Example reference implementation](#Example-reference-implementation)
1. [Trust Onboard Available Certificate Extraction](#Trust-Onboard-Available-Certificate-Extraction)

[Validating Requests on the Azure IoT cloud](cloud-support/azure-iot/README.md)

# Overview

The Breakout Trust Onboard SDK offers tools and examples on how to utilize the `available` X.509 certificate available on Twilio Wireless Trust Onboard (ToB) enabled SIM cards.

We also document a reference implementation hardware and software setup to show just one of many ways to leverage Trust Onboard with your IoT project.

Twilio Trust Onboard depends on hardware and software working together in three parts:

- The SIM has two X.509 certificates provisioned to it as well as SIM applets which provide access to and services which utilize the certificates.
- A compatible cellular module provides AT commands capable of communicating with the SIM applets and reporting the results back to the calling software.
- A library on the host platform or micro controller (referred to simply as `host` in this document) that is compatible with the cellular module's AT command set and the applets loaded on the SIM.

Twilio Trust Onboard provisions two X.509 certificates on ToB SIM cards.  An `available` certificate and a `signing` certificate.  The `available` certificate has both the X.509 public certificate and its associated private key available for export and use by your applications.  The `signing` certificate only allows the X.509 public certificate to be exported.  Cryptographic signing using this `signing` certificate only ever occurs within the SIM applet itself for security.  While the `available` certificate can be used for just about any client authentication you may wish to do that uses X.509 certificates, the `signing` certificate is primarily useful when access to the ToB SIM is built into an existing security framework, such as embedtls, openssl or wolfssl.  Plugins supporting `signing` use-cases are planned but not available today.

A typical Twilio Trust Onboard flow for utilizing the `available` certificate is as follows:

- The cellular module/development board is initialized, providing one or more serial interfaces to the host.
- A library communicates with the SIMs applets using the SIMs PIN and the path of the certificate and keys to access.  This occurs using the cellular module's serial interface and supported AT commands for accessing the SIM directly.
- The library provides the PEM encoded certificate chain and (in the case of the `available` certificate) the DER encoded private key.
- Your applications make use of the public certificate and private key for authentication with your chosen backend services.

# Example Reference Implementation

We have found multiple cellular modules to be compatible with the AT commands and access to the SIM needed to work with Twilio ToB SIMs, but the following is a combination which we have researched and built a tutorial around: [Twilio Wireless Broadband IoT Developer Kit](https://github.com/twilio/Wireless_Broadband_IoT_Dev_Kit)

# Trust Onboard Available Certificate Extraction

In the project you build above, you should now have an executable `bin/trust_onboard_tool`.  We can use this tool to extract the Trust Onboard `available` X.509 certificate and private key.  Since the output of this CLI is two files, we recommend writing the files out to a sub-directory.  **Important:** If you are provided a different default PIN or have changed it, be sure to replace the PIN `0000`, or you may block your SIM card.  The password `mypassword` below is the passphrase that will be used to secure the P12 certificate and key bundle the `convert_certs.sh` script generates.  This assumes you are using our pre-built image or ran `make install` for the Breakout_Trust_Onboard_SDK.

    mkdir temp
    trust_onboard_tool /dev/ttyACM1 0000 temp/certificate-chain.pem temp/key.pem

The certificate and private key can now be used in your applications.  We also offer direct access to the Trust Onboard SDK to access the certificate and private key in your code without an intermediate file.  See the [BreakoutTrustOnboardSDK.h](include/BreakoutTrustOnboardSDK.h) header for more details.
