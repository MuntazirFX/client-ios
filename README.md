# client-ios-native

CMake-based native library, built for iOS (arm64) via GitHub Actions.

## Structure

```
.
├── CMakeLists.txt
├── include/
│   └── client.h          # Public headers — your API
├── src/
│   └── client.cpp        # Your implementation
├── .github/workflows/
│   └── ios-build.yml      # CI: build + sign + upload artifact
├── exportOptions.plist    # Used only if exporting a full .ipa
└── README.md
```

Add your own `.c`/`.cpp` files under `src/` and matching headers under
`include/`. `CMakeLists.txt` globs everything in `src/` automatically.

## Building locally (macOS, Apple Silicon)

```bash
curl -L -o ios.toolchain.cmake \
  https://raw.githubusercontent.com/leetal/ios-cmake/master/ios.toolchain.cmake

cmake -B build-ios -G Xcode \
  -DCMAKE_TOOLCHAIN_FILE=ios.toolchain.cmake \
  -DPLATFORM=OS64 \
  -DDEPLOYMENT_TARGET=15.0

cmake --build build-ios --config Release
```

Output: `build-ios/Release-iphoneos/libclient_arm64.dylib`

## CI/CD (GitHub Actions)

`.github/workflows/ios-build.yml` runs on `macos-14` (Apple Silicon
runner) and:

1. Checks out the repo
2. Pulls the `ios-cmake` toolchain
3. Configures + builds with CMake/Xcode for `OS64` (device, arm64)
4. If signing secrets are present, code-signs the resulting `.dylib`
5. Uploads the artifact

### Required secrets (for code signing)

Set these under **Settings → Secrets and variables → Actions**:

| Secret | Description |
|---|---|
| `BUILD_CERTIFICATE_BASE64` | Your distribution `.p12`, base64-encoded (`base64 -i cert.p12`) |
| `P12_PASSWORD` | Password for the `.p12` |
| `BUILD_PROVISION_PROFILE_BASE64` | Your `.mobileprovision`, base64-encoded |
| `KEYCHAIN_PASSWORD` | Any throwaway password for the temp CI keychain |
| `CODESIGN_IDENTITY` | e.g. `Apple Distribution: Your Name (TEAMID)` |

Without these secrets, the workflow still builds the unsigned `.dylib`
and skips the signing step.

### Producing a full `.ipa`

A bare `.dylib` cannot become an `.ipa` by itself — an `.ipa` wraps a
signed `.app` bundle (an actual Xcode application target, with
`Info.plist`, entitlements, etc). The `package-ipa` job in the
workflow is a template for that: point `-scheme` at your real Xcode
app scheme, fill in `exportOptions.plist` with your Team ID, and flip
`if: false` off once you have that target.

## Notes

- This template assumes you own or have a legal license for all
  source code placed in `src/`/`include/`.
- Requires a paid Apple Developer account + valid signing
  certificate/provisioning profile for on-device signing.
