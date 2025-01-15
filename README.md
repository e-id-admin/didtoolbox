![Public Beta banner](https://github.com/e-id-admin/eidch-public-beta/blob/main/assets/github-banner-publicbeta.jpg)

# DID toolbox

An official Swiss Government project made by
the [Federal Office of Information Technology, Systems and Telecommunication FOITT](https://www.bit.admin.ch/)
as part of the electronic identity (e-ID) project.

This project implements the following things:

- General util structs reused by other libraries of e-id-admin
- Trust DID web according to the specification [trust-did-web](https://bcgov.github.io/trustdidweb/)

## Using the library

The library can be used either directly in rust as is or through the different built bindings which are published in
different submodules.

### Rust

The library can be used directly in rust by adding the following dependency to your `Cargo.toml`:

````toml
[dependencies]
didtoolbox = { git = "https://github.com/e-id-admin/didtoolbox", branch = "main" }

# Optional: For manipulating the json content in the example
serde_json = "1.0.133"
````

### Additional language bindings

> General information how the bindings are generated can be found in
> the [UniFFI user guide](https://mozilla.github.io/uniffi-rs/latest/)

Although indirectly, the library is to a certain extent also available in other languages. Please consult the
documentation of the subsequent repositories for more information:

- [Kotlin / Java](https://github.com/e-id-admin/didresolver-kotlin)
- [Kotlin (Android)](https://github.com/e-id-admin/didresolver-kotlin-android/)
- [Swift](https://github.com/e-id-admin/didresolver-swift)

## Example

In the example the following steps are shown:

1. Create a new did:tdw by initializing a DID doc. In this DID doc an ed25519 key is used as controller and to create
   the integrity proofs
2. Add another verification method to the existing DID doc
3. Update the DID log

```rust
use didtoolbox::ed25519::Ed25519KeyPair;
use didtoolbox::did_tdw::TrustDidWeb;

fn main() {
    // Base url on which base the did will be created. This is legacy logic from the first version of the tdw specification
    let base_url = String::from("https://someservice.bit.admin.ch");
    // Keypair which is used to sign the did document and isn't used for actual credential issuing
    let key_pair = Ed25519KeyPair::generate();

    // Create genesis did document which contains the public key of "key_pair" as controller and an according verification method entry
    let tdw_v1 = TrustDidWeb::create(
        base_url,
        &key_pair,
        Some(false)
    ).unwrap();

    // Updating the did document by adding a new verification method
    let did_doc_v1_str = tdw_v1.get_did_doc();
    println!("DID Doc v1: {}", did_doc_v1_str);
    let mut did_doc_v1: serde_json::Value = serde_json::from_str(&did_doc_v1_str).unwrap();;
    match &did_doc_v1["verificationMethod"] {
        serde_json::Value::Array(v) => v.iter().for_each(|x| println!("{}", x)),
        _ => panic!("Should fail")
    };
    did_doc_v1["verificationMethod"].as_array_mut().unwrap().push(serde_json::json!({
        "id": "<some unique identifer e.g. {did}#{usage}>",
        "type": "<public key identifier according to specification registry>",
        "controller": "<e.g. some controller did>",
        "publicKeyMultibase": "<some fancy multibase encoded public key>"
    }));
    let did_doc_v2_str = did_doc_v1.to_string();
    // updating DID log goes here
}

```

## License

This project is licensed under the terms of the MIT license. See the [LICENSE](LICENSE.md) file for details.
