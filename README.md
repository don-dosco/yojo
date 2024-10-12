<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Don Dosco - Juego de la Vendimia Mejorado</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: 'Arial', sans-serif;
            text-align: center;
            background-color: #ececec;
        }

        #game-container {
            width: 90%;
            max-width: 500px;
            height: 400px;
            position: relative;
            margin: 20px auto;
            background-image: url('f/viñedos.jpg');
            background-size: cover;
            border: 2px solid #333;
            overflow: hidden;
            border-radius: 15px;
        }

        #don-dosco {
            width: 50px;
            height: 50px;
            position: absolute;
            background-image: url('f/don\ 2.jpeg');
            background-size: cover;
            bottom: 10px;
            left: calc(50% - 25px);
        }

        .grape, .bad-grape, .bonus {
            width: 30px;
            height: 30px;
            position: absolute;
            background-size: cover;
        }

        .grape {
            background-image: url('f/uva.jpg');
        }

        .bad-grape {
            background-image: url('f/podri\ 1.jpg');
        }

        .bonus {
            background-image: url('f/bonus.jpg');
        }

        #score-board, #game-info {
            margin: 20px;
            font-size: 18px;
        }

        #leaderboard {
            list-style-type: none;
            padding: 0;
        }

        #leaderboard li {
            margin: 5px 0;
        }

        button {
            padding: 10px 20px;
            font-size: 16px;
            background-color: #4CAF50;
            color: white;
            border: none;
            cursor: pointer;
            border-radius: 5px;
        }

        button:hover {
            background-color: #45a049;
        }

        #name-container {
            margin: 20px;
        }

        #name-container input {
            padding: 10px;
            font-size: 16px;
            border-radius: 5px;
            border: 1px solid #ccc;
        }

        #life-bar {
            width: 100%;
            background-color: #ddd;
            border-radius: 10px;
            margin: 10px 0;
        }

        #life {
            width: 100%;
            height: 20px;
            background-color: green;
            border-radius: 10px;
        }

        /* Responsive Design */
        @media (max-width: 600px) {
            #game-container {
                width: 100%;
                height: 300px;
            }

            #don-dosco {
                width: 40px;
                height: 40px;
            }

            .grape, .bad-grape, .bonus {
                width: 25px;
                height: 25px;
            }

            button {
                font-size: 14px;
                padding: 8px 16px;
            }
        }
    </style>
