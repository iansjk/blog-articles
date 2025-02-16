While working on [Sanity;Gone](https://sanitygone.help) I was trying to determine what minimum version of Safari to use. I say Safari specifically because it's the last major browser that has updates tied to the operating system version, unlike Firefox, Chrome, and Edge (the "evergreen" browsers). So I've compiled this brief list based on some major features we would use that cannot be easily transpiled[1].

**Note:** A version change like 13.0 → 13.1 is [considered a *major* version update by the Safari team](https://github.com/web-platform-dx/web-features/issues/173#issuecomment-1555386336), *not* a minor version like [semver](https://semver.org/). As such, 13.0 and 13.1 are listed separately.
# Safari 10
- [`Intl.NumberFormat` basic support](https://caniuse.com/mdn-javascript_builtins_intl_numberformat), e.g. `format()`
# Safari 13
## 13.0
- `Intl.NumberFormat` `formatToParts()`
# Safari 14
## 14.0
- `Intl.RelativeTimeFormat`
## 14.1
- `gap, row-gap, column-gap` inside of flex layout
- `Intl.ListFormat`
# Safari 15
## 15.0
- `focus()` with `{ preventScroll: true }`
## 15.4
- `:has`
- `Intl.NumberFormat` `formatRange()` , `formatRangeToParts()`
- `<dialog>` `showModal()
- `revert-layer`
	- Note: be extremely careful about using this when using this in Safari 15.x in combination with `contenteditable` (can cause them to be uneditable due to reverting `-webkit-user-modify`). In Safari 16+ it should be fine
## 15.5
- `inert`
# Safari 17
## 17.0
- `popover`, `popovertarget`
## 17.4
- `text-wrap: wrap`, `text-wrap: nowrap`
## 17.5
- [`text-wrap: balance, stable`](https://developer.mozilla.org/en-US/docs/Web/CSS/text-wrap#browser_compatibility)
	- `pretty` not supported in any version of Firefox/Safari yet
# Technical Preview
- `overflow-anchor`

---
[1] For example, operators like `??` and `??=` can be trivially transpiled, but features like the `inert` attribute or the `Intl` API require polyfills on browsers that do not support them natively.