
```
static const int kAesIvSize = AES_BLOCK_SIZE; // AES_BLOCK_SIZE is from openssl/aes.h and equals 16
AesGcmCryptor::AesGcmCryptor(const Byte* aesKey, size_t keySizeInBytes, const Bytes_vt& initializationVector)
: m_aesKey(aesKey, aesKey + keySizeInBytes)
, m_iv(initializationVector) {}
void aesGcmEncrypt(const Byte* plainBytes, size_t plainBytesSize, Bytes_vt& cipherBytes) {
cipherBytes.resize(plainBytesSize);
CipherContextGuard ctxGuard(EVP_CIPHER_CTX_new());
EVP_EncryptInit_ex(ctxGuard.getContext(),
EVP_aes_256_gcm(),
NULL,
NULL,
NULL);
/* Set IV length if default 12 bytes (96 bits) is not appropriate */
EVP_CIPHER_CTX_ctrl(ctxGuard.getContext(),
EVP_CTRL_GCM_SET_IVLEN,
static_cast<int>(m_iv.size()),
NULL);
/* Initialise key and IV */
EVP_EncryptInit_ex(ctxGuard.getContext(), NULL, NULL,
reinterpret_cast<unsigned char*>(&m_aesKey.at(0)),
reinterpret_cast<unsigned char*>(&m_iv.at(0)));
/* Provide the message to be encrypted, and obtain the encrypted output.
* EVP_EncryptUpdate can be called multiple times if necessary */
int len;
EVP_EncryptUpdate(ctxGuard.getContext(),
reinterpret_cast<unsigned char*>(&cipherBytes.at(0)),
&len,
reinterpret_cast<const unsigned char*>(plainBytes),
static_cast<int>(plainBytesSize));
/* Finalize the encryption. Normally ciphertext bytes may be written at
* this stage, but this does not occur in GCM mode */
EVP_EncryptFinal_ex(ctxGuard.getContext(),
reinterpret_cast<unsigned char*>(&cipherBytes.at(0)) + len,
&len);
/* Get the tag */
Bytes_vt tagBytes(kAesGcmTagSize);
EVP_CIPHER_CTX_ctrl(ctxGuard.getContext(),
EVP_CTRL_GCM_GET_TAG,
kAesGcmTagSize,
reinterpret_cast<unsigned char*>(&tagBytes.at(0)));
cipherBytes.insert(cipherBytes.end(), tagBytes.begin(), tagBytes.end());
}
```

下面一段代码展示了一个 AES 解密过程。

```
void aesGcmDecrypt(const Byte* cipherBytes, size_t cipherBytesSize, Bytes_vt& plainBytes) {
size_t plainBytesSize = cipherBytesSize - kAesGcmTagSize;
plainBytes.resize(plainBytesSize);
CipherContextGuard ctxGuard(EVP_CIPHER_CTX_new());
EVP_DecryptInit_ex(ctxGuard.getContext(), EVP_aes_256_gcm(), 0, 0, 0);
/* Set IV length if default 12 bytes (96 bits) is not appropriate */
EVP_CIPHER_CTX_ctrl(ctxGuard.getContext(),
EVP_CTRL_GCM_SET_IVLEN,
static_cast<int>(m_iv.size()),
NULL);
/* Initialize key and IV */
EVP_DecryptInit_ex(ctxGuard.getContext(),
NULL,
NULL,
reinterpret_cast<unsigned char*>(&m_aesKey.at(0)),
reinterpret_cast<unsigned char*>(&m_iv.at(0)));
/* Provide the message to be decrypted, and obtain the plaintext output.
* EVP_DecryptUpdate can be called multiple times if necessary */
int len;
EVP_DecryptUpdate(ctxGuard.getContext(),
reinterpret_cast<unsigned char*>(plainBytes.data()),
&len,
reinterpret_cast<const unsigned char*>(cipherBytes),
static_cast<int>(plainBytesSize));
int plaintext_len = len;
/* Set expected tag value.*/
char tag[kAesGcmTagSize];
memcpy(tag, cipherBytes + cipherBytesSize - kAesGcmTagSize, kAesGcmTagSize);
EVP_CIPHER_CTX_ctrl(ctxGuard.getContext(), EVP_CTRL_GCM_SET_TAG, kAesGcmTagSize, tag);
/* Finalize the decryption. A positive return value indicates success,
* anything else is a failure - the plaintext is not trustworthy.
*/
EVP_DecryptFinal_ex(ctxGuard.getContext(),
reinterpret_cast<unsigned char*>(plainBytes.data()) + len,
&len);
plaintext_len += len;
plainBytes.resize(plaintext_len);
}
```