</head>
<body>
    <h1>Juego de la Vendimia - Don Dosco</h1>

    <!-- Formulario para el nombre del jugador -->
    <div id="name-container">
        <p>Ingresa tu nombre:</p>
        <input type="text" id="player-name" placeholder="Tu nombre">
        <button id="start-button">Iniciar Juego</button>
    </div>

    <!-- Barra de vida -->
    <div id="life-bar">
        <div id="life"></div>
    </div>

    <!-- Contenedor del juego -->
    <div id="game-container">
        <div id="don-dosco"></div>
    </div>

    <!-- Tabla de clasificación -->
    <div id="score-board">
        <h2>Tabla de Clasificación</h2>
        <ul id="leaderboard">
            <!-- Aquí se agregarán los jugadores y sus puntajes -->
        </ul>
    </div>

    <div id="game-info">
        <p>Tiempo restante: <span id="timer">30</span> segundos</p>
        <p>Puntaje: <span id="score">0</span></p>
    </div>

    <script>
        document.addEventListener("DOMContentLoaded", () => {
            const donDosco = document.getElementById("don-dosco");
            const gameContainer = document.getElementById("game-container");
            const scoreElement = document.getElementById("score");
            const timerElement = document.getElementById("timer");
            const leaderboard = document.getElementById("leaderboard");
            const startButton = document.getElementById("start-button");
            const playerNameInput = document.getElementById("player-name");
            const lifeBar = document.getElementById("life");

            let score = 0;
            let timeLeft = 30;
            let gameInterval;
            let movementSpeed = 20;
            let playerName = "";
            let grapeInterval;
            let badGrapeInterval;
            let bonusInterval;
            let life = 100;
            let level = 1;

            // Recuperar puntuación guardada
            const savedPlayers = JSON.parse(localStorage.getItem('leaderboard')) || [];

            // Mostrar puntuaciones guardadas
            savedPlayers.forEach(player => {
                const playerEntry = document.createElement("li");
                playerEntry.textContent = `${player.name}: ${player.score}`;
                leaderboard.appendChild(playerEntry);
            });

            // Mover a Don Dosco
            document.addEventListener("keydown", (event) => {
                let left = parseInt(getComputedStyle(donDosco).left);

                if (event.key === "ArrowLeft" && left > 0) {
                    donDosco.style.left = `${left - movementSpeed}px`;
                } else if (event.key === "ArrowRight" && left < 450) {
                    donDosco.style.left = `${left + movementSpeed}px`;
                }
            });

            // Iniciar el juego
            startButton.addEventListener("click", () => {
                playerName = playerNameInput.value;
                if (!playerName) {
                    alert("Por favor ingresa tu nombre.");
                    return;
                }

                score = 0;
                timeLeft = 30;
                life = 100;
                level = 1;
                scoreElement.textContent = score;
                timerElement.textContent = timeLeft;
                startButton.disabled = true;
                lifeBar.style.width = `${life}%`;

                // Generar uvas, uvas podridas y bonos
                grapeInterval = setInterval(createGrape, 1000 / level);
                badGrapeInterval = setInterval(createBadGrape, 3000 / level);
                bonusInterval = setInterval(createBonus, 5000 / level);

                // Contador de tiempo
                gameInterval = setInterval(() => {
                    timeLeft--;
                    timerElement.textContent = timeLeft;

                    if (timeLeft <= 0 || life <= 0) {
                        clearInterval(gameInterval);
                        clearInterval(grapeInterval);
                        clearInterval(badGrapeInterval);
                        clearInterval(bonusInterval);
                        alert(`¡Juego terminado! Puntaje final: ${score}`);
                        startButton.disabled = false;
                        updateLeaderboard(playerName, score);
                    }
                }, 1000);
            });

            // Crear uvas
            function createGrape() {
                const grape = document.createElement("div");
                grape.classList.add("grape");
                grape.style.top = `${Math.random() * 370}px`;
                grape.style.left = `${Math.random() * 470}px`;
                gameContainer.appendChild(grape);

                grape.addEventListener("click", () => {
                    gameContainer.removeChild(grape);
                    score++;
                    scoreElement.textContent = score;
                    if (score % 10 === 0) {
                        levelUp();
                    }
                });
            }

            // Crear uvas podridas (restan puntos)
            function createBadGrape() {
                const badGrape = document.createElement("div");
                badGrape.classList.add("bad-grape");
                badGrape.style.top = `${Math.random() * 370}px`;
                badGrape.style.left = `${Math.random() * 470}px`;
                gameContainer.appendChild(badGrape);

                badGrape.addEventListener("click", () => {
                    gameContainer.removeChild(badGrape);
                    score--;
                    scoreElement.textContent = score;
                    life -= 10;
                    lifeBar.style.width = `${life}%`;
                });
            }

            // Crear bono (da puntos extra)
            function createBonus() {
                const bonus = document.createElement("div");
                bonus.classList.add("bonus");
                bonus.style.top = `${Math.random() * 370}px`;
                bonus.style.left = `${Math.random() * 470}px`;
                gameContainer.appendChild(bonus);

                bonus.addEventListener("click", () => {
                    gameContainer.removeChild(bonus);
                    score += 5;
                    scoreElement.textContent = score;
                });
            }

            // Aumentar la dificultad con el nivel
            function levelUp() {
                level++;
                clearInterval(grapeInterval);
                clearInterval(badGrapeInterval);
                clearInterval(bonusInterval);
                grapeInterval = setInterval(createGrape, 1000 / level);
                badGrapeInterval = setInterval(createBadGrape, 3000 / level);
                bonusInterval = setInterval(createBonus, 5000 / level);
            }

            // Actualizar tabla de clasificación
            function updateLeaderboard(name, score) {
                const playerEntry = document.createElement("li");
                playerEntry.textContent = `${name}: ${score}`;
                leaderboard.appendChild(playerEntry);

                savedPlayers.push({ name, score });
                localStorage.setItem("leaderboard", JSON.stringify(savedPlayers));
            }
        });
    </script>
</body>
</html>
