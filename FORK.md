# Why this fork exists

Upstream vscode-js-debug **cannot debug a Chrome extension**, and that is
deliberate: browser-extension support is closed as `*out-of-scope`
([#945](https://github.com/microsoft/vscode-js-debug/issues/945),
[#1794](https://github.com/microsoft/vscode-js-debug/issues/1794)). The
maintainer's position:

> We need to attach to browser-level frames and service workers, currently we
> filter and only attach to `page` types. This is easy to fix. […] Sources in
> extensions get some random URL prefix like `chrome-extension://gmocg…/…`. We
> don't have any way to map this in the debugger […] I don't plan to support
> this in the foreseeable future.

Since every distribution (mason included) ships upstream releases, there is no
packaged adapter that can do it. This fork carries
[PR #2361](https://github.com/microsoft/vscode-js-debug/pull/2361) —
"add WebExtension debugging support via extensionPath config" — on top of
upstream so the built adapter can.

## Layout

- `main` — tracks upstream, unmodified.
- `webext` — upstream + PR #2361, plus `.github/workflows/webext-release.yml`
  and this file. **Releases are cut from here.**

Nothing else is changed. When PR #2361 lands upstream, this fork has done its
job and should be archived.

## Releasing

Tag `webext` as `v<upstream version>-webext.<n>` and push, or run the
**webext release** workflow manually. It builds `gulp dapDebugServer`, asserts
the extension support is actually present in the bundle, and publishes
`js-debug-dap-webext.tar.gz`.

The tarball's layout matches upstream's own `js-debug-dap` asset, so it is a
drop-in replacement for what mason installs.

## Using it

```sh
mkdir -p ~/.local/share/nvim/js-debug-webext
curl -L https://github.com/huiyu/vscode-js-debug/releases/latest/download/js-debug-dap-webext.tar.gz \
  | tar -xz -C ~/.local/share/nvim/js-debug-webext
```

Point the debug adapter at
`~/.local/share/nvim/js-debug-webext/src/dapDebugServer.js` and give the launch
configuration an `extensionPath` pointing at the extension's **build output**
directory. No `webRoot` or `sourceMapPathOverrides` needed — the PR derives the
extension id, target filtering and sourcemap mapping from that one path.

## Known gap

PR #2361's **launch** mode passes `--load-extension`, which Chrome removed in
137+ (`--enable-unsafe-extension-debugging` does not bring it back). Attach mode
is unaffected. Routing initial load through CDP `Extensions.loadUnpacked` — which
the PR already uses for hot reload — would fix launch mode, and is worth sending
upstream to the PR author.
