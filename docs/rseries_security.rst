====================================
Securing / Hardening F5OS on rSeries
====================================

F5OS tenants follow the standard hardening/security best practices that are outlined in the following solution article:

`K53108777: Hardening your F5 system <https://support.f5.com/csp/article/K53108777>`_

This section will focus on how to harden/secure the F5OS layer of the rSeries appliances. 

F5OS Platform Layer Isolation
=============================

Management of the new F5OS platform layer is completely isolated from in-band traffic, networking, and VLANs. It is purposely isolated so that it is only accessible via the out-of-band management network. In fact, there are no in-band IP addresses assigned to the F5OS layer, only tenants will have in-band management IP addresses and access. Tenants also have out-of-band connectivity so they can be managed via the out-of-band network.

This allows customers to run a secure/locked-down out-of-band management network where access is tightly restricted. The diagram below shows the out-of-band management access entering the rSeries appliance through **MGMT** port. The external MGMT port is bridged to an internal out-of-band network that connects to all tenants within the rSeries appliance. 

.. image:: images/rseries_security/image1.png
  :align: center

Allow List for F5OS Management
===============================

F5OS only allows management access via a single out-of-band management interface. Access to that single management IP address may be restricted to specific IP addresses (both IPv4 and IPv6), subnets (via Prefix Length), as well as protocols - 443 (HTTPS), 80 (HTTP), 8888 (RESTCONF), 161 (SNMP), 7001 (VCONSOLE), and 22 (SSH). An administrator can add one or more Allow List entries via the CLI, webUI or API to lock down access to specific endpoints.

By default, all ports except for 161 (SNMP) are enabled for access, meaning ports 80, 443, 8888, 7001, and 22 are allowed access. Port 80 is only open to allow a redirect to port 443 in case someone tries to access the webUI over port 80. The webUI itself is not accessible over port 80. Port 161 is typically viewed as un-secure, and is therefore not accessible until an allow list entry is created for the endpoint trying to access F5OS using SNMP queries. Ideally SNMPv3 should be utilized to provide additional layers of security on an otherwise un-secure protocol. VCONSOLE access also has to be explicitly configured before access to the tenants is possible over port 7001. 

To further lock down access you may add an Allow List entry including an IP address and optional prefix for each of the protocols listed above. As an example, if you wanted to restrict API and webUI access to a particular IP address and/or subnet, you can add an Allow List entry for the desired IP or subnet (using the prefix length), specify port 443 and all access from other IP endpoints will be prevented.


Adding Allow List Entries via CLI
-----------------------------------

If you would like to lock down one of the protocols to either a single IP address or subnet, use the **system allowed-ips** command. Be sure to commit any changes. The **prefix-length** parameter is optional. If you omit it, then you will lock down access to a specific IP endpoint, if you add it you can lock down access to a specific subnet.

.. code-block:: bash

    r10900-2(config)# system allowed-ips allowed-ip snmp config ipv4 address 10.255.0.0 prefix-length 24 port 161
    r10900-2(config-allowed-ip-snmp)# commit
    Commit complete.

Currently you can add one ip address/port pair per **allowed-ip** name with an optional prefix length to specify a CIDR block containing multiple addresses. If you require more than one non-contiguous IP address or subnets you can add it under another name as seen below. 

.. code-block:: bash

    appliance-1(config)# system allowed-ips allowed-ip SNMP-144 config ipv4 address 10.255.0.144 port 161 
    appliance-1(config-allowed-ip-SNMP)# commit
    Commit complete.
    appliance-1(config-allowed-ip-SNMP)# 


    appliance-1(config)# system allowed-ips allowed-ip SNMP-145 config ipv4 address 10.255.2.145 port 161 
    appliance-1(config-allowed-ip-SNMP)# commit
    Commit complete.
    appliance-1(config-allowed-ip-SNMP)# 


Adding Allow List Entries via API
-----------------------------------

Below is an example of allowing multiple SNMP endpoints (port 161) to query SNMP on the F5OS platform layer.

.. code-block:: bash

    POST https://{{rseries_appliance1_ip}}:8888/restconf/data/openconfig-system:system/f5-allowed-ips:allowed-ips

Within the body of the API call, specific IP address/port combinations can be added under a given name. In the current release, you are limited to one IP address/port per name. 

.. code-block:: json

    {
        "allowed-ip": [
            {
                "name": "SNMP-142",
                "config": {
                    "ipv4": {
                        "address": "10.255.0.142",
                        "port": 161
                    }
                }
            },
            {
                "name": "SNMP-143",
                "config": {
                    "ipv4": {
                        "address": "10.255.0.143",
                        "port": 161
                    }
                }
            },
            {
                "name": "SNMP-144",
                "config": {
                    "ipv4": {
                        "address": "10.255.0.144",
                        "port": 161
                    }
                }
            }
        ]
    }



To view the allowed IP's in the API, use the following call.

.. code-block:: bash

    GET https://{{rseries_appliance1_ip}}:8888/restconf/data/openconfig-system:system/f5-allowed-ips:allowed-ips

The output will show the previously configured allowed-ip's.


.. code-block:: json

    {
        "f5-allowed-ips:allowed-ips": {
            "allowed-ip": [
                {
                    "name": "SNMP-142",
                    "config": {
                        "ipv4": {
                            "address": "10.255.0.142",
                            "port": 161
                        }
                    }
                },
                {
                    "name": "SNMP-143",
                    "config": {
                        "ipv4": {
                            "address": "10.255.0.143",
                            "port": 161
                        }
                    }
                },
                {
                    "name": "SNMP-144",
                    "config": {
                        "ipv4": {
                            "address": "10.255.0.144",
                            "port": 161
                        }
                    }
                }
            ]
        }
    }

Adding Allow List Entries via webUI
-----------------------------------

You can configure the **Allow List** in the webUI under the **System Settings** section. 

.. image:: images/rseries_security/image2.png
  :align: center
  :scale: 70%

Below is an example of allowing any SNMP endpoint at 10.255.0.0 (prefix length of 24) to query the F5OS layer on port 161.

.. image:: images/rseries_security/image3.png
  :align: center
  :scale: 70%

Setting F5OS Primary Key
======================== 

The F5 rSeries system uses a primary key to perform encryption and decryption of highly sensitive passwords/passphrases in the configuration database. You should periodically reset this primary key for additional security. You should set this primary key prior to performing any configuration backup if you have not already done so. In the case of a configuration migration such as moving configuration to a replacement device due to RMA, it is important to set the primary key to a known value so that the same key can be used to decrypt the passwords/passphrases in the configuration restored on the replacement device. More details are provided in the solution article below.

`K47512994: Back up and restore the F5OS-A configuration on an rSeries system <https://my.f5.com/manage/s/article/K47512994>`_

To set the primary-key issue the following command in config mode.

.. code-block:: bash

    system aaa primary-key set passphrase <passphrase string> confirm-passphrase <passphrase string> salt <salt string> confirm-salt <salt string>

Note that the hash key can be used to check and compare the status of the primary-key on both the source and the replacement devices if restoring to a different device. To view the current primary-key hash, issue the following CLI command.

.. code-block:: bash

    r10900-1# show system aaa primary-key 
    system aaa primary-key state hash gK/F47uQfi7JWYFirStCVhIaGcuoctpbGpx63MNy/korwigBW6piKx9TldiRazHmE8Y+qylGY4MOcs9IZ+KG4Q==
    system aaa primary-key state status NONE
    r10900-1# 


Certificates for Device Management
==================================

F5OS supports TLS device certificates and keys to secure connections to the management interface. You can either create a self-signed certificate, or load your own certificates and keys into the system. In F5OS-A 1.4.0 an admin can now optionally enter a passphrase with the encrypted private key. More details can be found in the link below.

`rSeries Certificate Management Overview <https://techdocs.f5.com/en-us/f5os-a-1-3-0/f5-rseries-systems-administration-configuration/title-system-settings.html#cert-mgmt-overview>`_


Managing Device Certificates, Keys, CSRs, and CAs via CLI
--------------------------------------------------------

By default, F5OS uses a self-signed certificate and key for device management. If you would like to create your own private key and self-signed certificate use the following CLI command:

.. code-block:: bash

    r10900-1(config)# system aaa tls create-self-signed-cert name jim email jim@f5.com city Boston region MA country US organization F5 unit Sales version 1 days-valid 365 key-type encrypted-ecdsa curve-name secp384r1 store-tls true key-passphrase 
    Value for 'key-passphrase' (<string, min: 6 chars, max: 255 chars>): **************
    Value for 'confirm-key-passphrase' (<string, min: 6 chars, max: 255 chars>): **************
    r10900-1(config)#


The **store-tls** option when set to **true**, stores the private key and self-signed certificate in system/aaa/tls/config/key and system/aaa/tls/config/certificate instead of returning the vlaues only in the CLI output. If you would prefer to have the keys returned in the CLI output and not stored in the system, then set **store-tls false** as seen below.

.. code-block:: bash

    r10900-1(config)# system aaa tls create-self-signed-cert name jim email jim@f5.com city Boston region MA country US organization F5 unit Sales version 1 days-valid 365 key-type encrypted-ecdsa curve-name secp384r1 store-tls false key-passphrase 
    Value for 'key-passphrase' (<string, min: 6 chars, max: 255 chars>): **************
    Value for 'confirm-key-passphrase' (<string, min: 6 chars, max: 255 chars>): **************
    key-response 
    -----BEGIN EC PRIVATE KEY-----
    Proc-Type: 4,ENCRYPTED
    DEK-Info: AES-256-CBC,BA7ECF55A14EBD39F5DB48EBB6BBB53E

    IF6Uk2tLE6LzIu3mEgy3VB/uADkN53HO4LE7P8QDTLBRt5f81LjxhP5MFJlKFk2a
    iYpZEqzhZwCAfOetcaK+LFv+z26NzUSdHLmEvM+qG3B5s6U7eQbes6mMPAyOFZcj
    +1El1olDrHfn+xmcbUFlM7lUVRgIhABy+Y3WT6GaH7CaYghDjKkRoppiiQs3KwXf
    /ZdO7QFRAWr0Lfi8iBtVZKBqL2CHsBQxfggvP0EB+9o=
    -----END EC PRIVATE KEY-----

    cert-response 
    -----BEGIN CERTIFICATE-----
    MIICDjCCAZUCCQCRNihj9kub1zAKBggqhkjOPQQDAjBxMQwwCgYDVQQDDANqaW0x
    CzAJBgNVBAYTAlVTMQswCQYDVQQIDAJNQTEPMA0GA1UEBwwGQm9zdG9uMQswCQYD
    VQQKDAJGNTEOMAwGA1UECwwFU2FsZXMxGTAXBgkqhkiG9w0BCQEWCmppbUBmNS5j
    b20wHhcNMjMwMjIzMDUwMDE0WhcNMjQwMjIzMDUwMDE0WjBxMQwwCgYDVQQDDANq
    aW0xCzAJBgNVBAYTAlVTMQswCQYDVQQIDAJNQTEPMA0GA1UEBwwGQm9zdG9uMQsw
    CQYDVQQKDAJGNTEOMAwGA1UECwwFU2FsZXMxGTAXBgkqhkiG9w0BCQEWCmppbUBm
    NS5jb20wdjAQBgcqhkjOPQIBBgUrgQQAIgNiAATDLVWBq7s1nwkZy27DGbqNEkHM
    /WTXwKo2i+uzoB2fL6DXGlgKJo1WIY5sFMYGv1lNsDte5Ztr11331rmcWghVOHkr
    FndFmeEnSNRyHZoqHXzVIkp60JAsv2Yv2ZafGJEwCgYIKoZIzj0EAwIDZwAwZAIw
    EluMBf0X9Zotm6pWMiajR5AL8Z2PMIE3hqpc3IREeSs09xf8ADKoCEEudRMHB1lc
    AjBelhJIkUoiZBtfAdf6NrUDWQdrN7kvC4h8DLm1XV9lr4Wxh5Es1WSwF1PoTRMt
    Mqs=
    -----END CERTIFICATE-----
    r10900-1(config)# 

The management interface will now use the self-signed certifcate you just created. You can verify by connecting to the F5OS management interface via a browser and then examining the certificate.

.. image:: images/rseries_security/imagecert.png
  :align: center
  :scale: 70%


To create a Certificate Signing Request (CSR) via the CLI use the **system aaa tls create-csr** command.

.. code-block:: bash

    r10900-1(config)# system aaa tls create-csr name r10900-1.f5demo.net email jim@f5.com city Boston country US organization F5 region MA unit Sales version 1 
    response 
    -----BEGIN CERTIFICATE REQUEST-----
    MIIBezCCAQECAQEwgYExHDAaBgNVBAMME3IxMDkwMC0xLmY1ZGVtby5uZXQxCzAJ
    BgNVBAYTAlVTMQswCQYDVQQIDAJNQTEPMA0GA1UEBwwGQm9zdG9uMQswCQYDVQQK
    DAJGNTEOMAwGA1UECwwFU2FsZXMxGTAXBgkqhkiG9w0BCQEWCmppbUBmNS5jb20w
    djAQBgcqhkjOPQIBBgUrgQQAIgNiAAQ/8UzZtEGMJ+vtmkEUsgiv2hL8r81sKwB3
    clwqnXKl08vFCNr4wy7TB28b4EszAQDTBhIipHuC5L2GpetjNsFywkDqZuoJAvmx
    nrqYQe5z9bDUpO6AJsAaohLG0sc9E4WgADAKBggqhkjOPQQDAgNoADBlAjEAsTST
    M43RDyve46QJtHf3ofCVuhmxZ8lAcWBX5W3JsDiZcdaNCeXgSk4pX5nwSrDnAjAH
    GPjWc5CcyCBh8+RyV9zNL7I5WlIsZj1aUAA3PD1CSgFHxaXV6cpHP8H8kQiJjjE=
    -----END CERTIFICATE REQUEST-----
    r10900-1(config)# 

To create a CA bundle via the CLI use the **system aaa tls ca-bundle** command.

.. code-block:: bash

    r10900-1(config)# system aaa tls ca-bundles ca-bundle ?
    Possible completions:
    <Reference to configured name of the CA Bundle.>
    r10900-1(config)# system aaa tls ca-bundles ca-bundle    


To create a Client Revocation List (CRL) via the CLI issue the following command.

.. code-block:: bash

    r10900-1(config)# system aaa tls crls crl ?
    Possible completions:
    <Reference to configured name of the CRL.>
    r10900-1(config)# system aaa tls crls crl

You can display the current certificate, keys, and passpharases using the CLI command **show system aaa tls**.

