<script setup>
import { ref, computed, onMounted } from 'vue'

const squareClassRef = ref(Array.from(Array(8), () => Array(8).fill('')))
const cellRef = computed(() => {
  return board.cells
})
const evaluation = ref(0)
const message = ref('')
const whiteAI = ref(false)
const blackAI = ref(true)

const pieceImages = {
  'WP': new URL('./assets/pieceWP.png', import.meta.url),
  'WN': new URL('./assets/pieceWN.png', import.meta.url),
  'WB': new URL('./assets/pieceWB.png', import.meta.url),
  'WR': new URL('./assets/pieceWR.png', import.meta.url),
  'WQ': new URL('./assets/pieceWQ.png', import.meta.url),
  'WK': new URL('./assets/pieceWK.png', import.meta.url),
  'BP': new URL('./assets/pieceBP.png', import.meta.url),
  'BN': new URL('./assets/pieceBN.png', import.meta.url),
  'BB': new URL('./assets/pieceBB.png', import.meta.url),
  'BR': new URL('./assets/pieceBR.png', import.meta.url),
  'BQ': new URL('./assets/pieceBQ.png', import.meta.url),
  'BK': new URL('./assets/pieceBK.png', import.meta.url),
}
const pawnTable = [
  [0, 50, 10, 5, 0, 5, 5, 0],
  [0, 50, 10, 5, 0, -5, 10, 0],
  [0, 50, 20, 10, 0, -10, 10, 0],
  [0, 50, 30, 25, 20, 0, -20, 0],
  [0, 50, 30, 25, 20, 0, -20, 0],
  [0, 50, 20, 10, 0, -10, 10, 0],
  [0, 50, 10, 5, 0, -5, 10, 0],
  [0, 50, 10, 5, 0, 5, 5, 0],
]
const knightTable = [
  [-50, -40, -30, -30, -30, -30, -40, -50],
  [-40, -20, 0, 5, 0, 5, -20, -40],
  [-30, 0, 10, 15, 15, 10, 0, -30],
  [-30, 0, 15, 20, 20, 15, 5, -30],
  [-30, 0, 15, 20, 20, 15, 5, -30],
  [-30, 0, 10, 15, 15, 10, 0, -30],
  [-40, -20, 0, 5, 0, 5, -20, -40],
  [-50, -40, -30, -30, -30, -30, -40, -50],
]
const bishopTable = [
  [-20, -10, -10, -10, -10, -10, -10, -20],
  [-10, 0, 0, 5, 0, 10, 5, -10],
  [-10, 0, 5, 5, 10, 10, 0, -10],
  [-10, 0, 10, 10, 10, 10, 0, -10],
  [-10, 0, 10, 10, 10, 10, 0, -10],
  [-10, 0, 5, 5, 10, 10, 0, -10],
  [-10, 0, 0, 5, 0, 10, 5, -10],
  [-20, -10, -10, -10, -10, -10, -10, -20],
]
const rookTable = [
  [0, 5, -5, -5, -5, -5, -5, 0],
  [0, 10, 0, 0, 0, 0, 0, 0],
  [0, 10, 0, 0, 0, 0, 0, 0],
  [0, 10, 0, 0, 0, 0, 0, 5],
  [0, 10, 0, 0, 0, 0, 0, 5],
  [0, 10, 0, 0, 0, 0, 0, 0],
  [0, 10, 0, 0, 0, 0, 0, 0],
  [0, 5, -5, -5, -5, -5, -5, 0],
]
const queenTable = [
  [-20, -10, -10, -5, 0, -10, -10, -20],
  [-10, 0, 0, 0, 0, 5, 0, -10],
  [-10, 0, 5, 5, 5, 5, 5, -10],
  [-5, 0, 5, 5, 5, 5, 0, -5],
  [-5, 0, 5, 5, 5, 5, 0, -5],
  [-10, 0, 5, 5, 5, 5, 0, -10],
  [-10, 0, 0, 0, 0, 0, 0, -10],
  [-20, -10, -10, -5, -5, -10, -10, -20],
]
const kingTable = [
  [-30, -30, -30, -30, -20, -10, 20, 20],
  [-40, -40, -40, -40, -30, -20, 20, 30],
  [-40, -40, -40, -40, -30, -20, 0, 10],
  [-50, -50, -50, -50, -40, -20, 0, 0],
  [-50, -50, -50, -50, -40, -20, 0, 0],
  [-40, -40, -40, -40, -30, -20, 0, 10],
  [-40, -40, -40, -40, -30, -20, 20, 30],
  [-30, -30, -30, -30, -20, -10, 20, 20],
]
const pieceTables = {
  'P':pawnTable,
  'N':knightTable,
  'B':bishopTable,
  'R':rookTable,
  'Q':queenTable,
  'K':kingTable
}
const board = {
  cells: Array.from(Array(8), () => Array(8).fill('')),
  moves: [],
  playerToMove: 'W',
  movesSinceProgress: 0,
  enPassantVulnerability: null,
  castlePrivelegeWK: true,
  castlePrivelegeWQ: true,
  castlePrivelegeBK: true,
  castlePrivelegeBQ: true,
}
let pieceHighlight = null

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
  updateMoves(board)
}

