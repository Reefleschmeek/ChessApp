<script setup>
import { ref, computed, onMounted } from 'vue'

const squareClassRef = ref(Array.from(Array(8), () => Array(8).fill('')))
const cellRef = computed(() => {
  return board.cells
})
const evaluation = ref(0)

const board = {
  cells: Array.from(Array(8), () => Array(8).fill('')),
  playerToMove: 'W',
  movesSinceProgress: 0,
  enPassantVulnerability: null,
  castlePrivelegeWK: true,
  castlePrivelegeWQ: true,
  castlePrivelegeBK: true,
  castlePrivelegeBQ: true,
}
let pieceHighlight = []

function resetBoardState() {
  for (let y=0;y<8;y++) {
    for (let x=0;x<8;x++) {
      let piece = ''
      if (y == 0) {
        if (x == 0 || x == 7) piece = 'BR'
        if (x == 1 || x == 6) piece = 'BN'
        if (x == 2 || x == 5) piece = 'BB'
        if (x == 3) piece = 'BQ'
        if (x == 4) piece = 'BK'
      }
      if (y == 1) piece = 'BP'
      if (y == 6) piece = 'WP'
      if (y == 7) {
        if (x == 0 || x == 7) piece = 'WR'
        if (x == 1 || x == 6) piece = 'WN'
        if (x == 2 || x == 5) piece = 'WB'
        if (x == 3) piece = 'WQ'
        if (x == 4) piece = 'WK'
      }
      
      board.cells[x][y] = piece
    }
  }
}

function getMoves(boardState, verify) {
  const moves = []
  for (let y=0;y<8;y++) {
    for (let x=0;x<8;x++) {
      if (boardState.cells[x][y][0] == boardState.playerToMove) {
        for (let halfMove of getHalfMovesFromPiece(boardState, x, y, verify)) {
          moves.push({xFrom:x, yFrom:y, xTo:halfMove[0], yTo:halfMove[1]})
        }
      }
    }
  }
  return moves
}

function getHalfMovesFromPiece(boardState, x, y, verify) {
  const moves = []
  let piece = boardState.cells[x][y]
  let color = piece[0]
  let type = piece[1]
  if (type == 'P') {
    if (color == 'W') {
      if (!boardState.cells[x][y-1]) {
        moves.push([x, y-1])
        if (y == 6 && !boardState.cells[x][y-2]) {
          moves.push([x, y-2])
        }
      }
      if (x > 0 && boardState.cells[x-1][y-1][0] == 'B') {
        moves.push([x-1, y-1])
      }
      if (x < 7 && boardState.cells[x+1][y-1][0] == 'B') {
        moves.push([x+1, y-1])
      }
    }
    if (color == 'B') {
      if (!boardState.cells[x][y+1]) {
        moves.push([x, y+1])
        if (y == 1 && !boardState.cells[x][y+2]) {
          moves.push([x, y+2])
        }
      }
      if (x > 0 && boardState.cells[x-1][y+1][0] == 'W') {
        moves.push([x-1, y+1])
      }
      if (x < 7 && boardState.cells[x+1][y+1][0] == 'W') {
        moves.push([x+1, y+1])
      }
    }
  }

  if (type == 'R' || type == 'Q') {
    if (x > 0) {
      for (let i=x-1;i>=0;i--) {
        if (boardState.cells[i][y]) {
          if (boardState.cells[i][y][0] != color) {
            moves.push([i, y])
          }
          break
        }
        moves.push([i, y])
      }
    }
    if (x < 7) {
      for (let i=x+1;i<=7;i++) {
        if (boardState.cells[i][y]) {
          if (boardState.cells[i][y][0] != color) {
            moves.push([i, y])
          }
          break
        }
        moves.push([i, y])
      }
    }
    if (y > 0) {
      for (let i=y-1;i>=0;i--) {
        if (boardState.cells[x][i]) {
          if (boardState.cells[x][i][0] != color) {
            moves.push([x, i])
          }
          break
        }
        moves.push([x, i])
      }
    }
    if (y < 7) {
      for (let i=y+1;i<=7;i++) {
        if (boardState.cells[x][i]) {
          if (boardState.cells[x][i][0] != color) {
            moves.push([x, i])
          }
          break
        }
        moves.push([x, i])
      }
    }
  }

  if (type == 'B' || type == 'Q') {
    for (let xIncrement of [-1, 1]) {
      for (let yIncrement of [-1, 1]) {
        let i = x + xIncrement
        let j = y + yIncrement
        while (i >= 0 && i <= 7 && j >= 0 && j <= 7) {
          if (boardState.cells[i][j]) {
            if (boardState.cells[i][j][0] != color) {
              moves.push([i, j])
            }
            break
          }
          moves.push([i, j])
          i += xIncrement
          j += yIncrement
        }
      }
    }
  }
  
  if (type == 'N') {
    const steps = [[2, 1], [2, -1], [1, 2], [1, -2], [-2, 1], [-2, -1], [-1, 2], [-1, -2]]
    for (let step of steps) {
      let i = x + step[0]
      let j = y + step[1]
      if (i >= 0 && i <= 7 && j >= 0 && j <= 7 && boardState.cells[i][j][0] != color) {
        moves.push([i, j])
      }
    }
  }

  if (type == 'K') {
    const steps = [[1, 1], [1, 0], [1, -1], [0, -1], [-1, -1], [-1, 0], [-1, 1], [0, 1]]
    for (let step of steps) {
      let i = x + step[0]
      let j = y + step[1]
      if (i >= 0 && i <= 7 && j >= 0 && j <= 7 && boardState.cells[i][j][0] != color) {
        moves.push([i, j])
      }
    }
  }

  if (!verify)
    return moves
  
  const verifiedHalfMoves = []
  for (let halfMove of moves) {
    const potentialBoardState = structuredClone(boardState)
    const move = {xFrom:x, yFrom:y, xTo:halfMove[0], yTo:halfMove[1]}
    makeMove(potentialBoardState, move)
    if (isValidConfiguration(potentialBoardState)) {
      verifiedHalfMoves.push(halfMove)
    }
  }

  return verifiedHalfMoves
}

