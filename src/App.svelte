<script>
  import { onMount } from "svelte";
  import svelteLogo from "./assets/svelte.svg";
  import viteLogo from "/vite.svg";
  import Counter from "./lib/Counter.svelte";
  import * as d3 from "d3";

  import Scroller from "./layout/Scroller.svelte";
  import Scatterplot from "./graphs/Scatterplot.svelte";
  import Layout from "./layout/layout.svelte";
  import { MapboxOverlay as DeckOverlay, MapboxLayer } from "@deck.gl/mapbox";
  import { GeoJsonLayer, ArcLayer, ScatterplotLayer } from "@deck.gl/layers";
  import * as deck from "deck.gl";

  import mapboxgl from "mapbox-gl";
  const mapbox_api = import.meta.env.VITE_mapbox_api_key;
  import data from "./Data/tourism_2019.json";
  // components
  import Button from "./components/Button.svelte";

  let data_2019;

  // scales for groups

  // scales for nb of tourist
  console.log();
  let colorScale;
  let widthScale;
  // var myColor = d3.scaleLinear().domain([1,10])
  // .range(["white", "blue"])

  const AIR_PORTS =
    "https://d2ad6b4ur7yvpq.cloudfront.net/naturalearth-3.3.0/ne_10m_airports.geojson";

  const sections = [
    {
      label: "Philippines",
      center: [120.984212, 14.599512],
      zoom: 2,
      pitch: 60,
      bearing: 0,
    },
    {
      label: "korea",
      center: [120.984212, 14.599512],
      zoom: 2,
      pitch: 60,
      bearing: -90,
    },
    {
      label: "Europe",
      center: [54, 15],
      zoom: 3,
      pitch: 45,
      bearing: 120,
    },
  ];

  function animateArcs(layer) {
    var coef = 0;
    layer.opacity = 1;
    const animationInterval = setInterval(() => {
      coef += 0.005;
      if (coef >= 1.0) {
        coef = 1;
        clearInterval(animationInterval);
      }
      layer.setProps({ coef });
    }, 5);
    console.log(layer, layer.props.coef);
  }

  function animateBack(layer) {
    console.log(layer.props.coef);
    var coef = 1;
    if (layer.props.coef == 1) {
      const animationInterval = setInterval(() => {
        coef -= 0.005;
        if (coef <= 0) {
          coef = 0;
          clearInterval(animationInterval);
        }
        layer.setProps({ coef });
      }, 5);
    }
  }

  // const arcsLayer = new deck.MapboxLayer({
  //   type: ArcBrushingLayer,
  //   id: `arcs`,
  //   data: data,
  //   opacity: 1,
  //   coef: 0,
  //   getSourcePosition: (d) => [d.longitude_source, d.latitude_source],
  //   getTargetPosition: (d) => [d.longitude_target, d.latitude_target],
  //   getSourceColor: (d) => colorScale(d.category),
  //   getTargetColor: (d) => colorScale(d.category),
  //   getStrokeWidth: 3,
  // });

  let value;
  const steps = [
    "<p>This is a dynamic, responsive scatterplot that uses Russell Goldenberg's <a href='	https://twitter.com/codenberg/status/1432774653139984387' target='_blank'><code>Scrolly</code></a> to update its points' values on scroll.</p>",
    "<p>The scatterplot uses tweened values to automatically update your points with smooth transitions. It also binds to the width of the container <code>div</code>, so its responsive by default.</p>",
    "<p>Try resizing me to see the 'side-by-side' version, compared to the 'text-on-top' version that appears on small screens.</p><p>Want it to always appear 'text-on-top'? Just comment out the media query at the bottom of our styles (as in, leave the styles but comment out the surrounding <code>media</code> query).</p>",
  ];

  mapboxgl.accessToken = mapbox_api;

  let map;
  let arcs_test_layer;
  class ArcBrushingLayer extends ArcLayer {
    getShaders() {
      return Object.assign({}, super.getShaders(), {
        inject: {
          "vs:#decl": `uniform float coef;`,
          "vs:#main-end": `
            if (coef > 0.0) {
              vec4 pct = vec4(segmentRatio);
              pct.a = step(coef, segmentRatio);
              vec4 colorA = instanceTargetColors;
              vec4 colorB = vec4(instanceTargetColors.r, instanceTargetColors.g, instanceTargetColors.b, 0.0);
              vec4 color = mix(instanceSourceColors, colorB, pct.a);
              vColor = color;
              DECKGL_FILTER_COLOR(vColor, geometry);
            }
          `,
        },
      });
    }

    draw(opts) {
      const { coef } = this.props;

      this.state.model.setUniforms({ coef });
      super.draw(opts);
    }
    s;
  }

  arcs_test_layer = new MapboxLayer({
    id: "deckgl-arc",
    type: ArcBrushingLayer,
    data: data,
    opacity: 0,
    coef: 0,
    getSourcePosition: (d) => [d.long, d.lat],
    getTargetPosition: [120.984212, 14.599512],
    getSourceColor: (d) => colorScale(d["Total 2019"]),
    getTargetColor: (d) => colorScale(d["Total 2019"]),
    getWidth: (d) => widthScale(d["Total 2019"]),
  });

  onMount(() => {
    let data_2019 = data;
    colorScale = d3
      .scaleLinear()
      .domain(d3.extent(data_2019.map((d) => d["Total 2019"])))
      .range([
        [0, 0, 255],
        [255, 0, 0],
      ]);
    widthScale = d3
      .scaleLinear()
      .domain(d3.extent(data_2019.map((d) => d["Total 2019"])))
      .range([0.5, 10]);
    initMap();
    console.log(data_2019);
    console.log();
  });

  const handle_click = (data) => {
    console.log(data);
    if (data.label == "korea") {
      animateBack(arcs_test_layer);
    } else {
      animateArcs(arcs_test_layer);
    }

    map.flyTo({
      center: data.center,
      zoom: data.zoom,
      bearing: data.bearing,
      pitch: data.pitch,
      duration: 5000,
    });
  };

  function initMap() {
    map = new mapboxgl.Map({
      container: "viz_mapbox",
      style: "mapbox://styles/mapbox/light-v9",
      center: [120.984212, 14.599512],
      zoom: 10,
      bearing: 0,
      pitch: 0,
    });

    // load all layers.

    map.on("load", () => {
      const firstLabelLayerId = map
        .getStyle()
        .layers.find((layer) => layer.type === "symbol").id;

      map.addLayer(
        new MapboxLayer({
          id: "deckgl-circle",
          type: ScatterplotLayer,
          data: [
            { position: [-122.402, 37.79], color: [255, 0, 0], radius: 1000 },
          ],
          getPosition: (d) => d.position,
          getFillColor: (d) => d.color,
          getRadius: (d) => d.radius,
          opacity: 0.3,
        }),
        firstLabelLayerId
      );
      map.addLayer(arcs_test_layer);
    });
  }