.. code-block:: bash

    r10900-1# show system aaa tls
    system aaa tls state certificate Certificate:
                                        Data:
                                            Version: 1 (0x0)
                                            Serial Number:
                                                c9:79:f0:b2:3e:9e:d2:a1
                                        Signature Algorithm: ecdsa-with-SHA256
                                            Issuer: CN=jim2, C=US, ST=MA, L=Boston, O=F5, OU=Sales/emailAddress=jim@f5.com
                                            Validity
                                                Not Before: Feb 24 21:35:31 2023 GMT
                                                Not After : Feb 24 21:35:31 2024 GMT
                                            Subject: CN=jim2, C=US, ST=MA, L=Boston, O=F5, OU=Sales/emailAddress=jim@f5.com
                                            Subject Public Key Info:
                                                Public Key Algorithm: id-ecPublicKey
                                                    Public-Key: (384 bit)
                                                    pub: 
                                                        04:3f:f1:4c:d9:b4:41:8c:27:eb:ed:9a:41:14:b2:
                                                        08:af:da:12:fc:af:cd:6c:2b:00:77:72:5c:2a:9d:
                                                        72:a5:d3:cb:c5:08:da:f8:c3:2e:d3:07:6f:1b:e0:
                                                        4b:33:01:00:d3:06:12:22:a4:7b:82:e4:bd:86:a5:
                                                        eb:63:36:c1:72:c2:40:ea:66:ea:09:02:f9:b1:9e:
                                                        ba:98:41:ee:73:f5:b0:d4:a4:ee:80:26:c0:1a:a2:
                                                        12:c6:d2:c7:3d:13:85
                                                    ASN1 OID: secp384r1
                                                    NIST CURVE: P-384
                                        Signature Algorithm: ecdsa-with-SHA256
                                            30:66:02:31:00:ad:83:1c:be:06:49:b7:16:36:57:aa:20:f5:
                                            73:b6:59:2a:48:01:cd:18:3f:8a:65:87:4c:02:17:14:32:47:
                                            02:db:c6:c7:28:48:ac:6c:9a:fc:e2:88:40:71:1c:31:45:02:
                                            31:00:b3:06:dc:eb:60:42:df:d7:a6:b2:21:aa:ad:15:e9:70:
                                            1f:76:d6:1d:2d:25:5a:d0:0f:53:ab:1c:1a:3c:ce:e3:9a:6d:
                                            c4:e0:1f:38:58:d0:b3:dc:94:6a:02:47:a8:d0
                                    
    system aaa tls state verify-client false
    system aaa tls state verify-client-depth 1
    r10900-1# 


Managing Device Certificates, Keys, CSRs, and CAs via webUI
-----------------------------------------------------------

In the F5OS webUI you can manage device certificates for the management interface via the **System Settings -> Certificate Management** page. There are options to view the TLS certificates, keys, and details. You may also create self-signed certificates, create certificate signing requests (CSRs), and CA bundles.

.. image:: images/rseries_security/imagecert2.png
  :align: center
  :scale: 70%

The screen below shows the options when creating a self signed certificate. 

.. image:: images/rseries_security/imagecert3.png
  :align: center
  :scale: 70%

If you choose the **Store TLS** option of **False** then the certifcate details will be displayed, and you will be given the option to copy them to the clipboard. If you want to store them on the system, then set the **Store TLS** option to **True**.

.. image:: images/rseries_security/imagecert4.png
  :align: center
  :scale: 70%

You can then use the **Show** options to display the current certificate, key, and details. Paste the text into the respective text boxes to add a certificate. TLS Key Passphrase is only required if TLS Key is in encrypted format. 

.. image:: images/rseries_security/imagecert5.png
  :align: center
  :scale: 70%

.. image:: images/rseries_security/imagecert6.png
  :align: center
  :scale: 70%

You can also create a Certificate Signing Request (CSR) for the self-signed certificate for use when submiting the certificate to the Certificate Authourity (CA).

.. image:: images/rseries_security/imagecsr1.png
  :align: center
  :scale: 70%

After clicking **Save** the CSR will appear, and you will be able to **Copy to Clipboard** so you can submit the singning request.

.. image:: images/rseries_security/imagecsr2.png
  :align: center
  :scale: 70%

When you install an SSL certificate on the system, you also install a certificate authority (CA) bundle, which is a file that contains root and intermediate certificates. The combination of these two files complete the SSL chain of trust.

.. image:: images/rseries_security/imageca1.png
  :align: center
  :scale: 70%

Managing Device Certificates, Keys, CSRs, and CAs via API
-------------------------------------

You can view the current certificates, keys and passphrases via the API using the following API call.

.. code-block:: bash

    GET https://{{rseries_appliance1_ip}}:8888/restconf/data/openconfig-system:system/aaa/f5-openconfig-aaa-tls:tls

In the response you will notice the certificate, key, and optional passphrase as well as the state.

.. code-block:: json

    {
        "f5-openconfig-aaa-tls:tls": {
            "config": {
                "certificate": "-----BEGIN CERTIFICATE-----\nMIICEjCCAZcCCQDJefCyPp7SoTAKBggqhkjOPQQDAjByMQ0wCwYDVQQDDARqaW0y\nMQswCQYDVQQGEwJVUzELMAkGA1UECAwCTUExDzANBgNVBAcMBkJvc3RvbjELMAkG\nA1UECgwCRjUxDjAMBgNVBAsMBVNhbGVzMRkwFwYJKoZIhvcNAQkBFgpqaW1AZjUu\nY29tMB4XDTIzMDIyNDIxMzUzMVoXDTI0MDIyNDIxMzUzMVowcjENMAsGA1UEAwwE\namltMjELMAkGA1UEBhMCVVMxCzAJBgNVBAgMAk1BMQ8wDQYDVQQHDAZCb3N0b24x\nCzAJBgNVBAoMAkY1MQ4wDAYDVQQLDAVTYWxlczEZMBcGCSqGSIb3DQEJARYKamlt\nQGY1LmNvbTB2MBAGByqGSM49AgEGBSuBBAAiA2IABD/xTNm0QYwn6+2aQRSyCK/a\nEvyvzWwrAHdyXCqdcqXTy8UI2vjDLtMHbxvgSzMBANMGEiKke4LkvYal62M2wXLC\nQOpm6gkC+bGeuphB7nP1sNSk7oAmwBqiEsbSxz0ThTAKBggqhkjOPQQDAgNpADBm\nAjEArYMcvgZJtxY2V6og9XO2WSpIAc0YP4plh0wCFxQyRwLbxscoSKxsmvziiEBx\nHDFFAjEAswbc62BC39emsiGqrRXpcB921h0tJVrQD1OrHBo8zuOabcTgHzhY0LPc\nlGoCR6jQ\n-----END CERTIFICATE-----",
                "key": "$8$LzRR+5tiwtRDLQI2NFQwJ3aVjXDZw8MAmMEvqO/uM9wPHjzq5AEKf8yWMQWIsmspS8GuYWhi\n4UwWBjRnhmuViENZLm5RXjA02Lr42vzHv05skcnnFfCiRL+L8goee8wI+tbI06x4iDnsYhD2\nAAUW1mV8Kb6zAIJ1/AeobAhgY/MvJdVrRpYAY6CWpRQQiCHJbnIsvw82HXqT8fEcKfNeAvLC\nPeLPXJltU89jGlylj899cWUN+CyxTDxko6mvvRaB2MeJSZ5jwnR8bhIubr/hlG1FPlGaOIbm\nP5BYZmhVmFliwQUzlVp+36AxtGG52amLZmudmW5xskOmnhEze5NcbFp8aIF6yUa7AyKE9Rc9\n0kv4W7gNmm2+0YXaMknj1ahTSYESf5sDxN5R6knz0pFf5fF7caun7gmS5Jfqs4OIwVtDjL7J\n2j4rT7hZuwnzIWbUKGu0N9620mWFpF6S9aI2keLzhwYcad1aPMEF6PabEtQPpZMZ9kJVDROe\n5bvf+8pBvNBCtLRCX7+MpKLeFYTzMQ==",
                "passphrase": "$8$4hyAzRD/Wy3WCyocZXv6K4XeM8qDmgfX0CIHtfJYZDY=",
                "verify-client": false,
                "verify-client-depth": 1
            },
            "state": {
                "certificate": "Certificate:\n    Data:\n        Version: 1 (0x0)\n        Serial Number:\n            c9:79:f0:b2:3e:9e:d2:a1\n    Signature Algorithm: ecdsa-with-SHA256\n        Issuer: CN=jim2, C=US, ST=MA, L=Boston, O=F5, OU=Sales/emailAddress=jim@f5.com\n        Validity\n            Not Before: Feb 24 21:35:31 2023 GMT\n            Not After : Feb 24 21:35:31 2024 GMT\n        Subject: CN=jim2, C=US, ST=MA, L=Boston, O=F5, OU=Sales/emailAddress=jim@f5.com\n        Subject Public Key Info:\n            Public Key Algorithm: id-ecPublicKey\n                Public-Key: (384 bit)\n                pub: \n                    04:3f:f1:4c:d9:b4:41:8c:27:eb:ed:9a:41:14:b2:\n                    08:af:da:12:fc:af:cd:6c:2b:00:77:72:5c:2a:9d:\n                    72:a5:d3:cb:c5:08:da:f8:c3:2e:d3:07:6f:1b:e0:\n                    4b:33:01:00:d3:06:12:22:a4:7b:82:e4:bd:86:a5:\n                    eb:63:36:c1:72:c2:40:ea:66:ea:09:02:f9:b1:9e:\n                    ba:98:41:ee:73:f5:b0:d4:a4:ee:80:26:c0:1a:a2:\n                    12:c6:d2:c7:3d:13:85\n                ASN1 OID: secp384r1\n                NIST CURVE: P-384\n    Signature Algorithm: ecdsa-with-SHA256\n         30:66:02:31:00:ad:83:1c:be:06:49:b7:16:36:57:aa:20:f5:\n         73:b6:59:2a:48:01:cd:18:3f:8a:65:87:4c:02:17:14:32:47:\n         02:db:c6:c7:28:48:ac:6c:9a:fc:e2:88:40:71:1c:31:45:02:\n         31:00:b3:06:dc:eb:60:42:df:d7:a6:b2:21:aa:ad:15:e9:70:\n         1f:76:d6:1d:2d:25:5a:d0:0f:53:ab:1c:1a:3c:ce:e3:9a:6d:\n         c4:e0:1f:38:58:d0:b3:dc:94:6a:02:47:a8:d0\n",
                "verify-client": false,
                "verify-client-depth": 1
            }
        }
    }

If you would like to upload a certificate, key, and passphrase you can issue the following API PUT command.

.. code-block:: bash

    PUT https://{{rseries_appliance1_ip}}:8888/restconf/data/openconfig-system:system/aaa/f5-openconfig-aaa-tls:tls

In the body of the API call enter the following JSON syntax.

.. code-block:: json

    {
        "f5-openconfig-aaa-tls:tls": {
            "config": {
                "certificate": "-----BEGIN CERTIFICATE-----\nMIICEjCCAZcCCQDJefCyPp7SoTAKBggqhkjOPQQDAjByMQ0wCwYDVQQDDARqaW0y\nMQswCQYDVQQGEwJVUzELMAkGA1UECAwCTUExDzANBgNVBAcMBkJvc3RvbjELMAkG\nA1UECgwCRjUxDjAMBgNVBAsMBVNhbGVzMRkwFwYJKoZIhvcNAQkBFgpqaW1AZjUu\nY29tMB4XDTIzMDIyNDIxMzUzMVoXDTI0MDIyNDIxMzUzMVowcjENMAsGA1UEAwwE\namltMjELMAkGA1UEBhMCVVMxCzAJBgNVBAgMAk1BMQ8wDQYDVQQHDAZCb3N0b24x\nCzAJBgNVBAoMAkY1MQ4wDAYDVQQLDAVTYWxlczEZMBcGCSqGSIb3DQEJARYKamlt\nQGY1LmNvbTB2MBAGByqGSM49AgEGBSuBBAAiA2IABD/xTNm0QYwn6+2aQRSyCK/a\nEvyvzWwrAHdyXCqdcqXTy8UI2vjDLtMHbxvgSzMBANMGEiKke4LkvYal62M2wXLC\nQOpm6gkC+bGeuphB7nP1sNSk7oAmwBqiEsbSxz0ThTAKBggqhkjOPQQDAgNpADBm\nAjEArYMcvgZJtxY2V6og9XO2WSpIAc0YP4plh0wCFxQyRwLbxscoSKxsmvziiEBx\nHDFFAjEAswbc62BC39emsiGqrRXpcB921h0tJVrQD1OrHBo8zuOabcTgHzhY0LPc\nlGoCR6jQ\n-----END CERTIFICATE-----",
                "key": "$8$LzRR+5tiwtRDLQI2NFQwJ3aVjXDZw8MAmMEvqO/uM9wPHjzq5AEKf8yWMQWIsmspS8GuYWhi\n4UwWBjRnhmuViENZLm5RXjA02Lr42vzHv05skcnnFfCiRL+L8goee8wI+tbI06x4iDnsYhD2\nAAUW1mV8Kb6zAIJ1/AeobAhgY/MvJdVrRpYAY6CWpRQQiCHJbnIsvw82HXqT8fEcKfNeAvLC\nPeLPXJltU89jGlylj899cWUN+CyxTDxko6mvvRaB2MeJSZ5jwnR8bhIubr/hlG1FPlGaOIbm\nP5BYZmhVmFliwQUzlVp+36AxtGG52amLZmudmW5xskOmnhEze5NcbFp8aIF6yUa7AyKE9Rc9\n0kv4W7gNmm2+0YXaMknj1ahTSYESf5sDxN5R6knz0pFf5fF7caun7gmS5Jfqs4OIwVtDjL7J\n2j4rT7hZuwnzIWbUKGu0N9620mWFpF6S9aI2keLzhwYcad1aPMEF6PabEtQPpZMZ9kJVDROe\n5bvf+8pBvNBCtLRCX7+MpKLeFYTzMQ==",
                "passphrase": "$8$4hyAzRD/Wy3WCyocZXv6K4XeM8qDmgfX0CIHtfJYZDY=",
                "verify-client": false,
                "verify-client-depth": 1
            }
        }
    }


Encrypt Management TLS Private Key
=======================

Previously, F5OS allowed an admin to import a TLS certificate and key in clear text. In F5OS-A 1.4.0 an admin can now optionally enter a passphrase with the encrypted private key. This is simlar to the BIG-IP functionality defined in the link below.

`K14912: Adding and removing encryption from private SSL keys (11.x - 16.x) <https://my.f5.com/manage/s/article/K14912>`_


Appliance Mode for F5OS
=======================

If you would like to prevent root / bash level access to the F5OS layer, you can enable **Appliance Mode**, which operates in a similar manner as TMOS appliance mode. Enabling Appliance mode will disable the root account, and access to the underlying bash shell is disabled. The admin account to the F5OS CLI is still enabled. This is viewed as a more secure setting as many vulnerabilites can be avoided by not allowing access to the bash shell. In some heavily audited environments, this setting may be mandatory, but it may prevent lower level debugging from occurring directly in the bash shell. It can be disabled on a temporary basis to do advanced troubleshooting, and then re-enabled when finished.

