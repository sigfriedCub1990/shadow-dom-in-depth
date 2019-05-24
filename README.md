# Shadow DOM

> Atención! Todo es acerca de V1 Spec.

Este documento es más tuyo que mío. Me hace feliz que haya podido ayudar a las personas. Para hacerlo mejor moví este document de [the original gist](https://gist.github.com/praveenpuglia/0832da687ed5a5d7a0907046c9ef1813) a este repositorio para que varias personas puedan colaborarr y mejorarlo.

Si te gusta lo que encuentras aquí, por favor crea un `issue` con ideas para saber qué otras cosas podemos agregar a este repositorio. Como ejemeplos, imágenes, representaciones gráficas para las terminologías, etc. Vía `issues`.  Hagamos que todos amen la plataforma :). 

## Qué hay de nuevo?

### Traducciones
Inglés (original) - https://github.com/praveenpuglia/shadow-dom-in-depth/blob/master/README.md
Chino - https://github.com/Tencent/omi/blob/master/tutorial/shadow-dom-in-depth.cn.md - Thanks [@eyea](https://github.com/eyea).

* Ejemplos. Estoy poniendo ejemplos que ayudarán a todo el mundo a comprender mejor, paso por paso [Míralos.](./examples)

Déjame saber si el documento "Todo lo que necesitas saber sobre Elementos Personalizados" como este te ayudaría. Si es así, pondré uno arriba 👨‍💻.

## Soporte para navegadores

* Chrome : Funciona
* Firefox : Funciona
* Opera : Funciona
* Safari : Funciona pero con algunos defectos.
* Edge : Bajo consideración.

Información comprensiva acerca del soporte de los navegadores puede encontrase en: https://caniuse.com/#feat=shadowdomv1.

## Introducción

En resúmen, `Shadow DOM` permite ámbitos locales para HTML & CSS.

> `Shadow DOM` arregla CSS y DOM. Introduce ámbitos locales para los estilos en la plataforma web. Sin herramientas o convenciones de nombres, puedes empacar CSS con HTML, esconder detalles de implementación, y crear componentes auto-contenidos en `Vanilla Javascrip`. - https://developers.google.com/web/fundamentals/getting-started/primers/shadowdom

Es como su pequenho mundo que raramente se ve afectado o se afecta por cambios externos.

Es lo que implementas como un **component author** para abstraer los detalles de la implementación de tu componente. También puede decidir qué hacer con el **light DOM** proveído por el usuario.

## Terminologías

**-DOM :** Lo que descargamos es una cadena de texto. Para renderizar algo en la pantalla, el navegador tiene que parsear esa cadena de texto y convertirla en un modelo de datos para así poder entenderla. También preserva la jerarquía de la cadena orgina poniendo los objetos parseados en una estructura de árbol.

Tenemos que hacer esto para que las máquinas interpreten mejor los documentos. Este modelo de datos en forma de árbol es llamado Document Object Model.

**- Component Author :** La persona que crea un componente y define cómo trabaja. Generalmente la persona que escribe mucho código `shadow DOM`. Example - Los que hacen navegadores crearon el elemento `input`.

**- Component User :** Los que usan los componentes creadores por los autores. Pueden pasar `light DOM` y poner atributos y propiedades en tu componente. Incluso pueden extender el funcionamiento del componente si así lo desean. Ejemplo - nosotros, los usuarios que usamos el elemento `input`.

