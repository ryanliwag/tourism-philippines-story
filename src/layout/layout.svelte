<script>
  // import libraries
  import gsap from "gsap";
  import { onMount } from "svelte";
  import { Flip } from "gsap/Flip";

  gsap.registerPlugin(Flip);
  // constants
  export let title = "VISUAL STORIES";
  export let quotetitle = "Ad Astra Per Astera";
  let links = [
    { link: "www.google.com", title: "Behance" },
    {
      link: "https://www.linkedin.com/in/ryan-joshua-liwag-4a6124138/",
      title: "Linkedin",
    },
    {
      link: "https://github.com/ryanliwag",
      title: "Github",
    },
    { link: "https://twitter.com/ryanorchous", title: "Twitter" },
  ];

  let info_details = [
    "Ryan Liwag",
    "www.ryanliwag.com",
    "+63 9178622124",
    "Manila, PH",
    "Mandaluyong 1550",
  ];

  $: headers = title.split(" ");

  let content;
  let text_content;

  function elevator_inside() {
    var tl = gsap.timeline();
    return tl;
  }

  function elevator_doors() {
    var tl = gsap.timeline();
    return tl;
  }
  // master.add(elevator_inside())
  //       .add(elevator_doors());

  var layout_header = gsap;
  onMount(() => {
    layout_header = gsap.timeline();
    layout_header
      .from(".header_title", {
        duration: 1,
        yPercent: 100,
        opacity: 0,
        stagger: function (index, target, list) {
          // your custom logic here. Return the delay from the start (not between each)
          return index * 0.5;
        },
      })
      .from(
        ".header_quote",
        {
          duration: 1,
          yPercent: -100,
          opacity: 0,
        },
        "<"
      )
      .from(
        ".details_box",
        {
          duration: 1,
          yPercent: 100,
          ease: "power4.inOut",
          stagger: function (index, target, list) {
            // your custom logic here. Return the delay from the start (not between each)
            return index * 0.1;
          },
          opacity: 0,
        },
        "<"
      );

    layout_header.play();
  });

  function handle_click(event) {
    console.log("gg");
    event.stopPropagation();
    console.log(layout_header.reversed());
    layout_header.reversed() ? layout_header.play() : layout_header.reverse();
  }
</script>

<section class="layout-container">
  <section class="header-container">
    <div class="layout-container__header">
      {#each headers as header}
        <h1 class="header_title">{header}</h1>
      {/each}
    </div>
    <div class="layout-container__text">
      <h1 class="header_quote">{quotetitle}</h1>
    </div>
    <div class="layout-container__links">
      <ul>
        {#each links as link}
          <li class="details_box">{link.title}</li>
        {/each}
      </ul>
    </div>
    <div class="layout-container__contacts">
      <ul bind:this={text_content}>
        {#each info_details as info_detail}
          <li class="details_box">{info_detail}</li>
        {/each}
      </ul>
    </div>
  </section>
  <div bind:this={content} class="layout-container__content">
    <slot />
    <!-- gg
    <button on:click={handle_click}>reverse</button> -->
  </div>
</section>

<style scoped lang="scss">
  $breakpoint-tablet: 768px;

  .layout-container {
    display: grid;
    min-height: 100vh;
    margin: 0 2rem;
    grid-template-columns: 1fr;
    grid-gap: 20px;
    margin: 0;
  }

  .header-container {
    position: absolute;
    top: 0;
    left: -200px;
    max-height: 100vh;
  }

  .layout-container__content {
    // background-color: whitesmoke;
    margin: 5px;
  }

  @media (min-width: $breakpoint-tablet) {
    .layout-container {
      grid-template-columns: 100px 1fr;
      grid-gap: 50px;
      margin: 0 0 0 20px;
    }

    .header-container {
      position: sticky;
      top: 0;
      left: 0;
      display: grid;
      grid-template-rows: 50px 1fr 2fr 1fr 1fr 1fr;
    }

    .layout-container__content {
      margin: 50px 0 0 0;
    }
  }

  .layout-container__header {
    grid-row: 2/3;
    writing-mode: tb-rl;
    transform: rotate(-180deg);
    display: flex;
    flex-direction: column;
    justify-content: start;
    align-items: flex-end;
    line-height: 1rem;
    h1 {
      font-size: 4rem;
      margin: 0 1.5rem;
    }
  }

  .layout-container__text {
    grid-row: 3/4;
    display: flex;
    flex-direction: column;
    justify-content: start;
    align-items: center;
    font-size: 0.875rem;
    writing-mode: tb-rl;
    transform: rotate(-180deg);
    margin: 0;
    h1 {
      font-size: 2rem;
      line-height: 0rem;
    }
  }

  .layout-container__links {
    grid-row: 4/5;
    display: flex;
    flex-direction: column;
    justify-content: start;
    align-items: flex-start;
    font-size: 0.875rem;
    line-height: 1.2rem;
    writing-mode: tb-rl;
    transform: rotate(-180deg);
  }

  .layout-container__contacts {
    grid-row: 5/6;
    display: flex;
    flex-direction: column;
    justify-content: start;
    align-items: flex-start;
    font-size: 0.875rem;
    line-height: 1.2rem;
    writing-mode: tb-rl;
    transform: rotate(-180deg);
  }
  //dssd
  ul {
    padding: 0;
    margin: 0;
    & li {
      list-style: none;
    }
  }
</style>