Enabling Appliance Mode via the CLI
-----------------------------------

Appliance mode can be enabled or disabled via the CLI using the command **system appliance-mode config** and entering either **enabled** or **disabled**. The command **show system appliance-mode** will display the current status. Be sure to commit any changes. 

.. code-block:: bash

    r10900(config)# system appliance-mode config enabled 
    r10900(config)# commit
    Commit complete.
    r10900(config)# 

To display the current status.

.. code-block:: bash

    r10900(config)# do show system appliance-mode       
    system appliance-mode state enabled
    r10900(config)# 

If you then try to login as root, you will get a permission denied error. You can still login as admin to gain access to the F5OS CLI.

To disable appliance mode.

.. code-block:: bash

    r10900(config)# system appliance-mode config disabled 
    r10900(config)# commit
    Commit complete.
    r10900(config)#

Enabling Appliance Mode via the webUI
------------------------------------- 

Appliance mode can be enabled or disabled via the webUI under the **System Settings -> General** page.

.. image:: images/rseries_security/image4.png
  :align: center
  :scale: 70%


Enabling Appliance Mode via the API
-----------------------------------

Appliance mode can be enabled or disabled via the API. To view the current status of appliance mode use the following API call.

.. code-block:: bash

    GET https://{{rseries_appliance1_ip}}:8888/restconf/data/openconfig-system:system/f5-security-appliance-mode:appliance-mode


You will see output similar to the response below showing the config and state of appliance mode for F5OS.

.. code-block:: json

    {
        "f5-security-appliance-mode:appliance-mode": {
            "config": {
                "enabled": false
            },
            "state": {
                "enabled": false
            }
        }
    }

To change the mode from disabled to enabled, use the following API call.

.. code-block:: bash

    PATCH https://{{rseries_appliance1_ip}}:8888/restconf/data/openconfig-system:system/f5-security-appliance-mode:appliance-mode/f5-security-appliance-mode:config

In the body of the API call add the following:

.. code-block:: json

    {
        "f5-security-appliance-mode:config": {
            "f5-security-appliance-mode:enabled": "true"
        }
    }

Session Timeouts and Token Lifetime
===================================

Idle timeouts were configurable in previous releases, but the configuration only applied to the current session and was not persistent. F5OS-A 1.3.0 added the ability to configure persistent idle timeouts for F5OS for both the CLI and webUI. The F5OS CLI timeout is configured under system settings, and is controlled via the **idle-timeout** option. This will logout idle sessions to the F5OS CLI whether they are logged in from the console or over SSH.

In F5OS-A 1.4.0, a new **sshd-idle-timeout** option has been added that will control idle-timeouts for both root sessions to the bash shell over SSH, as well as F5OS CLI sessions over SSH. When the idle-timeout and sshd-idle-timeout are both configured, the shorter interval should take precedence. As an example, if the idle-timeout is configured for three minutes, but the sshd-idle-timeout is set to 2 minutes, then an idle connection that is connected over SSH will disconnect in two minutes, which is the shorter of the two configured options. An idle connection to the F5OS CLI over the console will disconnect in three minutes, because the sshd-idle-timeout doesn't apply to console sessions. 

There is one case that is not covered by either of the above idle-timeout settings. When connecting over the console to the bash shell as root, neither of these settings will disconnect an idle session. Only console connections to the F5OS CLI are covered via the idle-timeout setting. An enhancement has been filed, and in the future this case will be addressed. If this is a concern, then applaince mode could be enabled preventing root/bash access to the system.

For the webUI, a token based timeout is now configurable under the **system aaa** settings. A restconf-token config lifetime option has been added. Once a client to the webUI has a token they are allowed to refresh it up to five times. If the token lifetime is set to 1 minute, then a timeout won't occur until five times that value, or five minutes later. This is because the token refresh has to fail five times before disconnecting the client.  

Configuring SSH and CLI Timeouts via CLI
-----------------------------------------

To configure the F5OS CLI timeout via the CLI, use the command **system settings config idle-timeout <value-in-seconds>**. Be sure to issue a commit to save the changes. In the case below, a CLI session to the F5OS CLI should disconnect after 300 seconds of inactivity. This will apply to connections to the F5OS CLI over both console and SSH.

.. code-block:: bash

    r10900(config)# system settings config idle-timeout 300
    r10900(config)# commit
    Commit complete.     

To configure the SSH timeout via the CLI, use the command **system settings config sshd-idle-timeout <value-in-seconds>**. This idle-timeout will apply to both bash sessions over SSH, as well as F5OS CLI sessions over SSH. Be sure to issue a commit to save the changes. In the case below, the CLI session should disconnect after 300 seconds of inactivity.


.. code-block:: bash

    r10900(config)# system settings config ssh-idle-timeout 300
    r10900(config)# commit
    Commit complete.      
 
Both timeout settings can be viewed using the **show system settings** command.

.. code-block:: bash

    r10900-1# show system settings 
    system settings state idle-timeout 300
    system settings state sshd-idle-timeout 300
    system settings dag state gtp-u teid-hash disabled
    r10900-1#


 
Configuring SSH and CLI Timeouts via API
----------------------------------------

To configure the CLI or SSH timeouts via the API, use the PATCH API call below. In the case below, the CLI session should disconnect after 300 seconds of inactivity.

.. code-block:: bash

    PATCH https://{{rseries_appliance1_ip}}:8888/restconf/data/openconfig-system:system/f5-system-settings:settings

Below is the payload in the API call above to set the idle-timeout.

.. code-block:: json

    {
        "f5-system-settings:settings": {
            "f5-system-settings:config": {
                "f5-system-settings:idle-timeout": 300
            }
        }
    }

To view the current idle-timeout settings, issue the following GET API call.

.. code-block:: bash

    GET https://{{rseries_appliance1_ip}}:8888/restconf/data/openconfig-system:system/f5-system-settings:settings/config


You'll see output similar to the example below.

.. code-block:: json

    {
        "f5-system-settings:config": {
            "idle-timeout": "40",
            "sshd-idle-timeout": "20"
        }
    }


Configuring SSH and CLI Timeouts via webUI
------------------------------------------

Currently only the HTTPS token lifetime is configurable in the webUI. SSH and CLI timeouts are not currently configurable via the webUI.

.. image:: images/rseries_security/imagetoken1.png
  :align: center
  :scale: 70%

Token Lifetime via CLI
----------------------

As mentioned in the introduction, the webUI and API use token based authentication and the timeout is based on five token refreshes failing, so the value is essentially five times the configured token lifetime. Use the command **system aaa restconf-token config lifetime <value-in-minutes>** to set the token lifetime. You may configure the restconf-token lifetime via the CLI. The value is in minutes, and the client is able to refresh the token five times before it expires. As an example, if the restconf-token lifetime is set to 1 minute, an inactive webUI session will have a token expire after one minute, but it can be refreshed a maximum of five times. This will result in a webUI session or API timing out after 5 minutes.

.. code-block:: bash

    r10900(config)# system aaa restconf-token config lifetime 1 
    r10900(config)# commit
    Commit complete.
    r10900(config)# 

To display the current restconf-token lifetime setting, use the command **show system aaa***.

.. code-block:: bash

    r10900(config)# do show system aaa
    system aaa restconf-token state lifetime 1
    system aaa primary-key state hash gK/F47uQfi7JWYFirStCVhIaGcuoctpbGpx63MNy/korwigBW6piKx9TldiRazHmE8Y+qylGY4MOcs9IZ+KG4Q==
    system aaa primary-key state status NONE
    system aaa authentication state basic enabled
            LAST        TALLY  EXPIRY                  
    USERNAME  CHANGE      COUNT  DATE    ROLE            
    -----------------------------------------------------
    admin     2022-06-02  0      -1      admin           
    jim-test  2022-09-02  10     -1      admin           
    operator  2022-10-11  0      -1      operator        
    root      2022-06-02  0      -1      root            
    tenant1   0           0      1       tenant-console  
    tenant2   0           0      1       tenant-console  

    ROLENAME        GID   USERS  
    -----------------------------
    admin           9000  -      
    operator        9001  -      
    tenant-console  9100  -      

    NAME    NAME    TYPE    
    ------------------------
    tacacs  tacacs  TACACS  

    system aaa tls state verify-client false
    system aaa tls state verify-client-depth 1

Token Lifetime via webUI
------------------------

You may configure the restconf-token lifetime via the webUI (new feature added in F5OS-A 1.4.0). The value is in minutes, and the client is able to refresh the token five times before it expires. As an example, if the token lifetime is set to 1 minute, an inactive webUI session will have a token expire after one minute, but it can be refreshed a maximum of five times. This will result in the webUI session timing out after 5 minutes.

.. image:: images/rseries_security/image6.png
  :align: center
  :scale: 70%

Token Lifetime via API
----------------------

You may configure the restconf-token lifetime via the API. The value is in minutes, and the client is able to refresh the token five times before it expires. As an example, if the token lifetime is set to 1 minute, an inactive webUI session or API session will have a token expire after one minute, but it can be refreshed a maximum of five times. This will result in the webUI session timing out after 5 minutes.

Use the following API PATCH call to set the restconf-token lifetime, or any other password policy parameter.

.. code-block:: bash

    PATCH https://{{rseries_appliance1_ip}}:8888/restconf/data/openconfig-system:system/aaa

In the body of the API call adjust the restconf-token lifetime setting to the desired timeout in minutes. The example below is 10 minutes, and the session will timeout at five times the value of the lifetime setting due to token refresh.

.. code-block:: json

    {
        "openconfig-system:aaa": {
            "authentication": {
                "config": {
                    "f5-aaa-confd-restconf-token:basic": {
                        "enabled": true
                    }
                }
            },
            "f5-aaa-confd-restconf-token:restconf-token": {
                "config": {
                    "lifetime": 10
                }
            },
            "f5-openconfig-aaa-password-policy:password-policy": {
                "config": {
                    "min-length": 6,
                    "required-numeric": 0,
                    "required-uppercase": 0,
                    "required-lowercase": 0,
                    "required-special": 0,
                    "required-differences": 8,
                    "reject-username": false,
                    "apply-to-root": true,
                    "retries": 3,
                    "max-login-failures": 10,
                    "unlock-time": 60,
                    "root-lockout": true,
                    "root-unlock-time": 60,
                    "max-age": 0
                }
            }
        }
    }


Disabling Basic Authentication
==============================

F5OS utilizes basic authentication (username/password) as well as token based authentication for both the API and the webUI. Generally, username/password is issued by the client in order to obtain a token from F5OS, which is then used to make further inquiries or changes. Tokens have a relatively short lifetime for security reasons, and the user is allowed to refresh that token a certain number of times before they are forced to re-authenticate using basic authentication again. Although token based authentication is supported, basic authentication can still be utilized to access F5OS and make changes by default. A new option was added in F5OS-A 1.3.0 to allow basic authentication to be disabled, except for the means of obtaining a token. Once a token is issued to a client, it will be the only way to make changes via the webUI or the API. 


Disabling Basic Auth via the CLI
--------------------------------

The default setting for basic auth is enabled, and the current state can be seen by entering the **show system aaa** command. The line **system aaa authentication state basic enabled** indicates that basic authentication is still enabled. 

.. code-block:: bash

    r10900# show system aaa
    system aaa restconf-token state lifetime 15
    system aaa primary-key state hash gK/F47uQfi7JWYFirStCVhIaGcuoctpbGpx63MNy/korwigBW6piKx9TldiRazHmE8Y+qylGY4MOcs9IZ+KG4Q==
    system aaa primary-key state status NONE
    system aaa authentication state basic enabled
            LAST        TALLY  EXPIRY                  
    USERNAME  CHANGE      COUNT  DATE    ROLE            
    -----------------------------------------------------
    admin     2022-06-02  0      -1      admin           
    jim-test  2022-09-02  10     -1      admin           
    operator  2022-10-11  0      -1      operator        
    root      2022-06-02  0      -1      root            
    tenant1   0           0      1       tenant-console  
    tenant2   0           0      1       tenant-console  

    ROLENAME        GID   USERS  
    -----------------------------
    admin           9000  -      
    operator        9001  -      
    root            0     -      
    tenant-console  9100  -      

    NAME    NAME    TYPE    
    ------------------------
    tacacs  tacacs  TACACS  

    r10900# 

You may disable basic authentication by issuing the cli command **system aaa authentication config basic disabled**, and then committing the change.

.. code-block:: bash

    r10900(config)# system aaa authentication config basic disabled 
    r10900(config)# commit
    Commit complete.
    r10900(config)#

To re-enable basic authentication, change the state to enabled and commit.

.. code-block:: bash

    r10900(config)# system aaa authentication config basic enabled 
    r10900(config)# commit
    Commit complete.
    r10900(config)#



Disabling Basic Auth via the API
--------------------------------

You may enable or disable basic authentication via the API. The default setting for basic authentication is enabled, and the current state can be seen by entering the following API call.

.. code-block:: bash

    GET https://{{rseries_appliance1_ip}}:8888/restconf/data/openconfig-system:system/aaa/authentication/config

You should see the returned output below with the basic authentication state set to either **true** or **false**.

