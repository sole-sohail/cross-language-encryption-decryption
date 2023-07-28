# Cross-Language Encryption and Decryption

## Introduction

The "Cross-Language Encryption-Decryption" project provides encryption and decryption methods implemented in three different programming languages: C#, Flutter (Dart), and PHP. The project allows you to encrypt a string in any of the supported languages and decrypt it in any other given language, facilitating secure communication and data exchange across various platforms. This project is open-source and can be freely used by the community.

## Encryption and Decryption Algorithms

The project utilizes the AES (Advanced Encryption Standard) with CBC (Cipher Block Chaining) mode for encryption and decryption. The encryption key and initialization vector (IV) are derived from a user-defined passphrase using SHA-256 hashing. The derived key is then truncated to the required length (32 bytes for the encryption key and 16 bytes for the IV).

## C# Implementation

The C# implementation of encryption and decryption uses the .NET framework's cryptographic classes. It provides a class named "Encryption" with static methods for encrypting and decrypting data.

```csharp
using System;
using System.IO;
using System.Security.Cryptography;
using System.Text;

public class Encryption
{
    byte[] key;
    byte[] iv;

    private Encryption instance;
    public Encryption Instance
    {
        get
        {
            if (instance == null)
                instance = new Encryption();
            return instance;
        }
    }

    private Encryption()
    {
        var Key = "ThisIsASecuredKey";
        var Iv = "ThisIsASecuredBlock";

        using (var sha256 = SHA256.Create())
        {
            key = Encoding.UTF8.GetBytes(ToHex(sha256.ComputeHash(Encoding.UTF8.GetBytes(Key))).Substring(0, 32));
            iv = Encoding.UTF8.GetBytes(ToHex(sha256.ComputeHash(Encoding.UTF8.GetBytes(Iv))).Substring(0, 16));
        }
    }
    
    public string Encrypt(string input)
    {
        using (var aesManaged = new AesManaged())
        using (var ms = new MemoryStream())
        {
            using (var cs = new CryptoStream(ms, aesManaged.CreateEncryptor(key, iv), CryptoStreamMode.Write))
            {
                var inputBytes = Encoding.UTF8.GetBytes(input);
                cs.Write(inputBytes, 0, inputBytes.Length);
            }

            return Convert.ToBase64String(ms.ToArray());
        }
    }

    public string Decrypt(string base64value)
    {
        using (var aesManaged = new AesManaged())
        using (var decryptor = aesManaged.CreateDecryptor(key, iv))
        using (var ms = new MemoryStream(Convert.FromBase64String(base64value)))
        using (var cs = new CryptoStream(ms, decryptor, CryptoStreamMode.Read))
        using (var sr = new StreamReader(cs))
        {
            return sr.ReadToEnd();
        }
    }

    private string ToHex(byte[] bytes, bool upperCase = false)
    {
        var result = new StringBuilder(bytes.Length * 2);
        for (int i = 0; i < bytes.Length; i++)
            result.Append(bytes[i].ToString(upperCase ? "X2" : "x2"));
        return result.ToString();
    }
}

```

## Flutter (Dart) Implementation

The Flutter implementation of encryption and decryption uses the encrypt package for AES encryption and decryption. It offers a singleton class "Encryption" with methods for encrypting and decrypting data.

```dart

import 'dart:convert';

import 'package:encrypt/encrypt.dart';
import 'package:crypto/crypto.dart';

class Encryption {
  static final Encryption instance = Encryption._();
  
  late IV _iv;
  late Encrypter _encrypter;

  Encryption._() {
    const mykey = 'ThisIsASecuredKey';
    const myiv = 'ThisIsASecuredBlock';
    final keyUtf8 = utf8.encode(mykey);
    final ivUtf8 = utf8.encode(myiv);
    final key = sha256.convert(keyUtf8).toString().substring(0, 32);
    final iv = sha256.convert(ivUtf8).toString().substring(0, 16);
    _iv = IV.fromUtf8(iv);

    _encrypter = Encrypter(AES(Key.fromUtf8(key), mode: AESMode.cbc));
  }

  String encrypt(String value) {
    return _encrypter.encrypt(value, iv: _iv).base64;
  }

  String decrypt(String base64value) {
    final encrypted = Encrypted.fromBase64(base64value);
    return _encrypter.decrypt(encrypted, iv: _iv);
  }
}

```

## PHP Implementation

The PHP implementation of encryption and decryption uses the OpenSSL extension for AES encryption and decryption. It provides a class named "Encryption" with methods for encrypting and decrypting data.

```php

<?php

class Encryption
{
    private string $encryptMethod = 'AES-256-CBC';
    private string $key;
    private string $iv;

    public function __construct()
    {
        $mykey = 'ThisIsASecuredKey';
        $myiv = 'ThisIsASecuredBlock';
        $this->key = substr(hash('sha256', $mykey), 0, 32);
        $this->iv = substr(hash('sha256', $myiv), 0, 16);
    }

    public function encrypt(string $value): string
    {
        return openssl_encrypt($value, $this->encryptMethod, $this->key, 0, $this->iv);
    }

    public function decrypt(string $base64Value): string
    {
        return openssl_decrypt($base64Value, $this->encryptMethod, $this->key, 0, $this->iv);
    }
}

```


## Usage

To encrypt and decrypt data across different platforms, follow these steps:

## Encrypt Data
1. Create an instance of the "Encryption" class in your desired programming language (C#, Flutter, or PHP).
2. Call the "encrypt" method with the data you want to encrypt as an argument.
3. The method will return the encrypted data as a string (Base64-encoded in PHP and Dart, and a regular string in C#).

## Decrypt Data
1. Create an instance of the "Encryption" class in the desired programming language (C#, Flutter, or PHP).
2. Call the "decrypt" method with the encrypted data as an argument (Base64-encoded in PHP and Dart, and a regular string in C#).
3. The method will return the decrypted data as a string.

## Example: Encrypting and Decrypting in C#
``` csharp
var encryption = new Encryption();
string originalText = "This is the sensitive data to be encrypted.";

string encryptedText = encryption.Encrypt(originalText);
Console.WriteLine("Encrypted: " + encryptedText);

string decryptedText = encryption.Decrypt(encryptedText);
Console.WriteLine("Decrypted: " + decryptedText);
```

## Example: Encrypting and Decrypting in Flutter (Dart)
``` dart
final encryption = Encryption.instance;
String originalText = "This is the sensitive data to be encrypted.";

String encryptedText = encryption.encrypt(originalText);
print("Encrypted: $encryptedText");

String decryptedText = encryption.decrypt(encryptedText);
print("Decrypted: $decryptedText");
```

## Example: Encrypting and Decrypting in PHP
``` php
$encryption = new Encryption();
$originalText = "This is the sensitive data to be encrypted.";

$encryptedText = $encryption->encrypt($originalText);
echo "Encrypted: " . $encryptedText . PHP_EOL;

$decryptedText = $encryption->decrypt($encryptedText);
echo "Decrypted: " . $decryptedText . PHP_EOL;
```

## Contribution and Usage
This project is open-source and available to the community for free use and contributions. You can integrate these encryption and decryption methods into your applications, contribute to the project's codebase, report issues, or suggest improvements through GitHub or other relevant platforms.

Happy coding and secure communication! 🚀





