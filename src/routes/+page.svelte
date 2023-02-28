<script>
	import P5 from 'p5-svelte';
	import Pressure from 'pressure';

	let width = 1280;
	let height = 720;
	let brushSize = 30;
	let color = [0, 0, 0, 100];
	let pressure = 0;

	let sketch = (p5) => {
		const interpolateLine = (startX, startY, endX, endY, circleSize, numCircles) => {
			let xDiff = endX - startX;
			let yDiff = endY - startY;
			let xStep = xDiff / numCircles;
			let yStep = yDiff / numCircles;
			for (let i = 0; i < numCircles; i++) {
				p5.ellipse(startX + xStep * i, startY + yStep * i, circleSize, circleSize);
			}
		};

		function clearCanvas() {
			p5.background(50);
		}

		p5.setup = () => {
			p5.createCanvas(width, height);
			clearCanvas();
			p5.noStroke();

			// set up pressure.js
			Pressure.set(
				'canvas',
				{
					change: (force, event) => {
						pressure = force;
					},
					end: () => {
						pressure = 0;
					}
				},
				{
					polyfill: false
				}
			);
		};

		p5.mouseDragged = () => {
			// check that the mouse is on the canvas
			if (p5.mouseX < 0 || p5.mouseX > width || p5.mouseY < 0 || p5.mouseY > height) {
				return;
			}
			p5.fill(color[0], color[1], color[2], color[3]);
			interpolateLine(p5.pmouseX, p5.pmouseY, p5.mouseX, p5.mouseY, brushSize * (pressure + 1), 15);
		};

		p5.keyPressed = () => {
			// clear the canvas
			if (p5.keyCode === 32) {
				clearCanvas();
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

		let clearButton = document.getElementById('clearButton');
		clearButton.addEventListener('click', () => {
			clearCanvas(p5);
		});
	};
</script>

<div class="w-full h-screen text-white text-lg p-4 flex flex-col xl:flex-row bg-neutral-900">
	<div class="flex flex-row xl:flex-col gap-1">
		Color:<input
			type="color"
			value="#000000"
			on:change={(e) => {
				let hexColor = e.target.value;
				color[0] = parseInt(hexColor.slice(1, 3), 16);
				color[1] = parseInt(hexColor.slice(3, 5), 16);
				color[2] = parseInt(hexColor.slice(5, 7), 16);
			}}
		/>
		Size ({Math.floor(brushSize)}):
		<input id="brushSlider" type="range" min="1" max="150" class="slider" />
		Opacity ({color[3]}):
		<input id="opacitySlider" type="range" min="1" max="75" value={color[3]} class="slider" />
		Pressure: {Math.floor(pressure * 100)}
		<button id="clearButton" class="bg-blue-500 p-2 rounded text-white font-bold hover:bg-blue-700"
			>Clear</button
		>
	</div>
	<div class="w-full flex justify-center p-2">
		<P5 {sketch} />
	</div>
</div>
