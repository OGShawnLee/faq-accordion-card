# Frontend Mentor - FAQ Accordion Card Solution

This is a solution to the [FAQ accordion card challenge on Frontend Mentor](https://www.frontendmentor.io/challenges/faq-accordion-card-XlyjD0Oam). Frontend Mentor challenges help you improve your coding skills by building realistic projects.

## Table of contents

- [Frontend Mentor - FAQ Accordion Card Solution](#frontend-mentor---faq-accordion-card-solution)
  - [Table of contents](#table-of-contents)
  - [Overview](#overview)
    - [The challenge](#the-challenge)
    - [Screenshot](#screenshot)
    - [Links](#links)
  - [My process](#my-process)
    - [Built with](#built-with)
    - [What I learned](#what-i-learned)
    - [Useful resources](#useful-resources)
  - [Author](#author)

## Overview

### The challenge

Users should be able to

- View the optimal layout for the component depending on their device's screen size
- See hover states for all interactive elements on the page
- Hide/Show the answer to a question when the question is clicked

### Screenshot

![Desktop View](./screenshots/Screenshot%202022-05-14%20at%2009-14-37%20Frontend%20Mentor%20FAQ%20Accordion%20Card.png)

### Links

- Solution URL: [Right here!](https://www.frontendmentor.io/solutions/animated-faq-section-with-fully-functional-accordion-component-S1F6YuRI5)
- Live Site URL: [Deployed on Vercel](https://faq-accordion-card-eta-orpin.vercel.app/)

## My process

### Built with

- Semantic HTML5 markup
- WindiCSS + Flexbox
- [Malachite UI](https://github.com/OGShawnLee/malachite-ui) - Component Library
- Svelte + Transition API
- Vite

### What I learned

For this project I had to **build the Accordion component from the ground up for Malachite UI** so that I could just install, import, style and chill knowing that I won't worry about accessibility nor functionality since it is all handled for me, and on top of that animations are handled by Svelte... ahhh what a time to be alive.

```html
<Accordion class="flex flex-col gap-3">
  {#each questions as { question, answer }, index}
    <AccordionItem let:isOpen let:header let:panel open={index === 1}>
      <h2 use:header class="h-12">
        <AccordionButton
          class={{
            base: 'w-full py-2 px-0 | button-reset border-b-blue-50 outline-none transition duration-150 ease-in',
            open: { on: 'font-bold focus:text-soft-violet', off: 'focus:text-soft-red' },
          }}>
          <span class="flex items-center justify-between">
            <span class="text-[13.5px] sm:text-base md:text-lg">
              {question}
            </span>
            <img
              class="transform duration-150 ease-in"
              class:rotate-180={isOpen}
              src={iconArrow}
              alt="" />
          </span>
        </AccordionButton>
      </h2>
      <div slot="panel" use:panel transition:slide={{ duration: 175, easing: quadOut }}>
        <p class="text-xs sm:text-sm md:text-base">{answer}</p>
      </div>
    </AccordionItem>
  {/each}
</Accordion>
```

```ts
export default class Accordion extends Component {
  protected readonly Buttons: Ordered<ItemInstance & Nav.Member>;
  protected readonly Navigable: Navigable<ItemInstance>;

  readonly Open: Readable<boolean>;
  protected readonly Items = new Hashable<number, Toggleable>();
  readonly Finite: Store<Readable<boolean>>;
  readonly ShouldOrder: Store<Readable<boolean>>;

  protected isInitialised = false;
  protected previousOpenItem: ItemInstance | undefined;

  constructor({ Finite, ShouldOrder }: Expand<Options>) {
    super({ component: 'accordion', index: Accordion.generateIndex() });

    this.Finite = Finite;
    this.ShouldOrder = ShouldOrder;

    this.Buttons = new Ordered(ShouldOrder);
    this.Navigable = new Navigable({ Ordered: this.Buttons, Finite, Vertical: true });

    this.Open = derived(this.Buttons.Members, (items) => {
      return items.some(({ isOpen }) => isOpen);
    });

    Context.setContext({
      accordion: this.accordion,
      initItem: this.initItem.bind(this),
    });
  }

  get accordion() {
    return this.defineActionComponent({
      onMount: this.name,
      destroy: ({ element }) => [
        this.Navigable.initNavigation(element, {
          handler() {
            return Navigable.initNavigationHandler(element, ({ event, code, ctrlKey }) => {
              if (!this.isWithin(document.activeElement)) return;
              switch (code) {
                case 'ArrowDown':
                  event.preventDefault();
                  return this.handleNextKey(code, ctrlKey);
                case 'ArrowUp':
                  event.preventDefault();
                  return this.handleBackKey(code, ctrlKey);
                case 'End':
                  return this.goLast();
                case 'Home':
                  return this.goFirst();
              }
            });
          },
        }),
      ],
    });
  }

  protected handleAriaLevel(header: HTMLElement, level: string | number | undefined) {
    if (isNumber(level) || isString(level)) return (header.ariaLevel = level.toString());
    const [h, number] = header.tagName;
    if (header.tagName.length === 2 && h === 'H' && isNumber(Number(number))) {
      header.ariaLevel = number;
    } else header.ariaLevel = '2';
  }

  protected handleButtonFocus(button: HTMLElement) {
    return useListener(button, 'focus', async () => {
      await tick();
      const index = this.Navigable.indexOf(button);
      if (index < 0 || this.Navigable.isSelected(index)) return;
      this.Navigable.set(index);
    });
  }

  protected handleUnique(button: HTMLElement, Toggleable: Toggleable) {
    return Toggleable.subscribe((isOpen) => {
      if (!isOpen) return;
      const item = this.Buttons.get(button);
      if (button !== this.previousOpenItem?.button) this.previousOpenItem?.close();
      this.previousOpenItem = item;
    });
  }

  initItem({ Toggleable, initialOpen }: { Toggleable: Toggleable; initialOpen: boolean }) {
    const { destroy, index } = this.Items.push(Toggleable);
    onDestroy(() => destroy());

    if (initialOpen && !this.isInitialised) {
      this.isInitialised = true;
      Toggleable.open();
    }

    const [Button, Header, Panel] = generate(3, () => new Bridge());
    const { nameChild } = useName({ parent: this.name, component: 'item', index });

    return ItemContext.setContext({
      Open: makeReadable(Toggleable),
      close: Toggleable.close.bind(Toggleable),
      button: defineActionComponent({
        Bridge: Button,
        onMount: ({ element }) => {
          setAttribute(element, ['tabIndex', '0'], {
            overwrite: true,
            predicate: () => element.tabIndex <= 0,
          });
          return nameChild('button');
        },
        destroy: ({ element }) => [
          Toggleable.button(element, {
            onChange: (isOpen) => {
              element.ariaExpanded = String(isOpen);
            },
          }),
          this.Navigable.initItem(element, {
            Bridge: Button,
            Value: {
              Index: writable(this.Buttons.size),
              button: element,
              close: () => Toggleable.set(false),
              isOpen: Toggleable.isOpen,
            },
          }),
          this.handleUnique(element, Toggleable),
          this.handleButtonFocus(element),
          Panel.Name.subscribe((id) => {
            if (id) element.setAttribute('aria-controls', id);
            else element.removeAttribute('aria-controls');
          }),
          usePair(Button.Disabled, Panel).subscribe(([isDisabled, panel]) => {
            if (isDisabled && panel) element.ariaDisabled = 'true';
            else element.ariaDisabled = null;
          }),
        ],
      }),
      header: defineActionComponentWithParams<number | string>({
        Bridge: Header,
        onMount: ({ element, parameter: level }) => {
          this.handleAriaLevel(element, level);
          element.setAttribute('role', 'heading');
          return nameChild('header');
        },
        onUpdate: ({ element, parameter: level }) => {
          this.handleAriaLevel(element, level);
        },
      }),
      panel: defineActionComponent({
        Bridge: Panel,
        onMount: () => {
          return nameChild('panel');
        },
        destroy: ({ element }) => [
          Toggleable.panel(element, {
            plugins: [usePreventInternalFocus],
          }),
          Button.Name.subscribe((id) => {
            if (id) element.setAttribute('aria-labelledby', id);
            else element.removeAttribute('aria-labelledby');
          }),
          this.Items.Size.subscribe((size) => {
            if (size <= 6) element.setAttribute('role', 'region');
            else element.removeAttribute('role');
          }),
        ],
      }),
    });
  }

  private static generateIndex = this.initIndexGenerator();
}
```

### Useful resources

- [WAI-ARIA Authoring Practices 1.1](https://www.w3.org/TR/wai-aria-practices-1.1/#accordion) - I followed this awesome pattern for the Accordion.

## Author

- Frontend Mentor - [@Shawn Lee](https://www.frontendmentor.io/profile/OGShawnLee)
