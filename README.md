<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Galton Board Simulation</title>
    <script src="https://unpkg.com/react@17.0.2/umd/react.development.js"></script>
    <script src="https://unpkg.com/react-dom@17.0.2/umd/react-dom.development.js"></script>
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
    <style>
        body {
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
            background-color: #F5DEB3;
        }
    </style>
</head>
<body>
    <div id="root"></div>
    <script type="text/babel">
        const { useState, useEffect, useRef } = React;

        const SCREEN_WIDTH = 400;
        const SCREEN_HEIGHT = 600;
        const BOARD_WIDTH = SCREEN_WIDTH * 0.9;
        const BOARD_HEIGHT = SCREEN_HEIGHT * 0.7;
        const PIN_RADIUS = 5;
        const BALL_RADIUS = 7;
        const ROWS = 10;

        const GaltonBoard = () => {
          const [ballCount, setBallCount] = useState(50);
          const [isRunning, setIsRunning] = useState(false);
          const [balls, setBalls] = useState([]);
          const canvasRef = useRef(null);

          const createBall = () => ({
            x: BOARD_WIDTH / 2,
            y: 0,
            vx: 0,
            vy: 0,
          });

          const moveBall = (ball) => {
            if (ball.y >= BOARD_HEIGHT - BALL_RADIUS) {
              ball.vy = 0;
              ball.vx = 0;
              return ball;
            }

            const row = Math.floor(ball.y / (BOARD_HEIGHT / ROWS));
            const pinX = (BOARD_WIDTH / (ROWS + 1)) * (row + 1);
            const pinY = (BOARD_HEIGHT / ROWS) * row;

            if (Math.abs(ball.y - pinY) < PIN_RADIUS + BALL_RADIUS && Math.abs(ball.x - pinX) < PIN_RADIUS + BALL_RADIUS) {
              ball.vx = Math.random() > 0.5 ? 2 : -2;
            }

            ball.vy += 0.2;
            ball.x += ball.vx;
            ball.y += ball.vy;

            return ball;
          };

          useEffect(() => {
            if (isRunning) {
              const interval = setInterval(() => {
                setBalls(prevBalls => prevBalls.map(moveBall));
              }, 16);
              return () => clearInterval(interval);
            }
          }, [isRunning]);

          useEffect(() => {
            const canvas = canvasRef.current;
            const ctx = canvas.getContext('2d');

            const drawBoard = () => {
              ctx.fillStyle = '#8B4513';
              ctx.fillRect(0, 0, BOARD_WIDTH, BOARD_HEIGHT);
            };

            const drawPins = () => {
              ctx.fillStyle = '#D2691E';
              for (let i = 0; i < ROWS; i++) {
                for (let j = 0; j <= i; j++) {
                  const x = (BOARD_WIDTH / (i + 2)) * (j + 1);
                  const y = (BOARD_HEIGHT / ROWS) * i;
                  ctx.beginPath();
                  ctx.arc(x, y, PIN_RADIUS, 0, Math.PI * 2);
                  ctx.fill();
                }
              }
            };

            const drawBalls = () => {
              ctx.fillStyle = 'red';
              balls.forEach(ball => {
                ctx.beginPath();
                ctx.arc(ball.x, ball.y, BALL_RADIUS, 0, Math.PI * 2);
                ctx.fill();
              });
            };

            const render = () => {
              ctx.clearRect(0, 0, BOARD_WIDTH, BOARD_HEIGHT);
              drawBoard();
              drawPins();
              drawBalls();
              requestAnimationFrame(render);
            };

            render();
          }, [balls]);

          return (
            <div style={{ display: 'flex', flexDirection: 'column', alignItems: 'center' }}>
              <canvas ref={canvasRef} width={BOARD_WIDTH} height={BOARD_HEIGHT} />
              <div style={{ marginTop: '20px' }}>
                <button onClick={() => setBallCount(Math.max(1, ballCount - 10))}>-10 Balls</button>
                <span style={{ margin: '0 10px' }}>{ballCount} Balls</span>
                <button onClick={() => setBallCount(ballCount + 10)}>+10 Balls</button>
              </div>
              <button
                style={{ marginTop: '20px' }}
                onClick={() => {
                  if (!isRunning) {
                    setBalls(Array.from({ length: ballCount }, createBall));
                    setIsRunning(true);
                  } else {
                    setIsRunning(false);
                    setBalls([]);
                  }
                }}
              >
                {isRunning ? 'Stop' : 'Start'}
              </button>
            </div>
          );
        };

        ReactDOM.render(<GaltonBoard />, document.getElementById('root'));
    </script>
</body>
</html>