function updateMoves(boardState) {
  boardState.moves = getMoves(boardState)
}

function getMoves(boardState) {
  const moves = []
  if (boardState.movesSinceProgress >= 100)
    return moves
  for (let y=0;y<8;y++) {
    for (let x=0;x<8;x++) {
      if (boardState.cells[x][y][0] == boardState.playerToMove) {
        for (let halfMove of getHalfMovesFromPiece(boardState, x, y)) {
          moves.push({xFrom:x, yFrom:y, xTo:halfMove.xTo, yTo:halfMove.yTo, flags:halfMove.flags})
        }
      }
    }
  }
  return moves
}

function getHalfMovesFromPiece(boardState, x, y) {

  //Flag dictionary:
  //X_ = Capture of _ piece, DP = Double pawn move, EC = En passant capture, E_ = En passant vulnerability on file _ expired, P_ = Pawn promotion to _ piece,
  //CK = King side castle, CQ = Queen side castle, LK = Lose king side castle privelege, LQ = Lose queen side castle privelege
  //Some are applied when verifying the moves

  const halfMoves = []
  let color = boardState.cells[x][y][0]
  let type = boardState.cells[x][y][1]

  if (type == 'P') { //Pawn moves
    if (color == 'W') { //White pawns
      if (!boardState.cells[x][y-1]) { //Single move
        const flags = []
        if (y == 1)
          flags.push('PQ')
        halfMoves.push({xTo:x, yTo:y-1, flags:flags})
        if (y == 6 && !boardState.cells[x][4]) { //Double move
          halfMoves.push({xTo:x, yTo:4, flags:['DP']})
        }
      }
      if (x > 0 && boardState.cells[x-1][y-1][0] == 'B') { //Capture left
        const flags = []
        if (y == 1)
          flags.push('PQ')
        halfMoves.push({xTo:x-1, yTo:y-1, flags:flags})
      }
      if (x < 7 && boardState.cells[x+1][y-1][0] == 'B') { //Capture right
        const flags = []
        if (y == 1)
          flags.push('PQ')
        halfMoves.push({xTo:x+1, yTo:y-1, flags:flags})
      }
      if (boardState.enPassantVulnerability !== null) { //Capture en passant
        if (y == 3 && (x == boardState.enPassantVulnerability-1 || x == boardState.enPassantVulnerability+1)){
          halfMoves.push({xTo:boardState.enPassantVulnerability, yTo:2, flags:['EX']})
        }
      }
    } else { //Black pawns
      if (!boardState.cells[x][y+1]) { //Single move
        const flags = []
        if (y==6)
          flags.push('PQ')
        halfMoves.push({xTo:x, yTo:y+1, flags:flags})
        if (y == 1 && !boardState.cells[x][3]) { //Double move
          halfMoves.push({xTo:x, yTo:3, flags:['DP']})
        }
      }
      if (x > 0 && boardState.cells[x-1][y+1][0] == 'W') { //Capture left
        halfMoves.push({xTo:x-1, yTo:y+1, flags:[]})
      }
      if (x < 7 && boardState.cells[x+1][y+1][0] == 'W') { //Capture right
        halfMoves.push({xTo:x+1, yTo:y+1, flags:[]})
      }
      if (boardState.enPassantVulnerability != null) { //Capture en passant
        if (y == 4 && (x == boardState.enPassantVulnerability-1 || x == boardState.enPassantVulnerability+1)){
          halfMoves.push({xTo:boardState.enPassantVulnerability, yTo:5, flags:['EX']})
        }
      }
    }
  }

  if (type == 'R' || type == 'Q') { //Rook+Queen moves
    const flags = []
    if (type == 'R') { //Castle privelege flags for moving rook
      if (boardState.castlePrivelegeWK && x == 7 && color == 'W')
        flags.push('LK')
      if (boardState.castlePrivelegeWQ && x == 0 && color == 'W')
        flags.push('LQ')
      if (boardState.castlePrivelegeBK && x == 7 && color == 'B')
        flags.push('LK')
      if (boardState.castlePrivelegeBQ && x == 0 && color == 'B')
        flags.push('LQ')
    }
    if (x > 0) {
      for (let i=x-1;i>=0;i--) {
        if (boardState.cells[i][y]) {
          if (boardState.cells[i][y][0] != color) {
            halfMoves.push({xTo:i, yTo:y, flags:flags})
          }
          break
        }
        halfMoves.push({xTo:i, yTo:y, flags:flags})
      }
    }
    if (x < 7) {
      for (let i=x+1;i<=7;i++) {
        if (boardState.cells[i][y]) {
          if (boardState.cells[i][y][0] != color) {
            halfMoves.push({xTo:i, yTo:y, flags:flags})
          }
          break
        }
        halfMoves.push({xTo:i, yTo:y, flags:flags})
      }
    }
    if (y > 0) {
      for (let i=y-1;i>=0;i--) {
        if (boardState.cells[x][i]) {
          if (boardState.cells[x][i][0] != color) {
            halfMoves.push({xTo:x, yTo:i, flags:flags})
          }
          break
        }
        halfMoves.push({xTo:x, yTo:i, flags:flags})
      }
    }
    if (y < 7) {
      for (let i=y+1;i<=7;i++) {
        if (boardState.cells[x][i]) {
          if (boardState.cells[x][i][0] != color) {
            halfMoves.push({xTo:x, yTo:i, flags:flags})
          }
          break
        }
        halfMoves.push({xTo:x, yTo:i, flags:flags})
      }
    }
  }

  if (type == 'B' || type == 'Q') { //Bishop+Queen moves
    for (let xIncrement of [-1, 1]) {
      for (let yIncrement of [-1, 1]) {
        let i = x + xIncrement
        let j = y + yIncrement
        while (i >= 0 && i <= 7 && j >= 0 && j <= 7) {
          if (boardState.cells[i][j]) {
            if (boardState.cells[i][j][0] != color) {
              halfMoves.push({xTo:i, yTo:j, flags:[]})
            }
            break
          }
          halfMoves.push({xTo:i, yTo:j, flags:[]})
          i += xIncrement
          j += yIncrement
        }
      }
    }
  }
  
  if (type == 'N') { //Knight moves
    const steps = [[2, 1], [2, -1], [1, 2], [1, -2], [-2, 1], [-2, -1], [-1, 2], [-1, -2]]
    for (let step of steps) {
      let i = x + step[0]
      let j = y + step[1]
      if (i >= 0 && i <= 7 && j >= 0 && j <= 7 && boardState.cells[i][j][0] != color) {
        halfMoves.push({xTo:i, yTo:j, flags:[]})
      }
    }
  }

  if (type == 'K') { //King moves
    const flags = []
    if (color == 'W') { //White king castling
      if (boardState.castlePrivelegeWK)
        flags.push('LK')
      if (boardState.castlePrivelegeWQ)
        flags.push('LQ')
      if (boardState.castlePrivelegeWK && !isKingInCheck(boardState, 'W')) { //King side
        if (!boardState.cells[5][7] && !boardState.cells[6][7]) {
          const intermediateBoardState = structuredClone(boardState)
          intermediateBoardState.cells[4][7] = ''
          intermediateBoardState.cells[5][7] = 'WK'
          if (!isKingInCheck(intermediateBoardState, 'W')) {
            halfMoves.push({xTo:6, yTo:7, flags:flags.concat(['CK'])})
          }
        }
      }
      if (boardState.castlePrivelegeWQ && !isKingInCheck(boardState, 'W')) { //Queen side
        if (!boardState.cells[1][7] && !boardState.cells[2][7] && !boardState.cells[3][7]) {
          const intermediateBoardState = structuredClone(boardState)
          intermediateBoardState.cells[4][7] = ''
          intermediateBoardState.cells[3][7] = 'WK'
          if (!isKingInCheck(intermediateBoardState, 'W')) {
            halfMoves.push({xTo:2, yTo:7, flags:flags.concat(['CQ'])})
          }
        }
      }
    } else { //Black king castling
      if (boardState.castlePrivelegeBK)
        flags.push('LK')
      if (boardState.castlePrivelegeBQ)
        flags.push('LQ')
      if (boardState.castlePrivelegeBK && !isKingInCheck(boardState, 'B')) { //King side
        if (!boardState.cells[5][0] && !boardState.cells[6][0]) {
          const intermediateBoardState = structuredClone(boardState)
          intermediateBoardState.cells[4][0] = ''
          intermediateBoardState.cells[5][0] = 'BK'
          if (!isKingInCheck(intermediateBoardState, 'B')) {
            halfMoves.push({xTo:6, yTo:0, flags:flags.concat(['CK'])})
          }
        }
      }
      if (boardState.castlePrivelegeBQ && !isKingInCheck(boardState, 'B')) { //Queen side
        if (!boardState.cells[1][0] && !boardState.cells[2][0] && !boardState.cells[3][0]) {
          const intermediateBoardState = structuredClone(boardState)
          intermediateBoardState.cells[4][0] = ''
          intermediateBoardState.cells[3][0] = 'BK'
          if (!isKingInCheck(intermediateBoardState, 'B')) {
            halfMoves.push({xTo:2, yTo:0, flags:flags.concat(['CQ'])})
          }
        }
      }
    }
    //Regular moves
    const steps = [[1, 1], [1, 0], [1, -1], [0, -1], [-1, -1], [-1, 0], [-1, 1], [0, 1]]
    for (let step of steps) {
      let i = x + step[0]
      let j = y + step[1]
      if (i >= 0 && i <= 7 && j >= 0 && j <= 7 && boardState.cells[i][j][0] != color) {
        halfMoves.push({xTo:i, yTo:j, flags:flags})
      }
    }
  }
  
  //Verify potential moves, discarding those that place the moving player's king under threat
  //Apply capture flags
  const verifiedHalfMoves = []
  for (let halfMove of halfMoves) {
    if (boardState.cells[halfMove.xTo][halfMove.yTo]) {
      halfMove.flags.push('X' + boardState.cells[halfMove.xTo][halfMove.yTo][1])
    }
    const potentialBoardState = structuredClone(boardState)
    const move = {xFrom:x, yFrom:y, xTo:halfMove.xTo, yTo:halfMove.yTo, flags:halfMove.flags}
    makeMove(potentialBoardState, move)
    if (!isKingInCheck(potentialBoardState, color)) {
      verifiedHalfMoves.push(halfMove)
    }
  }

  return verifiedHalfMoves
}