下面一段代码展示了加密密钥对，其中的密钥对是用上面的函数生成的。

```
Bytes_vt EncryptBytesUsingAesGcm(const Byte* plainBytes,
size_t plainBytesSize,
const Bytes_vt& aesKey,
const Bytes_vt& iv)
{
Bytes_vt cipherBytes;
crypto::AesGcmCryptor cryptor(aesKey.data(), aesKey.size(), iv);
cryptor.encrypt(plainBytes, plainBytesSize, cipherBytes);
return cipherBytes;
}
Bytes_vt encryptPrivateKeys(const RsaKeyPair& encryptDecryptKeyPair,
const RsaKeyPair& signVerifyKeyPair,
const Bytes_vt& encryptionKey)
{
Bytes_vt pdkIv(kAesIvSize);
GenerateRandomBytes(pdkIv.data(), pdkIv.size());
Bytes_vt pskIv(kAesIvSize);
GenerateRandomBytes(pskIv.data(), pskIv.size());
std::string encryptDecryptString = KeyToString(encryptDecryptKeyPair);
std::string signVerifyString = KeyToString(signVerifyKeyPair);
Bytes_vt encryptedPdk = EncryptBytesUsingAesGcm(encryptDecryptString.c_str(),
encryptDecryptString.size(),
encryptionKey,
pdkIv);
Bytes_vt encrytedPsk = EncryptBytesUsingAesGcm(signVerifyString.c_str(),
signVerifyString.size(),
encryptionKey,
pskIv);
uint16_t encryptedPdkLen = static_cast<uint16_t>(encryptedPdk.size());
Bytes_vt privateKeysBlob(sizeof(encryptedPdkLen) + pdkIv.size() + pskIv.size() +
encryptedPdk.size() + encrytedPsk.size());
privateKeysBlob[0] = static_cast<Byte>(encryptedPdkLen % 0x100);
privateKeysBlob[1] = static_cast<Byte>(encryptedPdkLen / 0x100);
auto it = privateKeysBlob.begin() + sizeof(encryptedPdkLen);
it = std::copy(pdkIv.begin(), pdkIv.end(), it);
it = std::copy(pskIv.begin(), pskIv.end(), it);
it = std::copy(encryptedPdk.begin(), encryptedPdk.end(), it);
it = std::copy(encrytedPsk.begin(), encrytedPsk.end(), it);
return privateKeysBlob;
}
```

下面一段代码展示了如何使用 RSA 对将要发送的数据进行签名。

```
void rsaSign(
const Byte* source,
size_t sourceSize,
Bytes_vt& signature,
RSA* rsa)
{
unsigned char hash[SHA256_DIGEST_LENGTH];
SHA256_CTX sha256Context = {0};
SHA256_Init(&sha256Context);
SHA256_Update(&sha256Context, source, sourceSize);
SHA256_Final(hash, &sha256Context);
signature.resize(RSA_size(m_rsa.get()));
unsigned int signatureResultSize = 0;
int result = RSA_sign(NID_sha256,
hash,
sizeof(hash),
reinterpret_cast<unsigned char*> (&signature.at(0)),
&signatureResultSize,
rsa);
if (1 != result)
{
_THROW_WITH_DEFAULT_INFO;
}
if (signatureResultSize != signature.size())
{
_THROW(cryptoErrorSigning);
}
}
```

下面一段代码展示了接收数据的一方是如何进行检验的。

