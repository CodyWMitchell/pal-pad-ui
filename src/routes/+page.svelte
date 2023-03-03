<script>
	import P5 from 'p5-svelte';
	import Pressure from 'pressure';
	import { page } from '$app/stores';
	import { goto } from '$app/navigation';
	import { browser } from '$app/environment';
	import Fa from 'svelte-fa/src/fa.svelte';
	import { faCopy } from '@fortawesome/free-regular-svg-icons';
	import { io } from 'socket.io-client';
	import { compute_rest_props } from 'svelte/internal';

	let width = 1280;
	let height = 720;
	let brushSize = 30;
	let color = [0, 0, 0, 100];
	let pressure = 0;
	let paperTexture;
	
	let roomID;
	if (browser) {
			// check if the room urlParam is set, if not then generate a random room
		roomID = $page.url.searchParams.get('room');
		if (!roomID) {
			roomID = Math.random().toString(36).substring(2, 15) + Math.random().toString(36).substring(2, 15);
			$page.url.searchParams.set('room', roomID);
			goto(`?${$page.url.searchParams.toString()}`);
		}
	}

	const socket = io('https://pal-pad.codymitchell.dev');
	
	let sketch = (p5) => {
		let currentLine = [];

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
			p5.image(paperTexture, 0, 0, width, height);
		}

		p5.preload = () => {
			paperTexture = p5.loadImage('/paper.jpg');
		};

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
			
			let pointData = {
				startX: p5.pmouseX,
				startY: p5.pmouseY,
				endX: p5.mouseX,
				endY: p5.mouseY,
				size: brushSize * (pressure + 1)
			};

			currentLine.push(pointData);
		};

		p5.mouseReleased = () => {
			// check that the mouse is on the canvas
			if (currentLine.length === 0) {
				return;
			}


			console.log('SOCKET EMIT: line')
			console.log({
				"sketch_id": roomID,
				"line": {
					"color": color,
					"points": currentLine
				}
			})

			socket.emit('line', roomID, {
				"color": color,
				"points": currentLine
			});

			currentLine = [];
		};

		p5.keyPressed = () => {
			// clear the canvas
			if (p5.keyCode === 32) {
				socket.emit('clear', roomID);
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
			console.log("SOCKET EMIT: clear")
			socket.emit('clear', roomID);
			clearCanvas(p5);
		});

		// join the room
		socket.emit('join', roomID);

		// draws the previous lines on the server
		socket.on('sketch', (data) => {
			console.log('SOCKET: sketch')
			console.log(data);
			if (data) {
				data.lines.forEach((line) => {
					p5.fill(line.color[0], line.color[1], line.color[2], line.color[3]);
					line.points.forEach((point) => {
						interpolateLine(point.startX, point.startY, point.endX, point.endY, point.size, 15);
					});
				});
			}
		});

		socket.on('clear', () => {
			console.log('SOCKET: clear')
			clearCanvas(p5);
		});

		socket.on('sync', (data) => {
			console.log('SOCKET: sync')
			console.log(data);
			if (data) {
				p5.fill(data.color[0], data.color[1], data.color[2], data.color[3]);
				data.points.forEach((point) => {
					interpolateLine(point.startX, point.startY, point.endX, point.endY, point.size, 15);
				});
			}
		});
	};
</script>

<div class="w-full h-screen text-white flex flex-col xl:flex-row bg-neutral-900">
	<div class="flex flex-row xl:flex-col gap-2 bg-neutral-800 p-4 justify-center items-center">
		<input
			type="color"
			value="#000000"
			on:change={(e) => {
				let hexColor = e.target.value;
				color[0] = parseInt(hexColor.slice(1, 3), 16);
				color[1] = parseInt(hexColor.slice(3, 5), 16);
				color[2] = parseInt(hexColor.slice(5, 7), 16);
			}}
		/>
		<input id="brushSlider" type="range" min="1" max="150" class="xl:vertical-slider" />
		<input
			id="opacitySlider"
			type="range"
			min="1"
			max="75"
			value={color[3]}
			class="xl:vertical-slider"
		/>
		<button id="clearButton" class="bg-neutral-700 w-12 aspect-square rounded text-white font-bold hover:bg-neutral-900"
			>clear</button
		>
	</div>
	<div class="w-full flex justify-center items-center p-2">
		<P5 {sketch} />
	</div>
	<div class="text-2xl">
		<button class="text-blue-500" on:click={() => navigator.clipboard.writeText(`${window.location.origin}/?room=${roomID}`)}>
			<Fa class="m-2" size="1.5x" icon={faCopy} />
		</button>
	</div>
</div>