function isKingInCheck(boardState, color) {
  let oppositeColor = getOppositeColor(color)
  let kingX
  let kingY
  for (let y=0;y<8;y++) {
    for (let x=0;x<8;x++) {
      if (boardState.cells[x][y] == color+'K') {
        kingX = x
        kingY = y
        break
      }
    }
  }
  if (color == 'W' && kingY > 1) {
    if ((kingX > 0 && boardState.cells[kingX-1][kingY-1] == 'BP') || (kingX < 7 && boardState.cells[kingX+1][kingY-1] == 'BP'))
      return true
  }
  if (color == 'B' && kingY < 6) {
    if ((kingX > 0 && boardState.cells[kingX-1][kingY+1] == 'WP') || (kingX < 7 && boardState.cells[kingX+1][kingY+1] == 'WP'))
      return true
  }
  let knightSteps = [[2, 1], [2, -1], [1, 2], [1, -2], [-2, 1], [-2, -1], [-1, 2], [-1, -2]]
  let bishopSteps = [[1, 1], [1, -1], [-1, 1], [-1, -1]]
  let rookSteps = [[1, 0], [-1, 0], [0, 1], [0, -1]]
  let kingSteps = [[1, 1], [1, 0], [1, -1], [0, -1], [-1, -1], [-1, 0], [-1, 1], [0, 1]]
  for (let step of knightSteps) {
    let x = kingX + step[0]
    let y = kingY + step[1]
    if (x < 0 || x > 7 || y < 0 || y > 7)
      continue
    if (boardState.cells[x][y][1] == 'N' && boardState.cells[x][y][0] == oppositeColor)
      return true
  }
  for (let step of bishopSteps) {
    let x = kingX + step[0]
    let y = kingY + step[1]
    while (x >= 0 && x <= 7 && y >= 0 && y <= 7) {
      if (boardState.cells[x][y]) {
        if ((boardState.cells[x][y][1] == 'B' || boardState.cells[x][y][1] == 'Q') && boardState.cells[x][y][0] == oppositeColor)
          return true
        break
      }
      x += step[0]
      y += step[1]
    }
  }
  for (let step of rookSteps) {
    let x = kingX + step[0]
    let y = kingY + step[1]
    while (x >= 0 && x <= 7 && y >= 0 && y <= 7) {
      if (boardState.cells[x][y]) {
        if ((boardState.cells[x][y][1] == 'R' || boardState.cells[x][y][1] == 'Q') && boardState.cells[x][y][0] == oppositeColor)
          return true
        break
      }
      x += step[0]
      y += step[1]
    }
  }
  for (let step of kingSteps) {
    let x = kingX + step[0]
    let y = kingY + step[1]
    if (x < 0 || x > 7 || y < 0 || y > 7)
      continue
    if (boardState.cells[x][y][1] == 'K' && boardState.cells[x][y][0] == oppositeColor)
      return true
  }
  return false
}

