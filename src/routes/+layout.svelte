<script>
	import P5 from 'p5-svelte';
	import lodash from 'lodash';


	class ParticleQueue {
		constructor(particles, limit) {
			this.particles = particles;
			this.queue = [];
			this.limit = limit*particles;

			// Create a proxy for custom indexing
			this.proxy = new Proxy(this.queue, {
				get: (target, prop) => {
					if (typeof prop === 'string' && /^\d+$/.test(prop)) {
						const index = parseInt(prop);
						return this.queue[index];
					} else {
						return target[prop];
					}
				}
			});
		}

		enqueue(particle_list) {
			if (particle_list instanceof Array) {
				this.queue.push(...particle_list);
			} else {
				this.queue.push(particle_list);
			}

			if (this.queue.length > this.limit) {
				let startIndex = this.queue.length - this.limit;
				this.queue.splice(0, startIndex);
			}
		}

		getFirst() {
			// return this.queue.slice(0, this.particles);
			return lodash.cloneDeep(this.queue.slice(-this.particles));
			
		}

		getAll() {
			return this.queue;
		}

		getLength() {
			return this.queue.length;
		}
	}

	const sketch = (p5) => {
		const numParticles = 50;
		const trailLength = 50;
		let particleQueue = new ParticleQueue(numParticles, trailLength);
		const fps = 15;
		const noiseScale = 0.01;
		const repellingRadius = 100;
		const minNumber = -50;
		const maxNumber = 50;

		p5.windowResized = () => {
			const clientHeight = p5.windowHeight;
			const clientWidth = p5.windowWidth;
			p5.resizeCanvas(clientWidth, clientHeight);
		};

		p5.setup = () => {
			const clientHeight = p5.windowHeight;
			const clientWidth = p5.windowWidth;
			p5.createCanvas(clientWidth, clientHeight);
			p5.frameRate(fps);
			for (let i = 0; i < numParticles; i++) {
				particleQueue.enqueue(p5.createVector(p5.random(p5.width), p5.random(p5.height)));
			}
			p5.stroke(150);
			// p5.background(20, 20, 40, 10);
		};

		p5.draw = () => {
			// make background blue
			p5.background(20, 20, 40, 10);

			let particles = particleQueue.getFirst();
			

			for (let i = 0; i < numParticles; i++) {
				//create point
				let thisParticle = particles[i];

				// //repel from mouse cursor
				let adjuster = 1;
				//apply force from mouse cursor
				let distance = p5.dist(p5.mouseX, p5.mouseY, thisParticle.x, thisParticle.y);
				if (distance < repellingRadius) {
					let angle = p5.atan2(thisParticle.y - p5.mouseY, thisParticle.x - p5.mouseX);
					thisParticle.x += p5.cos(angle) * 0.9;
					thisParticle.y += p5.sin(angle) * 0.9;
					adjuster = 400;
				}

				let thisNoiseScale = noiseScale * adjuster;

				// noise based movement
				let n = p5.noise(thisParticle.x * thisNoiseScale, thisParticle.y * thisNoiseScale);
				let a = p5.TAU * n;
				thisParticle.x += p5.cos(a);
				thisParticle.y += p5.sin(a);

				offScreen(thisParticle);
			}
			
			particleQueue.enqueue(particles);
			particles = particleQueue.getAll();

			// Batch drawing
			p5.beginShape();
			for (let i = 0; i < particleQueue.getLength(); i++) {
				p5.point(particles[i].x, particles[i].y);
			}
			p5.endShape(p5.CLOSE);
		};

		function offScreen(vect) {
			if (vect.x > 0 && vect.x < p5.width && vect.y > 0 && vect.y < p5.height) {
				return;
			}
			vect.x = p5.random(p5.width);
			vect.y = p5.random(p5.height);
			return;
		}

		// function offScreenX(vect) {
		// 	if (vect.x > 0 && vect.x < p5.width) {
		// 		return;
		// 	}
		// 	let randomInt = p5.floor(p5.random(minNumber, maxNumber + 1));
		// 	if (vect.x <= 0) {
		// 		vect.x = p5.width - 1;
		// 		vect.y = vect.y + randomInt;
		// 		return;
		// 	}
		// 	if (vect.x > p5.width) {
		// 		vect.x = 1;
		// 		vect.y = vect.y + randomInt;
		// 		return;
		// 	}
		// }

		// function offScreenY(vect) {
		// 	if (vect.y > 0 && vect.y < p5.height) {
		// 		return;
		// 	}
		// 	let randomInt = p5.floor(p5.random(minNumber, maxNumber + 1));
		// 	if (vect.y <= 0) {
		// 		vect.y = p5.height - 1;
		// 		vect.x = vect.x + randomInt;
		// 		return;
		// 	}
		// 	if (vect.y > p5.height) {
		// 		vect.y = 1;
		// 		vect.x = vect.x + randomInt;
		// 		return;
		// 	}
		// }

		// function offScreenXResize(vect) {
		// 	if (vect.x > 0 && vect.x < p5.width) {
		// 		return;
		// 	}
		// 	vect.x = p5.random(p5.width);
		// 	vect.y = p5.random(p5.height);
		// 	return;
		// }

		// function offScreenYResize(vect) {
		// 	if (vect.y > 0 && vect.y < p5.height) {
		// 		return;
		// 	}
		// 	vect.x = p5.random(p5.width);
		// 	vect.y = p5.random(p5.height);
		// 	return;
		// }
	};

	function toggle() {
		window.document.body.classList.toggle('light-mode');
	}
