# v86 Emulator (GitHub Pages)

A self-contained, browser-based x86 PC emulator built on [v86](https://github.com/copy/v86) (WebAssembly x86 emulator + JIT). No server, no backend — everything, including the "disk," lives in your browser tab.

Live features:

- **RAM**: slider + free-typed exact-MB input, no artificial cap tied to the browser's ~2GB `ArrayBuffer` limit (that limit only bites disk images, not emulated RAM — see below).
- **Boot media**: upload any ISO for the virtual CD-ROM, upload an existing raw disk image for the hard drive, or generate a blank hard drive image of (almost) any size on the fly.
- **Runtime controls**: pause/resume, reset, fullscreen, save/restore machine state, eject/swap CD.

## Why the 2GB thing doesn't limit your RAM

Two totally different limits get conflated with v86:

- **Emulated RAM** (the `memory_size` config option) is just a number of bytes v86's WASM core allocates as guest memory. It's not bound by 2GB — it's bound by whatever your device and browser tab can actually spare. Multi-GB is fine if your machine has it.
- **Disk images**, historically, could hit trouble if code naively read the whole image into one `ArrayBuffer` — some browser/engine contexts cap a single `ArrayBuffer` around 2GB, or at minimum choke badly on the allocation. v86's own `buffer_from_object` already avoids this for uploaded files ≥256MB by switching to `AsyncFileBuffer`, which streams reads lazily off the `File`/`Blob` via `FileReader` + `.slice()` instead of allocating the whole thing up front.

This page leans on that existing async path for uploads, and uses the same trick manually for **blank disk creation** (composing a large `File` out of reused zero-filled `Blob` chunks, flagged `async: true`) so generating a 64GB blank image doesn't try to allocate 64GB of live JS memory.

## Known limits

- Blank images are zero-filled raw disks — you still need to partition/format them from inside the guest OS (or via an OS installer booted from CD), same as a brand-new physical drive.
- Only raw disk image formats are understood natively (`.img`/`.raw`). Formats like `.qcow2` or `.vdi` need converting to raw first (e.g. via `qemu-img convert -O raw in.qcow2 out.img`) before uploading.
- Networking is off by default. v86 needs a WebSocket relay server for guest networking (see [v86's networking docs](https://github.com/copy/v86/blob/master/docs/networking.md)) — there's a field to plug one in, but this repo doesn't run one for you.
- Performance is JIT-compiled x86-in-WASM, so expect roughly "an old, honest PC," not bare-metal speed. Fine for DOS, small Linux distros, retro OSes; don't expect to compile the kernel in record time.

## Local structure

```
index.html        the whole app
build/libv86.js    prebuilt v86 runtime (from the official npm package)
build/v86.wasm     prebuilt v86 core
bios/*.bin         SeaBIOS + Bochs VGA BIOS (from copy/v86)
```

All vendored files are pulled directly from the official [copy/v86](https://github.com/copy/v86) project (BSD-2-Clause) and its npm distribution — nothing here is a fork of v86's core, just a configuration UI wrapped around it.


yeah yeah yeah its made by AI but it WORKS and it's useful so haha