function makeMove(boardState, move) {
  //Keep track of moves for the 50 move rule
  if (boardState.cells[move.xFrom][move.yFrom][1] == 'P' || boardState.cells[move.xTo][move.yTo]) {
    boardState.movesSinceProgress = 0
  } else  {
    boardState.movesSinceProgress += 1
  }
  //Keep track of en passant vulnerabilities
  if (move.yFrom == 6 && move.yTo == 4 && boardState.cells[move.xFrom][move.yFrom] == 'WP') {
    boardState.enPassantVulnerability = move.xFrom
  } else if (move.yFrom == 1 && move.yTo == 3 && boardState.cells[move.xFrom][move.yFrom] == 'BP') {
    boardState.enPassantVulnerability = move.xFrom
  } else{
    boardState.enPassantVulnerability = null
  }
  //Keep track of castling priveleges
  if ((move.xFrom==0 && move.yFrom==0) || (move.xTo==0 && move.yTo==0)) {
    boardState.castlePrivelegeBQ = false
  }
  if ((move.xFrom==7 && move.yFrom==0) || (move.xTo==7 && move.yTo==0)) {
    boardState.castlePrivelegeBK = false
  }
  if ((move.xFrom==0 && move.yFrom==7) || (move.xTo==0 && move.yTo==7)) {
    boardState.castlePrivelegeWQ = false
  }
  if ((move.xFrom==7 && move.yFrom==7) || (move.xTo==7 && move.yTo==7)) {
    boardState.castlePrivelegeWK = false
  }
  //Resolve en passant moves
  if (boardState.cells[move.xFrom][move.yFrom] == 'WP' && !boardState.cells[move.xTo][move.yTo] && move.xFrom != move.xTo) {
    boardState.cells[move.xTo][3] = ''
  }
  if (boardState.cells[move.xFrom][move.yFrom] == 'BP' && !boardState.cells[move.xTo][move.yTo] && move.xFrom != move.xTo) {
    boardState.cells[move.xTo][4] = ''
  }
  //Resolve castling moves
  if (boardState.cells[move.xFrom][move.yFrom] == 'WK') {
    boardState.castlePrivelegeWK = false
    boardState.castlePrivelegeWQ = false
    if (move.xFrom == 4 && move.xTo == 2) {
      boardState.cells[0][7] = ''
      boardState.cells[3][7] = 'WR'
    }
    if (move.xFrom == 4 && move.xTo == 6) {
      boardState.cells[7][7] = ''
      boardState.cells[5][7] = 'WR'
    }
  }
  if (boardState.cells[move.xFrom][move.yFrom] == 'BK') {
    boardState.castlePrivelegeBK = false
    boardState.castlePrivelegeBQ = false
    if (move.xFrom == 4 && move.xTo == 2) {
      boardState.cells[0][0] = ''
      boardState.cells[3][0] = 'BR'
    }
    if (move.xFrom == 4 && move.xTo == 6) {
      boardState.cells[7][0] = ''
      boardState.cells[5][0] = 'BR'
    }
  }
  //Move the piece
  boardState.cells[move.xTo][move.yTo] = boardState.cells[move.xFrom][move.yFrom]
  boardState.cells[move.xFrom][move.yFrom] = ''
  //Resolve pawn promotions
  if (move.yTo == 0 && boardState.cells[move.xTo][move.yTo] == 'WP') {
    boardState.cells[move.xTo][move.yTo] = 'WQ'
  }
  if (move.yTo == 7 && boardState.cells[move.xTo][move.yTo] == 'BP') {
    boardState.cells[move.xTo][move.yTo] = 'BQ'
  }
  //Change who moves next
  boardState.playerToMove = getOppositeColor(boardState.playerToMove)
}

