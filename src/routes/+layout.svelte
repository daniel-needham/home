<script>
	import P5 from 'p5-svelte';


	const sketch = (p5) => {
		let particles = [];
		const numParticles = 500;
		const noiseScale = 0.01;
		const repellingRadius = 100;
		const minNumber = -50;
		const maxNumber = 50;

		p5.setup = () => {
			let clientHeight = window.innerHeight;
			let clientWidth = window.innerWidth;
			
			p5.createCanvas(clientWidth, clientHeight);
			p5.frameRate(30);
			for (let i = 0; i < numParticles; i++) {
				particles.push(p5.createVector(p5.random(p5.width), p5.random(p5.height)));
			}
			p5.stroke(150);
		}

		p5.draw = () =>{
			p5.background(0,5);
			
			let mousePos = p5.createVector(p5.mouseX, p5.mouseY);
			
			for (let i = 0; i < numParticles; i++) {
				
				//create point
				let thisParticle = particles[i];
				p5.point(thisParticle.x, thisParticle.y);

				// //repel from mouse cursor
				let adjuster = 1;
				//apply force from mouse cursor
				let distance = p5.dist(p5.mouseX, p5.mouseY, thisParticle.x, thisParticle.y);
				if (distance < repellingRadius) {
					let angle = p5.atan2(thisParticle.y - p5.mouseY, thisParticle.x - p5.mouseX);
					thisParticle.x += p5.cos(angle) * 0.4;
					thisParticle.y += p5.sin(angle) * 0.4;
					adjuster = 1000;
				} else {
				}

				let thisNoiseScale = noiseScale * adjuster;

				//noise based movement
				let n = p5.noise(thisParticle.x * thisNoiseScale, thisParticle.y * thisNoiseScale);
				let a = p5.TAU * n;
				thisParticle.x += p5.cos(a);
				thisParticle.y += p5.sin(a);

				//check if particle off screen
				offScreenX(thisParticle);
				offScreenY(thisParticle);
			}
		}

		function offScreenX(vect) {
			if (vect.x > 0 && vect.x < p5.width) {
				return;
			}
			let randomInt = p5.floor(p5.random(minNumber, maxNumber + 1));
			if (vect.x <= 0) {
				vect.x = p5.width - 1;
				vect.y = vect.y + randomInt;
				return;
			}
			if (vect.x > p5.width) {
				vect.x = 1;
				vect.y = vect.y + randomInt;
				return;
			}
		}

		function offScreenY(vect) {
			if (vect.y > 0 && vect.y < p5.height) {
				return;
			}
			let randomInt = p5.floor(p5.random(minNumber, maxNumber + 1));
			if (vect.y <= 0) {
				vect.y = p5.height - 1;
				vect.x = vect.x + randomInt;
				return;
			}
			if (vect.y > p5.height) {
				vect.y = 1;
				vect.x = vect.x + randomInt;
				return;
			}
		}
	};
</script>

<nav class="layout">
	<a prefetch href="/">Home</a>
	<a prefetch href="/projects">Projects</a>
	<a prefetch href="/contact">Contact</a>
</nav>

<div class="p5-wrapper">
	<P5 {sketch} id="P5" />
  </div>

<main class="layout">
	<slot />
</main>

<footer class="layout">© Acme Corporation 2022</footer>

<style>
	/* Optional styles for the canvas */

	nav {
		display: flex;
		justify-content: center;
		background: rgba(0, 0, 0, 0.4);
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
		background: rgb(20, 20, 40);
		/* scrollbar-gutter: stable; */
		overflow-y: scroll;
	}

	:global(.layout) {
		--width: 700px;
		padding-left: max(1rem, calc(calc(100vw - var(--width)) / 2));
		padding-right: max(1rem, calc(calc(100vw - var(--width)) / 2));
	}

	main {
		margin-bottom: 5rem;
		margin-top: 5vw;
	}

	.p5-wrapper {
		position: fixed;
		top: 0;
		left: 0;
		z-index: -1;
	}

	footer {
		max-height: 40px;
		text-align: center;
		margin: 0;
		color: rgba(255, 255, 255, 0.4);
	}
</style>
