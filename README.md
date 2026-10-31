<div align=center>

<img width="616" height="353" alt="capsule_616x353" src="https://github.com/user-attachments/assets/54108a00-ed00-4292-a191-c0c96868cc35" />

</div>
<h1 align=center>shank1-nx installer</h1>

The installer page of [shank1-nx](https://github.com/zhtipu/shank1-nx), the native Nintendo Switch port of Shank:
**https://zhtipu.github.io/shank1-nx-installer/**

You give it your own copy of the Xbox Live Arcade game (its content package, or a folder you extracted it into) and it
gives you a zip ready to extract into `sdmc:/switch/`. Everything runs in your browser: the game files never leave
your computer.

## What it does

<div align=center>
<img width="240" height="224" alt="spider-man-tobey-maguire" src="https://github.com/user-attachments/assets/5fa01c77-f0cd-49c1-a133-7a2308bfc8e1" />
</div>


1. Finds the game in what you chose: its content package (from `Content/0000000000000000/58410A62/000D0000/`, read in
   place, nothing is extracted to disk), the `58410A62` folder of an Xbox 360's `Content` folder, or an extracted game
   with its `default.xex`.
2. Identifies the game by the SHA-256 of `default.xex` (or of its decompressed image, for an executable another tool
   rebuilt) and picks its build from `release/manifest.json`.
3. Downloads that build's NRO and checks its SHA-256.
4. Makes the game's shaders from its own files: the shader containers of the effect files in
   `data-360/bootstrap/shaders` (`*.fxc`), translated to HLSL by Renut's translator
   (`wasm/renut_hlsl`), compiled to SPIR-V by DXC, turned into GLSL by SPIRV-Cross (`wasm/spirv_cross`), converted
   and optimised for deko3d (`lib/deko3d_glsl.js`, the build's passes) and packed (`lib/deko3d_package.js`) into
   `deko3d_glsl.bin`. The package must match the build's SHA-256, or nothing is written.
5. Packs the game files into `game_root.nxpack` + `game_root.nxidx` (the SDK's asset pack, `lib/nxpack.js`) and
   streams `shank1-nx.zip` to disk through the service worker, so the 2 GB package never has to fit in memory.

The update zip (`shank1-nx-update.zip`) holds only `shank1.nro`, its licenses and the shaders.

## What goes on the SD card

```text
/switch/shank1-nx/
  shank1.nro
  LICENSES.txt
  deko3d_glsl.bin           (the game's shaders, made from your copy)
  deko3d_shaders.bin        (the shaders compiled for the console; written as you play)
  game_root.nxpack          (the files of your game, in one file)
  game_root.nxidx
  shank1.toml               (optional settings)
```

Start it with the NSP from the [shank1-nx releases](https://github.com/zhtipu/shank1-nx/releases) (it holds only
the program and reads everything else from this folder), from the Homebrew Menu in title takeover mode, or with a
39-bit forwarder.

## Publishing a new build

1. Publish a release of shank1-nx with `shank1.nro` as an asset.
2. Update `release/manifest.json` with the page's own code (copy the NRO to `release/` first):

   ```text
   node test/release_node.mjs <game package or folder> release/shank1.nro <tested deko3d_glsl.bin> --version x.y.z --write
   ```

   The tested `deko3d_glsl.bin` is shank1-nx's `port/out/renut_switch_release/sd/deko3d_glsl.bin`: it names the
   shaders the NRO knows.
3. Push: the workflow downloads the NRO from the latest release, checks it against the manifest and deploys the page.

`node test/package_node.mjs <out.zip> <game>` builds the whole zip in Node with the same code, and
`node test/shank1_compare_node.mjs <dump> <glsl_opt>` compares the page's shaders with the native chain's.

## License

GPL-3.0 (`LICENSE`). Third-party parts: `NOTICES.md`.

Shank is a trademark of Klei Entertainment. This project is not affiliated with or endorsed by Klei
Entertainment, Nintendo or Microsoft, and it does not host or distribute any game data.
