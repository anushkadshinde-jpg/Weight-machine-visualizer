<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>LED Spiral Visualizer</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/animejs/3.2.1/anime.min.js"></script>
    <style>
        body {
            margin: 0;
            padding: 0;
            background-color: #0a0a0a;
            color: #fff;
            font-family: sans-serif;
            display: flex;
            flex-direction: column;
            align-items: center;
            height: 100vh;
            overflow: hidden;
        }
        #controls {
            padding: 20px;
            background: #1a1a1a;
            width: 100%;
            display: flex;
            justify-content: center;
            gap: 20px;
            box-shadow: 0 4px 6px rgba(0,0,0,0.3);
            z-index: 10;
        }
        .control-group {
            display: flex;
            flex-direction: column;
            gap: 5px;
        }
        label { font-size: 12px; text-transform: uppercase; letter-spacing: 1px; color: #aaa; }
        input[type="range"] { cursor: pointer; }
        
        #canvas-container {
            flex-grow: 1;
            width: 100%;
            display: flex;
            justify-content: center;
            align-items: center;
            position: relative;
        }
        canvas {
            display: block;
        }
    </style>
</head>
<body>

    <div id="controls">
        <div class="control-group">
            <label for="density">Bulb Density</label>
            <input type="range" id="density" min="2" max="15" value="8">
        </div>
        <div class="control-group">
            <label for="speed">Animation Speed</label>
            <input type="range" id="speed" min="500" max="3000" value="1500" dir="rtl">
        </div>
    </div>

    <div id="canvas-container">
        <canvas id="ledCanvas"></canvas>
    </div>

    <script>
        const canvas = document.getElementById('ledCanvas');
        const ctx = canvas.getContext('2d');
        const container = document.getElementById('canvas-container');
        
        let bulbs = [];
        let animationSequence = null;

        // Resize canvas to fit the container
        function resizeCanvas() {
            canvas.width = container.clientWidth;
            canvas.height = container.clientHeight;
            generatePattern();
        }

        window.addEventListener('resize', resizeCanvas);

        // Render loop
        function draw() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            
            bulbs.forEach(b => {
                // Outer Glow (Diffused light)
                const glow = ctx.createRadialGradient(b.x, b.y, 0, b.x, b.y, b.radius * 3);
                glow.addColorStop(0, `rgba(255, 180, 50, ${b.opacity * 0.8})`); // Warm orange/yellow
                glow.addColorStop(1, `rgba(255, 50, 0, 0)`);

                ctx.beginPath();
                ctx.arc(b.x, b.y, b.radius * 3, 0, Math.PI * 2);
                ctx.fillStyle = glow;
                ctx.fill();

                // Inner Dome (The physical LED bulb)
                ctx.beginPath();
                ctx.arc(b.x, b.y, b.radius, 0, Math.PI * 2);
                // When "off" it has a dark base state, when "on" it turns bright white/yellow
                const innerIntensity = 0.1 + (b.opacity * 0.9);
                ctx.fillStyle = `rgba(255, 240, 200, ${innerIntensity})`; 
                ctx.fill();
                
                // Add a small highlight to make it look like a glass dome
                ctx.beginPath();
                ctx.arc(b.x - b.radius*0.3, b.y - b.radius*0.3, b.radius*0.3, 0, Math.PI * 2);
                ctx.fillStyle = `rgba(255, 255, 255, ${0.2 + b.opacity*0.3})`;
                ctx.fill();
            });
            
            requestAnimationFrame(draw);
        }

        // Procedural Generation of the Spiral
        function generatePattern() {
            bulbs = [];
            const density = parseInt(document.getElementById('density').value);
            
            const centerX = canvas.width / 2;
            const centerY = canvas.height / 2;
            
            // Ensure pattern fits within the screen (prevent cropping)
            const maxRadius = Math.min(centerX, centerY) * 0.90; 
            const totalBulbs = density * 40; 
            
            // Generate Fermat's Spiral for natural packing
            const c = maxRadius / Math.sqrt(totalBulbs); 

            for (let i = 1; i <= totalBulbs; i++) {
                const angle = i * 137.5 * (Math.PI / 180); // Golden angle
                const r = c * Math.sqrt(i);
                
                const x = centerX + r * Math.cos(angle);
                const y = centerY + r * Math.sin(angle);
                
                // Dynamic bulb sizing - slightly larger towards the outside
                const baseRadius = 3 + (density * 0.2);
                const sizeMult = 1 + (r / maxRadius) * 0.5;

                bulbs.push({ 
                    id: i,
                    x: x, 
                    y: y, 
                    radius: baseRadius * sizeMult, 
                    opacity: 0.05 // Initial "off" state
                });
            }
            
            playAnimation();
        }

        // Handle the sequential blinking via Anime.js
        function playAnimation() {
            if (animationSequence) animationSequence.pause();
            
            const speed = parseInt(document.getElementById('speed').value);

            animationSequence = anime({
                targets: bulbs,
                opacity: [0.05, 1, 0.05], // Fade in and out
                delay: anime.stagger(speed / 50, {start: 0, from: 'center'}), // Sequential outward ripple
                loop: true,
                duration: speed,
                easing: 'easeInOutSine'
            });
        }

        // Event Listeners for Controls
        document.getElementById('density').addEventListener('input', generatePattern);
        document.getElementById('speed').addEventListener('input', playAnimation);

        // Initialize
        resizeCanvas();
        requestAnimationFrame(draw);

    </script>
</body>
</html>
