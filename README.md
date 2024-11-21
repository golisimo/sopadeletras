# sopadeletras <!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Sopa de Letras</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            text-align: center;
        }
        .grid {
            display: grid;
            grid-template-columns: repeat(10, 30px);
            gap: 5px;
            justify-content: center;
            margin-top: 20px;
        }
        .cell {
            width: 30px;
            height: 30px;
            display: flex;
            align-items: center;
            justify-content: center;
            background-color: #f0f0f0;
            border: 1px solid #ccc;
            font-size: 18px;
            font-weight: bold;
            text-transform: uppercase;
            user-select: none;
        }
        .cell.selected {
            background-color: #ffd700;
        }
        .cell.found {
            background-color: #90ee90;
            color: #fff;
        }
        .words {
            margin-top: 20px;
        }
        .found-word {
            text-decoration: line-through;
            color: green;
        }
        #ganaste {
            display: none;
            margin-top: 20px;
            font-size: 24px;
            color: green;
            font-weight: bold;
        }
    </style>
</head>
<body>
    <h1>Sopa de Letras</h1>
    <div id="sopa" class="grid"></div>
    <div class="words">
        <h3>Palabras a encontrar:</h3>
        <ul id="palabras"></ul>
    </div>
    <div id="ganaste">¡Ganaste! Felicidades 🎉</div>
    <script>
        const palabras = ["YAYO", "CHUPACABRAS", "SANTIAGO", "BOCA", "BRISA","PALOMITA"];
        const tamaño = 10;
        let seleccionadas = [];
        let encontradas = [];

        function crearSopaDeLetras(tamaño, palabras) {
            const sopa = Array.from({ length: tamaño }, () => Array(tamaño).fill(' '));
            const letras = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ';

            palabras.forEach(palabra => {
                let colocada = false;
                while (!colocada) {
                    const direccion = ['H', 'V', 'D'][Math.floor(Math.random() * 3)];
                    const fila = Math.floor(Math.random() * tamaño);
                    const columna = Math.floor(Math.random() * tamaño);

                    if (direccion === 'H' && columna + palabra.length <= tamaño) {
                        if (sopa[fila].slice(columna, columna + palabra.length).every((c, i) => c === ' ' || c === palabra[i])) {
                            palabra.split('').forEach((letra, i) => sopa[fila][columna + i] = letra);
                            colocada = true;
                        }
                    } else if (direccion === 'V' && fila + palabra.length <= tamaño) {
                        if (sopa.slice(fila, fila + palabra.length).every((filaSopa, i) => filaSopa[columna] === ' ' || filaSopa[columna] === palabra[i])) {
                            palabra.split('').forEach((letra, i) => sopa[fila + i][columna] = letra);
                            colocada = true;
                        }
                    } else if (direccion === 'D' && fila + palabra.length <= tamaño && columna + palabra.length <= tamaño) {
                        if (Array.from({ length: palabra.length }, (_, i) => sopa[fila + i][columna + i]).every((c, i) => c === ' ' || c === palabra[i])) {
                            palabra.split('').forEach((letra, i) => sopa[fila + i][columna + i] = letra);
                            colocada = true;
                        }
                    }
                }
            });

            for (let i = 0; i < tamaño; i++) {
                for (let j = 0; j < tamaño; j++) {
                    if (sopa[i][j] === ' ') {
                        sopa[i][j] = letras[Math.floor(Math.random() * letras.length)];
                    }
                }
            }

            return sopa;
        }

        function mostrarSopa(sopa) {
            const contenedor = document.getElementById('sopa');
            sopa.forEach((fila, filaIndex) => {
                fila.forEach((letra, columnaIndex) => {
                    const celda = document.createElement('div');
                    celda.classList.add('cell');
                    celda.textContent = letra;
                    celda.dataset.fila = filaIndex;
                    celda.dataset.columna = columnaIndex;
                    celda.addEventListener('click', () => seleccionarCelda(celda));
                    contenedor.appendChild(celda);
                });
            });
        }

        function mostrarPalabras(palabras) {
            const lista = document.getElementById('palabras');
            palabras.forEach(palabra => {
                const item = document.createElement('li');
                item.textContent = palabra;
                item.id = `palabra-${palabra}`;
                lista.appendChild(item);
            });
        }

        function seleccionarCelda(celda) {
            const fila = parseInt(celda.dataset.fila);
            const columna = parseInt(celda.dataset.columna);

            if (seleccionadas.some(sel => sel.fila === fila && sel.columna === columna)) {
                celda.classList.remove('selected');
                seleccionadas = seleccionadas.filter(sel => !(sel.fila === fila && sel.columna === columna));
            } else {
                celda.classList.add('selected');
                seleccionadas.push({ fila, columna, letra: celda.textContent });
            }

            verificarPalabra();
        }

        function verificarPalabra() {
            const palabraActual = seleccionadas.map(sel => sel.letra).join('');
            const palabraReversa = seleccionadas.map(sel => sel.letra).reverse().join('');

            if (palabras.includes(palabraActual) || palabras.includes(palabraReversa)) {
                seleccionadas.forEach(sel => {
                    const celda = document.querySelector(`.cell[data-fila="${sel.fila}"][data-columna="${sel.columna}"]`);
                    celda.classList.add('found');
                    celda.classList.remove('selected');
                });
                const palabraEncontrada = palabras.includes(palabraActual) ? palabraActual : palabraReversa;
                document.getElementById(`palabra-${palabraEncontrada}`).classList.add('found-word');
                encontradas.push(palabraEncontrada);
                seleccionadas = [];
                verificarGanador();
            }
        }

        function verificarGanador() {
            if (encontradas.length === palabras.length) {
                document.getElementById('ganaste').style.display = 'block';
            }
        }

        const sopa = crearSopaDeLetras(tamaño, palabras);
        mostrarSopa(sopa);
        mostrarPalabras(palabras);
    </script>
</body>
</html>