**- Shadow Root:** Es lo que se adjunta a un elemento para definir su `shadow DOM`. Tecnicamente es un nodo no-elemento, un caso especial de [_DocumentFragment_](https://developer.mozilla.org/en-US/docs/Web/API/DocumentFragment).

```html
<custom-picture>
    ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ ISOLATION
    #shadow-root
        ...
    ____________________________________________ DOCUMENT FRAGMENT

    <!--LIGHT DOM-->
</custom-picture>
```
A través del documento he puesto el `shadow root` dentro de esos extranhos caracteres ASCII. Esto pondrá más énfasis en pensar cómo ellos son realmente `Document Fragments` que tienen un muro a su alrededor.

**- Shadow Host:** El elemento al que se adjunta el `shadow root`. Un `host` puede acceder a su `shadow root` mediante la propiedad `.shadowRoot`.

**- Shadow Tree:** Todos los elementos que van dentro del `Shadow Root`, que tienen un ámbito aislado al del mundo exterior es llamado `Shadow Tree`.

> Los elementos en el `Shadow Tree` **no** son descendientes del `Shadow Host` en general (incluido para el propósito de los Selectores como el descendiente combinado) - Spec

**- Light DOM:** - El conjunto de elementos del DOM que pueden estar entre las etiquetas de apertura y cierre. - El DOM que reside fuera del `Shadow DOM`. - El DOM, la parte que escribe el usuario. - El DOM que es el hijo actual de tu elemento.

```html
<custom-picture>
    ^^^^^^^^^^^^^^^^^^^^^^^^^^^
    #shadow-root
    ___________________________

    <!--Light DOM-->
    <img src="https://path.to/a-kitten.png">
    <cite>A Nice Kitten!</cite>
    <!--Light DOM ends-->
</custom-picture>
```

**- DocumentFragment:**

> The DocumentFragment interface represents a minimal document object that has no parent. It is used as a lightweight version of Document to store a segment of a document structure comprised of nodes just like a standard document. The key difference is that because the document fragment isn't part of the actual DOM's structure, changes made to the fragment don't affect the document, cause reflow, or incur any performance impact that can occur when changes are made. - [MDN](https://developer.mozilla.org/en/docs/Web/API/DocumentFragment)

## How to create Shadow DOM?

```html
<div class="dom"></div>
```

```js
let el = document.querySelector('.dom');
el.attachShadow({ mode: 'open' });
// Just like prototype & constructor bi-directional references, we have...
el.shadowRoot; // the shadow root.
el.shadowRoot.host; // the element itself.

// Put something in shadow DOM
el.shadowRoot.innerHTML = 'Hi I am shadowed!';

// Like any other normal DOM operation.
let hello = document.createElement('span');
hello.textContent = 'Hi I am shadowed but wrapped in span';
el.shadowRoot.appendChild(hello);
```

### Off Topic Question - Could we use `append()` instead of `appendChild()` ?

Yes! But here are the differences from MDN.

* `ParentNode.append()` allows you to also append DOMString object, whereas `Node.appendChild()` only accepts Node objects.
* `ParentNode.append()` has no return value, whereas `Node.appendChild()` returns the appended Node object.
* `ParentNode.append()` can append several nodes and strings, whereas `Node.appendChild()` can only append one node.

### What happens if we use an `input` element instead of the div to attach the shadow DOM?

Well, it doesn't work. Because the browser already hosts its own shadow DOM for those elements. Bunch of red colored english alphabets will be thrown at console's face. 😰

## Notes & Tips

* Shadow DOM cannot be removed once created; it can only be replaced with a new one.
* If you are creating a custom element, you should be creating the shadowRoot in its constructor. It can also be probably called in `connectedCallback()` but I am not sure if that introduces performance problems or any other problems. 🤷‍♂️
* To see how browsers implement shadow DOM for elements like `input` or `textarea`, Go to `DevTools > Settings > Elements > [x] Show user agent shadow DOM`.

## Shadow DOM Modes

### Open

You saw the `{mode: "open"}` in the `attachShadow()` method right? Yeah! That's it. What open mode does is that it provides a way for us to reach into the shadow DOM to access the element's contents. It also lets us access the host element from within the shadow DOM.

This is done by the two implicit properties created when we call `attachShadow()` in `open` mode.

1.  The element gets a property called `shadowRoot` which points to the shadow DOM being attached to it.
2.  The `shadowRoot` gets a property called `host` pointing to the element itself.

```js
// From the "How to create shadow DOM" example
el.attachShadow({ mode: 'open' });
// Just like prototype & constructor bi-directional references, we have...
el.shadowRoot; // the shadow root.
el.shadowRoot.host; // the el itself.
```

### Closed

Pass `{mode: "closed"}` to `attachShadow()` to create a closed shadow DOM. It makes the shadow DOM inaccessible from JS.

```js
el.shadowRoot; // null
```

### So which one should I use?

Almost always use `open` mode shadow DOMs because they make it possible for both the component author and user to change things how they want.

Remember we did `el.shadowRoot` stuff up there? Yeah! That won't work with `closed` mode. The element doesn't get any reference to its shadow DOM, which is a problem when you want to access the shadow DOM for manipulation.

```js
class CustomPicture extends HTMLElement {
  constructor() {
    this.attachShadow({ mode: 'open' }); // this.shadowRoot exists. Add or remove stuff in there using this ref.
    this.attachShadow({ mode: 'closed' }); // this.shadowRoot returns null. Bummer!
  }
}
// You could always do the following in your constructor.
// but it totally defies the purpose of using closed mode.
this._shadowRoot = this.attachShadow({ mode: 'closed' });
```

Also, closed mode isn't a security mechanism. It just gives a fake sense of security. Nobody can stop someone from modifying how `Element.prototype.attachShadow()` works.

## Styles inside Shadow DOM.

* They are scoped.
* They don't leak out.
* They can have simple names.
* They are cool, be like them. 😎

```html
<custom-picture>
    #shadow-root
        ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
        <style>
            /*Applies only to spans inside shadow DOM. Doesn't leak out.*/
            span {
                color: red;
            }
        </style>
        <span>Hello!</span>
        __________________________________________________________________
</custom-picture>
```

### Oh! oh! So can we also include a stylesheet?

Yeeeeaaaah.. but not in all browsers. 😕

```html
<custom-picture>
    ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    #shadow-root
        <!--All styles coming from custom-picture.css will be scoped inside this shadow root-->
        <link rel="stylesheet" href="custom-picture.css">
        <span>Hello!</span>
    _____________________________________________________
```

### Does it get affected by global CSS?

Yeah. In some ways. Only the properties that are inherited will make their way through the shadow DOM boundary. Examples:

* color
* background
* font-family, etc.

The `*` selector also affects things because `*` means all elements and that includes the element to which you are attaching the shadow root to (the host element). Things which get applied to the host and can be inherited will pass the shadow DOM boundary to apply to inner elements.

## Styles Terminologies

* :host: targets the host element. BUT!

```css
/* winner */
custom-picture {
    background: red;
}
/* loser */
#shadow-root
    <style>
        :host {
            background: green;
        }
    </style>
```

* :host(`<selector>`): Does the component **host** match the **selector** ? Basically, allows us to target different states of the same host. Examples:

```css
:host([disabled]) {
  ...;
}

:host(:focus) {
  ...;
}

:host(:focus) span {
  /*change all spans inside the element when the host has focus*/
}
```

* :host-context(`<selector>`): Is the **host** a descendant of **selector** ? Let us change a component's styles based on how the parent looks. General application could be in theming. Examples:

```css
:host-context(.light-theme) {
  background: lightgray;
}

:host-context(.dark-theme) {
  background: darkgray;
}

/*You can also do...*/
:host-context(.aqua-theme) > * {
  color: aqua; /* lame */
}
```

### A note about :host() and :host-context()

Both of these functional pseudo classes can only take `<compound-selector>` but not a combinator like space or ">" or "+" etc.
In case of `:host()` that means you can only choose the attributes and other aspects of the host element.

In case of `:host-context()` that means you can choose the attributes and other aspects of one particular element which is an ancestor of the `:host`. Only one!

### Do the modes in `attachShadow()` affect how styles get applied/cascaded?

Nope! Only affects how we do things in JS.

### How does the user-agent styles get applied to even the shadow DOM elements?

Based on how shadow root or in general DocumentFragment works, user-agent styles (global) shouldn't have applied to all the elements inside a shadow root. So how do they work?

From the spec...

> `Window`s gain a private slot `[[defaultElementStylesMap]]` which is a map of local names to stylesheets. This makes it possible to write elements inside shadow root and still get the default browser styles applied to them.

### Which styles to put in shadow DOM ?

The purpose of having styles inside a shadow DOM is to just have default styles and provide hooks via CSS custom properties so component users can make changes to those default styles via, [CSS custom properties](https://developer.mozilla.org/en-US/docs/Web/CSS/--*) (a.k.a CSS variables).

```html
<business-card>
    ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    #shadow-root
        <h1 class="card-title">Hardcoded Title - </h1>
    ---------------------------------------------------
</business-card>
```

```css
/*Inside shadow DOM*/
.card-title {
  color: var(--card-title-color, #000);
}

/*Component users can then override this color as*/
business-card {
  --card-title-color: magenta;
}
```

### Notes from the Specs

* A shadow host is outside of the shadow tree it hosts, and so would ordinarily be untargettable by any selectors evaluated in the context of the shadow tree (as selectors are limited to a single tree), but it is sometimes useful to be able to style it from inside the shadow tree context.
* To work around that problem, the shadow host is treated as replacing the shadow root node.
* When considered within its own shadow trees, the shadow host is featureless. Only the `:host`, `:host()`, and `:host-context()` pseudo classes are allowed to match it.

## Events in Shadow DOM

Events work towards maintaining the encapsulation provided by the shadow DOM. Essentially, if an event occurs somewhere in the shadow DOM, to outside world it'll look as if that event has triggered from the host element itself and not a specific part of the shadow DOM. This is called **re-targeting of the event**.

Inside the shadow DOM, however, the events aren't retargeted and we can find out which specific element an event was associated with.

Say our flattened DOM tree looks like this:

```html
<body>
    <custom-picture>
        ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
        #shadow-root
            <button> Hello </button>
        ---------------------------------
    </custom-picture>
</body>
```

On click of the button, to body, or anywhere outside custom-picture, the `event.target` will point to the `<custom-picture>` itself.

> If the shadow tree is open, calling `event.composedPath()` will return an array of nodes that the event traveled through.

Inside `<custom-picture>` however, the event target will be the button which was really clicked.

Most events bubble out of the shadow DOM boundary, and when they do they get re-targered. Some events aren't allowed to pass that boundary. Precisely these:

* abort
* error
* select
* change
* load
* reset
* resize
* scroll
* selectstart

## Slots

Slots are a pretty big thing in Shadow DOM.

> Slots are placeholders inside your component that users can fill with their own markup. - https://developers.google.com/web/fundamentals/getting-started/primers/shadowdom#slots

When creating custom components, we want to be able to provide only the necessary markup that goes into a particular component and use/group/style that as we want to as component authors.

The DOM that a component user provides is called light DOM and slots are the way we arrange, style, and group those elements.

There are two aspects of a slot:

* **Light DOM Elements :** They say which slot they wanna go into.

```html
<custom-picture>
    <!--Light DOM <img> saying it should be put into the "profile-picture" slot-->
    <img src="assets/user.svg" slot="profile-picture">
</custom-picture>
```

* **The Actual Slot :** The `<slot>` element, residing somewhere in shadow DOM with a name for itself to be found by light DOM.

```html
<custom-picture>
    ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    #shadow-root
        <slot name="profile-picture">
            <!--The <img> from the light DOM gets rendered here!-->
        </slot>
    _________________________________________
```

### A very important note about how light DOM and slots play together.

> Elements are allowed to "cross" the shadow DOM boundary when a &lt;slot&gt; invites them in. These elements are called distributed nodes. Conceptually, distributed nodes can seem a bit bizarre. Slots don't physically move DOM; they render it at another location inside the shadow DOM. - https://developers.google.com/web/fundamentals/getting-started/primers/shadowdom#slots

### What if I don't provide the `slot` attribute in the `<img>` in `<custom-picture>`?

You'll see nothing rendered. Here's why:

1.  A host element that has shadow DOM, only renders stuff that goes inside its shadow DOM.
2.  In order to get light DOM elements rendered, they need to be part of the shadow DOM.
3.  The way we make them part of the shadow DOM is by putting them in slots.
4.  In our example above, there's no element in light DOM that wants to go into a slot named `profile-picture`.
5.  Since there's no one, the `<img>` from light DOM is not rendered.
6.  Takeaway: named slots accommodate only those light DOM elements which specify they want to go into that particular slot.

### What if I want to render all elements that don't say where they should go?

This will require us to have a general purpose slot in our shadow DOM. A slot without a name.
Example -

```html
<custom-picture>
    ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    #shadow-root
        <!--General purpose slot, render every element from light DOM that doesn't mention a slot name, here.-->
        <slot>
            <!--The <img> from the light DOM gets rendered here!-->
        </slot>
    _________________________________________
    <!--Light DOM-->
    <img src="assets/user.svg">
</custom-picture>
```

### What if I add two unnamed slots?

Woah! 😲
Actually, we can have duplicate unnamed or named slots but they are essentially ignored, since light DOM elements will go into the first slot they match.

```html
<custom-picture>
    ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    #shadow-root
        <slot>
            <!--The <img> from the light DOM gets rendered here! Winner!-->
        </slot>
        <slot>
            <!--Doesn't come here!-->
        </slot>
    _________________________________________
    <!--Light DOM-->
    <img src="assets/user.svg">
</custom-picture>
```

### What if there's a slot but no light DOM elements want to go there?

Nothing will be rendered unless there's fallback content provided by the slot itself. Providing fallback content is easy:

```html
#shadow-root
    <slot name="nobody-comes-here">
        <h1> I'll show up when no slot content is provided!</h1>
    </slot>

    <style>
        /*And that fallback can be styled from within the shadow DOM just like we do styles*/
        slot[name="nobody-comes-here"] h1 {
            color: #bada55;
        }
    </style>
```

### So what are slotted elements and how can we style them?

Light DOM elements that go into a slot are called slotted elements. As mentioned above, these are also called distributed elements which cross the shadow DOM boundary.

These slotted elements can be styled using the `::slotted()` functional pseudo element. The syntax is as follows:

```css
::slotted(<compound-selector >) {
  /* styles */
}
```

Example:

```html
<custom-picture>
    ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    #shadow-root
        <slot>
            <!--The <img> from the light DOM gets rendered here!-->
        </slot>

        <style>
            /* find the slotted image and set their width and height */
            ::slotted(img) {
                width: 256px;
                height: 256px;
            }
        </style>
    _________________________________________
    <!--Light DOM-->
    <img src="assets/user.svg">
</custom-picture>
```

Here's how the spec formally defines it:

> The ::slotted() pseudo-element represents the elements assigned, after flattening, to a slot. This pseudo-element only exists on slots.

Flattened trees are [here](https://developers.google.com/web/fundamentals/getting-started/primers/shadowdom#lightdom).

An important thing to remember is that only direct children of the host element can be assigned to a slot. For example:

```html
<custom-picture>
    <div class="picture-wrapper">
        <!--This won't work! Slots can't pick descendants out of the host element's light DOM tree and put them in.-->
        <img src="assets/user.svg" slot="assign-me" />
    </div>
</custom-picture>
```

I, however, don't know why that's not possible and what the reasons are behind it, so I created [this bug](https://github.com/w3c/csswg-drafts/issues/1530).

### How can I pass a light DOM element multiple levels down?

It may sound like we don't need to think about this scenario but it's often required.

```html
<parent-element>
    <!--parent-element uses child-element in its shadow DOM and we want this span to render inside that child-element's shadow DOM-->
    <span slot="parent-slot">Finally</span>
</parent-element>
```

Here's what our custom elements look like:

```js
class ParentElement extends HTMLElement {
  constructor() {
    super();
    this.attachShadow({ mode: 'open' });
    this.shadowRoot.innerHTML = `
        <!--We specify a slot property on the slot itself. Which specifies where it goes in the child-element's shadow DOM-->
        <child-element>
            <slot name="parent-slot" slot="child-slot"></slot>
        </child-element>
    `;
  }
}

class ChildElement extends HTMLElement {
  constructor() {
    super();
    this.attachShadow({ mode: 'open' });
    this.shadowRoot.innerHTML = `
      <slot name="child-slot">
        <!--The span with the thext "Finally" gets rendered here!-->
      </slot>
    `;
  }
}

window.customElements.define('parent-element', ParentElement);
window.customElements.define('child-element', ChildElement);
```

### Slots have JavaScript API

* To find the elements that went into a slot - [`slot.assignedNodes()`](https://developer.mozilla.org/en-US/docs/Web/API/HTMLSlotElement/assignedNodes);
* To find out which slot an light DOM element is assigned to - [`element.assignedSlot`](https://developer.mozilla.org/en-US/docs/Web/API/Element/assignedSlot);

## Referencias

* http://robdodson.me/shadow-dom-css-cheat-sheet/
* https://developers.google.com/web/fundamentals/getting-started/primers/shadowdom
* https://drafts.csswg.org/css-scoping/

## Más consultas & problemas

* https://stackoverflow.com/questions/44564366/inheritance-inside-a-shadow-dom-slot
* https://github.com/w3c/csswg-drafts/issues/1535
* https://github.com/w3c/csswg-drafts/issues/1530
* https://twitter.com/praveenpuglia/status/876677801146896384
