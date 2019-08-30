# 3 Semantics, structure, and APIs of HTML documents

## 3.1 Documents

> Interactive user agents typically expose the Document object's URL in their user interface. This is the primary mechanism by which a user can tell if a site is attempting to impersonate another.

### 3.1.1 `The Document object` Document 对象

DOM defines a Document interface, which this specification extends significantly.

```interface
enum DocumentReadyState { "loading", "interactive", "complete" };
typedef (HTMLScriptElement or SVGScriptElement) HTMLOrSVGScriptElement;

[OverrideBuiltins]
partial interface Document {
  // resource metadata management
  [PutForwards=href, Unforgeable] readonly attribute Location? location;
  attribute USVString domain;
  readonly attribute USVString referrer;
  attribute USVString cookie;
  readonly attribute DOMString lastModified;
  readonly attribute DocumentReadyState readyState;

  // DOM tree accessors
  getter object (DOMString name);
  [CEReactions] attribute DOMString title;
  [CEReactions] attribute DOMString dir;
  [CEReactions] attribute HTMLElement? body;
  readonly attribute HTMLHeadElement? head;
  [SameObject] readonly attribute HTMLCollection images;
  [SameObject] readonly attribute HTMLCollection embeds;
  [SameObject] readonly attribute HTMLCollection plugins;
  [SameObject] readonly attribute HTMLCollection links;
  [SameObject] readonly attribute HTMLCollection forms;
  [SameObject] readonly attribute HTMLCollection scripts;
  NodeList getElementsByName(DOMString elementName);
  readonly attribute HTMLOrSVGScriptElement? currentScript; // classic scripts in a document tree only

  // dynamic markup insertion
  [CEReactions] Document open(optional DOMString unused1, optional DOMString unused2); // both arguments are ignored
  WindowProxy? open(USVString url, DOMString name, DOMString features);
  [CEReactions] void close();
  [CEReactions] void write(DOMString... text);
  [CEReactions] void writeln(DOMString... text);

  // user interaction
  readonly attribute WindowProxy? defaultView;
  boolean hasFocus();
  [CEReactions] attribute DOMString designMode;
  [CEReactions] boolean execCommand(DOMString commandId, optional boolean showUI = false, optional DOMString value = "");
  boolean queryCommandEnabled(DOMString commandId);
  boolean queryCommandIndeterm(DOMString commandId);
  boolean queryCommandState(DOMString commandId);
  boolean queryCommandSupported(DOMString commandId);
  DOMString queryCommandValue(DOMString commandId);

  // special event handler IDL attributes that only apply to Document objects
  [LenientThis] attribute EventHandler onreadystatechange;

  // also has obsolete members
};
Document includes GlobalEventHandlers;
Document includes DocumentAndElementEventHandlers;
```

`The Document has an HTTPS state (an HTTPS state value), initially "none", which represents the security properties of the network channel used to deliver the Document's data.`
Document 有一个 HTTPS 状态 （一个 HTTPS 状态值）， 初始为 "none"，它表示传递 Document 的数据的网络频道的安全属性。
The Document has a referrer policy (a referrer policy), initially the empty string, which represents the default referrer policy used by fetches initiated by the Document.

`The Document has a CSP list, which is a CSP list containing all of the Content Security Policy objects active for the document. The list is empty unless otherwise specified.`
Document 有一个 引荐来源策略 （一个 引荐来源策略）， 初始为空字符串，它表示由 Document 发起的 获取 的默认 引荐来源策略。

`The Document has a feature policy, which is a feature policy, which is initially empty.`
Document 有一个 CSP 列表，它是一个在该上下文中活动的 内容安全策略 对象的列表。 该列表在没有指定时是空的。

`The Document has a module map, which is a module map, initially empty.`
Document 有一个 模块映射， 是一个初始为空的 模块映射。

### 3.1.2 The DocumentOrShadowRoot interface

DOM defines the DocumentOrShadowRoot mixin, which this specification extends.

```interface
partial interface mixin DocumentOrShadowRoot {
  readonly attribute Element? activeElement;
};
```

### 3.1.3 Resource metadata management

#### document.referrer

The referrer attribute must return the document's referrer.

#### document.cookie

The cookie attribute represents the cookies of the resource identified by the document's URL.

A Document object that falls into one of the following conditions is a cookie-averse Document object:

- A Document object whose browsing context is null.
- A Document whose URL's scheme is not a network scheme.

On getting, if the document is a cookie-averse Document object, then the user agent must return the empty string. Otherwise, if the Document's origin is an opaque origin, the user agent must throw a "SecurityError" DOMException. Otherwise, the user agent must return the cookie-string for the document's URL for a "non-HTTP" API, decoded using UTF-8 decode without BOM. [COOKIES]