function isValidConfiguration(boardState) {
  const moves = getMoves(boardState, false)
  for (const move of moves) {
    if (boardState.cells[move.xTo][move.yTo][1] == 'K')
      return false
  }
  return true
}

function makeMove(boardState, move) {
  if (boardState.cells[move.xFrom][move.yFrom][1] == 'P' || boardState.cells[move.xTo][move.yTo]) {
    boardState.movesSinceProgress = 0
  } else  {
    boardState.movesSinceProgress += 1
  }
  boardState.cells[move.xTo][move.yTo] = boardState.cells[move.xFrom][move.yFrom]
  boardState.cells[move.xFrom][move.yFrom] = ''
  boardState.playerToMove = (boardState.playerToMove == 'W') ? 'B' : 'W'

}

function evaluate(boardState) {
  const values = {'P':100, 'N':310, 'B':330, 'R':500, 'Q':900, 'K':20000}
  let evaluation = 0
  for (let y=0;y<8;y++) {
    for (let x=0;x<8;x++) {
      if (boardState.cells[x][y]) {
        let piece = boardState.cells[x][y]
        let color = piece[0]
        let type = piece[1]
        evaluation += values[type] * ((color == 'W') ? 1 : -1)
      }
    }
  }
  return evaluation
}

function resetSquareClassRef() {
  for (let y=0;y<8;y++) {
    for (let x=0;x<8;x++) {
      squareClassRef.value[x][y] = (x+y)%2 ? 'EvenSquare' : 'OddSquare'
    }
  }
}

function clickCell(event) {
  let index = parseInt(event.target.id)
  let x = index % 8
  let y = Math.floor(index/8)
  if (squareClassRef.value[x][y] == 'HighlightSquare') {
    resetSquareClassRef()
    return
  }
  if (squareClassRef.value[x][y] == 'ThreatSquare') {
    resetSquareClassRef()
    const move = {xFrom:pieceHighlight[0], yFrom:pieceHighlight[1], xTo:x, yTo:y}
    makeMove(board, move)
    squareClassRef.value[move.xFrom][move.yFrom] = 'HighlightSquare'
    squareClassRef.value[move.xTo][move.yTo] = 'HighlightSquare'
    evaluation.value = evaluate(board)
    return
  }
  resetSquareClassRef()
  let piece = board.cells[x][y]
  if (piece[0] == board.playerToMove) {
    squareClassRef.value[x][y] = 'HighlightSquare'
    pieceHighlight = [x, y]
    const moves = getHalfMovesFromPiece(board, x, y, true)
    for (let move of moves) {
      squareClassRef.value[move[0]][move[1]] = 'ThreatSquare'
    }
  }
}

onMounted(() => {
  resetBoardState()
  resetSquareClassRef()
})

</script>

<template>
  <div class='Board'>
    <div v-for='index in 64' :class='squareClassRef[(index-1)%8][Math.floor((index-1)/8)]' :key='index-1' :id='index-1' @click='clickCell'>{{ cellRef[(index-1)%8][Math.floor((index-1)/8)] }}</div>
  </div>
  <p>{{ board.playerToMove=='W' ? 'White' : 'Black' }} to move. Eval: {{ evaluation }}</p>
</template>

<style>

.Board {
  display: grid;
  grid-template-columns: repeat(8, 64px);
  grid-template-rows: repeat(8, 64px);
  gap: 2px;
  width: 526px;
  background-color: black;
  padding: 2px;
}

.EvenSquare {
  width: 64px;
  height: 64px;
  background-color:burlywood;
  color: black;
}

.OddSquare {
  width: 64px;
  height: 64px;
  background-color: blanchedalmond;
  color: black;
}

.HighlightSquare {
  width: 64px;
  height: 64px;
  background-color: rgb(255, 255, 123);
  color: black;
}

.ThreatSquare {
  width: 64px;
  height: 64px;
  background-color: rgb(255, 122, 122);
  color: black;
}

</style>