</script>

<Layout>
  <div class="section-container">
    <div class="steps-container">
      <Scroller bind:value>
        {#each steps as text, i}
          <div class="step" class:active={value === i}>
            <div class="step-content">{@html text}</div>
          </div>
        {/each}
        <div class="spacer" />
      </Scroller>
    </div>
    <div class="sticky">
      <Scatterplot step={value} />
    </div>
  </div>
  <div>
    Lorem ipsum dolor sit amet consectetur adipisicing elit. Neque culpa
    veritatis pariatur velit repellat at harum! Hic voluptatem tempora
    perferendis maxime pariatur cum accusamus quo enim. Animi laudantium ad
    omnis? Sequi aliquam temporibus, culpa maiores, suscipit tenetur dolorem,
    eaque placeat sunt eius praesentium. Quae qui quasi, numquam consectetur,
    veritatis, magnam dignissimos similique distinctio ducimus sed quis
    molestiae. Natus ducimus, minus itaque aut, hic et beatae vitae consequatur
    quod atque tempora possimus suscipit adipisci repudiandae neque assumenda
    officia incidunt eius minima deserunt a ut nulla exercitationem! Recusandae
    sint nihil earum ducimus quia molestiae deserunt saepe tempora at eius nulla
    tempore magnam obcaecati est quisquam soluta dolore quasi reprehenderit rem,
    aliquid error enim modi asperiores. Facere id minima vero aut sint, mollitia
    ad, alias quo dolores ratione qui ab recusandae nihil nisi earum obcaecati
    officia quos itaque. Praesentium dolor sapiente vero quos commodi quibusdam,
    ad cum nobis veniam qui quas placeat dicta nulla eaque iste ea quidem
    recusandae provident et, aliquam quasi dolorum quaerat saepe? Modi eos dolor
    iusto, corporis adipisci saepe. Assumenda laboriosam error perspiciatis nam
    aliquam facere impedit fugit officia sit iure. Ipsa odit suscipit molestias
    nulla nesciunt ducimus, nemo cupiditate, esse assumenda deleniti iure nobis
    voluptatum illum mollitia voluptate aperiam totam provident veniam iusto
    eligendi perferendis reprehenderit. Culpa excepturi fuga odit dolor, saepe
    ea nam explicabo veritatis exercitationem pariatur unde vel doloribus
    assumenda nesciunt, modi nulla natus hic reiciendis, impedit nemo laudantium
    ipsa accusamus commodi? Explicabo totam dolor suscipit ad blanditiis
    obcaecati aliquid minus maiores harum sunt doloribus, laudantium earum
    laborum reprehenderit. Excepturi, culpa qui, totam similique consequuntur,
    natus sed ex autem quam deserunt pariatur tempore odit fugiat magni nam
    quisquam distinctio aliquam incidunt quibusdam ut rerum doloremque omnis ab
    voluptatum. Itaque mollitia quibusdam ipsa fuga aliquam explicabo odit
    perferendis ea, dolorum animi aut accusantium eos? Vel fugiat nam odio
    necessitatibus quod earum provident nihil quis fugit aliquid reprehenderit
    ad magnam ducimus qui, temporibus cumque enim. Harum modi nihil atque? Non
    maiores sed omnis, voluptas iste odio fugiat tempore ex, vero enim,
    similique tempora eligendi autem doloremque atque ratione? Cumque laudantium
    perferendis itaque expedita unde et reiciendis, excepturi libero sed,
    necessitatibus architecto commodi tenetur omnis debitis quis voluptatum,
    harum officia hic blanditiis dignissimos! Eaque laboriosam optio quisquam
    aspernatur iure nostrum repudiandae ad debitis similique dicta voluptates
    reiciendis temporibus dolorum tempore neque ipsam autem, nisi fuga,
    consequuntur quos est quidem, molestias sunt nulla! Assumenda quo quisquam
    voluptates eius ad, voluptate provident, amet sunt deserunt, eveniet
    deleniti fugiat. Labore veritatis aliquid porro optio ipsum neque minus! Qui
    quod ea harum in magni corporis eveniet! Quasi maxime suscipit obcaecati
    vero id facere odit ducimus corrupti? Exercitationem tempore qui quis
    voluptas quod? Optio maxime doloremque libero eaque sit natus nesciunt
    exercitationem quas reprehenderit autem, dolores tenetur, officia placeat
    ea, minus veritatis debitis facilis quos fuga laborum ipsa ducimus repellat
    atque omnis. Aut facere reprehenderit, laborum laudantium ea quam, provident
    autem quaerat perspiciatis eligendi excepturi quos, odio dolore ratione
    magnam explicabo inventore porro sed? Assumenda sit iusto molestiae totam
    voluptate, ipsum unde quo, magni velit enim consequatur quibusdam doloribus,
    repellendus id? Officiis tempore temporibus earum quisquam ducimus omnis
    nobis ex laudantium sint nihil? Consequatur, soluta nisi sed sit deleniti
    obcaecati illum, quisquam a maiores in id doloribus repudiandae adipisci
    fugit eius harum tenetur ex atque autem? Saepe esse similique libero dolores
    corrupti minus omnis repellendus iusto fugit qui nesciunt laborum provident
    tempore voluptates sunt, assumenda, quasi odit voluptatibus dolor numquam.
    Fugiat distinctio aspernatur natus aut omnis adipisci pariatur quae ipsam
    architecto exercitationem hic quia voluptas, dolorem odit tempora et? Illo
    laudantium qui atque, odio veniam voluptatem sunt rem nobis enim nam aut
    consectetur, impedit, autem nesciunt nisi tempore delectus fuga debitis
    fugit dolorum? Quas voluptates tempora dolorem temporibus. Eum id laudantium
    dignissimos soluta nisi quia, corporis mollitia a voluptate reiciendis.
    Quibusdam nemo quisquam ipsa, debitis aliquid veniam corporis quam suscipit
    aperiam, architecto natus excepturi reiciendis dignissimos repellat
    perspiciatis libero commodi soluta facere voluptas eos tempore at. Odit
    maiores voluptatibus assumenda excepturi ullam delectus sunt, eaque beatae
    fugiat nobis cum aspernatur blanditiis corrupti, voluptates inventore eos.
    Harum, voluptate? Error blanditiis animi corporis inventore possimus ipsa
    nisi molestias assumenda praesentium distinctio mollitia minima alias nam
    dolores magni adipisci repellat unde aut illum, tempore omnis facere enim
    fugit iure! Aliquam at, quas eum qui corporis, praesentium laudantium
    officia beatae nobis tempore tenetur omnis ducimus temporibus quis debitis
    laboriosam, aliquid fuga. Tempora neque ex dolorum error. Possimus ex iste
    quidem adipisci voluptate? Veritatis deserunt libero cum necessitatibus, et
    quod. Rem doloremque modi sed excepturi voluptas sit temporibus accusantium
    quae deleniti suscipit. Quibusdam, ipsam velit earum asperiores vitae nihil
    unde dolor sint voluptatum molestiae error repellat ab eveniet totam ut
    exercitationem deleniti, ea alias, odio odit dolores incidunt? Accusamus rem
    aliquam mollitia vel ipsam quis perferendis hic veniam itaque culpa
    obcaecati autem laborum quae voluptatum dolorum veritatis, animi laudantium
    fugiat id quam error voluptas magnam nihil pariatur. Vitae, totam.
    Distinctio eveniet odio provident vero animi ducimus sit perspiciatis optio
    quasi incidunt dolores id, facere neque accusamus. Saepe, ea quisquam nobis
    suscipit blanditiis assumenda, labore cumque maxime vitae perferendis
    consectetur veniam molestias rem repellat? Natus iusto eaque aut inventore
    debitis dolorum, ipsa nemo quae, laborum alias consectetur officiis eos
    doloremque temporibus dolores accusamus reiciendis veritatis atque
    blanditiis, voluptate corrupti! Nam obcaecati eaque mollitia sequi expedita
    vero ipsam animi perspiciatis officiis omnis, suscipit sit ad nulla ullam
    nemo ratione consequuntur perferendis repellendus molestiae ut veritatis
    placeat et eos distinctio. Quisquam, maiores ut ipsa temporibus, blanditiis
    quo praesentium illo earum ad sunt impedit quod quaerat pariatur nesciunt
    facilis magni possimus at eius aperiam ipsum ducimus culpa voluptate beatae.
    Assumenda dolor, adipisci sequi eos, vel distinctio amet voluptatum expedita
    ipsum minus, culpa quibusdam excepturi! Excepturi, odio assumenda ex ut
    nostrum non porro in rem esse rerum ad blanditiis dicta dolore consequuntur
    deserunt ullam necessitatibus velit nobis nesciunt soluta itaque dolor
    libero neque quo? Quibusdam ea placeat porro deleniti pariatur quos impedit
    sed! Voluptates culpa dignissimos eaque accusantium impedit commodi itaque
    molestiae eveniet ut, nihil sapiente corrupti veniam nemo tempore, nam velit
    magnam quaerat.
  </div>
</Layout>

<svelte:head>
  <style lang="scss">
    @import "./styles/vars.scss";
    @import url("https://fonts.googleapis.com/css?family=Bebas+Neue&display=swap");
    @import url("https://fonts.googleapis.com/css?family=Inter&display=swap");

    h1,
    h2,
    h3,
    h4,
    h5 {
      font-family: "Bebas Neue";
    }

    p {
      font-family: "Inter";
    }
    li {
      font-family: "Inter";
    }
  </style>
</svelte:head>

<style lang="scss">
  @import "./styles/vars.scss";

  .spacer {
    height: 50vh;
  }

  .sticky {
    position: sticky;
    top: 0%;
    padding: 0;
    margin: 0;
    height: 100%;
    width: auto;
  }

  .section-container {
    /* margin-top: 1em; */
    background-color: $color;
    width: inherit;
    height: inherit;
    text-align: center;
    transition: background 100ms;
    display: flex;
    flex-direction: column-reverse;
  }

  .step {
    height: 80vh;
    display: flex;
    place-items: center;
    justify-content: center;
    background-color: $color;
  }

  .step-content {
    font-size: 1rem;
    background: whitesmoke;
    color: #ccc;
    border-radius: 5px;
    padding: 0.5rem 1rem;
    display: flex;
    flex-direction: column;
    justify-content: center;
    transition: background 500ms ease;
    box-shadow: 1px 1px 10px rgba(0, 0, 0, 0.2);
    text-align: left;
    width: 75%;
    margin: auto;
    max-width: 500px;
  }

  .step.active .step-content {
    background: white;
    color: black;
  }

  .steps-container,
  .sticky {
    height: 100%;
  }

  .steps-container {
    /* flex: 1 1 40%; */
    z-index: 10;
    width: 100%;
  }

  /* Comment out the following line to always make it 'text-on-top' */
  /* @media screen and (max-width: 768px) {
    .section-container {
      flex-direction: column-reverse;
    }
    .sticky {
      width: 95%;
			margin: auto;
    }
  } */
</style>
