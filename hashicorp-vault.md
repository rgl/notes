# Azure Key Vault Auto-Unseal

See https://www.vaultproject.io/docs/concepts/seal#auto-unseal

Code path:

[go-kms-wrapping wrappers/azurekeyvault/azurekeyvault.go](https://github.com/hashicorp/vault/blob/v1.5.0/vendor/github.com/hashicorp/go-kms-wrapping/wrappers/azurekeyvault/azurekeyvault.go):

```go
type azurekeyvault.Wrapper
  func (v *Wrapper) Encrypt(ctx context.Context, plaintext, aad []byte) (blob *wrapping.EncryptedBlobInfo, err error) {
    return EncryptedBlobInfo {
      cipherText, IV = AES-256-GCM-AEAD(key, plaintext, aad)
      keyID, wrappedKey = azureKeyVault.wrapKey(RSA-OAEP-256, key) // https://docs.microsoft.com/en-us/rest/api/keyvault/wrapkey/wrapkey
    }
```

[vault command/server.go](https://github.com/hashicorp/vault/blob/v1.5.0/command/server.go):

```go
seal = vault.NewAutoSeal(&vaultseal.Access{
  Wrapper: wrapper,
})

// uses the azurekeyvault.Wrapper to wrap the hashicorp vault recovery-key with the key stored in azure key vault.
func (d *autoSeal) SetRecoveryKey(ctx context.Context, key []byte) error

// uses the azurekeyvault.Wrapper to unwrap the hashicorp vault recovery-key with the key stored in azure key vault.
func (d *autoSeal) RecoveryKey(ctx context.Context) ([]byte, error)
```

* the recovery key is actually the master key.
* the master key is encrypted by the key stored in azure key vault
* the encrypted master key in stored inside the hashicorp vault.
