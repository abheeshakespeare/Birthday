<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Fireworks Animation</title>
    <style>
        body {
            margin: 0;
            background: #020202;
            cursor: crosshair;
            overflow: hidden;
        }
        canvas {
            display: block;
        }
        h3 {
            position: absolute;
            top: 40%;
            left: 50%;
            transform: translate(-50%, -50%);
            color: #fff;
            font-family: "Source Sans Pro", sans-serif;
            font-size: 5em;
            font-weight: 900;
            -webkit-user-select: none;
            user-select: none;
            text-shadow: 0 0 15px rgba(255, 255, 255, 0.8); /* Increased text shadow */
            z-index: 1;
            transition: transform 0.05s ease; /* Faster transition for dancing effect */
        }
    </style>
</head>
<body>
    <h3 id="message">Happy Birthday Jagriti!</h3>
    <canvas id="birthday"></canvas>

    <script>
        // Helper functions
        const PI2 = Math.PI * 2;
        const random = (min, max) => Math.random() * (max - min + 1) + min | 0;
        const timestamp = _ => new Date().getTime();

        // Container
        class Birthday {
            constructor() {
                this.resize();
                this.fireworks = [];
                this.counter = 0;
                this.messageAngle = 0; // To control the dancing effect
            }

            resize() {
                this.width = canvas.width = window.innerWidth;
                let center = this.width / 2 | 0;
                this.spawnA = center - center / 4 | 0;
                this.spawnB = center + center / 4 | 0;

                this.height = canvas.height = window.innerHeight;
                this.spawnC = this.height * .1;
                this.spawnD = this.height * .5;
            }

            onClick(evt) {
                let x = evt.clientX || evt.touches && evt.touches[0].pageX;
                let y = evt.clientY || evt.touches && evt.touches[0].pageY;

                let count = random(3, 5);
                for (let i = 0; i < count; i++) this.fireworks.push(new Firework(
                    random(this.spawnA, this.spawnB),
                    this.height,
                    x,
                    y,
                    random(0, 360), // Changed color range for more variety
                    random(30, 110)
                ));

                this.counter = -1;
            }

            update(delta) {
                ctx.globalCompositeOperation = 'hard-light';
                ctx.fillStyle = `rgba(20,20,20,${7 * delta})`;
                ctx.fillRect(0, 0, this.width, this.height);

                ctx.globalCompositeOperation = 'lighter';
                for (let firework of this.fireworks) firework.update(delta);

                this.counter += delta * 3;
                if (this.counter >= 1) {
                    this.fireworks.push(new Firework(
                        random(this.spawnA, this.spawnB),
                        this.height,
                        random(0, this.width),
                        random(this.spawnC, this.spawnD),
                        random(0, 360),
                        random(30, 110)
                    ));
                    this.counter = 0;
                }

                if (this.fireworks.length > 1000) this.fireworks = this.fireworks.filter(firework => !firework.dead);

                this.messageAngle += delta * 4; // Increased dancing speed
                this.updateMessage();
            }

            updateMessage() {
                const message = document.getElementById('message');
                const amplitude = 30; // Increased amplitude for more noticeable movement
                const frequency = 0.3; // Increased frequency for faster dancing
                const scale = 1 + 0.2 * Math.sin(frequency * this.messageAngle); // Increased scale effect
                const offsetX = amplitude * Math.sin(frequency * this.messageAngle);
                const offsetY = amplitude * Math.cos(frequency * this.messageAngle);

                message.style.transform = `translate(-50%, -50%) translate(${offsetX}px, ${offsetY}px) scale(${scale}) rotate(${Math.sin(frequency * this.messageAngle) * 15}deg)`;
            }
        }

        class Firework {
            constructor(x, y, targetX, targetY, shade, offsprings) {
                this.dead = false;
                this.offsprings = offsprings;
                this.x = x;
                this.y = y;
                this.targetX = targetX;
                this.targetY = targetY;
                this.shade = shade;
                this.history = [];
            }
            update(delta) {
                if (this.dead) return;

                let xDiff = this.targetX - this.x;
                let yDiff = this.targetY - this.y;
                if (Math.abs(xDiff) > 3 || Math.abs(yDiff) > 3) {
                    this.x += xDiff * 2 * delta;
                    this.y += yDiff * 2 * delta;

                    this.history.push({ x: this.x, y: this.y });

                    if (this.history.length > 20) this.history.shift();
                } else {
                    if (this.offsprings && !this.madeChilds) {
                        let babies = this.offsprings / 2;
                        for (let i = 0; i < babies; i++) {
                            let targetX = this.x + this.offsprings * Math.cos(PI2 * i / babies) | 0;
                            let targetY = this.y + this.offsprings * Math.sin(PI2 * i / babies) | 0;

                            birthday.fireworks.push(new Firework(this.x, this.y, targetX, targetY, this.shade, 0));
                        }
                    }
                    this.madeChilds = true;
                    this.history.shift();
                }

                if (this.history.length === 0) this.dead = true;
                else if (this.offsprings) {
                    for (let i = 0; this.history.length > i; i++) {
                        let point = this.history[i];
                        ctx.beginPath();
                        ctx.fillStyle = 'hsl(' + this.shade + ',100%,' + Math.min(i * 5, 100) + '%)'; // Increased brightness
                        ctx.arc(point.x, point.y, 2, 0, PI2, false); // Increased size
                        ctx.fill();
                    }
                } else {
                    ctx.beginPath();
                    ctx.fillStyle = 'hsl(' + this.shade + ',100%,50%)';
                    ctx.arc(this.x, this.y, 2, 0, PI2, false); // Increased size
                    ctx.fill();
                }
            }
        }

        let canvas = document.getElementById('birthday');
        let ctx = canvas.getContext('2d');

        let then = timestamp();
        let birthday = new Birthday();
        window.onresize = () => birthday.resize();
        document.onclick = evt => birthday.onClick(evt);
        document.ontouchstart = evt => birthday.onClick(evt);

        (function loop() {
            requestAnimationFrame(loop);
            let now = timestamp();
            let delta = now - then;
            then = now;
            birthday.update(delta / 1000);
        })();
    </script>
</body>
</html>