On setting, if the document is a cookie-averse Document object, then the user agent must do nothing. Otherwise, if the Document's origin is an opaque origin, the user agent must throw a "SecurityError" DOMException. Otherwise, the user agent must act as it would when receiving a set-cookie-string for the document's URL via a "non-HTTP" API, consisting of the new value encoded as UTF-8. [COOKIES][encoding]

> Since the cookie attribute is accessible across frames, the path restrictions on cookies are only a tool to help manage which cookies are sent to which parts of the site, and are not in any way a security feature.

> The cookie attribute's getter and setter synchronously access shared state. Since there is no locking mechanism, other browsing contexts in a multiprocess user agent can modify cookies while scripts are running. A site could, for instance, try to read a cookie, increment its value, then write it back out, using the new value of the cookie as a unique identifier for the session; if the site does this twice in two different browser windows at the same time, it might end up using the same "unique" identifier for both sessions, with potentially disastrous effects.

#### document.lastModified

Returns the date of the last modification to the document, as reported by the server, in the form "MM/DD/YYYY hh:mm:ss", in the user's local time zone.If the last modification date is not known, the current time is returned instead.

#### document.readyState

Returns "loading" while the Document is loading, "interactive" once it is finished parsing but still loading subresources, and "complete" once it has loaded.

The readystatechange event fires on the Document object when this value changes.

The DOMContentLoaded event fires after the transition to "interactive" but before the transition to "complete", at the point where all subresources apart from async script elements have loaded.

### 3.1.4 `DOM tree accessors`DOM 树访问器

#### document.head

The head element of a document is the first head element that is a child of the html element, if there is one, or null otherwise.

#### document.title [ = value ]

Returns the document's title, as given by the title element for HTML and as given by the SVG title element for SVG.
Can be set, to update the document's title. If there is no appropriate element to update, the new value is ignored.

The title attribute must, **on getting**, run the following algorithm:

1. If the document element is an SVG svg element, then let value be the child text content of the first SVG title element that is a child of the document element.
2. Otherwise, let value be the child text content of the title element, or the empty string if the title element is null.
3. Strip and collapse ASCII whitespace in value.
4. Return value.

**On setting**, the steps corresponding to the first matching condition in the following list must be run:
**If the document element is an SVG svg element**

1. If there is an SVG title element that is a child of the document element, let element be the first such element.
2. Otherwise:
   1. Let element be the result of creating an element given the document element's node document, title, and the SVG namespace.
   2. Insert element as the first child of the document element.
3. String replace all with the given value within element.

If the document element is in the HTML namespace

1. If the title element is null and the head element is null, then return.
2. If the title element is non-null, let element be the title element.
3. Otherwise:
   1. Let element be the result of creating an element given the document element's node document, title, and the HTML namespace.
   2. Append element to the head element.
4. String replace all with the given value within element.

#### document.body [ = value ]

The body attribute, on getting, must return the body element of the document (either a body element, a frameset element, or null). On setting, the following algorithm must be run:

1. If the new value is not a body or frameset element, then throw a "HierarchyRequestError" DOMException.
2. Otherwise, if the new value is the same as the body element, return.
3. Otherwise, if the body element is not null, then replace the body element with the new value within the body element's parent and return.
4. Otherwise, if there is no document element, throw a "HierarchyRequestError" DOMException.
5. Otherwise, the body element is null, but there's a document element. Append the new value to the document element.

#### HTMLCollection

- document.images
- document.embeds
- document.plugins
- document.links
- document.forms
- document.scripts

## 3.2 Elements

### 3.2.1 Semantics

### 3.2.2 Elements in the DOM

The basic interface, from which all the HTML elements' interfaces inherit, and which must be used by elements that have no additional requirements, is the HTMLElement interface.

```interface
[Exposed=Window,
 HTMLConstructor]
interface HTMLElement : Element {
  // metadata attributes
  [CEReactions] attribute DOMString title;
  [CEReactions] attribute DOMString lang;
  [CEReactions] attribute boolean translate;
  [CEReactions] attribute DOMString dir;

  // user interaction
  [CEReactions] attribute boolean hidden;
  void click();
  [CEReactions] attribute DOMString accessKey;
  readonly attribute DOMString accessKeyLabel;
  [CEReactions] attribute boolean draggable;
  [CEReactions] attribute boolean spellcheck;
  [CEReactions] attribute DOMString autocapitalize;

  [CEReactions] attribute [TreatNullAs=EmptyString] DOMString innerText;

  ElementInternals attachInternals();
};

HTMLElement includes GlobalEventHandlers;
HTMLElement includes DocumentAndElementEventHandlers;
HTMLElement includes ElementContentEditable;
HTMLElement includes HTMLOrSVGElement;

// Note: intentionally not [HTMLConstructor]
[Exposed=Window]
interface HTMLUnknownElement : HTMLElement { };
```