.. code-block:: json

    {`
        "openconfig-system:config": {
            "f5-aaa-confd-restconf-token:basic": {
                "enabled": true
            }
        }
    }

Use the following API PATCH call to set the restconf-token:basic setting to **true** or **false**, or to adjust any other password policy parameter.

.. code-block:: bash

    PATCH https://{{rseries_appliance1_ip}}:8888/restconf/data/openconfig-system:system/aaa

In the body of the API call adjust the restconf-token:basic setting to **true** or **false**.

.. code-block:: json

    {
        "openconfig-system:aaa": {
            "authentication": {
                "config": {
                    "f5-aaa-confd-restconf-token:basic": {
                        "enabled": true
                    }
                }
            },
            "f5-aaa-confd-restconf-token:restconf-token": {
                "config": {
                    "lifetime": 10
                }
            },
            "f5-openconfig-aaa-password-policy:password-policy": {
                "config": {
                    "min-length": 6,
                    "required-numeric": 0,
                    "required-uppercase": 0,
                    "required-lowercase": 0,
                    "required-special": 0,
                    "required-differences": 8,
                    "reject-username": false,
                    "apply-to-root": true,
                    "retries": 3,
                    "max-login-failures": 10,
                    "unlock-time": 60,
                    "root-lockout": true,
                    "root-unlock-time": 60,
                    "max-age": 0
                }
            }
        }
    }


Disabling Basic Auth via the webUI
----------------------------------

Disabling basic authentication via the webUI is a new feature that has been added in F5OS-A 1.4.0. In the webUI go to **User Management -> Authentication Settings** and you'll see a drop down box to enable or disable **Basic Authentication**.

.. image:: images/rseries_security/image5.png
  :align: center
  :scale: 70%


Setting Password Policies
=========================

You may configure the local password policy to ensure secure passwords are utilized, re-use is minimized, and to limit the amount of failures/retries. Below are some of the settings that can be set.

- **Minimum Password Length** - For Minimum Length, specify the minimum number of characters (6 to 255) required for a valid password.
- **Password Required Characters** - For Required Characters, specify the minimum number of Numeric, Uppercase, Lowercase, and Special characters that are required in a valid password.
- **New/Old Password Differential** - For New/Old Password Differential, specify the number of character changes in the new password that differentiate it from the old password. The default value is 8.
- **Disallow Username** - For Disallow Username, set to True to check whether the name of the user in forward or reversed form is contained in the password. The default value is False.
- **Apply Password Policy to Root Account** - For Apply Password Policy to Root Account, set to True to use the same password policy for the root account. The default value is True.
- **Maximum Password Retries** - For Maximum Password Retries, specify the number of times that a user can try to create an acceptable password. The default value is 3.
- **Maximum Login Attempts** - For Maximum Login Attempts, specify the number of times a user can attempt to log in before the account is temporarily suspended. The default value is 10; 0 means no limit.
- **Lockout Duration** - For Lockout Duration, specify the duration, in seconds, an account is locked out. The default value is 60.
- **Maximum Password Age** - For Max Password Age, specify the number of days after which the password will expire after being changed. 0 means never expires.

Setting Password Policies via CLI
---------------------------------

Local Password Policies can be set in the CLI using the **system aaa password-policy config** command. Adding a question mark after the command will show all the configurable options. Be sure to commit after making any changes.

.. code-block:: bash

    r10900-2(config)# system aaa password-policy config ?
    Possible completions:
    apply-to-root          Apply password restrictions to root accounts.
    max-age                Number of days after which the user will have to change the password.
    max-login-failures     Number of unsuccessful login attempts allowed before lockout.
    min-length             Minimum length of a new password.
    reject-username        Reject passwords that contain the username.
    required-differences   Required number of differences between the old and new passwords.
    required-lowercase     Required number of lowercase characters in password.
    required-numeric       Required number of numeric digits in password.
    required-special       Required number of 'special' characters in password.
    required-uppercase     Required number of uppercase character in password.
    retries                Number of times to prompt before failing.
    root-lockout           Enable lockout of root users.
    root-unlock-time       Time (seconds) before the root account is automatically unlocked.
    unlock-time            Time (seconds) before a locked account is automatically unlocked.
    r10900-2(config)# 

Setting Password Policies via webUI
---------------------------------

Local Password Policies can be set in the **User Management -> Authentication Settings** page in the webUI.

.. image:: images/rseries_security/passwordpolicy1.png
  :align: center
  :scale: 70%

Setting Password Policies via API
---------------------------------

Local Password Policies can be viewed or set via the API using the following API calls. To view the current password policy settings issue the following GET API call.

.. code-block:: bash

    GET https://{{rseries_appliance1_ip}}:8888/restconf/data/openconfig-system:system/aaa/f5-openconfig-aaa-password-policy:password-policy

The JSON output will reflect the current settings.

.. code-block:: json

    {
        "f5-openconfig-aaa-password-policy:password-policy": {
            "config": {
                "min-length": 6,
                "required-numeric": 0,
                "required-uppercase": 0,
                "required-lowercase": 0,
                "required-special": 0,
                "required-differences": 8,
                "reject-username": false,
                "apply-to-root": true,
                "retries": 3,
                "max-login-failures": 10,
                "unlock-time": 60,
                "root-lockout": true,
                "root-unlock-time": 60,
                "max-age": 0
            }
        }
    }

To change any of the password policy parameters, use the following API GET call.

.. code-block:: bash

    PATCH https://{{rseries_appliance1_ip}}:8888/restconf/data/openconfig-system:system/aaa

In the payload of the API call adjust the appropriate parameters under **f5-openconfig-aaa-password-policy:password-policy**.


.. code-block:: json

    {
        "openconfig-system:aaa": {
            "authentication": {
                "config": {
                    "f5-aaa-confd-restconf-token:basic": {
                        "enabled": true
                    }
                }
            },
            "f5-aaa-confd-restconf-token:restconf-token": {
                "config": {
                    "lifetime": 10
                }
            },
            "f5-openconfig-aaa-password-policy:password-policy": {
                "config": {
                    "min-length": 6,
                    "required-numeric": 0,
                    "required-uppercase": 0,
                    "required-lowercase": 0,
                    "required-special": 0,
                    "required-differences": 8,
                    "reject-username": false,
                    "apply-to-root": true,
                    "retries": 3,
                    "max-login-failures": 10,
                    "unlock-time": 60,
                    "root-lockout": true,
                    "root-unlock-time": 60,
                    "max-age": 0
                }
            }
        }
    }

Remote Authentication
=====================

The F5OS platform layer supports both local and remote authentication. By default, there are local users enabled for both admin and root access. You will be forced to change passwords for both of these accounts on intial login. Many users will prefer to configure the F5OS layer to use remote authentication via LDAP, RADIUS, AD, or TACACS+. The F5OS TMOS based tenants maintain their own local or remote authentication, and details are covered in standard TMOS documentation.

`Configuring Remote User Authentication and Authorization on TMOS <https://techdocs.f5.com/kb/en-us/products/big-ip_ltm/manuals/product/tmos-implementations-13-0-0/10.html>`_

In versions prior to F5OS-A 1.4.0, F5OS only supported static pre-defined roles which in turn map to specific group IDs. Users created and managed on external LDAP, Active Directory, RADIUS, or TACACS+ servers must have the same group IDs on the external authentication servers as they do within F5OS based systems to allow authentication and authorization to occur. Users created on external LDAP, Active Directory, RADIUS, or TACACS+ servers must be associated with one of these group IDs on the system. The supported F5OS static group IDs and the roles they map to are seen in the table below. User defined roles are not supported in version prior to F5OS-A 1.4.0.

+----------------+----------+
| Role           | Group ID | 
+================+==========+
| admin          | 9000     | 
+----------------+----------+
| operator       | 9001     |
+----------------+----------+
| root           | 0        | 
+----------------+----------+
| tenant-console | 9100     | 
+----------------+----------+

From a high level the **admin** role (group ID 9000) is a read/write role with full access to the system to make changes. The **operator** role (group ID 9001) is a read-only role and is prevented form making any configuration changes. The **root** role (group ID 0) gives full access to the bash shell, and in some environments this role will be disabled by enabling appliance mode. Note that the root role is not allowed access via remote authentication. The last role is **tenant-console** (group ID 9100) and this role is used to provide remote access directly to the tenant console as noted here:

` Console Access to Tenant via Built-In Terminal Server <https://clouddocs.f5.com/training/community/rseries-training/html/rseries_diagnostics.html#console-access-via-built-in-terminal-server>`_

The group IDs are typically specified in a user configuration file on the external server (file locations vary on different servers). You can assign these F5 user attributes: 

.. code-block:: bash

    F5-F5OS-UID=1001 

    F5-F5OS-GID=9000   <-- THIS MUST MATCH /etc/group items    

    F5-F5OS-HOMEDIR=/tmp  <-- Optional; prevents sshd warning msgs  

    F5-F5OS-USERINFO=test_user  <-- Optional user info  

    F5-F5OS-SHELL=/bin/bash    <--  Ignored; always set to /var/lib/controller/f5_confd_cli 

Setting F5-F5OS-HOMEDIR=/tmp is a good idea to avoid warning messages from sshd that the directory does not exist. Also, the source address in the TACACS+ configuration is not used by the rSeries system. 

If F5-F5OS-UID is not set, it defaults to 1001. If F5-F5OS-GID is not set, it defaults to 0 (disallowed for authentication). The F5-F5OS-USERINFO is a comment field. Essentially, F5-F5OS-GID is the only hard requirement and must coincide with group ID's user role (except for the root role where the GID is 0). 

More specific configuration details can be found in the **User Management** section of the **rSeries System Administration Guide**.

`F5OS User Management <https://techdocs.f5.com/en-us/f5os-a-1-3-0/f5-rseries-systems-administration-configuration/title-user-mgmt.html#user-management>`_

The **gidNumber** attribute needs to either be on the user or on a group the user is a member of. The **gidNumber** must be one of those listed (9000, 9001, 9100). [The root role is not externally accessible for obvious reasons.] 

The current implementation relies on AD “unix attributes” being installed into the directory.

AD groups are not currently queried. The role IDs are fixed. As noted above, the IDs are configurable in F5OS-A 1.4.0, but this is still based on numeric GIDs not group names. 

Currently the role numbers (9000, 9001, 9100) are fixed and hard-coded. 

Roles are mutually exclusive. While it is theoretically possible to assign a user to multiple role groups, It is up to confd to resolve how the roles present to it are assigned, and it doesn’t always choose the most logical answer. For that reason, you should consider them mutually exclusive and put the user in the role with the least access necessary to do their work. More details, on configuration of F5OS-A 1.3.0 can be found below.

`LDAP/AD configuration overview <https://techdocs.f5.com/en-us/f5os-a-1-3-0/f5-rseries-systems-administration-configuration/title-user-mgmt.html#ldap-config-overview>`_

Changing Group ID Mapping via CLI (F5OS-A 1.4.0 and Later)
---------------------------------------------------------

F5OS-A 1.4.0 has added the ability to customize the Group ID mapping to the remote authentication server. In previous releases the Group IDs were static, now they can be changed to map to user selectable Group IDs. Below is an example of changing the remote Group ID for the admin account to a custom value of 9200.

.. code-block:: bash

    r10900-1(config)# system aaa authentication roles role admin config remote-gid 9200 
    r10900-1(config-role-admin)# commit
    Commit complete.
    r10900-1(config-role-admin)# 

To view the current mappings use the **show system aaa authentication roles** CLI command.

.. code-block:: bash

    r10900-1# show system aaa authentication roles
                        REMOTE         
    ROLENAME        GID   GID     USERS  
    -------------------------------------
    admin           9000  9200    -      
    operator        9001  -       -      
    resource-admin  9003  -       -      
    tenant-console  9100  -       -      

    r10900-1# 


Login Banner / Message of the Day
===================

Some environments require warning or acceptance messages to be displayed to clients connecting to the F5OS layer at initial connection time and/or upon successful login. The F5OS layer supports configurable Message of the Day (MoTD) and Login Banners that are displayed to clients connecting to the F5OS layer via both CLI and the webUI. The MoTD and Login Banner can be configured via CLI, webUI, or API. The Login Banner is displayed at initial connect time and is commonly used to notify users they are connecting to a specific resource, and that they should not connect if they are not authorized. The MoTD is displayed after successful login, and may also display some information about the resource the user is connecting to.

Configuring Login Banner / MoTD via CLI
---------------------------------------

Enter config mode and use the command **system config login-banner** to configure the login banner via the CLI. You must commit the change afterwards.

.. code-block:: bash

    r10900(config)# system config login-banner "This is a restricted resource. Unauthorized access is prohibited. Please disconnect now if you are not authorized."                                                 
    r10900(config)# commit
    Commit complete.
    r10900(config)# 

Enter config mode and use the command **system config motd-banner** to configure the Message of the Day banner via the CLI. You must commit the change afterwards.

.. code-block:: bash

    r10900(config)# system config motd-banner "Welcome to the GSA r10900 unit#1, do not make any changes to configuration without a ticket." 
    r10900(config)# commit
    Commit complete.
    r10900(config)# 

To display both settings, use the **show system state** command.

.. code-block:: bash

    r10900# show system state 
    system state hostname r10900.f5demo.net
    system state login-banner This is a restricted resource. Unauthorized access is prohibited. Please disconnect now if you are not authorized.
    system state motd-banner Welcome to the GSA r10900 unit#1, do not make any changes to configuration without a ticket.
    system state current-datetime "2022-11-29 11:12:27-05:00"
    system state base-mac 00:94:a1:69:59:00
    system state mac-pool-size 256
    r10900# 



Configuring Login Banner / MoTD via webUI
-----------------------------------------

You may configure both the Login Banner and the Message of the Day Banner via the webUI on the **System Settings -> General** page.

.. image:: images/rseries_security/image7.png
  :align: center
  :scale: 70%



Configuring Login Banner / MoTD via API
---------------------------------------

You may configure both the Login Banner and the Message of the Day Banner via the API using the following API calls.

.. code-block:: bash

    PATCH https://{{rseries_appliance1_ip}}:8888/restconf/data/openconfig-system:system

In the body of the API call configure the desired message of the day and login banner settings.

.. code-block:: json

    {
        "openconfig-system:system": {
            "config": {
                "hostname": "r10900-1.f5demo.net",
                "login-banner": "This is the Global Solution Architect's rSeries r10900 unit-1 in the Boston Lab. Unauthorized use is prohibited. Please reach out to Jim McCarron with any questions.",
                "motd-banner": "Welcome to the GSA r10900 Unit 1 in Boston"
            }
        }
    }

To view the currently configured MoTD and login banner, issue the folowing GET API request.

.. code-block:: bash

    GET https://{{rseries_appliance1_ip}}:8888/restconf/data/openconfig-system:system/config

The output will contain the current MoTD and login banner configuration.

.. code-block:: json

    {
        "openconfig-system:config": {
            "hostname": "r10900.f5demo.net",
            "login-banner": "This is a restricted resource. Unauthorized access is prohibited. Please disconnect now if you are not authorized.",
            "motd-banner": "This is a test"
        }
    }


Display of Login Banner and MoTD
--------------------------------

Below is an example of the Login Banner being displayed before the user is prompted for a password during an SSH connection to the F5OS platform layer. After a successful user login, the MoTD is then displayed. 

.. code-block:: bash

    prompt:~ user$ ssh -l admin 10.255.0.132
    This is a restricted resource. Unauthorized access is prohibited. Please disconnect now if you are not authorized.
    admin@10.255.0.132's password: 
    Last login: Tue Nov 29 10:41:06 2022 from 10.10.10.16
    Welcome to the GSA r10900 unit#1, do not make any changes to configuration without a ticket.
    System Time: 2022-11-29 11:17:00 EST
    Welcome to the Management CLI
    User admin last logged in 2022-11-29T16:17:00.008317+00:00, to appliance-1, from 10.10.10.16 using cli-ssh
    admin connected from 10.10.10.16 using ssh on r10900.f5demo.net
    r10900# 

Below is an example of the Login Banner being displayed before the user is prompted for a password during a webUI connection to the F5OS platform layer. After a successful user login, the MoTD is then displayed.


.. image:: images/rseries_security/image8.png
  :align: center
  :scale: 70%


.. image:: images/rseries_security/image9.png
  :align: center
  :scale: 70%  


SNMPv3
=======

F5OS-A 1.2.0 added support for SNMPv3. Earlier versions of F5OS-A only supported SNMPv1/v2c. SNMPv3 provides a more secure monitoring environment through the use of authenticated access. More details can be found here:

`rSeries F5OS-A SNMP Monitoring and Alerting <https://clouddocs.f5.com/training/community/rseries-training/html/rseries_monitoring_snmp.html>`_


NTP Authentication
==================

NTP Authentication can be enabled to provide a secure communication channel for Network Time Protocol queries from the F5OS platform layer. In order to utilize NTP authentication you must first enable NTP authentication and then add keys in order to secure communication to your NTP servers.

Enabling NTP Authentication via CLI
-----------------------------------

To enable NTP authentication use the **system ntp config enable-ntp-auth true** command in the CLI, and then commit the change.

.. code-block:: bash

    r10900(config)# system ntp config enable-ntp-auth true 
    r10900(config)# commit
    Commit complete.
    r10900(config)# 

Next you'll need to add keys for NTP Authentication

.. code-block:: bash

    r10900(config)# system ntp ntp-keys ntp-key 11 config key-id 11 key-type F5_NTP_AUTH_SHA1 key-value HEX:E27611234BB5E7CDFC8A8ACE55B567FC5CA7C890

The key ID, key type, and key value on this client system must match the server exactly. Lastly, you'll need to associate the key with an NTP server using the configured key-id above.

.. code-block:: bash

    r10900(config)# system ntp servers server 10.255.0.139
    r10900(config-server-10.255.0.139)# config key-id 11

Enabling NTP Authentication via webUI
-------------------------------------

To enable NTP authentication in the webUI use the **System Settings -> Time Settings** page. You'll need to enable NTP authentication then add the appropriate keys, and then associate those keys with an NTP server.

.. image:: images/rseries_security/ntpauth1.png
  :align: center
  :scale: 70%  

Enabling NTP Authentication via API
-----------------------------------

NTP authentication can also be set and viewed using the F5OS API. To view the current NTP setting use the following API call.

.. code-block:: bash

    GET https://{{rseries_appliance1_ip}}:8888/restconf/data/openconfig-system:system/ntp

The output will display the current NTP configuration state including authentication and keys.

.. code-block:: json

    {
        "openconfig-system:ntp": {
            "config": {
                "enabled": true,
                "enable-ntp-auth": true
            },
            "state": {
                "enabled": true,
                "enable-ntp-auth": true
            },
            "ntp-keys": {
                "ntp-key": [
                    {
                        "key-id": 11,
                        "config": {
                            "key-id": 11,
                            "key-type": "f5-system-ntp:F5_NTP_AUTH_SHA1",
                            "key-value": "$8$IIACWGpGPUYzian06FdH5PpH/sbSNQmre6DVsBZ2zxCv6S5vM3cXUkn8NwD0BABSeT3Drnmm\npLCQibKafAFFPg=="
                        },
                        "state": {
                            "key-id": 11,
                            "key-type": "F5_NTP_AUTH_SHA1",
                            "key-value": "$8$IIACWGpGPUYzian06FdH5PpH/sbSNQmre6DVsBZ2zxCv6S5vM3cXUkn8NwD0BABSeT3Drnmm\npLCQibKafAFFPg=="
                        }
                    }
                ]
            },
            "servers": {
                "server": [
                    {
                        "address": "10.255.0.139",
                        "config": {
                            "address": "10.255.0.139",
                            "port": 123,
                            "version": 4,
                            "association-type": "SERVER",
                            "iburst": false,
                            "prefer": false,
                            "f5-openconfig-system-ntp:key-id": 11
                        },
                        "state": {
                            "address": "10.255.0.139",
                            "port": 123,
                            "version": 4,
                            "association-type": "SERVER",
                            "iburst": false,
                            "prefer": false,
                            "f5-openconfig-system-ntp:key-id": 11,
                            "f5-openconfig-system-ntp:authenticated": false
                        }
                    },
                    {
                        "address": "time.f5net.com",
                        "config": {
                            "address": "time.f5net.com",
                            "port": 123,
                            "version": 4,
                            "association-type": "SERVER",
                            "iburst": false,
                            "prefer": false
                        },
                        "state": {
                            "address": "time.f5net.com",
                            "port": 123,
                            "version": 4,
                            "association-type": "SERVER",
                            "iburst": false,
                            "prefer": false,
                            "f5-openconfig-system-ntp:authenticated": false
                        }
                    }
                ]
            }
        }
    }

To enable NTP authentication via the F5OS API use the following API call.

.. code-block:: bash

    PATCH https://{{rseries_appliance1_ip}}:8888/restconf/data/openconfig-system:system/ntp

In the body of the API call you can enable NTP authentication, add keys, and associate those keys with an NTP server using the key-id.

.. code-block:: json

    {
        "openconfig-system:ntp": {
            "config": {
                "enabled": true,
                "enable-ntp-auth": true
            },
            "ntp-keys": {
                "ntp-key": [
                    {
                        "key-id": 11,
                        "config": {
                            "key-id": 11,
                            "key-type": "f5-system-ntp:F5_NTP_AUTH_SHA1",
                            "key-value": "$8$IIACWGpGPUYzian06FdH5PpH/sbSNQmre6DVsBZ2zxCv6S5vM3cXUkn8NwD0BABSeT3Drnmm\npLCQibKafAFFPg=="
                        }
                    }
                ]
            },
            "servers": {
                "server": [
                    {
                        "address": "10.255.0.139",
                        "config": {
                            "address": "10.255.0.139",
                            "port": 123,
                            "version": 4,
                            "association-type": "SERVER",
                            "iburst": false,
                            "prefer": false,
                            "f5-openconfig-system-ntp:key-id": 11
                        }
                    }
                ]
            }
        }
    }




Configurable Management Ciphers
===============================

F5OS-A 1.4.0 added the ability to display and configure the ciphers used for the management interface of F5OS. The **show system security** CLI command will display the **ssl-ciphersuite** for the webUI/httpd management interface. It will also display the **ciphers** and **kexalgorithms** for the sshd service. Below is an example of the default settings. 

.. code-block:: bash

    r10900-1# show system security 
    system security services service httpd
    state ssl-ciphersuite ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:DHE-DSS-AES256-GCM-SHA384:DHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES256-SHA256:DHE-DSS-AES256-SHA256:DHE-RSA-AES256-SHA:DHE-DSS-AES256-SHA:DHE-RSA-CAMELLIA256-SHA:DHE-DSS-CAMELLIA256-SHA:ECDH-RSA-AES256-GCM-SHA384:ECDH-ECDSA-AES256-GCM-SHA384:ECDH-RSA-AES256-SHA384:ECDH-ECDSA-AES256-SHA384:ECDH-RSA-AES256-SHA:ECDH-ECDSA-AES256-SHA:AES256-GCM-SHA384:AES256-SHA256:AES256-SHA:CAMELLIA256-SHA:PSK-AES256-CBC-SHA:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:DHE-DSS-AES128-GCM-SHA256:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES128-SHA256:DHE-DSS-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-DSS-AES128-SHA:DHE-RSA-CAMELLIA128-SHA:DHE-DSS-CAMELLIA128-SHA:ECDH-RSA-AES128-GCM-SHA256:ECDH-ECDSA-AES128-GCM-SHA256:ECDH-RSA-AES128-SHA256:ECDH-ECDSA-AES128-SHA256:ECDH-RSA-AES128-SHA:ECDH-ECDSA-AES128-SHA:AES128-GCM-SHA256:AES128-SHA256:AES128-SHA:CAMELLIA128-SHA:PSK-AES128-CBC-SHA
    system security services service sshd
    state ciphers [ aes128-cbc aes128-ctr aes128-gcm@openssh.com aes256-cbc aes256-ctr aes256-gcm@openssh.com ]
    state kexalgorithms [ diffie-hellman-group14-sha1 diffie-hellman-group14-sha256 diffie-hellman-group16-sha512 ecdh-sha2-nistp256 ecdh-sha2-nistp384 ecdh-sha2-nistp521 ]
    r10900-1#

You can change the ciphers offered by F5OS to clients connecting to the httpd service by using the **system security services service httpd config ssl-ciphersuite** CLI command, and then choosing the ciphers you would like to enable. Be sure to commit any changes.

.. code-block:: bash

    r10900-1(config)# system security services service httpd config ssl-ciphersuite ?
    Description: User specified ssl-ciphersuite.
    Possible completions:
    <string>[ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:DHE-DSS-AES256-GCM-SHA384:DHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES2
    56-SHA256:DHE-DSS-AES256-SHA256:DHE-RSA-AES256-SHA:DHE-DSS-AES256-SHA:DHE-RSA-CAMELLIA256-SHA:DHE-DSS-CAMELLIA256-SHA:ECDH-RSA-AES256-GCM-SHA384:ECDH-ECDSA-AES256-GCM-SHA384:ECDH-RSA-AES256-SHA384:ECDH-ECDSA-AES256-SHA384:ECDH-
    RSA-AES256-SHA:ECDH-ECDSA-AES256-SHA:AES256-GCM-SHA384:AES256-SHA256:AES256-SHA:CAMELLIA256-SHA:PSK-AES256-CBC-SHA:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDH
    E-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:DHE-DSS-AES128-GCM-SHA256:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES128-SHA256:DHE-DSS-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-DSS-AES128-SHA:DHE-RSA-CAMELLIA128-SHA:DHE-DSS-CAMELLIA128-SHA:ECDH-
    RSA-AES128-GCM-SHA256:ECDH-ECDSA-AES128-GCM-SHA256:ECDH-RSA-AES128-SHA256:ECDH-ECDSA-AES128-SHA256:ECDH-RSA-AES128-SHA:ECDH-ECDSA-AES128-SHA:AES128-GCM-SHA256:AES128-SHA256:AES128-SHA:CAMELLIA128-SHA:PSK-AES128-CBC-SHA]
    r10900-1(config)# 
    
You can change the ciphers and kexalgorithms offered by F5OS to clients connecting to the sshd service by using the **system security services service sshd config ssl-ciphersuite** CLI command, and then choosing the ciphers you would like to enable. Be sure to commit any changes.

.. code-block:: bash

    r10900-1(config)# system security services service sshd config ?
    Possible completions:
        ciphers         User specified ciphers.
        kexalgorithms   User specified kexalgorithms.
        macs            User specified MACs.


Below are the current options for sshd cipers and kexalgorithms.

    system security services service sshd
    config ciphers [ aes128-cbc aes128-ctr aes128-gcm@openssh.com aes256-cbc aes256-ctr aes256-gcm@openssh.com ]
    config kexalgorithms [ diffie-hellman-group14-sha1 diffie-hellman-group14-sha256 diffie-hellman-group16-sha512 ecdh-sha2-nistp256 ecdh-sha2-nistp384 ecdh-sha2-nistp521 ]
    !


Client Certificate Based Auth
=============================

Coming in F5OS-A 1.5.0.

iHealth Proxy Server
====================

F5OS supports the ability to capture detailed logs and configuration using the qkView utility. To speed up support case resolution the qkView can be uploaded directly to F5's iHealth service, which will give F5 support personnel access to the detailed information to aid problem resolution. In some environments, F5 devices may not have the ability to access the Internet without going through a proxy. The F5OS-A 1.3.0 release added the ability to upload qkViews directly to iHealth through a proxy device.


Adding a Proxy Server via CLI
------------------------------

To add a proxy server for iHealth uploads via the CLI, use the **system diagnostics proxy** command.

.. code-block:: bash

    r10900(config)# system diagnostics proxy config proxy-username myusername proxy-server https://myproxy.com:3128 proxy-password 
    (<AES encrypted string>): **************
    r10900(config)# 

Adding a Proxy Server via webUI
-------------------------------

To add a proxy server for iHealth uploads via the webUI, go to the **Diagnostics -> iHealth Configuration** page. 

.. image:: images/rseries_security/imageproxy1.png
  :align: center
  :scale: 70%  

Adding a Proxy Server via API
------------------------------

To add a proxy server for iHealth uploads via the API, use the following API call.

.. code-block:: bash

    PATCH https://{{rseries_appliance1_ip}}:8888/restconf/data/openconfig-system:system/f5-system-diagnostics-qkview:diagnostics/f5-system-diagnostics-proxy:proxy

In the body of the API call add the username, password, and proxy server configuration.

.. code-block:: json


    {
        "f5-system-diagnostics-proxy:proxy": {
            "config": {
                "proxy-username": "username2",
                "proxy-password": "$8$8FudCujBpUpoTBaQQw4QaTeyUU8UHdkYAv90Dfx43SA=",
                "proxy-server": "https://myproxy2.demo.f5net"
            }
        }
    }


To view the current proxy configuration via the API use the following call.

.. code-block:: bash

    GET https://{{rseries_appliance1_ip}}:8888/restconf/data/openconfig-system:system/f5-system-diagnostics-qkview:diagnostics/f5-system-diagnostics-proxy:proxy

The API call should return output similar to what is seen below.

.. code-block:: json

    {
        "f5-system-diagnostics-proxy:proxy": {
            "state": {
                "proxy-username": "username",
                "proxy-server": "https://myproxy.demo.f5net"
            },
            "config": {
                "proxy-username": "username",
                "proxy-password": "$8$8FudCujBpUpoTBaQQw4QaTeyUU8UHdkYAv90Dfx43SA=",
                "proxy-server": "https://myproxy.demo.f5net"
            }
        }
    }


Audit Logging
=============

F5OS has the ability to log all configuration changes and access to the F5OS layer in audit logs. In versions prior to F5OS-A 1.4.0, all access and configuration changes are logged in one of two separate **audit.log** files. The files reside in the in one of the following paths in the F5OS filesystem when logged in as root; **/var/F5/system/log/audit.log** or **/var/log/audit/audit.log**. If you are logged into the F5OS CLI as admin, then the actual paths are simplified to **log/system/audit.log** and **/log/host/audit/audit.log**.

In versions prior to F5OS-A 1.4.0, the audit.log files may only be viewed locally within the F5OS layer, the audit logs cannot be sent to a remote syslog location. F5OS-A 1.4.0 adds the ability to allow audit.log entries to be redirected to a remote syslog location, as well as changing the log format to conform to standard F5OS syslog format of all audit related events. Details on the two different implementations are below.

Viewing Audit Logs via F5OS CLI (F5OS-A 1.4.0 and Later)
--------------------------------------------------------

Any information related to login/logout or configuration changes are logged in the **log/system/audit.log** location. By default these events are not sent to a configured remote syslog location. If you would like to send informational audit level messages to a remote syslog server, then you must explicitly enable audit events.

First you must configure the remote syslog destination. As part of that configuration, you will specify the IP address, port, and protocol of the remote syslog server. To send audit.log events to the remote server you must add the command **selectors selector AUTHPRIV DEBUG** as seen below.

.. code-block:: bash

    r10900(config)# system logging remote-servers remote-server 10.255.0.139
    r10900(config-remote-server-10.255.0.139)# config remote-port 514
    r10900(config-remote-server-10.255.0.139)# config proto udp
    r10900(config-remote-server-10.255.0.139)# selectors selector LOCAL0 INFORMATIONAL
    r10900(config-remote-server-10.255.0.139)# selectors selector AUTHPRIV DEBUG
    r10900(config-remote-server-10.255.0.139)# commit
    % No modifications to commit.
    r10900(config-remote-server-10.255.0.139)#

Then, you can control the level of events that will be logged to the local audit.log file by configuring the **audit-service** **sw-component**. By default all audit events will be logged, but you can turn down the level of events

.. code-block:: bash

    r10900# show running-config system logging sw-components sw-component audit-service
    system logging sw-components sw-component audit-service
    config name audit-service
    config description "Audit message handling service"
    config severity DEBUG
    !

The formatting of audit logs provide the date/time in UTC, the account and ID who performed the action, the type of event, the asset affected, the type of access, and success or failure of the request. Separate log entries provide details on user access (login/login failures) information such as IP address and port and whether access was granted or not.


Viewing Audit Logs via F5OS CLI
-------------------------------

Most audit events go to the **log/system/audit.log** location, while a few others such as CLI login failures are logged to **log/host/audit.log** in the current F5OS releases. In the F5OS CLI, the paths are simplified so that you don’t have to know the underlying directory structure. You can use the **file list path** command to see the files inside the **log/system/** directory; use the tab complete to see the options. You may choose either the **log/system** directory or the **log/host** directory. Note the **audit.log** file. 

.. code-block:: bash

    appliance-1# file list path log/
    Possible completions:
    confd/  host/  system/
    appliance-1# file list path log/system/
    Possible completions:
    audit.log                      confd.log          devel.log     devel.log.1    lcd.log           lcd.log.1           lcd.log.2.gz       
    lcd.log.3.gz                   lcd.log.4.gz       lcd.log.5.gz  logrotate.log  logrotate.log.1   logrotate.log.2.gz  platform.log       
    reprogram_chassis_network.log  rsyslogd_init.log  snmp.log      startup.log    startup.log.prev  trace/              vconsole_auth.log  
    vconsole_startup.log           velos.log          webUI/        
    appliance-1# file list path log/system/

To view the contents of the **audit.log** file, use the command **file show path /log/system/audit.log**. This will show the entire log file from the beginning, but may not be the best way to troubleshoot a recent event:

.. code-block:: bash

    r10900# file show log/system/audit.log
    <INFO> 9-Dec-2021::17:13:57.506 appliance-1 confd[106]: audit user: admin/20518 assigned to groups: admin
    <INFO> 9-Dec-2021::17:13:57.506 appliance-1 confd[106]: audit user: admin/20518 created new session via cli from 172.27.196.47:52582 with ssh
    <INFO> 9-Dec-2021::17:13:57.589 appliance-1 confd[106]: audit user: admin/20518 terminated session (reason: normal)
    <INFO> 9-Dec-2021::17:13:57.633 appliance-1 confd[106]: audit user: admin/20519 assigned to groups: admin
    <INFO> 9-Dec-2021::17:13:57.633 appliance-1 confd[106]: audit user: admin/20519 created new session via cli from 172.27.196.47:52582 with ssh
    <INFO> 9-Dec-2021::18:14:14.380 appliance-1 confd[106]: audit user: admin/20519 terminated session (reason: timeout)
    <INFO> 9-Dec-2021::18:19:38.135 appliance-1 confd[106]: audit user: admin/0 external authentication succeeded via rest from 172.18.3.162:0 with http, member of groups: admin
    <INFO> 9-Dec-2021::18:19:38.135 appliance-1 confd[106]: audit user: admin/0 logged in via rest from 172.18.3.162:0 with http using external authentication
    <INFO> 9-Dec-2021::18:19:38.136 appliance-1 confd[106]: audit user: admin/21353 assigned to groups: admin
    <INFO> 9-Dec-2021::18:19:38.136 appliance-1 confd[106]: audit user: admin/21353 created new session via rest from 172.18.3.162:0 with http
    <INFO> 9-Dec-2021::18:19:38.136 appliance-1 confd[106]: audit user: admin/21353 RESTCONF: request with http: GET /restconf/ HTTP/1.1
    <INFO> 9-Dec-2021::18:19:38.137 appliance-1 confd[106]: audit user: admin/21353 terminated session (reason: normal)
    <INFO> 9-Dec-2021::18:19:38.137 appliance-1 confd[106]: audit user: admin/21353 RESTCONF: response with http: HTTP/1.1 /restconf/ 200 duration 62361 ms


There are options to manipulate the output of the file. Add **| ?** to the command to see the options available to manipulate the file output.

.. code-block:: bash

    r10900# file show log/system/audit.log | ?
    Possible completions:
    append    Append output text to a file
    begin     Begin with the line that matches
    count     Count the number of lines in the output
    exclude   Exclude lines that match
    include   Include lines that match
    linnum    Enumerate lines in the output
    more      Paginate output
    nomore    Suppress pagination
    save      Save output text to a file
    until     End with the line that matches
    r10900# file show log/system/audit.log | 

There are other file options that allow the user to tail the log file using **file tail -f** for a live tail,  or **file tail -n <number of lines>** to view a specific number of the most recent lines.

.. code-block:: bash

    r10900# file tail -f log/system/audit.log
    <INFO> 7-Dec-2022::15:05:01.996 appliance-1 confd[125]: audit user: admin/13692368 assigned to groups: admin
    <INFO> 7-Dec-2022::15:05:01.996 appliance-1 confd[125]: audit user: admin/13692368 created new session via cli from 172.18.104.73:60301 with ssh
    <INFO> 7-Dec-2022::15:05:02.007 appliance-1 confd[125]: audit user: admin/13692368 CLI 'show system state hostname'
    <INFO> 7-Dec-2022::15:05:02.008 appliance-1 confd[125]: audit user: admin/13692368 CLI done
    <INFO> 7-Dec-2022::15:05:02.009 appliance-1 confd[125]: audit user: admin/13692368 terminated session (reason: normal)
    <INFO> 7-Dec-2022::15:05:02.052 appliance-1 confd[125]: audit user: admin/13692371 assigned to groups: admin
    <INFO> 7-Dec-2022::15:05:02.053 appliance-1 confd[125]: audit user: admin/13692371 created new session via cli from 172.18.104.73:60301 with ssh
    <INFO> 7-Dec-2022::15:05:19.428 appliance-1 confd[125]: audit user: admin/13692371 CLI 'file show log/system/audit.log'
    <INFO> 7-Dec-2022::15:05:21.784 appliance-1 confd[125]: audit user: admin/13692371 CLI done
    <INFO> 7-Dec-2022::15:08:59.462 appliance-1 confd[125]: audit user: admin/13692371 CLI 'file tail -f log/system/audit.log'



    r10900# file tail -n 20 log/system/audit.log
    <INFO> 7-Dec-2022::14:46:50.546 appliance-1 confd[125]: audit user: admin/13672920 RESTCONF: response with http: HTTP/1.1 /restconf/ 200 duration 37668 ms
    <INFO> 7-Dec-2022::14:47:05.976 appliance-1 confd[125]: audit user: admin/0 external token authentication succeeded via rest from 172.18.104.73:0 with http, member of groups: admin session-id:admin1670421700
    <INFO> 7-Dec-2022::14:47:05.976 appliance-1 confd[125]: audit user: admin/0 logged in via rest from 172.18.104.73:0 with http using externalvalidation authentication
    <INFO> 7-Dec-2022::14:47:05.976 appliance-1 confd[125]: audit user: admin/13673201 assigned to groups: admin
    <INFO> 7-Dec-2022::14:47:05.976 appliance-1 confd[125]: audit user: admin/13673201 created new session via rest from 172.18.104.73:0 with http
    <INFO> 7-Dec-2022::14:47:05.977 appliance-1 confd[125]: audit user: admin/13673201 RESTCONF: request with http: GET /restconf/ HTTP/1.1
    <INFO> 7-Dec-2022::14:47:05.980 appliance-1 confd[125]: audit user: admin/13673201 terminated session (reason: normal)
    <INFO> 7-Dec-2022::14:47:05.981 appliance-1 confd[125]: audit user: admin/13673201 RESTCONF: response with http: HTTP/1.1 /restconf/ 200 duration 35923 ms
    <INFO> 7-Dec-2022::15:05:01.996 appliance-1 confd[125]: audit user: admin/13692368 assigned to groups: admin
    <INFO> 7-Dec-2022::15:05:01.996 appliance-1 confd[125]: audit user: admin/13692368 created new session via cli from 172.18.104.73:60301 with ssh
    <INFO> 7-Dec-2022::15:05:02.007 appliance-1 confd[125]: audit user: admin/13692368 CLI 'show system state hostname'
    <INFO> 7-Dec-2022::15:05:02.008 appliance-1 confd[125]: audit user: admin/13692368 CLI done
    <INFO> 7-Dec-2022::15:05:02.009 appliance-1 confd[125]: audit user: admin/13692368 terminated session (reason: normal)
    <INFO> 7-Dec-2022::15:05:02.052 appliance-1 confd[125]: audit user: admin/13692371 assigned to groups: admin
    <INFO> 7-Dec-2022::15:05:02.053 appliance-1 confd[125]: audit user: admin/13692371 created new session via cli from 172.18.104.73:60301 with ssh
    <INFO> 7-Dec-2022::15:05:19.428 appliance-1 confd[125]: audit user: admin/13692371 CLI 'file show log/system/audit.log'
    <INFO> 7-Dec-2022::15:05:21.784 appliance-1 confd[125]: audit user: admin/13692371 CLI done
    <INFO> 7-Dec-2022::15:08:59.462 appliance-1 confd[125]: audit user: admin/13692371 CLI 'file tail -f log/system/audit.log'
    <INFO> 7-Dec-2022::15:09:22.907 appliance-1 confd[125]: audit user: admin/13692371 CLI done
    <INFO> 7-Dec-2022::15:09:31.142 appliance-1 confd[125]: audit user: admin/13692371 CLI 'file tail -n 20 log/system/audit.log' 

Within the bash shell if you are logged in as root, the path for the logging is different; **/var/F5/system/log**. Note that older audit.log files are gzipped and rotated.

.. code-block:: bash

    [root@appliance-1(r10900.f5demo.net) ~]# ls -al /var/F5/system/log/
    total 2541432
    drwxr-xr-x.  4 root root       4096 Dec  6 20:14 .
    drwxr-xr-x. 26 root root       4096 Nov 28 12:38 ..
    -rw-r--r--.  1 root root   71290161 Dec  7 10:10 audit.log
    -rw-r--r--.  1 root root    1743543 Dec  9  2021 audit.log.1
    -rw-r--r--.  1 root root         20 Dec  7  2021 audit.log.2.gz
    -rw-r--r--.  1 root root         20 Dec  7  2021 audit.log.3.gz
    -rw-r--r--.  1 root root    1847232 Dec  7  2021 audit.log.4.gz
    -rw-r--r--.  1 root root    9848782 Nov 28 12:35 confd.log
    -rw-r--r--.  1 root root      29979 Dec  9  2021 confd.log.1
    -rw-r--r--.  1 root root         20 Dec  7  2021 confd.log.2.gz
    -rw-r--r--.  1 root root         20 Dec  7  2021 confd.log.3.gz
    -rw-r--r--.  1 root root      33306 Dec  7  2021 confd.log.4.gz
    -rw-r--r--.  1 root root   81663088 Dec  7 10:10 devel.log
    -rw-r--r--.  1 root root  104858977 Nov 13 15:11 devel.log.1
    -rw-r--r--.  1 root root    4541548 Oct 14 02:37 devel.log.2.gz
    -rw-r--r--.  1 root root    4838903 Aug 23 01:14 devel.log.3.gz
    -rw-r--r--.  1 root root    4747221 Jun 22 18:45 devel.log.4.gz
    -rw-r--r--.  1 root root    4788922 Apr 13  2022 devel.log.5.gz
    -rw-r--r--.  1 root root   24263778 Nov 28 13:40 k3s_events.log
    -rw-r--r--.  1 root root  105344182 Nov 28 12:54 k3s_events.log.1
    -rw-r--r--.  1 root root    8073081 Sep 19 11:30 k3s_events.log.2.gz
    -rw-r--r--.  1 root root   68972233 Jan 23  2022 lacp_out_132
    -rw-r--r--.  1 root root   50821845 Dec  7 10:10 lcd.log
    -rw-r--r--.  1 root root  104858247 Oct  6 23:13 lcd.log.1
    -rw-r--r--.  1 root root    6501076 Jun 27 10:24 lcd.log.2.gz
    -rw-r--r--.  1 root root    6518411 Jun  8 00:41 lcd.log.3.gz
    -rw-r--r--.  1 root root    6541114 May 19  2022 lcd.log.4.gz
    -rw-r--r--.  1 root root    6561702 Apr 22  2022 lcd.log.5.gz
    -rw-r--r--.  1 root root    1909130 Dec  7 10:10 logrotate.log
    -rw-r--r--.  1 root root    5244641 Dec  6 20:14 logrotate.log.1
    -rw-r--r--.  1 root root      31197 Dec  5 05:57 logrotate.log.2.gz
    -rw-r--r--.  1 root root  607087556 Dec  7 10:09 platform.log
    -rw-r--r--.  1 root root 1073833624 Jan 12  2022 platform.log.1
    -rw-r--r--.  1 root root   60136728 Jan  4  2022 platform.log.2.gz
    -rw-r--r--.  1 root root     454400 Dec  8  2021 platform.log.3.gz
    -rw-r--r--.  1 root root        621 Dec  7  2021 platform.log.4.gz
    -rw-r--r--.  1 root root       7841 Dec  7  2021 platform.log.5.gz
    -rw-r--r--.  1 root root       7734 Dec  7  2021 platform.log.6.gz
    -rw-r--r--.  1 root root  152724547 Dec  7  2021 platform.log.7.gz
    -rw-r--r--.  1 root root          0 Sep 30  2021 reprogram_chassis_network.log
    -rw-r--r--.  1 root root      41122 Nov 28 12:34 rsyslogd_init.log
    -rw-r--r--.  1 root root   16070999 Dec  5 23:48 snmp.log
    -rw-r--r--.  1 root root          0 Dec  9  2021 snmp.log.1
    -rw-r--r--.  1 root root         20 Dec  7  2021 snmp.log.2.gz
    -rw-r--r--.  1 root root         20 Dec  7  2021 snmp.log.3.gz
    -rw-r--r--.  1 root root         20 Dec  7  2021 snmp.log.4.gz
    -rw-r--r--.  1 root root        435 Nov 28 12:34 startup.log
    -rw-r--r--.  1 root root        190 Nov 28 12:27 startup.log.prev
    drwxr-xr-x.  2 root root       4096 Sep 28  2021 trace
    -rw-r--r--.  1 root root       8424 Nov 28 12:34 vconsole_auth.log
    -rw-r--r--.  1 root root      31966 Nov 28 12:34 vconsole_startup.log
    -rw-r--r--.  1 root root          0 Dec  9  2021 velos.log
    -rw-r--r--.  1 root root          0 Dec  7  2021 velos.log.1
    -rw-r--r--.  1 root root         20 Dec  7  2021 velos.log.2.gz
    -rw-r--r--.  1 root root    5960344 Oct 18  2021 velos.log.3.gz
    -rw-r--r--.  1 root root       4096 Oct 15  2021 .velos.log.swp
    drwxr-xr-x.  2 root root       4096 Nov 28 12:34 webui
    [root@appliance-1(r10900.f5demo.net) ~]# 
  
Viewing Logs from the webUI
--------------------------

In the current F5OS releases, you cannot view the F5OS audit.log file directly from the webUI, although you can download it from the webUI. To view the audit.log, you can use the CLI or API, or download the files and then view. To download log files from the webUI, go to the **System Settings -> File Utilities** page. Here there are various logs directories you can download files from. You have the option to **Export** files to a remote HTTPS server, or **Download** the files directly to your client machine through the browser.

.. image:: images/rseries_security/image10.png
  :align: center
  :scale: 70%

If you want to download the main **audit.log**, select the directory **/log/system**.


.. image:: images/rseries_security/image11.png
  :align: center
  :scale: 70%


Viewing Audit Logs via F5OS API
-------------------------------

Example Audit Logging for Login, Logout, Login Failure, and Account Lockout
---------------------------------------------------------------------------


Below are examples seen on a remote syslog server for various login, logout, login failure, and account lockout events for F5OS. These examples are based on F5OS-A 1.4.0 or later.

----------------
Login Audit Logs
----------------

Below is an example of a client logging into the F5OS webUI. Note that the logs identify which user has logged in as well as what IP address they have logged in from.

.. code-block:: bash


    2023-01-06T16:56:17.475631-05:00 appliance-1 audit-service[12]: priority="Notice" version=1.0 msgid=0x1f03000000000002 msg="audit" user="admin/0" cmd="external token authentication succeeded via rest from 172.18.104.40:0 with http, member of groups: admin session-id:admin1673042162".
    2023-01-06T16:56:17.475645-05:00 appliance-1 audit-service[12]: priority="Notice" version=1.0 msgid=0x1f03000000000002 msg="audit" user="admin/0" cmd="logged in via rest from 172.18.104.40:0 with http using externalvalidation authentication".
    2023-01-06T16:56:17.475855-05:00 appliance-1 audit-service[12]: priority="Notice" version=1.0 msgid=0x1f03000000000002 msg="audit" user="admin/14985364" cmd="assigned to groups: admin".
    2023-01-06T16:56:17.476077-05:00 appliance-1 audit-service[12]: priority="Notice" version=1.0 msgid=0x1f03000000000002 msg="audit" user="admin/14985364" cmd="created new session via rest from 172.18.104.40:0 with http".
    2023-01-06T16:56:17.477346-05:00 appliance-1 audit-service[12]: priority="Info" version=1.0 msgid=0x1f03000000000001 msg="audit" user="admin/14985364" cmd="RESTCONF: request with http: GET /restconf/ HTTP/1.1".
    2023-01-06T16:56:17.479784-05:00 appliance-1 audit-service[12]: priority="Notice" version=1.0 msgid=0x1f03000000000002 msg="audit" user="admin/14985364" cmd="terminated session (reason: normal)".
    2023-01-06T16:56:17.480672-05:00 appliance-1 audit-service[12]: priority="Info" version=1.0 msgid=0x1f03000000000001 msg="audit" user="admin/14985364" cmd="RESTCONF: response with http: HTTP/1.1 /restconf/ 200 duration 50243 ms".

Below is an example of a client logging into the F5OS CLI. Note that the logs identify which user has logged in as well as what IP address they have logged in from.

.. code-block:: bash

    2023-01-06T17:06:57.717699-05:00 appliance-1 audit-service[12]: priority="Notice" version=1.0 msgid=0x1f03000000000002 msg="audit" user="admin/15001237" cmd="assigned to groups: admin".
    2023-01-06T17:06:57.717714-05:00 appliance-1 audit-service[12]: priority="Notice" version=1.0 msgid=0x1f03000000000002 msg="audit" user="admin/15001237" cmd="created new session via cli from 172.18.104.40:61769 with ssh".
    2023-01-06T17:06:57.728956-05:00 appliance-1 audit-service[12]: priority="Info" version=1.0 msgid=0x1f03000000000001 msg="audit" user="admin/15001237" cmd="CLI 'show system state hostname'".
    2023-01-06T17:06:57.730238-05:00 appliance-1 audit-service[12]: priority="Info" version=1.0 msgid=0x1f03000000000001 msg="audit" user="admin/15001237" cmd="CLI done".
    2023-01-06T17:06:57.732901-05:00 appliance-1 audit-service[12]: priority="Notice" version=1.0 msgid=0x1f03000000000002 msg="audit" user="admin/15001237" cmd="terminated session (reason: normal)".
    2023-01-06T17:06:57.775152-05:00 appliance-1 audit-service[12]: priority="Notice" version=1.0 msgid=0x1f03000000000002 msg="audit" user="admin/15001242" cmd="assigned to groups: admin".
    2023-01-06T17:06:57.775243-05:00 appliance-1 audit-service[12]: priority="Notice" version=1.0 msgid=0x1f03000000000002 msg="audit" user="admin/15001242" cmd="created new session via cli from 172.18.104.40:61769 with ssh".

Below is an example of a client authentication to the F5OS REST API. Note that the logs identify which user has accessed the API as well as what IP address they have sent the request from.

.. code-block:: bash

    2023-01-06T17:09:23.296905-05:00 appliance-1 audit-service[12]: priority="Notice" version=1.0 msgid=0x1f03000000000002 msg="audit" user="admin/0" cmd="external authentication succeeded via rest from 172.18.104.40:0 with http, member of groups: admin session-id:admin1673042963".
    2023-01-06T17:09:23.296919-05:00 appliance-1 audit-service[12]: priority="Notice" version=1.0 msgid=0x1f03000000000002 msg="audit" user="admin/0" cmd="logged in via rest from 172.18.104.40:0 with http using external authentication".
    2023-01-06T17:09:23.296926-05:00 appliance-1 audit-service[12]: priority="Notice" version=1.0 msgid=0x1f03000000000002 msg="audit" user="admin/15004795" cmd="assigned to groups: admin".
    2023-01-06T17:09:23.296969-05:00 appliance-1 audit-service[12]: priority="Notice" version=1.0 msgid=0x1f03000000000002 msg="audit" user="admin/15004795" cmd="created new session via rest from 172.18.104.40:0 with http".
    2023-01-06T17:09:23.297161-05:00 appliance-1 audit-service[12]: priority="Info" version=1.0 msgid=0x1f03000000000001 msg="audit" user="admin/15004795" cmd="RESTCONF: request with http: GET /restconf/data/openconfig-system:system/aaa HTTP/1.1".
    2023-01-06T17:09:23.389320-05:00 appliance-1 audit-service[12]: priority="Info" version=1.0 msgid=0x1f03000000000001 msg="audit" user="admin/15004795" cmd="RESTCONF: response with http: HTTP/1.1 /restconf/data/openconfig-system:system/aaa 200 duration 151730 ms".
    2023-01-06T17:09:23.390141-05:00 appliance-1 audit-service[12]: priority="Notice" version=1.0 msgid=0x1f03000000000002 msg="audit" user="admin/15004795" cmd="terminated session (reason: normal)".

----------------
Logout Audit Logs
----------------

Below is an example of a client logging out of the F5OS CLI. Note that the logs identify which user has logged out as well as what IP address they have logged out from.

.. code-block:: bash

    2023-01-06T17:16:05.536108-05:00 appliance-1 audit-service[12]: priority="Info" version=1.0 msgid=0x1f03000000000001 msg="audit" user="admin/15014425" cmd="CLI 'logout'".
    2023-01-06T17:16:05.736047-05:00 appliance-1 audit-service[12]: priority="Notice" version=1.0 msgid=0x1f03000000000002 msg="audit" user="admin/15014425" cmd="terminated session (reason: normal)".

Below is an example of a client logging out of the F5OS webUI. Note that the logs identify which user has logged out as well as what IP address they have logged out from.

**Do we log logout events from GUI?**

--------------------------
Account Lockout Audit Logs
--------------------------

In order to capture all events related to account lockout, you will need to configure F5OS to send both standard syslog events as well as host audit log events to a remote server. This is because some of the audit events related to account lockout are captured before the F5OS layer in the host/audit.log and by default that log is not sent remotely.

To forward the contents of the host audit logs add **config files file audit/audit.log** to the **system logging host-logs** configuration as seen below.

.. code-block:: bash

    default-1# show running-config system logging host-logs
    system logging host-logs
    config remote-forwarding enabled
    config selectors selector AUTHPRIV DEBUG
    config files file audit/audit.log
    !

In addtion, you'll want to ensure that **selectors selector AUTHPRIV INFORMATIONAL** is added to the **system logging remote-servers** configuration for your configured syslog location.

.. code-block:: bash

    default-1# show running-config system logging remote-servers
    system logging remote-servers remote-server 10.255.85.182
    config remote-port 514
    config proto udp
    selectors selector LOCAL0 DEBUG
    selectors selector AUTHPRIV INFORMATIONAL
    !

Below is remote syslog example of a client logging into the F5OS CLI and entering an invalid password multiple times, resulting in an account lockout event. The configured password-policy for **max-login-failures** has been set to two, meaning once the client issues two invalid passwords the account will be temporarily locked for the **unlock-time** of sixty seconds. 


.. code-block:: bash

    r10900-1# show running-config system aaa password-policy 
    system aaa password-policy config min-length 6
    system aaa password-policy config required-numeric 0
    system aaa password-policy config required-uppercase 0
    system aaa password-policy config required-lowercase 0
    system aaa password-policy config required-special 0
    system aaa password-policy config required-differences 8
    system aaa password-policy config reject-username true
    system aaa password-policy config apply-to-root true
    system aaa password-policy config retries 3
    system aaa password-policy config max-login-failures 2
    system aaa password-policy config unlock-time 60
    system aaa password-policy config root-lockout true
    system aaa password-policy config root-unlock-time 60
    system aaa password-policy config max-age 0
    r10900-1#

In the logs below, a local user **testuser** has entered two consecutive bad passwords resulting in a temporary lock of the account.

.. code-block:: bash

    2023-02-21T12:23:10.495053-05:00 appliance-1 unix_chkpwd[45741]:  password check failed for user (testuser)
    2023-02-21T12:23:10.495481-05:00 appliance-1 sshd[45026]:  pam_unix(sshd:auth): authentication failure; logname= uid=0 euid=0 tty=ssh ruser= rhost=172.18.105.83  user=testuser
    2023-02-21T12:23:18.298137-05:00 appliance-1 unix_chkpwd[46717]:  password check failed for user (testuser)
    2023-02-21T12:23:18.298942-05:00 appliance-1 sshd[45026]:  pam_faillock(sshd:auth): Consecutive login failures for user testuser account temporarily locked
    2023-02-21T12:23:20.223386-05:00 appliance-1 sshd[46957]:  pam_unix(sshd:session): session opened for user root by (uid=0)
    2023-02-21T12:23:20.274338-05:00 appliance-1 sshd[46957]:  pam_unix(sshd:session): session closed for user root
    2023-02-21T12:23:20.416710-05:00 appliance-1 HOST-audit/audit.log:  type=RESP_ACCT_UNLOCK_TIMED msg=audit(1677000190.495:250): pid=45026 uid=0 auid=4294967295 ses=4294967295 subj=system_u:system_r:sshd_t:s0-s0:c0.c1023 msg='pam_faillock uid=1003  exe="/usr/sbin/sshd" hostname=172.18.105.83 addr=172.18.105.83 terminal=ssh res=success'
    2023-02-21T12:23:20.416724-05:00 appliance-1 HOST-audit/audit.log:  type=ANOM_LOGIN_FAILURES msg=audit(1677000198.297:251): pid=45026 uid=0 auid=4294967295 ses=4294967295 subj=system_u:system_r:sshd_t:s0-s0:c0.c1023 msg='pam_faillock uid=1003  exe="/usr/sbin/sshd" hostname=? addr=? terminal=? res=success'
    2023-02-21T12:23:20.416727-05:00 appliance-1 HOST-audit/audit.log:  type=RESP_ACCT_LOCK msg=audit(1677000198.297:252): pid=45026 uid=0 auid=4294967295 ses=4294967295 subj=system_u:system_r:sshd_t:s0-s0:c0.c1023 msg='pam_faillock uid=1003  exe="/usr/sbin/sshd" hostname=? addr=? terminal=? res=success'

Example Audit Logging of Configuration Changes
----------------------------------------------

Below is an example audit log of the user **jim-test** entering config mode via the CLI and then changing the description for interface 20.0 and then committing the change.

.. code-block:: bash

    2023-01-06T17:44:16.790917-05:00 appliance-1 audit-service[12]: priority="Info" version=1.0 msgid=0x1f03000000000001 msg="audit" user="jim-test/15056017" cmd="CLI 'config'".
    2023-01-06T17:44:16.791664-05:00 appliance-1 audit-service[12]: priority="Info" version=1.0 msgid=0x1f03000000000001 msg="audit" user="jim-test/15056017" cmd="CLI done".
    2023-01-06T17:44:54.864806-05:00 appliance-1 audit-service[12]: priority="Info" version=1.0 msgid=0x1f03000000000001 msg="audit" user="jim-test/15056017" cmd="CLI 'interfaces interface 20.0 config description "This is a test"'".
    2023-01-06T17:44:54.864822-05:00 appliance-1 audit-service[12]: priority="Info" version=1.0 msgid=0x1f03000000000001 msg="audit" user="jim-test/15056017" cmd="CLI done".
    2023-01-06T17:44:59.392050-05:00 appliance-1 audit-service[12]: priority="Info" version=1.0 msgid=0x1f03000000000001 msg="audit" user="jim-test/15056017" cmd="CLI 'commit'".
    2023-01-06T17:44:59.412077-05:00 appliance-1 audit-service[12]: priority="Notice" version=1.0 msgid=0x1f03000000000005 msg="audit modify" ctx="CLI" user="jim-test/15056017" path="/interfaces/interface{20.0}".
    2023-01-06T17:44:59.412156-05:00 appliance-1 audit-service[12]: priority="Notice" version=1.0 msgid=0x1f03000000000006 msg="audit value set" ctx="CLI" user="jim-test/15056017" path="/interfaces/interface{20.0}/config/description" value="This is a test".
    2023-01-06T17:44:59.413541-05:00 appliance-1 audit-service[12]: priority="Info" version=1.0 msgid=0x1f03000000000001 msg="audit" user="jim-test/15056017" cmd="CLI done".

Example Audit Logging of webUI Changes
--------------------------------------

Below is an example audit log of the user **jim-test** using the webUI and then changing the VLAN membership for interface 20.0 and then committing the change.

.. code-block:: bash

    2023-01-06T17:50:43.011201-05:00 appliance-1 audit-service[12]: priority="Notice" version=1.0 msgid=0x1f03000000000002 msg="audit" user="jim-test/0" cmd="external token authentication succeeded via rest from 172.18.104.40:0 with http, member of groups: admin session-id:jim-test1673045408".
    2023-01-06T17:50:43.011215-05:00 appliance-1 audit-service[12]: priority="Notice" version=1.0 msgid=0x1f03000000000002 msg="audit" user="jim-test/0" cmd="logged in via rest from 172.18.104.40:0 with http using externalvalidation authentication".
    2023-01-06T17:50:43.014087-05:00 appliance-1 audit-service[12]: priority="Notice" version=1.0 msgid=0x1f03000000000002 msg="audit" user="jim-test/15065552" cmd="assigned to groups: admin".
    2023-01-06T17:50:43.014185-05:00 appliance-1 audit-service[12]: priority="Notice" version=1.0 msgid=0x1f03000000000002 msg="audit" user="jim-test/15065552" cmd="created new session via rest from 172.18.104.40:0 with http".
    2023-01-06T17:50:43.014470-05:00 appliance-1 audit-service[12]: priority="Info" version=1.0 msgid=0x1f03000000000001 msg="audit" user="jim-test/15065552" cmd="RESTCONF: request with http: GET /restconf/ HTTP/1.1".
    2023-01-06T17:50:43.015325-05:00 appliance-1 audit-service[12]: priority="Notice" version=1.0 msgid=0x1f03000000000002 msg="audit" user="jim-test/15065552" cmd="terminated session (reason: normal)".
    2023-01-06T17:50:43.016496-05:00 appliance-1 audit-service[12]: priority="Info" version=1.0 msgid=0x1f03000000000001 msg="audit" user="jim-test/15065552" cmd="RESTCONF: response with http: HTTP/1.1 /restconf/ 200 duration 42906 ms".
    2023-01-06T17:50:46.109658-05:00 appliance-1 audit-service[12]: priority="Notice" version=1.0 msgid=0x1f03000000000002 msg="audit" user="jim-test/0" cmd="external token authentication succeeded via rest from 172.18.104.40:0 with http, member of groups: admin session-id:jim-test1673045408".
    2023-01-06T17:50:46.110048-05:00 appliance-1 audit-service[12]: priority="Notice" version=1.0 msgid=0x1f03000000000002 msg="audit" user="jim-test/0" cmd="logged in via rest from 172.18.104.40:0 with http using externalvalidation authentication".
    2023-01-06T17:50:46.110850-05:00 appliance-1 audit-service[12]: priority="Notice" version=1.0 msgid=0x1f03000000000002 msg="audit" user="jim-test/15065637" cmd="assigned to groups: admin".
    2023-01-06T17:50:46.110956-05:00 appliance-1 audit-service[12]: priority="Notice" version=1.0 msgid=0x1f03000000000002 msg="audit" user="jim-test/15065637" cmd="created new session via rest from 172.18.104.40:0 with http".
    2023-01-06T17:50:46.111225-05:00 appliance-1 audit-service[12]: priority="Info" version=1.0 msgid=0x1f03000000000001 msg="audit" user="jim-test/15065637" cmd="RESTCONF: request with http: PUT /restconf/data/openconfig-interfaces:interfaces/interface=19.0/openconfig-if-ethernet:ethernet/openconfig-vlan:switched-vlan HTTP/1.1".
    2023-01-06T17:50:46.146634-05:00 appliance-1 audit-service[12]: priority="Notice" version=1.0 msgid=0x1f03000000000005 msg="audit modify" ctx="REST" user="jim-test/15065637" path="/interfaces/interface{19.0}".
    2023-01-06T17:50:46.146728-05:00 appliance-1 audit-service[12]: priority="Notice" version=1.0 msgid=0x1f03000000000003 msg="audit create" ctx="REST" user="jim-test/15065637" path="/interfaces/interface{19.0}/ethernet/switched-vlan/config/trunk-vlans{503}".
    2023-01-06T17:50:46.147281-05:00 appliance-1 audit-service[12]: priority="Notice" version=1.0 msgid=0x1f03000000000002 msg="audit" user="jim-test/15065637" cmd="terminated session (reason: normal)".
    2023-01-06T17:50:46.148887-05:00 appliance-1 audit-service[12]: priority="Info" version=1.0 msgid=0x1f03000000000001 msg="audit" user="jim-test/15065637" cmd="RESTCONF: response with http: HTTP/1.1 /restconf/data/openconfig-interfaces:interfaces/interface=19.0/openconfig-if-ethernet:ethernet/openconfig-vlan:switched-vlan 204 duration 69082 ms".
    2023-01-06T17:50:46.207531-05:00 appliance-1 audit-service[12]: priority="Notice" version=1.0 msgid=0x1f03000000000002 msg="audit" user="jim-test/0" cmd="external token authentication succeeded via rest from 172.18.104.40:0 with http, member of groups: admin session-id:jim-test1673045408".
    2023-01-06T17:50:46.207564-05:00 appliance-1 audit-service[12]: priority="Notice" version=1.0 msgid=0x1f03000000000002 msg="audit" user="jim-test/0" cmd="logged in via rest from 172.18.104.40:0 with http using externalvalidation authentication".
    2023-01-06T17:50:46.208310-05:00 appliance-1 audit-service[12]: priority="Notice" version=1.0 msgid=0x1f03000000000002 msg="audit" user="jim-test/15065642" cmd="assigned to groups: admin".
    2023-01-06T17:50:46.208414-05:00 appliance-1 audit-service[12]: priority="Notice" version=1.0 msgid=0x1f03000000000002 msg="audit" user="jim-test/15065642" cmd="created new session via rest from 172.18.104.40:0 with http".
    2023-01-06T17:50:46.208908-05:00 appliance-1 audit-service[12]: priority="Info" version=1.0 msgid=0x1f03000000000001 msg="audit" user="jim-test/15065642" cmd="RESTCONF: request with http: GET /restconf/data/openconfig-interfaces:interfaces HTTP/1.1".
    2023-01-06T17:50:46.404290-05:00 appliance-1 audit-service[12]: priority="Info" version=1.0 msgid=0x1f03000000000001 msg="audit" user="jim-test/15065642" cmd="RESTCONF: response with http: HTTP/1.1 /restconf/data/openconfig-interfaces:interfaces 200 duration 227159 ms".
    2023-01-06T17:50:46.404731-05:00 appliance-1 audit-service[12]: priority="Notice" version=1.0 msgid=0x1f03000000000002 msg="audit" user="jim-test/15065642" cmd="terminated session (reason: normal)".

Example Audit Logging of API Changes
------------------------------------

Below is an example audit log of the user **admin** using the API and then adding a new VLAN to the configuration. In F5OS release prior to F5OS-A 1.4.0 API audit logs captured configuration changes, but did not log the full configuration payload. 

.. code-block:: bash


    2023-02-17T17:28:10.290541-05:00 appliance-1 audit-service[11]: priority="Notice" version=1.0 msgid=0x1f03000000000002 msg="audit" user="admin/3104052" cmd="created new session via rest from 172.18.104.20:0 with http".
    2023-02-17T17:28:10.290769-05:00 appliance-1 audit-service[11]: priority="Info" version=1.0 msgid=0x1f03000000000001 msg="audit" user="admin/3104052" cmd="RESTCONF: request with http: PATCH /restconf/data/openconfig-vlan:vlans HTTP/1.1".
    2023-02-17T17:28:10.308174-05:00 appliance-1 audit-service[11]: priority="Notice" version=1.0 msgid=0x1f03000000000003 msg="audit create" ctx="REST" user="admin/3104052" path="/vlans/vlan{600}".
    2023-02-17T17:28:10.308217-05:00 appliance-1 audit-service[11]: priority="Notice" version=1.0 msgid=0x1f03000000000006 msg="audit value set" ctx="REST" user="admin/3104052" path="/vlans/vlan{600}/vlan-id" value="600".
    2023-02-17T17:28:10.308280-05:00 appliance-1 audit-service[11]: priority="Notice" version=1.0 msgid=0x1f03000000000006 msg="audit value set" ctx="REST" user="admin/3104052" path="/vlans/vlan{600}/config/vlan-id" value="600".
    2023-02-17T17:28:10.308319-05:00 appliance-1 audit-service[11]: priority="Notice" version=1.0 msgid=0x1f03000000000006 msg="audit value set" ctx="REST" user="admin/3104052" path="/vlans/vlan{600}/config/name" value="TEST600-VLAN".
    2023-02-17T17:28:10.308819-05:00 appliance-1 audit-service[11]: priority="Notice" version=1.0 msgid=0x1f03000000000002 msg="audit" user="admin/3104052" cmd="terminated session (reason: normal)".
    2023-02-17T17:28:10.310149-05:00 appliance-1 audit-service[11]: priority="Info" version=1.0 msgid=0x1f03000000000001 msg="audit" user="admin/3104052" cmd="RESTCONF: response with http: HTTP/1.1 /restconf/data/openconfig-vlan:vlans 204 duration 57569 ms".


Downloading Audit Logs via CLI
------------------------------

Audit logs can be sent to a remote server as outlined above, but they can also be downloaded from the system if needed. Before transfering a file using the CLI, use the **file list** command to see the contents of the  directory and ensure the file is there. There are two audit.log locations: **log/system/audit.log** where most of the audit.log events are logged, and **log/host/audit/audit.log** where some lower level events are logged.

The path below is the main audit.log.

.. code-block:: bash

    r10900-1# file list path log/system/audit.log
    entries {
        name audit.log
        date Sat Feb 25 21:38:45 UTC 2023
        size 11MB
    }
    r10900-1#

The path below is for lower level audit log events like account lockouts.

.. code-block:: bash

    r10900-1# file list path log/host/audit/audit.log
    entries {
        name audit.log
        date Thu Feb 23 05:05:14 UTC 2023
        size 50MB
    }
    r10900-1# 

To export copies of these files off the system you can use the **file export** command to transfer the file to a remote HTTPS server, or to a remote server using SFTP, or SCP. Below is an example of transferring the log/system/audit.log to a remote HTTPS server:

.. code-block:: bash

    r10900-1# file export local-file log/system/audit.log remote-host 10.255.0.142 remote-file /upload/upload.php username corpuser insecure
    Value for 'password' (<string>): ********
    result File transfer is initiated.(log/system/audit.log)
    r10900-1#

To check on status of the export use the **file transfer-status** command:

.. code-block:: bash

    r10900-1# file transfer-status 
    result 
    S.No.|Operation  |Protocol|Local File Path                                             |Remote Host         |Remote File Path                                            |Status            |Time                
    1    |Export file|HTTPS   |log/system/audit.log                                        |10.255.0.142        |/upload/upload.php                                          |         Completed|Sat Feb 25 16:46:28 2023

    r10900-1# 

You may also transfer from the CLI using SCP or SFTP protocols. Below is an example using SCP:

.. code-block:: bash

    r10900-1# file export local-file log/system/audit.log remote-host 10.255.0.142 protocol scp insecure remote-file r109001-audit.log username root
    Value for 'password' (<string>): *******
    result File transfer is initiated.(log/system/audit.log)
    r10900-1#

The file transfer-status command will show the upload of the SCP transfer as well as HTTPS or SFTP:

.. code-block:: bash

    r10900-1# file transfer-status
    result 
    S.No.|Operation  |Protocol|Local File Path                                             |Remote Host         |Remote File Path                                            |Status            |Time                
    1    |Export file|HTTPS   |log/system/audit.log                                        |10.255.0.142        |/upload/upload.php                                          |         Completed|Sat Feb 25 16:46:28 2023
    2    |Export file|SCP     |log/system/audit.log                                        |10.255.0.142        |r109001-audit.log                                           |         Completed|Sat Feb 25 16:50:06 2023

    r10900-1# 


Downloading Audit Logs via API
------------------------------

To copy the audit.log files from the appliance to a remote https server use the following API call, you can change the local-file path depending on which audit.log you want to export. Below is an API POST call to export the log/system/audit.log to a remote server.

.. code-block:: bash

    POST https://{{rseries_appliance1_ip}}:8888/restconf/data/f5-utils-file-transfer:file/export

The JSON body of the API call should contain the following syntax.

.. code-block:: json

    {
        "f5-utils-file-transfer:insecure": "",
        "f5-utils-file-transfer:protocol": "https",
        "f5-utils-file-transfer:username": "corpuser",
        "f5-utils-file-transfer:password": "password",
        "f5-utils-file-transfer:remote-host": "10.255.0.142",
        "f5-utils-file-transfer:remote-file": "/upload/upload.php",
        "f5-utils-file-transfer:local-file": "log/system/audit.log"
    }

You can then check on the status of the export via the following API call:

.. code-block:: bash

    POST https://{{rseries_appliance1_ip}}:8888/api/data/f5-utils-file-transfer:file/transfer-status

In the response the latest file trasnfer status will be displayed.

.. code-block:: json

    {
        "f5-utils-file-transfer:output": {
            "result": "\nS.No.|Operation  |Protocol|Local File Path                                             |Remote Host         |Remote File Path                                            |Status            |Time                \n1    |Export file|HTTPS   |log/system/audit.log                                        |10.255.0.142        |/upload/upload.php                                          |         Completed|Sat Feb 25 17:06:00 2023\n2    |Export file|SCP     |log/system/audit.log                                        |10.255.0.142        |r109001-audit.log                                           |         Completed|Sat Feb 25 16:50:06 2023\n"
        }
    }



Downloading Audit Logs via webUI
-------------------------------

You can download either of the audit.log files from the **System -> File Utilities** page in the webUI. In the drop down menu for the **Base Directory** select log/host, and then you can select the audit directory as seen below.  

.. image:: images/rseries_security/imageaudit1.png
  :align: center
  :scale: 70%

Inside the audit directory you can then select the audit.log and then either **Download** to copy the file to you local machine via the browser, or select **Export** to copy to a rmeote HTTPS server.

.. image:: images/rseries_security/imageaudit2.png
  :align: center
  :scale: 70%

You can also select the **log/system** path to download the system audit.log.

.. image:: images/rseries_security/imageaudit3.png
  :align: center
  :scale: 70%