```
bool rsaVerify(
const Byte* source,
size_t sourceSize,
const Byte* signature,
size_t signatureSize,
RSA* rsa)
{
unsigned char hash[SHA256_DIGEST_LENGTH];
SHA256_CTX sha256Context = {0};
SHA256_Init(&sha256Context);
SHA256_Update(&sha256Context, source, sourceSize);
SHA256_Final(hash, &sha256Context);
int result = RSA_verify(NID_sha256,
hash,
sizeof(hash),
reinterpret_cast<const unsigned char*> (signature),
static_cast<int> (signatureSize),
rsa);
return (1 == result);
}
```

# 总结

本文我们讲述了两名用户进行经过加密的即时消息交流背后的一些安全原则。OpenSSL 库在大多数情况下是一个很不错的选择。上文中所描述的加密方案在大多数情况下的效率表现是相当不错的，而且可以根据软件的需求方便地对其进行修改。

通信安全是一个很大的领域，要想覆盖所有的情况要耗费的时间或许是相当惊人的。我们在本文中仅仅讨论了如何在用户之间安全地传递消息，但还有很多东西是值得去思考的：

- 服务器端安全和如何在数据库中安全地存储密码(当然不能以明文存储了:)

- 客户端对于安全数据的缓存

- 生成 salt

- 如何选择加密算法

# 参考

<a href="elink@11eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4K9h3E0A6i4K6u0W2L8%4m8W2L8Y4y4K6L8q4)9J5k6h3!0J5k6H3`.`."><mark class="encrypted">aecK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4K9h3E0A6i4K6u0W2L8%4m8W2L8Y4y4K6L8q4)9J5k6h3!0J5k6H3`.`.</mark></a>

[<mark class="encrypted">81fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2L8W2)9J5k6i4N6A6K9$3W2H3k6h3c8A6j5g2)9J5k6h3!0J5k6#2)9J5c8Y4N6A6K9$3W2Q4x3V1k6q4L8r3I4A6M7s2c8A6j5#2)9#2k6V1y4#2M7Y4k6W2i4K6g2X3c8r3W2Y4K9i4c8S2L8q4)9#2k6W2y4A6k6$3&6S2N6s2g2J5k6g2)9#2k6V1q4D9k6$3!0J5K9i4c8Z5L8b7`.`.</mark>

](<mark class="encrypted">ce2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2L8W2)9J5k6i4N6A6K9$3W2H3k6h3c8A6j5g2)9J5k6h3!0J5k6#2)9J5c8Y4N6A6K9$3W2Q4x3V1k6q4L8r3I4A6M7s2c8A6j5#2)9#2k6V1y4#2M7Y4k6W2i4K6g2X3c8r3W2Y4K9i4c8S2L8q4)9#2k6W2y4A6k6$3&6S2N6s2g2J5k6g2)9#2k6V1q4D9k6$3!0J5K9i4c8Z5L8b7`.`.</mark>

)

<a href="elink@b7dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2L8W2)9J5k6i4N6A6K9$3W2H3k6h3c8A6j5g2)9J5k6h3!0J5k6#2)9J5c8Y4N6A6K9$3W2Q4x3V1k6q4L8r3I4A6M7s2c8A6j5#2)9#2k6X3y4#2M7Y4k6W2i4K6g2X3c8r3W2X3k6X3W2W2i4K6t1#2c8e0u0Q4x3U0f1^5x3q4)9J5y4e0V1K6d9r3g2D9L8r3#2S2L8R3`.`."><mark class="encrypted">99fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2L8W2)9J5k6i4N6A6K9$3W2H3k6h3c8A6j5g2)9J5k6h3!0J5k6#2)9J5c8Y4N6A6K9$3W2Q4x3V1k6q4L8r3I4A6M7s2c8A6j5#2)9#2k6X3y4#2M7Y4k6W2i4K6g2X3c8r3W2X3k6X3W2W2i4K6t1#2c8e0u0Q4x3U0f1^5x3q4)9J5y4e0V1K6d9r3g2D9L8r3#2S2L8R3`.`.</mark></a>
