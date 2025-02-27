---
pcx_content_type: how-to
title: Manage custom certificates
weight: 2
---

# Manage custom certificates

This page lists Cloudflare requirements for custom certificates and explains how to upload and update these certificates using Cloudflare dashboard or API.

## Certificate requirements

Before accepting custom certificates, Cloudflare parses them and checks for validity according to a list of requirements.

<details>
<summary>Full list of requirements</summary>
<div> 

Each custom certificate you upload must:

- Be encoded in PEM format (PEM, PKCS#7, or PKCS#12). See [Converting Using OpenSSL](https://www.sslshopper.com/article-most-common-openssl-commands.html) for conversion examples.
- Not have a [key file password](/ssl/edge-certificates/custom-certificates/remove-file-key-password/).
- Not be expiring in less than 14 days from time of upload.
- Have a subject alternative name (SAN) matching at least one hostname in the zone where it is being uploaded.
- Use a private key greater than or equal to a minimum length. Currently, 2048 bit for RSA and 225 bit for ECDSA.
- Be publicly trusted by a major browser. This does not apply for certificates that specify `User Defined` as their [bundling methodology](/ssl/edge-certificates/custom-certificates/bundling-methodologies/).
- Be one of the following certificate types:

  - Unified Communications Certificates (UCC)
  - Extended Validation (EV)
  - Domain Validated (DV)
  - Organization Validated (OV)
</div>
</details>


---

## Upload a custom certificate

{{<tabs labels="Dashboard | API">}}
{{<tab label="dashboard" no-code="true">}}

To upload a custom SSL certificate in the dashboard:

1.  Log in to the [Cloudflare dashboard](https://dash.cloudflare.com) and select your account.

2.  Select your application.

3.  Go to **SSL/TLS**.

4.  In **Edge Certificates**, select **Upload Custom SSL Certificate**.

5.  Copy and paste relevant values into **SSL Certificate** and **Private key** text areas (or select **Paste from file**).

    {{<Aside type="note">}}If doing this manually, include the `---BEGIN CERTIFICATE---` and `---END CERTIFICATE---` like the placeholder text.{{</Aside>}}

6.  Choose the appropriate [**Bundle Method**](/ssl/edge-certificates/custom-certificates/bundling-methodologies/).

7.  Select a value for [**Private Key Restriction**](/ssl/edge-certificates/custom-certificates/#geo-key-manager-private-key-restriction).

8.  Select a value for **Legacy Client Support**, which toggles [Server Name Indication (SNI)](/fundamentals/reference/glossary/#server-name-indication-sni) support:

    - **Modern (recommended)**: SNI only
    - **Legacy**: Supports non-SNI

9.  Select **Upload Custom Certificate**. If you see an error for `The key you provided does not match the certificate`, contact your Certificate Authority to ensure the private key matches the certificate.

10. (optional) [Add a CAA DNS record](/ssl/edge-certificates/caa-records/).
{{</tab>}}

{{<tab label="api" no-code="true">}}

The following call will upload a certificate for use with `app.example.com`. Cloudflare will automatically bundle the certificate with a certificate chain optimized for maximum compatibility with browsers.

{{<Aside type="warning">}}
Note that if you are using an ECC key generated by OpenSSL, you will need to first remove the `-----BEGIN EC PARAMETERS-----...-----END EC PARAMETERS-----` section of the file.
{{</Aside>}}

1. Update the file and build the payload

{{<render file="_custom-cert-file-example.md">}}

```bash
$ request_body=$(< <(cat <<EOF
{
	"certificate": "$MYCERT",
	"private_key": "$MYKEY",
	"bundle_method":"ubiquitous"
}
EOF
))
```

You can optionally add [geographic restrictions](https://blog.cloudflare.com/introducing-cloudflare-geo-key-manager/) that specify where your private key can physically be decrypted:

```bash
$ request_body=$(< <(cat <<EOF
{
	"certificate": "$MYCERT",
	"private_key": "$MYKEY",
	"bundle_method":"ubiquitous",
	"geo_restrictions":{"label":"us"}'
}
EOF
))
```

You can also enable support for legacy clients which do not include SNI in the TLS handshake.

```bash
$ request_body=$(< <(cat <<EOF
{
	"certificate": "$MYCERT",
	"private_key": "$MYKEY",
	"bundle_method":"ubiquitous",
	"geo_restrictions":{"label":"us"}',
	"type":"sni_custom"
}
EOF
))
```

`sni_custom` is recommended by Cloudflare. Use `legacy_custom` when a specific client requires non-SNI support. The Cloudflare API treats all Custom SSL certificates as Legacy by default.

2. Upload your certificate and key

Use the [POST](/api/operations/custom-ssl-for-a-zone-create-ssl-configuration) endpoint to upload your certificate and key.

```bash
$ curl -sX POST https://api.cloudflare.com/client/v4/zones/{zone_id}/custom_certificates \
     -H "X-Auth-Email: {email}" -H "X-Auth-Key: {key}" \
     -H "Content-Type: application/json" -d "$request_body"
```

3. (Optional) Add a CAA record.

{{<render file="_caa-records-definition.md">}}

For more guidance, refer to [Create a CAA record](/ssl/edge-certificates/caa-records/).

{{</tab>}}
{{</tabs>}}

---

## Update an existing custom certificate

Before you update an existing custom certificate, you might want to consider having active [universal](/ssl/edge-certificates/universal-ssl/) or [advanced](/ssl/edge-certificates/advanced-certificate-manager/) certificates as fallback options. Go to [**SSL/TLS** > **Edge Certificates**](https://dash.cloudflare.com/?to=/:account/:zone/ssl-tls/edge-certificates) to check a list of hostnames and status of the edge certificates in your zone.

If you are on an Enterprise plan and want to update a custom (modern) certificate, also consider requesting access to [Staging environment (Beta)](/ssl/edge-certificates/staging-environment/).

{{<tabs labels="Dashboard | API">}}
{{<tab label="dashboard" no-code="true">}}
 
To update a certificate in the dashboard:

1.  Log in to the [Cloudflare dashboard](https://dash.cloudflare.com) and select your account.
2.  Select your application.
3.  Go to **SSL/TLS**.
4.  In **Edge Certificates**, locate a custom certificate.
5.  Select the wrench icon and select **Replace SSL certificate and key**.
6.  Follow the same steps as [upload a new certificate](#upload-a-custom-certificate).
 
{{</tab>}}
{{<tab label="api" no-code="true">}}
 
To update a certificate using the API, send a [`PATCH`](/api/operations/custom-ssl-for-a-zone-edit-ssl-configuration) command.
 
{{</tab>}}
{{</tabs>}}

{{<Aside type="note">}}

To update the **Private Key Restriction** setting of a certificate, delete and re-add the certificate.

{{</Aside>}}
