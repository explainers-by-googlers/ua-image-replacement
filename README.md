# Explainer: User Agent Image Replacement API

This proposal is an early design sketch by an AI team in Chrome to describe the problem below and solicit
feedback on the proposed solution. It has not been approved to ship in Chrome.

## Authors:

- Jeremy Roman (Google LLC)

## Participate

- [explainers-by-googlers/ua-image-replacement (GitHub)](https://github.com/explainers-by-googlers/ua-image-replacement/issues)

## Table of Contents

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

- [Introduction](#introduction)
- [User-Facing Problem](#user-facing-problem)
  - [Goals](#goals)
  - [Non-goals](#non-goals)
- [Proposed Approach](#proposed-approach)
  - [Example](#example)
- [Alternatives considered](#alternatives-considered)
  - [CSS Pseudo-class only (:replaced-by-user-agent)](#css-pseudo-class-only-replaced-by-user-agent)
- [Web developer hints](#web-developer-hints)
- [Privacy & Security Considerations](#privacy--security-considerations)
- [Accessibility Considerations](#accessibility-considerations)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

---

## Introduction

Modern browsers can provide capabilities to augment the browsing experience by modifying media in the page on behalf of the user. The advent of generative AI makes it more likely that browsers will add such features.

Sometimes, the replacement content added at the user request might not match other content and functionality in the page, which could confuse the user. Even if this cannot be completely avoided, if authors can observe when replacement happens they can adjust the document to mitigate confusion (e.g., by hiding or adjusting other content).

For example, a user browsing an e-commerce site with a generic product image (e.g., a model wearing a jacket) might wish to imagine themselves wearing the item. The user agent uses generative AI technology to produce that image and present it in place of the model image. The page improves the user experience by removing text referring to the model's dimensions and the garment size depicted, as it may not be correct in the replacement image.

## User-Facing Problem

When a user uses a browser feature that affects media in the page, it's sometimes confusing whether other page content is appropriate for the original media, replacement media, or both. While user agents should make an effort to make the distinction between UA- and author-provided content clear, users get the best experience when both coordinate to present a harmonious experience in which this confusion is kept to a minimum.

A Chrome feature which offers image replacement has heard feedback from multiple websites that this capability would enable them to both avoid a confusing user experience and improve their own analytics to understand how users of these new features interact with their site (for example, whether they are more likely to browse or make a purchase).

### Goals

- **Simplicity**: An API which is simple, easy to adopt, and naturally degrades to existing behavior is going to have both a low barrier for adoption and minimize compatibility risk.
- **Privacy**: User agents should be empowered to limit the information available to the page in accordance with their own stance and user preferences. User agents can continue to withhold the fact that an image is replaced if this is sensitive, or surface only limited information about the nature of the replacement content, to reduce the exposure of additional information to the site.

### Non-goals

- **Replacement media access and isolation**: This API does not provide a mechanism for the page to access replacement media, nor does it provide media isolation itself. Instead, the mechanism of replacement—and whether and how replacement content is isolated from the page—is the responsibility of the underlying browser feature performing the replacement.
- **Content-driven replacement**: This API does not propose to allow authors to initiate such replacements of any kind.
- **Enumeration of possible image replacement features**: A future enhancement could provide information about what kind of replacement image is being used, but since this may have privacy and compatibility risk implications, it's left out of scope for now. It's likely that, if this information is added, it will be optional.

## Proposed Approach

We propose extending `HTMLImageElement` with two new DOM events and one read-only boolean attribute.

1. `uareplacestart` event: Fired when the user agent begins substituting the image content.
2. `uareplaceend` event: Fired when the substitution finishes, fails, or is reverted.
3. `replacedByUserAgent` boolean property: A read-only attribute on `HTMLImageElement` indicating whether the element currently displays substitute content provided by the user agent.

When image replacement occurs, what changes to the document cause image replacement to end, and whether pages are notified or this information is withheld from the page is user agent (and even feature) specific behavior and not covered here.

### Example

When an e-commerce clothing page renders a product image, it often attaches interactive overlays (such as a zoom viewer which uses other content to render an enlarged version, and would confuse the user if the original image reappeared). When the user agent starts replacing the product image with a virtual try-on render of the user, the page can remove this control.

```javascript
const imageEl = document.querySelector('img.product-view');
const zoomLens = document.querySelector('.zoom-lens');

// Listen for replacement start
imageEl.addEventListener('uareplacestart', () => {
  // Temporarily disable custom interactive zoom while substitution is in progress
  zoomLens.style.display = 'none';
});

// Listen for replacement end
imageEl.addEventListener('uareplaceend', () => {
  // Re-enable interactive zoom if replacement was reverted or canceled
  zoomLens.style.display = 'block';
});

// Check if the replacement happened before this script executed.
if (imageEl.replacedByUserAgent) {
  zoomLens.style.display = 'none';
}
```

Sites can adjust style by adding attributes, classes, etc. when these events fire.

A site can track which elements are replaced by using a capturing listener on the document for when replacements start, and then observing when replacement ends on each such element.

## Alternatives considered

### CSS Pseudo-class only (:replaced-by-user-agent)

While would be useful for making style changes, it is not ergonomic to monitor from script for other changes. This is considered as a possible future enhancement.

## Web developer hints

While this explainer focuses on observation primitives (`uareplacestart`, `uareplaceend`, and `replacedByUserAgent`), it's reasonable for developers to prefer that such features not be used, or possibly be used, on their sites, because it is impractical to adapt the site when the user uses them without significantly degrading functionality or the user's experience. The browser might then not offer that feature, or offer a different experience (e.g., in a side panel or pop-up window). Deciding on a mechanism for developer hints is out of scope for this initial proposal, but is an area for potential future work.

Future work in this direction should consider this sort of case and might want to establish a registry of different kinds of transformations that sites could hint about. For example, it might look something like `<html notransform="interior-decor-generation">` (or an element-scoped equivalent). Some effort would be required to make sure that vendors with similar features eventually supported interoperable tokens. This is somewhat analogous to the existing `translate` and `autocomplete` attributes.

If there is browser vendor and web developer interest, it would be useful at that time to also consider whether it would also be useful for hints to opt _in_ to offering particular features, as a signal to browsers that the feature is likely to be particularly beneficial, especially for browsers that support transformations of this kind but wish to take a more cautious approach to applying them.

## Privacy & Security Considerations
- **Data Protection and Image Isolation**: This API only notifies the page of lifecycle transitions and does not itself isolate replacement image data from script. The privacy sensitivity of replacement media depends on the specific browser feature:
  - Transformations that incorporate sensitive user data or intent (such as virtual try-on using a user's photo) require strong isolation. For such features, user agents are expected to isolate the replacement image so that page scripts cannot inspect or exfiltrate it (for example, by presenting the replacement within user agent shadow DOM or cross-origin contexts such that canvas pixel readback reflects only the original image resource).
  - Other transformations might not be privacy-sensitive (for example, client-side photo colorization, contrast enhancement, or super-resolution upscaling), where isolation from page scripts may not be necessary.
  It is therefore up to the user agent to evaluate the sensitivity of a given transformation and provide appropriate isolation.
- **Anti-Fingerprinting**: `uareplacestart` and `uareplaceend` events only fire when an actual replacement operation is initiated by explicit user intent or user-configured browser policy. Web pages MUST NOT be able to probe for replacement capabilities by observing side-effects without the user using such a feature.

## Accessibility Considerations
- **Alt Text and Screen Readers**: When a user agent replaces image visual content (e.g., text translation in image or virtual try-on), the UA may override the accessible description (`alt` text or ARIA attributes) so assistive technologies accurately convey the substitute content to visually impaired users.
