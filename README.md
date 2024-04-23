ansible-role-openssl
=========

Setup the simple PKI. It is possible to configure multi-tier architecture. Note. This role does not save id of the issued new cert and not implement the revocation list.


Role Variables
--------------

`openssl_ca_common_name`: The commonName field of the Certificate. "{{ ansible_hostname }}" default.

`openssl_ca_user`: CA certificates owner user. root default

`openssl_ca_group`: System group to grant the FS permissions. root default

`openssl_ca_permission`: Access mode to CA files. '0640' default

`openssl_ca_cert_path`: Directory to store CA publik keys. '/etc/selfsigned-ca' default

`openssl_ca_key_path`: Directory to store CA private keys. '/etc/selfsigned-ca' default

`openssl_ca_key_type`: Define CA private key algorithm. 'RSA' default

`openssl_ca_key_size`: CA private key size. 4096 default

`openssl_ca_key_passphrase`: Define passphrase for CA private key. '' default

`openssl_ca_name_constraints_critical`: Should the Name Constraints extension be considered as critical. false default

`openssl_ca_name_constraints_excluded`: This specifies a list of identifiers which describe subtrees of names that this CA is not allowed to issue certificates for. [] default
 
`openssl_ca_name_constraints_permitted`: A list of identifiers which describe subtrees of names that this CA is allowed to issue certificates for. [] default

`openssl_ca_basic_constraints`: Define basic constraints. [] default

`openssl_ca_basic_constraints_critical`: Should the basicConstraints extension be considered as critical. false default

`openssl_ca_extended_key_usage`: Additional restrictions (for example client authentication, server authentication) on the allowed purposes for which the public key may be used. ['serverAuth', 'clientAuth'] default.

`openssl_ca_extended_key_usage_critical`: Should the extkeyUsage extension be considered as critical. false default

`openssl_ca_key_usage`: This defines the purpose (for example encipherment, signature, certificate signing) of the key contained in the certificate. []

`openssl_ca_key_usage_critical`: Should the keyUsage extension be considered as critical. false default


###### Client
`openssl_client_common_name` The commonName field of the Certificate. {{ ansible_hostname }} default.
`openssl_client_user`: Client certificates owner user. root default

`openssl_client_group`: Client certificates owner group. root default

`openssl_client_permission`: Permission mode. "0640" default

`openssl_client_key_path`: Client private key directory. '/etc/ssl/private' default.

`openssl_client_cert_path`: Client publik key directory '/etc/ssl/certs' default.

`openssl_client_cert_expire`: Client cert expire. '+365d' default

`openssl_client_key_type`: Client private key algorithm. RSA default

`openssl_client_key_size`: Client private key size. 4096 default

`openssl_client_key_passphrase`: Protect client key passphrase . '' default

`openssl_client_subject_alt_name`: Subject Alternative Name (SAN) extension to attach to the certificate signing request. ["IP:{{ hostvars[inventory_hostname]['ansible_default_ipv4']['address'] }}", "DNS:{{ hostvars[inventory_hostname]['ansible_default_ipv4']['address'] }}"] default

`openssl_client_ca_bundle`: Create CA bundle on client. true default

`openssl_client_basic_constraints`: Define basic constraints. [] default

`openssl_client_basic_constraints_critical`: Should the basicConstraints extension be considered as critical. false default

`openssl_client_extended_key_usage`: Additional restrictions (for example client authentication, server authentication) on the allowed purposes for which the public key may be used. ['serverAuth', 'clientAuth'] default.

`openssl_client_extended_key_usage_critical`: Should the extkeyUsage extension be considered as critical. false default

`openssl_client_key_usage`: This defines the purpose (for example encipherment, signature, certificate signing) of the key contained in the certificate. []

`openssl_client_key_usage_critical`: Should the keyUsage extension be considered as critical. false default

`openssl_client_ocsp_must_staple`: Indicates that the certificate should contain the OCSP Must Staple extension. false default

