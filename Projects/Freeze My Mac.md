---
type: personal-project
status: experimental
stack: [Swift, macOS, XcodeGen]
code_path: "~/fun-projects/freeze-my-mac"
---

# Freeze My Mac

macOS app/extension — experimental.

## Structure
```
freeze-my-mac/
├── FreezeMyMac/           ← main app target
├── FreezeMyMacExtension/  ← extension target
├── Shared/                ← shared code
├── project.yml            ← XcodeGen project definition
└── logs.txt
```

## Stack
- Swift
- macOS
- XcodeGen (`project.yml` → generates `.xcodeproj`)

## Notes
- Uses XcodeGen — run `xcodegen generate` before opening in Xcode if `.xcodeproj` is not committed
