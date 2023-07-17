<script>
  import { MapboxOverlay as DeckOverlay, MapboxLayer } from "@deck.gl/mapbox";
  import { GeoJsonLayer, ArcLayer, ScatterplotLayer } from "@deck.gl/layers";
  import * as deck from "deck.gl";
  import mapboxgl from "mapbox-gl";

  const mapbox_api = import.meta.env.VITE_mapbox_api_key;

  export let data;

//    map.on("load", () => {
//       const firstLabelLayerId = map
//         .getStyle()
//         .layers.find((layer) => layer.type === "symbol").id;

//       map.addLayer(
//         new MapboxLayer({
//           id: "deckgl-circle",
//           type: ScatterplotLayer,
//           data: [
//             { position: [-122.402, 37.79], color: [255, 0, 0], radius: 1000 },
//           ],
//           getPosition: (d) => d.position,
//           getFillColor: (d) => d.color,
//           getRadius: (d) => d.radius,
//           opacity: 0.3,
//         }),
//         firstLabelLayerId
//       );
//       map.addLayer(arcs_test_layer);
//     });
//   }

mapboxgl.accessToken = mapbox_api;

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

</script>