`openssl_client_ocsp_must_staple_critical`: Should the OCSP Must Staple extension be considered as critical. false default

`openssl_create_subject_key_identifier`: Create the Subject Key Identifier from the public key. false default

`openssl_crl_distribution_points`:  Specift certificate revocation list information (optional). 
```yaml
openssl_crl_distribution_points:
- crl_issuer: ""
  full_name: ""
  reasons:                    # Reason of revocation
  - "key_compromise"
  - "ca_compromise"
  - "affiliation_changed"
  - "superseded"
  - "cessation_of_operation"
  - "certificate_hold"
  - "privilege_withdrawn"
  - "aa_compromise"
  relative_name: ""           # Crl relative name point
```

`openssl_csr_subject`  Certificate signing request subject
```yaml
openssl_csr_subject:
  country_name: ''
  email_address: 'admin@example.com'
  organization_name: 'Company name'
  organizational_unit_name: 'IT'
  state_or_province_name: ''
  locality_name: ''
```

`openssl_certs`: List of certs to create. Root CA can not contain field ca_issuer. Into ca_issuer we define cert/key/bundle paths, key passphrase and issued host. issued_ca_host must be the valid hostname from an inventory to be able to use with delegate_to:.
```yaml
openssl_certs:
- name: ca
  ca: yes
  key_type: Ed25519
  key_passphrase: "CAPrivateKeyPassphraseHere"
  key_path: "{{ openssl_ca_key_path }}/ca"
  cert_path: "{{ openssl_ca_cert_path }}/ca"
  cert_expire: "+1825d"
  cert_subject_alt_name:
  - "IP:127.0.0.1"
  - "IP:{{ hostvars[inventory_hostname]['ansible_default_ipv4']['address'] }}"
  - "DNS:{{ ansible_hostname }}"
  - "DNS:localhost"
  - "DNS:{{ hostvars[inventory_hostname]['ansible_default_ipv4']['address'] }}"
  cert_common_name: "{{ ansible_hostname }}"
  basic_constraints:
  - "CA:TRUE"
  - "pathlen:1"
  basic_constraints_critical: true
  key_usage: ["keyCertSign"]
  key_usage_critical: true

- name: client1
  key_path: "{{ openssl_client_key_path }}/client1"
  key_passphrase: "client1_PrivateKeyPassphraseHere"
  cert_path: "{{ openssl_client_cert_path }}/client1"
  subject_alt_name:
  - "IP:127.0.0.1"
  - "DNS:client1"
  - "DNS:localhost"
  common_name: "client1"
  key_usage: ['digitalSignature', 'keyEncipherment']
  key_usage_critical: true
  extended_key_usage: ['serverAuth', 'clientAuth']
  extended_key_usage_critical: true
  ca_issuer:
    cert_path: "{{ openssl_ca_key_path }}/ca/ca.crt"
    key_path: "{{ openssl_ca_key_path }}/ca/ca.key"
    key_passphrase: "CAPrivateKeyPassphraseHere"
    issued_ca_host: "localhost"
    bundle_path: "{{ openssl_ca_key_path }}/ca/ca-bundle.crt"
```

####### Copy ca bundle to host without creating client certs
```yaml
openssl_copy_ca:
  src_host: intermediate_ca_host            # CA host from where will be copyed ca bundle
  src: "/etc/openssl/certs/ca-bundle.crt"   # Path where to find ca bundle on the src_host
  dest: "/etc/openssl/certs/ca-bundle2.crt" # Path where to copy the bundle
```

`More examples see directory ./samples`

Dependencies
------------


Example Playbook
----------------

```yaml
- name: PKI
  hosts: all
  tasks:
    - name: "Include ansible-role-openssl"
      ansible.builtin.include_role:
        name: "ansible-role-openssl"
```

License
-------

MIT

Author Information
------------------

saydulaev.rb@gmail.com
