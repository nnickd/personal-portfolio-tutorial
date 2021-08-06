<!-- <script>
    import { onMount } from "svelte";
    
    let P5;
    
    onMount(async () => {
      const module = await import("p5-svelte");
      P5 = module.default;
    });
    
    let width = 55;
    let height = 55;
    
    const sketch = (p5) => {
      p5.setup = () => {
        p5.createCanvas(400, 400);
      };
    
      p5.draw = () => {
        p5.ellipse(p5.width / 2, p5.height / 2, width, height);
      };
    };
    </script>
    
  <label>
    Width
    <input type="range" bind:value={width} min="100" max="1000" step="0.01" />
    {width}
  </label>
  
  <label>
    Height
    <input type="range" bind:value={height} min="100" max="1000" step="0.01" />
    {height}
  </label>
  
  <svelte:component this={P5} {sketch} /> -->
























  