# sigstore-kms-alibabakms

A [sigstore](https://sigstore.dev) KMS plugin for [Alibaba Cloud KMS](https://www.alibabacloud.com/en/product/kms).

## Table of Contents

- [Installation](#installation)
- [Credentials](#credentials)
- [Usage](#usage)
   - [Key Generation and Management](#key-generation-and-management)
      - [Generate keys](#generate-keys)
      - [Retrieve the Public Key](#retrieve-the-public-key)
   - [Signing and Verification](#signing-and-verification)
      - [Sign an image](#sign-an-image)
      - [Verify an image](#verify-an-image)
- [URI Format](#uri-format)


## Installation

1. Download the latest release from the [Releases](https://github.com/mozillazg/sigstore-kms-alibabakms/releases) page.
2. Install the binary to your system's `PATH`.


## Credentials


The plugin locates Alibaba Cloud credentials by following this order:

1. Environment variables:  
   \- `ALIBABA_CLOUD_ACCESS_KEY_ID`  
   \- `ALIBABA_CLOUD_ACCESS_KEY_SECRET`  
   \- `ALIBABA_CLOUD_SECURITY_TOKEN` (optional)
2. [RAM Roles for Service Accounts (RRSA) OIDC Token](https://www.alibabacloud.com/help/en/container-service-for-kubernetes/latest/use-rrsa-to-enforce-access-control) (using:  
   \- `ALIBABA_CLOUD_ROLE_ARN`  
   \- `ALIBABA_CLOUD_OIDC_PROVIDER_ARN`  
   \- `ALIBABA_CLOUD_OIDC_TOKEN_FILE`)
3. VM metadata (if `ALIBABA_CLOUD_ECS_METADATA` is defined).
4. Local configuration file: `~/.aliyun/config`.
5. VM's metadata server as a fallback.


## Usage

### Key Generation and Management

#### Generate keys:

```shell
cosign generate-key-pair --kms alibabakms://kms.cn-hangzhou.aliyuncs.com//alias/cosign-key
```

<details>

```
$ cosign generate-key-pair --kms alibabakms://kms.cn-hangzhou.aliyuncs.com//alias/cosign-key
Public key written to cosign.pub
```
</details>

#### Retrieve the Public Key

Retrieve the public key directly from KMS:

```shell
cosign public-key --key alibabakms://kms.cn-hangzhou.aliyuncs.com//alias/cosign-key
```

<details>

```
$ cosign public-key --key alibabakms://kms.cn-hangzhou.aliyuncs.com//alias/cosign-key
-----BEGIN PUBLIC KEY-----
MFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAEDHw40rZ/xIEwOp2rKlb+zA80N38h
pvpEsyMEpJUCMLMgAzVm1rj9hJJFU4GoW7YwpgXSzj6cmNQ6+VAscdohhQ==
-----END PUBLIC KEY-----
```
</details>

Or using a key ID and version:

```shell
cosign public-key --key alibabakms://kms.cn-hangzhou.aliyuncs.com//218d6978-***/versions/bb0416cd-2086-4107-a4a1-d2ecba325080
```

<details>

```
$ cosign public-key --key alibabakms://kms.cn-hangzhou.aliyuncs.com//218d6978-***/versions/bb0416cd-2086-4107-a4a1-d2ecba325080
-----BEGIN PUBLIC KEY-----
MFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAEDHw40rZ/xIEwOp2rKlb+zA80N38h
pvpEsyMEpJUCMLMgAzVm1rj9hJJFU4GoW7YwpgXSzj6cmNQ6+VAscdohhQ==
-----END PUBLIC KEY-----
```
</details>


### Signing and Verification

#### Sign an image

Set the key and image digest as environment variables:

```shell
export KEY=alibabakms://kms.cn-hangzhou.aliyuncs.com//218d6978-***/versions/bb0416cd-2086-4107-a4a1-d2ecba325080
export IMAGE=registry.cn-beijing.aliyuncs.com/test-***/dummy:0.1.0
export IMAGE_DIGEST=$IMAGE@sha256:b12c7d46bc14b4260b9e42714688e2dbf5dee973b291a4c12e0e1539404d9f1d
```

Then sign the image:

```shell
cosign sign --key $KEY $IMAGE_DIGEST
```

Follow the prompts regarding the transparency log and confirmation details.
<details>

```
$ cosign sign --key $KEY $IMAGE_DIGEST
WARNING: "registry.cn-beijing.aliyuncs.com/test-***/dummy" appears to be a private repository, please confirm uploading to the transparency log at "https://rekor.sigstore.dev"
Are you sure you would like to continue? [y/N] y

        The sigstore service, hosted by sigstore a Series of LF Projects, LLC, is provided pursuant to the Hosted Project Tools Terms of Use, available at https://lfprojects.org/policies/hosted-project-tools-terms-of-use/.
        Note that if your submission includes personal data associated with this signed artifact, it will be part of an immutable record.
        This may include the email address associated with the account with which you authenticate your contractual Agreement.
        This information will be used for signing this artifact and will be stored in public transparency logs and cannot be removed later, and is subject to the Immutable Record notice at https://lfprojects.org/policies/hosted-project-tools-immutable-records/.

By typing 'y', you attest that (1) you are not submitting the personal data of any other person; and (2) you understand and agree to the statement and the Agreement terms at the URLs listed above.
Are you sure you would like to continue? [y/N] y
tlog entry created with index: 173545498
Pushing signature to: registry.cn-beijing.aliyuncs.com/test-***/dummy
```
</details>

#### Verify an image

Verify the signature using the public key:

```shell
cosign verify --key $KEY $IMAGE_DIGEST
```

<details>

```
$ cosign verify --key $KEY $IMAGE_DIGEST
Verification for registry.cn-beijing.aliyuncs.com/test-***/dummy@sha256:b12c7d46bc14b4260b9e42714688e2dbf5dee973b291a4c12e0e1539404d9f1d --
The following checks were performed on each of these signatures:
  - The cosign claims were validated
  - Existence of the claims in the transparency log was verified offline
  - The signatures were verified against the specified public key

[{"critical":{"identity":{"docker-reference":"registry.cn-beijing.aliyuncs.com/test-***/dummy"},"image":{"docker-manifest-digest":"sha256:b12c7d46bc14b4260b9e42714688e2dbf5dee973b291a4c12e0e1539404d9f1d"},"type":"cosign container image signature"},"optional":{"Bundle":{"SignedEntryTimestamp":"MEUCIAnDYUDBm3ufG7sN/D1dJLupYLhBkUcY6H3RTlYXxdAEAiEAikDsai+TsSU0PGC/gcds4ooDY/tmk6s+ZU5ttitf4g4=","Payload":{"body":"eyJhcGlWZXJzaW9uIjoiMC4wLjEiLCJraW5kIjoiaGFzaGVkcmVrb3JkIiwic3BlYyI6eyJkYXRhIjp7Imhhc2giOnsiYWxnb3JpdGhtIjoic2hhMjU2IiwidmFsdWUiOiI2MzE5Yjg1NzhkNWM5MjE2NmY0MzNmNzZjODg4NzZkOTA4Y2M2ZDgxOWFkMjYxODEzNGQ5NTcwYmFmNjhiYzg1In19LCJzaWduYXR1cmUiOnsiY29udGVudCI6Ik1FVUNJRHFnTmZ0MEF2ZUtpNVpuSnBlaTc4U01rZXZqNlkxMEt3bkY5cDhNWUxxYkFpRUE0bWlNZ0tqUkVKU3BJQ2JoZTVpbUorWE5sbUhJbll4dXV3bmlxZnlWZmU0PSIsInB1YmxpY0tleSI6eyJjb250ZW50IjoiTFMwdExTMUNSVWRKVGlCUVZVSk1TVU1nUzBWWkxTMHRMUzBLVFVacmQwVjNXVWhMYjFwSmVtb3dRMEZSV1VsTGIxcEplbW93UkVGUlkwUlJaMEZGUkVoM05EQnlXaTk0U1VWM1QzQXlja3RzWWl0NlFUZ3dUak00YUFwd2RuQkZjM2xOUlhCS1ZVTk5URTFuUVhwV2JURnlhamxvU2twR1ZUUkhiMWMzV1hkd1oxaFRlbW8yWTIxT1VUWXJWa0Z6WTJSdmFHaFJQVDBLTFMwdExTMUZUa1FnVUZWQ1RFbERJRXRGV1MwdExTMHRDZz09In19fX0=","integratedTime":1740278067,"logIndex":173545498,"logID":"c0d23d6ad406973f9559f3ba2d1ca01f84147d8ffc5b8445c224f98b9591801d"}}}}]
```
</details>

Alternatively, export the public key to a file and verify against it:

```shell
cosign public-key --key  $KEY > alibaba_kms.pub
cosign verify --key alibaba_kms.pub $IMAGE_DIGEST
```

<details>

```
$ cosign verify --key alibaba_kms.pub $IMAGE_DIGEST
Verification for registry.cn-beijing.aliyuncs.com/test-***/dummy@sha256:b12c7d46bc14b4260b9e42714688e2dbf5dee973b291a4c12e0e1539404d9f1d --
The following checks were performed on each of these signatures:
  - The cosign claims were validated
  - Existence of the claims in the transparency log was verified offline
  - The signatures were verified against the specified public key

[{"critical":{"identity":{"docker-reference":"registry.cn-beijing.aliyuncs.com/test-***/dummy"},"image":{"docker-manifest-digest":"sha256:b12c7d46bc14b4260b9e42714688e2dbf5dee973b291a4c12e0e1539404d9f1d"},"type":"cosign container image signature"},"optional":{"Bundle":{"SignedEntryTimestamp":"MEUCIAnDYUDBm3ufG7sN/D1dJLupYLhBkUcY6H3RTlYXxdAEAiEAikDsai+TsSU0PGC/gcds4ooDY/tmk6s+ZU5ttitf4g4=","Payload":{"body":"eyJhcGlWZXJzaW9uIjoiMC4wLjEiLCJraW5kIjoiaGFzaGVkcmVrb3JkIiwic3BlYyI6eyJkYXRhIjp7Imhhc2giOnsiYWxnb3JpdGhtIjoic2hhMjU2IiwidmFsdWUiOiI2MzE5Yjg1NzhkNWM5MjE2NmY0MzNmNzZjODg4NzZkOTA4Y2M2ZDgxOWFkMjYxODEzNGQ5NTcwYmFmNjhiYzg1In19LCJzaWduYXR1cmUiOnsiY29udGVudCI6Ik1FVUNJRHFnTmZ0MEF2ZUtpNVpuSnBlaTc4U01rZXZqNlkxMEt3bkY5cDhNWUxxYkFpRUE0bWlNZ0tqUkVKU3BJQ2JoZTVpbUorWE5sbUhJbll4dXV3bmlxZnlWZmU0PSIsInB1YmxpY0tleSI6eyJjb250ZW50IjoiTFMwdExTMUNSVWRKVGlCUVZVSk1TVU1nUzBWWkxTMHRMUzBLVFVacmQwVjNXVWhMYjFwSmVtb3dRMEZSV1VsTGIxcEplbW93UkVGUlkwUlJaMEZGUkVoM05EQnlXaTk0U1VWM1QzQXlja3RzWWl0NlFUZ3dUak00YUFwd2RuQkZjM2xOUlhCS1ZVTk5URTFuUVhwV2JURnlhamxvU2twR1ZUUkhiMWMzV1hkd1oxaFRlbW8yWTIxT1VUWXJWa0Z6WTJSdmFHaFJQVDBLTFMwdExTMUZUa1FnVUZWQ1RFbERJRXRGV1MwdExTMHRDZz09In19fX0=","integratedTime":1740278067,"logIndex":173545498,"logID":"c0d23d6ad406973f9559f3ba2d1ca01f84147d8ffc5b8445c224f98b9591801d"}}}}]
```
</details>


## URI Format

The URI used with the --key and --kms flag follows these formats:

* With alias name: `alibabakms://$ENDPOINT/[$INSTANCE_ID]/alias/$ALIAS_NAME`
* With key ID and version: `alibabakms://$ENDPOINT/[$INSTANCE_ID]/$KEY_ID/versions/$KEY_VERSION_ID`

Valid Examples:

* Alias name with endpoint: `alibabakms://kms.cn-hangzhou.aliyuncs.com//alias/cosign-key`
* Alias name with endpoint and instance ID: `alibabakms://kms.cn-hangzhou.aliyuncs.com/kst-foo-bar/alias/cosign-key`
* Key ID and version with endpoint: `alibabakms://kms.cn-hangzhou.aliyuncs.com//218d6978-***/versions/bb0416cd-2086-4107-a4a1-d2ecba325080`
* Key ID and version with endpoint and instance ID: `alibabakms://kms.cn-hangzhou.aliyuncs.com/kst-foo-bar/218d6978-***/versions/bb0416cd-2086-4107-a4a1-d2ecba325080`
