<script>
	import P5 from 'p5-svelte';

	let width = 700;
	let height = 700;

	let sketch = (p5) => {
		p5.setup = () => {
			p5.createCanvas(width, height);
			p5.background(200);
            p5.frameRate(60);
		};

		p5.mouseDragged = () => {
			p5.strokeWeight(4);
			p5.line(p5.mouseX, p5.mouseY, p5.pmouseX, p5.pmouseY);
		};

		p5.keyPressed = () => {
			if (p5.keyCode === 32) {
				p5.background(200);
			}
		};
	};
</script>

<div class="w-full text-center justify-center mt-4">
	<h1 class="text-xl font-bold">Welcome to Pal Pad</h1>
	<p>click and drag to draw, press [SPACE] to clear</p>
	<div class="w-full flex justify-center">
		<P5 {sketch} />
	</div>
	<p>
		Visit <a href="https://github.com/CodyWMitchell/pal-pad-ui" class="text-blue-500 underline"
			>Our GitHub</a
		> to read the documentation
	</p>
</div>
