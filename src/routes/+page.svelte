<script>
	import P5 from 'p5-svelte';

	let width = 700;
	let height = 700;
	let brushSize = 30;

	let sketch = (p5) => {
		p5.setup = () => {
			p5.createCanvas(width, height);
			p5.background(200);
			p5.frameRate(60);
		};

		p5.mouseDragged = () => {
			p5.strokeWeight(brushSize);
			p5.line(p5.mouseX, p5.mouseY, p5.pmouseX, p5.pmouseY);
		};

		p5.keyPressed = () => {
			if (p5.keyCode === 32) {
				p5.background(200);
			}
		};

		let randomColor = document.getElementById('randomColor');
		randomColor.addEventListener('click', () => {
			let color = [p5.random(255), p5.random(255), p5.random(255)];
			p5.stroke(color);

			randomColor.style.backgroundColor = `rgb(${color[0]}, ${color[1]}, ${color[2]})`;

			if (color[0] + color[1] + color[2] < (255 * 3) / 2) {
				randomColor.style.color = 'white';
			} else {
				randomColor.style.color = 'black';
			}
		});

		let brushSlider = document.getElementById('brushSlider');
		brushSlider.addEventListener('input', () => {
			brushSize = brushSlider.value;
		});
	};
</script>

<div class="w-full text-center justify-center mt-4">
	<h1 class="text-xl font-bold">Welcome to Pal Pad</h1>
	<p>click and drag to draw, press [SPACE] to clear</p>
	<div class="w-full flex justify-center p-2">
		<P5 {sketch} />
	</div>
	<button
		id="randomColor"
		class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded"
	>
		Random Color
	</button>
	Size: <input id="brushSlider" type="range" min="1" max="100" value="4" class="slider" />
	<p>
		Visit
		<a href="https://github.com/CodyWMitchell/pal-pad-ui" class="text-blue-500 underline"
			>Our GitHub</a
		>
		to read the documentation
	</p>
</div>
