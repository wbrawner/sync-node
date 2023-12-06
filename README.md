# Sync Node

Sync Node is a general-purpose backend sync server. It is designed to augment
[local-first] software by simply providing an endpoint with which to sync data
across devices and/or between users. Note that Sync Node does not concern itself
with end-to-end encryption; this is by design. Nothing is stopping you from
encrypting the data before submitting it to Sync Node, and in fact it's
encouraged to do so. Key management and encryption are exercises left to the
clients, however. If your use case requires a more "batteries included" approach
to end-to-end encryption, consider using something like [etebase] instead.

## Configuration

Configuration is handled exclusively through the following environment variables:

|  Environment Variable  |  Default Value  |
|:----------------------:|:---------------:|
|         `PORT`         |     `8080`      |
|     `STATIC_ROOT`      |    `./data`     |

## Goals

1. Provide a general-purpose backend that requires little-to-no configuration to
   enable syncing for [local-first] applications.
2. Minimize resource usage on the server
3. Maximize longevity of the project
4. Simplify installation/updates

## Data Model

```mermaid
erDiagram
    sync_account }|--o{ account_permission: ""
    sync_account ||--|{ sync_session: ""
    sync_account ||--o| password_reset_token: ""
    sync_store }|--|{ account_permission: ""
    sync_account {
        string id PK "The user name/email. Should be hashed for privacy."
        string password "The user password. Should be hashed for security."
        string key_salt UK "The salt used to hash key names"
    }
    account_permission {
        string sync_account_id PK, FK "The user identifier"
        string store_key PK, FK "The key to the store to which this permission is granted"
        string permission "The permission granted. Should be one of READ, WRITE, OWN"
    }
    sync_store {
        string(8000) store_key PK
        string store_file "The path on disk where the data for the associated key is stored"
        string(64) store_token "A token used to ensure that the client has the most recent copy of the data"
        string(8000) store_meta "Optional metadata to be associated with the store_file"
    }
    password_reset_token {
        string id PK, FK "The user identifier"
        string token UK
        int expiration "The timestamp for the expiration of the password reset token, in milliseconds since Unix Epoch"
    }
    sync_session {
        string token PK
        string id FK "The user identifier"
        int expiration "The timestamp for the expiration of the session token, in milliseconds since Unix Epoch"
    }
```

[etebase]: https://www.etebase.com/

[local-first]: https://www.inkandswitch.com/local-first/
