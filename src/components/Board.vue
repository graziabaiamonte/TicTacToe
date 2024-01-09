<template>
    <div class="container">

    
    <h1>Tic Tac Toe Game</h1>
    <div class="container">
        <div class="audio-toggle">
            <button class="audioToggleButton" @click="toggleAudio">
                <img :src="audioEnabled ? 'src/assets/audio.png' : 'src/assets/noAudio.png'" alt="Audio Toggle">
            </button>
        </div>

      <button class="toggleButton" @click="toggleGameMode">
        {{ SinglePlayerMode ? "Switch to Two Players" : "Switch to Single Player" }}
      </button>
      <h3 v-if="SinglePlayerMode">Game Mode: Single Player</h3>
      <h3 v-else>Game Mode: Two Players</h3>
  
      <div v-if="currentGame <= maxGames">
        <h2 v-if="winner">Winner: {{ winner }}</h2>
        <h2 v-else>Players Move: {{ player }}</h2>
        
      </div>
  
      <div v-else>
        <h2>Game over</h2>
        <button @click="restart" class="btn btn-primary mt-3 restartBtn">Restart</button>
      </div>
  
      <div class="grid-container">
          <div v-for="x in 3" :key="x" class="grid-row">
          <button v-for="y in 3" :key="y" class="square" @click="move(x - 1, y - 1); playClickSound()" 
          :class="{ winningCell: isWinningCell(x -1, y-1), 'player-X': squares[x - 1][y - 1] === 'X', 'player-O': squares[x - 1][y - 1] === 'O'  }">
            {{ squares[x - 1][y - 1] }}
          </button>
        </div>
      </div>
     

      <button v-show="!winner" @click="reset" class="btn btn-success mt-3 resetBtn">Reset</button>
  
      <h2 class="mt-5">History</h2>
      <div v-for="(game, idx) in history" :key="idx">
        Game {{ idx + 1 }}: {{ game }} won
      </div>
      
      
    </div>
    <audio ref="audioElement" src="src/assets/click.wav"></audio>
    <audio ref="winnerAudioElement" src="src/assets/winner.mp3"></audio>
</div>
  </template>
  
  <script>
import { computed, onMounted, ref, watch } from 'vue';

const calculateWinner = squares => {
  const lines = [
    [0, 1, 2],
    [3, 4, 5],
    [6, 7, 8],
    [0, 3, 6],
    [1, 4, 7],
    [2, 5, 8],
    [0, 4, 8],
    [2, 4, 6]
  ];
  for (let i = 0; i < lines.length; i++) {
    const [a, b, c] = lines[i];
    if (squares[a] && squares[a] === squares[b] && squares[a] === squares[c]) {
      return squares[a];
    }
  }
  return null;
};

