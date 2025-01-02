---
tags:
  - vault
  - hashicorp
---

# Tokens

Authentication with Vault is done via tokens. These tokens can either be created manually or dynamically via `auth methods`.

A token has a set of policies linked to it. These policies define the effective permissions of the token.

There are three types of tokens available.
* service tokens
* batch tokens
* recovery tokens

## Service tokens

Service tokens are prefixed with `hvs.` when they are generated.
These are considered the 'normal' tokens for everyday use.

## Batch tokens

Batch tokens are prefixed with `hvb.` when they are generated.
Batch tokens are encrypted blobs that carry enough information for them to be used for Vault actions, but they require no storage on disk to track them. As a result they are extremely lightweight and scalable, but lack most of the flexibility and features of service tokens.

## Recovery tokens

Recovery tokens are prefixed with `hvr.` when they are generated.

## Token hierarchy

The root token can do anything. This token will never expire, but should also not be used for day to day operations. Instead, this token should be used to do the initial setup.

A token is generated via a token holder. By default the generated token will be a child of the parent token. Whenever the parent token is revoked, all the child tokens are revoked as well.
Alternatively you can create an orphan token. This token does not have a parent.

## TTL

Every token except the root token has a TTL which determines how long the token will be valid. Once the TTL is expired, the token is no longer usable.

You can renew a token using the `vault token renew` command.

## Periodic tokens

For some tokens it is important that they are automatically renewed. This can be done by creating a periodic token. The token will have an initial TTL and once it is automatically renewed, the TTL will be reset. This effectively means that the token never expires, as long as the service is renewing it.

A token with both a period and an explicit max TTL will act like a periodic token but will be revoked when the explicit max TTL is reached