The element interface for an element with name name in the HTML namespace is determined as follows:

1. If name is applet, bgsound, blink, isindex, keygen, multicol, nextid, or spacer, then return HTMLUnknownElement.
2. If name is acronym, basefont, big, center, nobr, noembed, noframes, plaintext, rb, rtc, strike, or tt, then return HTMLElement.
3. If name is listing or xmp, then return HTMLPreElement.
4. Otherwise, if this specification defines an interface appropriate for the element type corresponding to the local name name, then return that interface.
5. If other applicable specifications define an appropriate interface for name, then return the interface they define.
6. If name is a valid custom element name, then return HTMLElement.
7. Return HTMLUnknownElement.

### 3.2.3 HTML element constructors

### 3.2.4 Element definitions

#### 3.2.4.1 Attributes

### 3.2.5 Content models

#### 3.2.5.1 The "nothing" content model

#### 3.2.5.2 Kinds of content

- 1.Metadata content

  Metadata content is content that sets up the presentation or behavior of the rest of the content, or that sets up the relationship of the document with other documents, or that conveys other "out of band" information.

  - base
  - link
  - meta
  - noscript
  - script
  - style
  - template
  - title

- 2.Flow content

  Most elements that are used in the body of documents and applications are categorized as flow content.

  ![ha]('../images/flow_content_elements.png')

- 3.Sectioning content

  Sectioning content is content that defines the scope of headings and footers.

  - article
  - aside
  - nav
  - section

- 4.Heading content

  Heading content defines the header of a section (whether explicitly marked up using sectioning content elements, or implied by the heading content itself).

  - h1
  - h2
  - h3
  - h4
  - h5
  - h6
  - hgroup

- 5.Phrasing content

  Phrasing content is the text of the document, as well as elements that mark up that text at the intra-paragraph level. Runs of phrasing content form paragraphs.

  - a
  - abbr
  - area (if it is a descendant of a map element)audio
  - b
  - bdi
  - bdo
  - br
  - button
  - canvas
  - cite
  - code
  - data
  - datalist
  - del
  - dfn
  - em
  - embed
  - i
  - iframe
  - img
  - input
  - ins
  - kbd
  - label
  - link (if it is allowed in the body)
  - map
  - mark
  - MathML math
  - meta (if the itemprop attribute is present)meter
  - noscript
  - object
  - output
  - picture
  - progress
  - q
  - ruby
  - s
  - samp
  - script
  - select
  - slot
  - small
  - span
  - strong
  - sub
  - sup
  - SVG svg
  - template
  - textarea
  - time
  - u
  - var
  - video
  - wbr
  - autonomous custom elements
  - text

- 6.Embedded content

  Embedded content is content that imports another resource into the document, or content from another vocabulary that is inserted into the document.

  - audio
  - canvas
  - embed
  - iframe
  - img
  - MathML math
  - object
  - picture
  - SVG svg
  - video

- 7.Interactive content

  Interactive content is content that is specifically intended for user interaction.

  - a (if the href attribute is present)
  - audio (if the controls attribute is present)
  - button
  - details
  - embed
  - iframe
  - img (if the usemap attribute is present)
  - input (if the type attribute is not in the Hidden state)
  - label
  - object (if the usemap attribute is present)
  - select
  - textarea
  - video (if the controls attribute is present)

- 8.Palpable content

  As a general rule, elements whose content model allows any flow content or phrasing content should have at least one node in its contents that is palpable content and that does not have the hidden attribute specified.

  > Palpable content makes an element non-empty by providing either some descendant non-empty text, or else something users can hear (audio elements) or view (video or img or canvas elements) or otherwise interact with (for example, interactive form controls).

  This requirement is not a hard requirement, however, as there are many cases where an element can be empty legitimately, for example when it is used as a placeholder which will later be filled in by a script, or when the element is part of a template and would on most pages be filled in but on some pages is not relevant.

  Conformance checkers are encouraged to provide a mechanism for authors to find elements that fail to fulfill this requirement, as an authoring aid.

  ```Palpable content
  a
  abbr
  address
  article
  aside
  audio (if the controls attribute is present)
  b
  bdi
  bdo
  block
  quote
  button
  canvas
  cite
  code
  data
  details
  dfn
  divdl (if the element's children include at least one name-value group)
  em
  embed
  fieldset
  figure
  footer
  form
  h1
  h2
  h3
  h4
  h5
  h6
  header
  hgroup
  i
  iframe
  img
  input (if the type attribute is not in the Hidden state)
  ins
  kbd
  label
  main
  map
  mark
  MathML math
  menu (if the element's children include at least one li element)
  meter
  nav
  object
  ol (if the element's children include at least one li element)
  output
  p
  pre
  progress
  q
  ruby
  s
  samp
  section
  select
  small
  span
  strong
  sub
  sup
  SVG svg
  table
  textarea
  time
  u
  ul (if the element's children include at least one li element)
  var
  video
  autonomous custom elements
  text that is not inter-element whitespace
  ```

