# Security & Privacy Self-Review Questionnaire

These are responses to the questions from the [W3C Self-Review Questionnaire](https://www.w3.org/TR/security-privacy-questionnaire/).

---

## 1. What information does this feature expose, and for what purposes?

This feature exposes one piece of information (namely, whether an `HTMLImageElement` is currently displaying substitute content). Authors can access its current value and events when it changes.

This allows:
- **UI Coordination & User Experience**: Allows web applications to adapt their UI when an image's visual representation is replaced (for example, removing interactive zoom lenses that would erroneously show the original image, hiding model dimensions/garment sizes that no longer apply to a virtual try-on image, or updating captions).
- **Analytics**: Allows sites to understand aggregate user engagement with image replacement features (e.g., whether users engaging with virtual try-on are more likely to make a purchase).


## 2. Do features in your specification expose the minimum amount of information necessary to implement the intended functionality?

Yes. The API is strictly minimal and exposes only coarse lifecycle notifications (`uareplacestart`, `uareplaceend`) and a single boolean property (`replacedByUserAgent`).

The API deliberately does **not** expose:
- The replacement image data or pixel bitmap
- URLs (`blob:`, `data:`, or `http:`) pointing to the generated/substituted asset
- User prompt text or generation configuration parameters
- Detailed categorization of the replacement feature (e.g., virtual try-on vs. translation)
- Underlying AI models or backend service metadata


## 3. Do the features in your specification expose personal information, personally-identifiable information (PII), or information derived from either?

No. The API does not expose any personal information, PII, or derived attributes to script. While the user agent may use personal data (e.g., a photo of the user for virtual try-on) to render replacement content, that content and the underlying data remain isolated inside the user agent and are not accessible to page scripts.


## 4. How do the features in your specification deal with sensitive information?

The API itself is strictly an observation and lifecycle notification mechanism and does not implement the image replacement or its isolation.

However, user agents implementing image replacement features must evaluate the sensitivity of the replacement contentr .

- **Media Isolation**: When replacement media incorporates private user data or intent (such as generative AI virtual try-on), user agents are expected to provide isolation that prevents page scripts from reading or exfiltrating the replacement pixels (e.g., isolating rendered content so canvas readback operations see only the original resource). For transformations that are not privacy-sensitive (such as auto-colorization or upscaling), user agents may determine that such isolation is not required.
- **No Input Exposure**: No user inputs, prompts, or personal photos used to create the replacement are exposed to the web page.
- **UA Discretion**: User agents may withhold firing events and leave `replacedByUserAgent` set to `false` if the fact of use of a replacement feature is itself too sensitive.


## 5. Does data exposed by your specification carry related but distinct information that may not be obvious to users?

Firing replacement events indicates that an image replacement feature was activated on a particular image element. In specific contexts, an origin might infer user interest in a specific browser feature (such as virtual try-on). However, the API provides only a coarse boolean signal without fine-grained user parameters or preferences.


## 6. Do the features in your specification introduce state that persists across browsing sessions?

No. All state introduced by this API (`replacedByUserAgent` and event listeners) is strictly in-memory and ephemeral, bound to the lifetime of the `HTMLImageElement` in the active document. No persistent client-side storage (cookies, `localStorage`, `indexedDB`, etc.) is created or modified.


## 7. Do the features in your specification expose information about the underlying platform to origins?

No, except to the extent that it is a platform on which an image replacement feature is supported by the user agent.


## 8. Does this specification allow an origin to send data to the underlying platform?

No. The API is strictly unidirectional from the user agent to the web page. Pages cannot trigger image replacements, provide prompts, or pass images to the browser's replacement engine via this API.


## 9. Do features in this specification enable access to device sensors?

No.


## 10. Do features in this specification enable new script execution/loading mechanisms?

No.


## 11. Do features in this specification allow an origin to access other devices?

No.


## 12. Do features in this specification allow an origin some measure of control over a user agent's native UI?

No. The user agent maintains full control over all native UI elements, prompts, and controls used to manage image replacements.


## 13. What temporary identifiers do the features in this specification create or expose to the web?

None.


## 14. How does this specification distinguish between behavior in first-party and third-party contexts?

The API operates on `HTMLImageElement`s in the DOM. Any script with access to the element (including third-party scripts embedded in the page) can observe the events and property. Standard cross-origin iframe boundaries apply, preventing cross-origin frames from inspecting elements in parent or sibling documents. User agents may also choose to restrict replacement features to top-level or same-origin contexts.


## 15. How do the features in this specification work in the context of a browser's Private Browsing or Incognito mode?

The API behaves identically in normal and Private Browsing / Incognito modes. Because no persistent storage or cross-session state is created, the API cannot be used to correlate sessions across private and normal browsing modes. However, some browsers may choose not to offer some image replacement features in private browsing modes (e.g., because they rely on user data, settings, or subscriptions linked to an account).


## 16. Does this specification have both "Security Considerations" and "Privacy Considerations" sections?

At present the explainer's coverage of this is cursory and the draft specification is a placeholder. However, these considerations are expected to be straightforward (as covered in this questionnaire).


## 17. Do features in your specification enable origins to downgrade default security protections?

No. The API does not relax the Same-Origin Policy, Content Security Policy (CSP), or any other web security protections.


## 18. What happens when a document that uses your feature is kept alive in BFCache (instead of getting destroyed) after navigation, and potentially gets reused on future navigations back to the document?

When a document is in the BFCache (not fully active):
- No `uareplacestart` or `uareplaceend` events will be dispatched to it.
- If an image was replaced prior to entering BFCache, the user agent may maintain or revert the replacement per its policy.
- When the document is restored and becomes fully active again, `img.replacedByUserAgent` will accurately reflect the visual state of the image element upon restoration.


## 20. Does your spec define when and how new kinds of errors should be raised?

The API does not define new JavaScript exceptions or error objects.

If an image replacement operation fails, is canceled by the user, or is aborted by the user agent, the UA dispatches `uareplaceend` (with `replacedByUserAgent` remaining `false` or reverting to `false`). No internal error messages, diagnostics, or stack traces are exposed to the page.


## 21. Does your feature allow sites to learn about the user's use of assistive technology?

If a user agent performs image replacement for accessibility purposes (e.g., enhancing contrast or translating embedded image text for a screen reader user), exposing replacement events could theoretically allow an origin to infer that an accessibility tool was used.

To mitigate this risk, user agents are explicitly permitted to treat accessibility-related replacements as internal rendering adaptations and withhold replacement events and the `replacedByUserAgent` flag from the document.


## 22. What should this questionnaire have asked?

No other questions are identified (though additional considerations may apply to particular features which integrate with the API).
