<script>
	import P5 from 'p5-svelte';

	let width = 700;
	let height = 700;
	let brushSize = 30;
    let color = [0, 0, 0, 100];

	let sketch = (p5) => {
		p5.setup = () => {
			p5.createCanvas(width, height);
			p5.background(200);
			p5.frameRate(60);
            p5.fill(0, color[3]);
            p5.noStroke();
		};

		p5.mouseDragged = () => {
            // check that the mouse is on the canvas
            if (p5.mouseX < 0 || p5.mouseX > width || p5.mouseY < 0 || p5.mouseY > height) {
                return;
            }
            p5.fill(color[0], color[1], color[2], color[3]);
			p5.ellipse(p5.mouseX, p5.mouseY, brushSize);
		};

		p5.keyPressed = () => {
            // clear the canvas
			if (p5.keyCode === 32) {
				p5.background(200);
			}
		};

		let brushSlider = document.getElementById('brushSlider');
		brushSlider.addEventListener('input', () => {
			brushSize = brushSlider.value;
		});

        let opacitySlider = document.getElementById('opacitySlider');
        opacitySlider.addEventListener('input', () => {
            color[3] = Number(opacitySlider.value);
        });
	};
</script>

<div class="w-full text-center justify-center mt-4">
	<h1 class="text-xl font-bold">Welcome to Pal Pad</h1>
	<p>click and drag to draw, press [SPACE] to clear</p>
	<div class="w-full flex justify-center p-2">
		<P5 {sketch} />
	</div>
    Color: <input type="color" value="#000000" on:change={(e) => {
        let hexColor = e.target.value;
        color[0] = parseInt(hexColor.slice(1, 3), 16);
        color[1] = parseInt(hexColor.slice(3, 5), 16);
        color[2] = parseInt(hexColor.slice(5, 7), 16);
    }} />
	Size ({brushSize}): <input id="brushSlider" type="range" min="1" max="100" value="4" class="slider" />
    Opacity ({color[3]}): <input id="opacitySlider" type="range" min="1" max="100" value={color[3]} class="slider" />
	<p>
		Visit
		<a href="https://github.com/CodyWMitchell/pal-pad-ui" class="text-blue-500 underline"
			>Our GitHub</a
		>
		to read the documentation
	</p>
</div>