- 9.Script-supporting elements

  Script-supporting elements are those that do not represent anything themselves (i.e. they are not rendered), but are used to support scripts, e.g. to provide functionality for the user.

  - script
  - template

#### 3.2.5.3 Transparent content models

ruby

#### 3.2.5.4 Paragraphs

p

### 3.2.6 Global attributes

The following attributes are common to and may be specified on all HTML elements (even those not defined in this specification):

- accesskey
- autocapitalize
- contenteditable
- dir
- draggable
- enterkeyhint
- hidden
- inputmode
- is
- itemid
- itemprop
- itemref
- itemscope
- itemtype
- lang
- nonce
- spellcheck
- style
- tabindex
- title
- translate

DOM standard defines the user agent requirements for the class, id, and slot attributes for any element in any namespace. [DOM]
The class, id, and slot attributes may be specified on all HTML elements.

When specified on HTML elements, the id attribute value must be unique amongst all the IDs in the element's tree and must contain at least one character. The value must not contain any ASCII whitespace.

The following event handler content attributes may be specified on any HTML element:

- onabort
- onauxclick
- onblur\*
- oncancel
- oncanplay
- oncanplaythrough
- onchange
- onclick
- onclose
- oncontextmenu
- oncopy
- oncuechange
- oncut
- ondblclick
- ondrag
- ondragend
- ondragenter
- ondragexit
- ondragleave
- ondragover
- ondragstart
- ondrop
- ondurationchange
- onemptied
- onended
- onerror\*
- onfocus\*
- onformdata
- oninput
- oninvalid
- onkeydown
- onkeypress
- onkeyup
- onload\*
- onloadeddata
- onloadedmetadata
- onloadstart
- onmousedown
- onmouseenter
- onmouseleave
- onmousemove
- onmouseout
- onmouseover
- onmouseup
- onpaste
- onpause
- onplay
- onplaying
- onprogress
- onratechange
- onreset
- onresize\*
- onscroll\*
- onsecuritypolicyviolation
- onseeked
- onseeking
- onselect
- onstalled
- onsubmit
- onsuspend
- ontimeupdate
- ontoggle
- onvolumechange
- onwaiting
- onwheel

Custom data attributes (e.g. data-foldername or data-msgid) can be specified on any HTML element, to store custom data, state, annotations, and similar, specific to the page.

#### 3.2.6.1 The title attribute

The title attribute represents advisory information for the element, such as would be appropriate for a tooltip. On a link, this could be the title or a description of the target resource; on an image, it could be the image credit or a description of the image; on a paragraph, it could be a footnote or commentary on the text; on a citation, it could be further information about the source; on interactive content, it could be a label for, or instructions for, use of the element; and so forth. The value is text.

The advisory information of an element is the value that the following algorithm returns, with the algorithm being aborted once a value is returned. When the algorithm returns the empty string, then there is no advisory information.

1. If the element has a title attribute, then return its value.
2. If the element has a parent element, then return the parent element's advisory information.
3. Return the empty string.

#### 3.2.6.2 The lang and xml:lang attributes

[The lang and xml:lang attributes](https://html.spec.whatwg.org/multipage/dom.html#the-lang-and-xml:lang-attributes)

#### 3.2.6.3 The translate attribute

[The translate attribute](https://html.spec.whatwg.org/multipage/dom.html#the-translate-attribute)

#### 3.2.6.4 The dir attribute

- ltr
- rtl

#### 3.2.6.5 The style attribute

**element.style:**
Returns a CSSStyleDeclaration object for the element's style attribute.

#### 3.2.6.6 Embedding custom non-visible data with the data-\* attributes

A custom data attribute is an attribute in no namespace whose name starts with the string "data-", has at least one character after the hyphen, is XML-compatible, and contains no ASCII upper alphas.

**element.dataset**
Returns a `DOMStringMap` object for the element's data-\* attributes.
Hyphenated names become camel-cased. For example, `data-foo-bar=""` becomes `element.dataset.fooBar`.

### 3.2.7 The innerText IDL attribute

**element.innerText [ = value ]**
Returns the element's text content "as rendered".
Can be set, to replace the element's children with the given value, but with line breaks converted to br elements.

### 3.2.8 Requirements relating to the bidirectional algorithm

#### 3.2.8.1 Authoring conformance criteria for bidirectional-algorithm formatting characters

#### 3.2.8.2 User agent conformance criteria

### 3.2.9 Requirements related to ARIA and to platform accessibility APIs
