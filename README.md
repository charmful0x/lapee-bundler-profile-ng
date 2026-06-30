# LapEE Bundler Profile NG

This repo contains the working profile JSON for the native P4 AO-paid LapEE bundler profile.

It intentionally tracks only:

- `lapee-bundler-profile-ng.json` - the deployable HyperBEAM/LapEE profile.
- `README.md` - the manifest for the profile, source repos, and proof run.

The device source lives in separate repos. Keeping this repo profile-only avoids duplicating Erlang device code and makes the profile artifact easy to review, pin, and publish.

## Device Source

- `ao-payment@1.0`: https://github.com/xylophonez/aopayment-device
- `arweave-byte-pricing@1.0`: https://github.com/xylophonez/arweave-byte-pricing-device
- `bundler-settlement@1.0`: https://github.com/xylophonez/bundler-settlement-device
- `lapee-bundler-gc@1.0`: https://github.com/xylophonez/lapee-bundler-gc-device
- `lapee-p4-bootstrap@1.0`: https://github.com/xylophonez/lapee-p4-bootstrap-device
- `pricing-router@1.0`: https://github.com/xylophonez/pricing-router-device
- `simple-oracle@1.0`: https://github.com/xylophonez/simple-oracle-device

The combined development mirror is:

- https://github.com/xylophonez/lapee-bundler-devices-mirror/tree/feat/p4-compat

## Profile Behavior

This profile boots through `lapee-p4-bootstrap@1.0`, disables remote device loading, and configures the paid bundler path so native P4 charges are settled directly through `ao-payment@1.0`.

The important runtime behavior is:

- `ao-payment@1.0` imports AO token payments into the local P4 ledger.
- Native P4 charges debit that local ledger when `~bundler@1.0/item` is posted.
- `ao-payment@1.0` auto-withdraws the charged AO quantity to `bundler-beneficiary`.
- `bundler-settlement@1.0` is no longer required for the immediate paid-bundle path.

## Proven Build

The JSON in this repo was copied from the local profile used in the QEMU proof run:

- Source path: `/home/fn/Dev/FWD/os/build/local-profiles/aopayment-bundler-p4-native-baked-local-20260629.json`
- Profiled image: `/home/fn/Dev/FWD/os/build/images/lapee-no-tme-aopayment-p4-autowithdraw-profile-20260629-signed.img`
- Preloaded device index: `AHEfwbPh0hq-20u5JNdicAEfjg60UwpbDCI7zsqRh24`
- Proof artifacts: `/home/fn/Dev/FWD/os/build/qemu-p4-flow-20260629-autowithdraw-profile`

QEMU proof summary:

- Payment message: `euwfJwDYGw5V0WzfG1NMF82zG7fOYJQWFUJwk1fHXTQ`
- Payment slot: `2495513`
- Bundle item: `ZNq0StxWO26ifWzc26Gfed_pou4_mtj8Ik_MunQBozM`
- Quote/payment quantity: `2509673921`
- Bundle status: `200`
- Beneficiary AO delta: `2509673921`
- Sender local ledger after charge: `0`
- Deposit AO after auto-withdraw: `0`

## Applying The Profile

From the LapEE OS repo, apply this profile to a signed image with:

```sh
scripts/apply-profile-to-image.sh \
  /path/to/lapee-bundler-profile-ng.json \
  build/images/lapee-no-tme-aopayment-p4-autowithdraw-20260629-signed.img \
  build/images/lapee-no-tme-aopayment-p4-autowithdraw-profile-20260629-signed.img
```

The device source should be baked into the image before profile application. This profile does not fetch remote device code at runtime.