function getOppositeColor(color) {
  return (color == 'W') ? 'B' : 'W'
}

function findBestMove(boardState, depth) {
  boardState.moves.sort(() => (Math.random()-0.5))
  let bestMove = board.moves[0]
  if (boardState.playerToMove == 'W') {
    let bestEval = -1000
    for (let newMove of boardState.moves) {
      const potentialBoardState = structuredClone(board)
      makeMove(potentialBoardState, newMove)
      updateMoves(potentialBoardState)
      let newEval = deepEvaluate(potentialBoardState, depth)
      if (newEval > bestEval) {
        bestEval = newEval
        bestMove = newMove
      }
    }
    evaluation.value = bestEval
  }
  if (boardState.playerToMove == 'B') {
    let bestEval = 1000
    for (let newMove of boardState.moves) {
      const potentialBoardState = structuredClone(boardState)
      makeMove(potentialBoardState, newMove)
      updateMoves(potentialBoardState)
      let newEval = deepEvaluate(potentialBoardState, depth)
      if (newEval < bestEval) {
        bestEval = newEval
        bestMove = newMove
      }
    }
    evaluation.value = bestEval
  }
  return bestMove
}

function deepEvaluate(boardState, depth) {
  if (depth == 0 || !boardState.moves.length) {
    return evaluate(boardState)
  }
  if (boardState.playerToMove == 'W') {
    let bestEval = -1000
    for (let move of boardState.moves) {
      const potentialBoardState = structuredClone(boardState)
      makeMove(potentialBoardState, move)
      updateMoves(potentialBoardState)
      let newEval = deepEvaluate(potentialBoardState, depth-1)
      if (newEval > bestEval) {
        bestEval = newEval
      }
    }
    return bestEval
  }
  if (boardState.playerToMove == 'B') {
    let bestEval = 1000
    for (let move of boardState.moves) {
      const potentialBoardState = structuredClone(boardState)
      makeMove(potentialBoardState, move)
      updateMoves(potentialBoardState)
      let newEval = deepEvaluate(potentialBoardState, depth-1)
      if (newEval < bestEval) {
        bestEval = newEval
      }
    }
    return bestEval
  }
}

