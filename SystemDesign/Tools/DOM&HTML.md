# [ARIA](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA) 

**Accessible Rich Internet Applications** attributes are special HTML attributes that add semantic meaning and context to web elements, making dynamic content (like sliders, tabs, modals) understandable and usable for people with disabilities using assistive technologies (ATs) like screen readers, bridging gaps where standard HTML isn't enough

Use semantic HTML first: Always prefer native HTML elements (like `<button>, <nav>`) when possible, as they have built-in ARIA roles.
Use sparingly: Only add ARIA when semantic HTML isn't sufficient for accessibility.

- `aria-label:` Provides a text label for an element (e.g., `<button aria-label="Close">X</button>`).
- `aria-expanded:` Indicates if a collapsible section is open or closed (e.g., `aria-expanded="true"` or `"false"`).

---
<br />
<br />
<br />

# DOM vs Document

The **DOM is the abstract, theoretical model/interface**, while the document object is the **actual, concrete object (implementation)** that you use in JavaScript to interact with that model. 

You can view this DOM as live tree like structure, in Elements tab.
The DOM API defines *how a document(actual html document) should be modeled as object*.


## SPA (Single Page Applications)

In modern sites like Gmail or Facebook, you only trigger a **Document** request **once** (when you first load the site). After that, as you click around, the site stays on the same page and uses **Fetch** to grab new data and update the view. This is why it feels fast—it's avoiding that heavy "Document" reload.