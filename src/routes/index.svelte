<script>
	import { onMount } from 'svelte';
	import SimplexNoise from 'simplex-noise';
	import { PI, bg, green, colors, createGrid, Circle } from '$lib/utils/canvas';

	let smplx = [];

	let animationFrame,
		canvas,
		columns,
		container,
		ctx,
		cR,
		field,
		observer,
		rows,
		simplex,
		size,
		wX,
		wY;
	let time = 0;
	const inc = 0.003;
	let circles = [];

	function setup() {
		wX = container.offsetWidth;
		wY = container.offsetHeight;
		size = wX < 600 ? 20 : 24;
		canvas.style.width = wX + 'px';
		canvas.style.height = wY + 'px';
		canvas.width = wX;
		canvas.height = wY;
		columns = createGrid(wX, size);
		rows = createGrid(wY, size);
		initField();
		cR = (wX / 3) * 2;
	}

	function draw() {
		animationFrame = window.requestAnimationFrame(draw);
		clear();
		calculateField();
		drawField();
		drawCircles();
		time += inc;
	}

	// setup the grid of vectors
	function initField() {
		field = new Array(columns);
		for (let x = 0; x < columns; x++) {
			field[x] = new Array(rows);
			for (let y = 0; y < rows; y++) {
				field[x][y] = [0, 0];
			}
		}
	}

	// clear the canvas at the beginning of each frame
	function clear() {
		ctx.fillStyle = bg;
		ctx.fillRect(0, 0, wX, wY);
	}

	// calculate each frame of the animation
	function calculateField() {
		for (let x = 0; x < columns; x++) {
			for (let y = 0; y < rows; y++) {
				const noiseVal = simplex.noise3D(x / 50, y / 50, time);
				field[x][y][0] = noiseVal * (PI * 2);
				field[x][y][1] = noiseVal;
			}
		}
	}

	// run the animation
	function drawField() {
		for (let x = 0; x < columns; x++) {
			for (let y = 0; y < rows; y++) {
				const angle = field[x][y][0];
				const length = field[x][y][1];
				ctx.save();
				ctx.translate(x * size, y * size);
				ctx.rotate(angle);
				ctx.strokeStyle = green;
				ctx.beginPath();
				ctx.moveTo(0, 0);
				ctx.lineTo(0, size * length);
				ctx.stroke();
				ctx.restore();
			}
		}
	}

	function createCircles() {
		for (let i = 0; i < 2; i++) {
			circles.push(new Circle(colors[i]));
		}
	}

	function drawCircles() {
		const sine = [Math.sin(time) * (wX / 2) + wX / 2, Math.sin(time + PI) * (wX / 2) + wX / 2];
		const cosine = [Math.cos(time) * (wX / 2) + wX / 2, Math.cos(time + PI) * (wX / 2) + wX / 2];
		for (const [i, circle] of circles.entries()) {
			circle.update(sine[i], cosine[i]);
			circle.show(ctx, cR);
		}
	}

	onMount(() => {
		simplex = new SimplexNoise();
		ctx = canvas.getContext('2d');
		setup();
		createCircles();
		animationFrame = window.requestAnimationFrame(draw);

		observer = new ResizeObserver(() => {
			setup();
		});
		observer.observe(container);

		return () => {
			window.cancelAnimationFrame(animationFrame);
			observer.disconnect();
		};
	});
</script>

<section class="hero">
	<div bind:this={container} class="container">
		<canvas bind:this={canvas} id="canvas" width="400" height="400" />
	</div>
	<div class="text-wrapper">
		<h1>This is a test</h1>
	</div>
</section>

<style>
	section.hero {
		position: relative;
		display: flex;
		justify-content: center;
		align-items: center;
		width: 100%;
		min-height: 100vh;
		background: #f9f6ee;
	}

	div.container {
		width: 80vh;
		height: 80vh;
		aspect-ratio: 1/1;
		border-radius: 50%;
		overflow: hidden;
		/* background: green; */
	}

	canvas {
		display: block;
	}

	.text-wrapper {
		position: absolute;
		top: 0;
		left: 0;
		display: flex;
		justify-content: center;
		align-items: center;
		width: 100%;
		min-height: 100vh;
	}

	h1 {
		font-size: 4rem;
		font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell,
			'Open Sans', 'Helvetica Neue', sans-serif;
		font-weight: 900;
	}
</style>
