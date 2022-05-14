<script lang="ts">
  import { Accordion, AccordionButton, AccordionItem } from 'malachite-ui/components';
  import {
    iconArrow,
    illustrationBox,
    illustrationWomanDesktop,
    illustrationWomanMobile,
    patternDesktop,
    patternMobile,
    questions,
  } from './assets';
  import { slide } from 'svelte/transition';
  import { quadOut } from 'svelte/easing';
</script>

<main class="w-xs mx-auto | sm:w-sm md:w-md lg:w-4xl">
  <div class="relative shadow-2xl">
    <img
      class="hidden absolute top-1/2 right-full transform -translate-y-3/12 translate-x-3/8 z-90 lg:block"
      src={illustrationBox}
      alt="" />
    <div
      class="relative pt-32 p-6 | bg-white shadow-lg rounded-2xl | md:(pt-32 p-8) lg:(p-18 grid grid-cols-12 overflow-hidden)">
      <div class="col-span-6">
        <picture aria-hidden="true" class="hidden lg:block">
          <img
            class="absolute top-1/2 right-full transform -translate-y-1/2 translate-x-6/8"
            src={illustrationWomanDesktop}
            alt="" />
          <img
            class="absolute top-1/2 right-full transform -translate-y-14/26 translate-x-40/100"
            src={patternDesktop}
            alt="" />
        </picture>
        <picture aria-hidden="true" class="lg:hidden">
          <img
            class="absolute w-9/12 max-w-[18rem] bottom-full left-1/2 transform -translate-x-1/2 translate-y-3/8"
            src={illustrationWomanMobile}
            alt="" />
          <img
            class="absolute w-9/12 max-w-[18rem] left-1/2 transform -top-2 -translate-x-1/2"
            src={patternMobile}
            alt="" />
        </picture>
      </div>

      <div class="col-span-6">
        <h1 class="mb-4 | uppercase text-3xl text-center font-bold | lg:(mb-8 text-left text-4xl)">
          FAQ
        </h1>
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
      </div>
    </div>
  </div>
</main>

<style global>
  .button-reset {
    background-color: transparent;
    font-family: inherit;
    text-align: left;
    cursor: pointer;
  }
</style>
