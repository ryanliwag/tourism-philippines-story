<script>
  import { ArcLayer } from "@deck.gl/layers";
  export let ArcBrushingLayer;

  ArcBrushingLayer = class ArcBrushingLayer extends ArcLayer {
    getShaders() {
      // here comes our custom shader
      // we will use step function to create opacity gradient with colorA and color B
      // for more information see https://thebookofshaders.com/05/
      return Object.assign({}, super.getShaders(), {
        inject: {
          "vs:#decl": `
        uniform float coef;
        `,
          "vs:#main-end": `
        if (coef > 0.0) {
          vec4 pct = vec4(segmentRatio);
          pct.a = step(coef, segmentRatio) - step(coef + 0.9, segmentRatio);
          vec4 colorA = instanceTargetColors;
          vec4 colorB = vec4(instanceTargetColors.r, instanceTargetColors.g, instanceTargetColors.b, 0.0);
          vec4 color = mix(colorA, colorB, pct) / 255.;
          vColor = color;
        }
        `,
          "fs:#main-start": `
        if (vColor.a == 0.0) discard;
        `,
        },
      });
    }

    draw(opts) {
      const { coef } = this.props;
      // add uniforms
      const uniforms = Object.assign({}, opts.uniforms, { coef: coef });
      super.draw(Object.assign({}, opts, { uniforms }));
    }
  }
</script>