export default {
  setup() {
    const player = ref('X');
    const squares = ref([
      ['', '', ''],
      ['', '', ''],
      ['', '', '']
    ]);
    const winner = computed(() => calculateWinner(squares.value.flat()));

    const maxGames = 3;
    const currentGame = ref(1);
    const SinglePlayerMode = ref(true);
    const audioElement = ref(null);
    const winnerAudioElement = ref(null);

    


  const handleCellClick = (x, y) => {
    // Riproduci il suono
    playClickSound();

    if (winner.value || (SinglePlayerMode.value && player.value === 'O')) return;

    squares.value[x][y] = player.value;

    if (calculateWinner(squares.value.flat())) {
      playWinnerSound();
      if (currentGame.value <= maxGames) {
        history.value.push(player.value);
        localStorage.setItem('history', JSON.stringify(history.value));
      }
      currentGame.value++;

      if (currentGame.value <= maxGames) {
        setTimeout(() => {
          reset();
        }, 1000);
      }
    } else {
      player.value = player.value === 'O' ? 'X' : 'O';
      if (SinglePlayerMode.value && player.value === 'O') {
        makeComputerMove();
      }
    }
  };

  const audioEnabled = ref(true);

  const toggleAudio = () => {
    audioEnabled.value = !audioEnabled.value;
  };
  
  const playClickSound = () => {
    if (audioEnabled.value) {
      const audio = audioElement.value;
      audio.currentTime = 0;
      audio.play();
    }
  };

  const playWinnerSound = () => {
    if (audioEnabled.value) {
      const winnerAudio = winnerAudioElement.value;
      console.log(winnerAudio);
      if (winnerAudio.paused) {
        winnerAudio.currentTime = 0;
        winnerAudio.play();
      }
    }
  };


    const makeComputerMove = () => {
    setTimeout(() => {
      if (!winner.value && SinglePlayerMode.value && player.value === 'O') {
        const emptySquares = [];
        for (let i = 0; i < 3; i++) {
          for (let j = 0; j < 3; j++) {
            if (squares.value[i][j] === '') {
              emptySquares.push({ x: i, y: j });
            }
          }
        }

        if (emptySquares.length > 0) {
          const randomIndex = Math.floor(Math.random() * emptySquares.length);
          const { x, y } = emptySquares[randomIndex];
          squares.value[x][y] = 'O';
          player.value = 'X'; // Torna al giocatore X
        }
      }
    }, 1000); // Ritardo di un secondo (1000 millisecondi)
  };

    const move = (x, y) => {
     

      if (winner.value || SinglePlayerMode.value && player.value === 'O') 
        return; // Evita che il giocatore O faccia mosse manuali
      

      squares.value[x][y] = player.value;

      if (calculateWinner(squares.value.flat())) {
        if (currentGame.value <= maxGames) {
          history.value.push(player.value);
          localStorage.setItem('history', JSON.stringify(history.value));
        }
        currentGame.value++;

        if (currentGame.value <= maxGames) {
          setTimeout(() => {
             reset(); 
          }, 1000);
          
        }
      } else {
          player.value = player.value === 'O'? 'X': 'O';
          if (SinglePlayerMode.value && player.value === 'O') {
            makeComputerMove();
          }
      }
    };

    const reset = () => {
      if (currentGame.value <= maxGames) {
        player.value = 'X';
        squares.value = [
          ['', '', ''],
          ['', '', ''],
          ['', '', '']
        ];
      }
    };

    const restart = () => {
      currentGame.value = 1;
      history.value = [];
      localStorage.removeItem('history');
      reset();
    };

    const toggleGameMode = () => {
      SinglePlayerMode.value = !SinglePlayerMode.value;
      reset();
    };


    const winningLineStyle = computed(() => {
    const winnerCells = calculateWinnerCells(squares.value.flat());
    if (winnerCells) {
      const style = {
        backgroundColor: 'yellow', // Puoi personalizzare lo stile di evidenziazione
      };
      return style;
    } else {
      return null;
    }
  });

  const isWinningCell = (x, y) => {
    const winnerCells = calculateWinnerCells(squares.value.flat());
    return winnerCells && winnerCells.includes(3 * x + y);
  };

  const calculateWinnerCells = (flatSquares) => {
    const lines = [
      [0, 1, 2],
      [3, 4, 5],
      [6, 7, 8],
      [0, 3, 6],
      [1, 4, 7],
      [2, 5, 8],
      [0, 4, 8],
      [2, 4, 6],
    ];

    for (let i = 0; i < lines.length; i++) {
      const [a, b, c] = lines[i];
      if (
        flatSquares[a] &&
        flatSquares[a] === flatSquares[b] &&
        flatSquares[a] === flatSquares[c]
      ) {
        return lines[i];
      }
    }

    return null;
  };




   







    const history = ref([]);
    watch(
      winner,
      (current, previous) => {
        if (current && !previous && currentGame.value <= maxGames){
            if (!history.value.includes(current)){
                history.value.push(current);
                localStorage.setItem('history', JSON.stringify(history.value));
            }
        }
        
      }
      
    );

    onMounted(() => {
      history.value = JSON.parse(localStorage.getItem('history')) ?? [];
    });

    return {
      winner,
      player,
      squares,
      move,
      reset,
      history,
      maxGames,
      currentGame,
      restart,
      SinglePlayerMode,
      toggleGameMode,
      makeComputerMove,
      winningLineStyle,
      isWinningCell,
      audioElement,
      handleCellClick,
      playClickSound,
      playWinnerSound,
      audioEnabled,
      toggleAudio
    };
  }
};
</script> 