function evaluate(boardState) {
  if (!boardState.moves.length) {
    if (boardState.playerToMove == 'W' && isKingInCheck(boardState, 'W'))
      return -100
    if (boardState.playerToMove == 'B' && isKingInCheck(boardState, 'B'))
      return 100
    return 0
  }
  const values = {'P':100, 'N':320, 'B':330, 'R':500, 'Q':900, 'K':20000}
  let evaluation = 0
  for (let y=0;y<8;y++) {
    for (let x=0;x<8;x++) {
      if (boardState.cells[x][y]) {
        let piece = boardState.cells[x][y]
        let color = piece[0]
        let type = piece[1]
        if (color == 'W') {
          evaluation += values[type] + pieceTables[type][x][y]
        } else {
          evaluation -= values[type] + pieceTables[type][x][7-y]
        }
        
      }
    }
  }
  return evaluation / 100
}

function resetSquareClassRef() {
  for (let y=0;y<8;y++) {
    for (let x=0;x<8;x++) {
      squareClassRef.value[x][y] = (x+y)%2 ? 'EvenSquare' : 'OddSquare'
    }
  }
}

function clickCell(event, cellNum) {

  //Get grid coords of clicked cell
  let x = cellNum% 8
  let y = Math.floor(cellNum/8)
  
  //Remove highlight if a piece is clicked a second time
  if (pieceHighlight && x == pieceHighlight[0] && y == pieceHighlight[1]) {
    pieceHighlight = null
    resetSquareClassRef()
    return
  }
  
  //Make move if valid move clicked, else update highlighting
  if (squareClassRef.value[x][y] == 'ThreatSquare') {
    const move = {xFrom:pieceHighlight[0], yFrom:pieceHighlight[1], xTo:x, yTo:y, flags:[]}
    makeRealMove(move)
  } else {
    resetSquareClassRef()
    let piece = board.cells[x][y]
    if (piece[0] == board.playerToMove) {
      squareClassRef.value[x][y] = 'HighlightSquare'
      pieceHighlight = [x, y]
      const moves = getHalfMovesFromPiece(board, x, y)
      for (let move of moves) {
        squareClassRef.value[move.xTo][move.yTo] = 'ThreatSquare'
      }
    }
  }
}

