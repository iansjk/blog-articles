---
title: Unexpected behavior with paragraph tags
tags:
  - "#languages"
created: 2025-01-11
---
**tl;dr**: be very careful with invalid HTML markup & tag auto-closure

I've been using [Kimchi Reader](https://kimchi-reader.app) to learn more Korean vocabulary. Usually this involves first going to YouTube and finding a video that looks interesting. Then if I encounter a word I don't understand, I'll save it using Kimchi Reader's browser extension, which also includes a screenshot of the video and the audio at that moment. Then I can review the word later in Anki using their Anki add-on. (This process is called [sentence mining](https://www.youtube.com/watch?v=QBcQJESGQvc&t=25s).)

I've sentence mined manually before by screenshotting my primary monitor, looking up the target word, and filling in Anki note fields manually, but it's quite time consuming. Kimchi Reader streamlines it into a few clicks, and as a result I've been able to mine a lot more vocabulary (around 700 entries at time of writing).[^1]

Overall the service and its extensions work well and I'm pretty happy with it. One downside is that some markup appears to be fixed and not customizable. For example, if I mined a [Sino-Korean word](https://en.wikipedia.org/wiki/Sino-Korean_vocabulary) like 의아하다 ("to be dubious, suspicious"), its corresponding hanja gets filled into a field like this:
```html
<span c-orange="">疑</span> (의) doubt, question, suspect<br><span c-orange="">訝</span> (아) express surprise, be surprised
```

(In the Kimchi Reader app itself--not in Anki--the `[c-orange]` portions are highlighted in orange, hence the attribute name.)

I already had a card template I was using that looked like this (where the target word and the English translation are in different fields):

> 의아하다 (疑訝): dubious; suspicious: Questionable and strange.

Since the markup isn't customizable, I wanted to avoid editing it since I'd have to edit all subsequent cards manually. I wanted the Kimchi Reader hanja markup to show up in parentheses and displayed inline, like the example quoted above. To do that, I ended up having to zero out the font size and then give the `[c-orange]` parts a non-zero font size to get the desired result:

<p class="codepen" data-height="300" data-default-tab="html,result" data-slug-hash="GgKxyjx" data-pen-title="Kimchi Reader hanja example" data-user="iansjk-the-decoder" style="height: 300px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
  <span>See the Pen <a href="https://codepen.io/iansjk-the-decoder/pen/GgKxyjx">
  Kimchi Reader hanja example</a> by Ian Kim (<a href="https://codepen.io/iansjk-the-decoder">@iansjk-the-decoder</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://public.codepenassets.com/embed/index.js"></script>

This looked like what I wanted, so I happily kept mining more words. But one day I noticed a card that looked like this instead:

![[Pasted image 20250111110029.png|Screenshot of Anki card showing broken layout due to multiple sets of hanja]]

The word had multiple sets of possible hanja, and I was seeing part of the original markup, despite trying to suppress it with `font-size: 0`. Confused, I tried reproducing it in CodePen by copy-pasting the Kimchi Reader markup along with my Anki card styles. Sure enough, the problem happens there too:

<p class="codepen" data-height="300" data-default-tab="html,result" data-slug-hash="azoEPvW" data-pen-title=".hanja wrapped in p" data-user="iansjk-the-decoder" style="height: 300px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
  <span>See the Pen <a href="https://codepen.io/iansjk-the-decoder/pen/azoEPvW">
  .hanja wrapped in p</a> by Ian Kim (<a href="https://codepen.io/iansjk-the-decoder">@iansjk-the-decoder</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://public.codepenassets.com/embed/index.js"></script>

I had noticed previously that Kimchi Reader's markup would use a `<hr>` to separate multiple hanja sets, so I added some styles to account for that. And I had used a `<span class="hanja">` to wrap the preexisting markup. But for some reason the `.hanja hr` styles didn't apply, and the second set of hanja weren't styled either.

The problem was that in cases with multiple sets of hanja, I now had invalid markup without realizing it. I had wrapped the whole thing (target word, hanja, and English) in a `<p>` tag. But `<hr>` is [*flow content*](https://developer.mozilla.org/en-US/docs/Web/HTML/Content_categories#flow_content), while paragraph tags can only contain [*phrasing content*](https://developer.mozilla.org/en-US/docs/Web/HTML/Content_categories#phrasing_content) (indeed, the linked MDN document even mentions that "sequences of phrasing content make up **paragraphs**"--emphasis mine). So the markup I had in my card template was invalid when there was an `<hr>` inside of the hanja field.

Now, I had made this mistake of invalid HTML nesting before, e.g. putting a `<div>` inside a `<span>`. In those cases, I might have gotten a console error from React about invalid markup, but didn't see anything obviously wrong.

But it turns out that paragraph tags allow **tag omission**--that is, the ending `</p>` tag can be omitted in some cases. [From MDN](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/p):

> **Tag omission**: The start tag is required. The end tag may be omitted if the `<p>` element is immediately followed by an [`<address>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/address), [`<article>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/article), [`<aside>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/aside), [`<blockquote>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/blockquote), [`<details>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/details), [`<div>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/div), [`<dl>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/dl), [`<fieldset>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/fieldset), [`<figcaption>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/figcaption), [`<figure>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/figure), [`<footer>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/footer), [`<form>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/form), [h1](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/Heading_Elements), [h2](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/Heading_Elements), [h3](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/Heading_Elements), [h4](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/Heading_Elements), [h5](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/Heading_Elements), [h6](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/Heading_Elements), [`<header>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/header), [`<hgroup>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/hgroup), [`<hr>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/hr), [`<main>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/main), [`<menu>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/menu), [`<nav>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/nav), [`<ol>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/ol), [`<pre>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/pre), [`<search>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/search), [`<section>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/section), [`<table>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/table), [`<ul>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/ul) or another `<p>` element, or if there is no more content in the parent element and the parent element is not an [`<a>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/a), [`<audio>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/audio), [`<del>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/del), [`<ins>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/ins), [`<map>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/map), [`<noscript>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/noscript) or [`<video>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/video) element, or an autonomous custom element.

And if you look closely, you'll see that `<hr>` is mentioned--if you open a paragraph tag with `<p>` and then put an `<hr>` inside of it, that *acts like `</p>`.* Indeed, in the CodePen reproduction, if you inspect the HTML this is what you see:

```html
<p>
  <span class="lemma">안배</span>
  <span class="hanja">
    <span c-orange="">按</span> (안) put hand on, press down with hand<br><span c-orange="">排</span> (배) row, rank, line
  </span>
</p> <!-- paragraph tag ends here -->
<hr>
<span c-orange="">按</span> (안) put hand on, press down with hand<br><span c-orange="">配</span> (배) match, pair; equal; blend
  : 
  <span class="english">distribution; assignment: An act of dividing up workload or handling the divided workload.</span>
<p></p> <!-- empty paragraph tag created -->
```

So that's why the `font-size: 0` style didn't apply to the second set of hanja: it was as if the paragraph tag ended early, so no styles could apply to it after the `<hr>`. This is also why something like `.hanja hr { display: none; }` would also have no effect.

As a bonus, the reason why `<div>` inside of a `<span>` doesn't have an obvious visual impact is because [`<span>` does not allow tag omission](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/span#technical_summary).

It's easy to forget that tag omission exists--just take a look at some of these other issues with folks discovering the same thing, especially with `p > div`:
- [Warn on invalid HTML hierarchy](https://github.com/preactjs/preact/issues/2399#issuecomment-596182535)
- [\[Bug\] Horizontal Rule (\<hr\>) inside paragraph (\<p\>) generates syntax error](https://github.com/adobe/brackets/issues/12971#top)
- [Nesting block level elements inside the \<p\> tag... right or wrong?](https://stackoverflow.com/questions/4291467/nesting-block-level-elements-inside-the-p-tag-right-or-wrong)

Even back in the days when I was writing HTML by hand, I don't think I ever (intentionally) used tag omission; and in JSX, it's simply not allowed, so it's easy to forget that it's [part of the HTML spec](https://html.spec.whatwg.org/multipage/syntax.html#syntax-tag-omission).

Inside of Anki specifically I couldn't tell that something was wrong in the markup because I didn't have access to the console, but since then I found and installed the [AnkiWebView Inspector](https://ankiweb.net/shared/info/317460320) add-on which should hopefully make debugging these card styling issues easier in the future.

---
[^1]: Unfortunately it doesn't plug into games (as far as I know), so if I wanted to use games for sentence mining (as I have done in the past) I'd still have to provide the image and audio myself. The image is easy enough but I wish I had an easy way to copy a few seconds of audio onto the clipboard to paste into Anki.