</script>

<background>
	<div class="p5-wrapper">
		<P5 {sketch} id="P5" />
	</div>
</background>

<body>
	<nav class="layout">
		<a prefetch href="/">Home</a>
		<a prefetch href="/projects">Projects</a>
		<a prefetch href="/contact">Contact</a>
		<button on:click={toggle}>Toggle mode</button>
	</nav>
	<main class="layout">
		<slot />
	</main>

	<footer class="layout">© Acme Corporation 2022</footer>
</body>

<style>
	/* Optional styles for the canvas */
	background {
		background-color: black;
		height: 100%;
		width: 100%;
		position: fixed;
		top: 0;
		left: 0;
		z-index: -1;
	}

	nav {
		display: flex;
		justify-content: center;
		padding-top: 1rem;
		padding-bottom: 1rem;
	}

	nav a {
		width: 20%; /* this sets the width of each anchor tag to 20%, so the total nav width is 40% */
		text-align: center; /* centers the text within the anchor tag */
	}
	nav,
	footer {
		text-transform: uppercase;
		font-size: 0.7rem;
		letter-spacing: 0.1px;
		font-weight: 500;
	}

	:global(html) {
		/* scrollbar-gutter: stable; */
		overflow-y: scroll;
		height: 100%;
		height: 100%;
	}

	:global(.layout) {
		--width: 700px;
		
		padding-left: max(1rem, calc(calc(100vw - var(--width)) / 2));
		padding-right: max(1rem, calc(calc(100vw - var(--width)) / 2));
	}

	:global(body) {
		background-color: rgb(0, 0, 0,0);
		color: white;
		margin: 0;
		height: 100%;
		width: 100%;
	}

	:global(body.light-mode) {
		background-color: #bfc2c7;
		color: black;
	}

	button {
		background-color: #f76027;
		color: white;
		border: none;
		border-radius: 4px;
		padding: 0.5rem;
		text-transform: uppercase;
	}

	:global(body.light-mode) button {
		background-color: white;
		color: black;
	}

	main {
		margin-bottom: 5rem;
		margin-top: 5vw;
	}

	.p5-wrapper {
		position: fixed;
		width: 100%;
		height: 100%;
		top: 0;
		left: 0;
		z-index: -1;
	}

	footer {
		max-height: 40px;
		text-align: center;
		margin: 0;
	}
</style>