function makeRealMove(move) {

  //Apply move to board
  makeMove(board, move)
  updateMoves(board)
  resetSquareClassRef()
  pieceHighlight = null
  squareClassRef.value[move.xFrom][move.yFrom] = 'HighlightSquare'
  squareClassRef.value[move.xTo][move.yTo] = 'HighlightSquare'

  //Check for game end conditions
  if (!board.moves.length) {
    if (isKingInCheck(board, board.playerToMove)) {
    message.value = 'Checkmate! '+((board.playerToMove == 'W') ? 'Black' : 'White')+' wins!'
    } else {
      message.value = 'Stalemate! Game ends in a draw.'
    }
    return
  }

  //Let AI take turn
  if ((board.playerToMove == 'B' && blackAI.value) || (board.playerToMove == 'W' && whiteAI.value)) {
    setTimeout(makeAIMove, 25)
  }
}

function makeAIMove() {
  let time1 = Date.now()
  const bestMove = findBestMove(board, 2)
  let time2 = Date.now()
  let time = (time2 - time1) / 1000
  console.log('Move search time: '+time+' seconds.')
  makeRealMove(bestMove)
}

onMounted(() => {
  resetBoardState()
  resetSquareClassRef()
  if (whiteAI.value) {
    makeAIMove()
  }
})

</script>

<template>
  <div class='Board' @click='resetSquareClassRef'>
    <div v-for='index in 64' :class='squareClassRef[(index-1)%8][Math.floor((index-1)/8)]' :key='index-1' @click.stop='(e) => clickCell(e, index-1)'><img :src='pieceImages[board.cells[(index-1)%8][Math.floor((index-1)/8)]]'></div>
  </div>
  <p>
    {{ board.playerToMove=='W' ? 'White' : 'Black' }} to move. Eval: {{ (evaluation >= 0 ? '+' : '')+evaluation }}<br>
    {{ message }}
  </p>
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