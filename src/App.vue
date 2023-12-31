<script setup>
import { ref, computed, onMounted } from 'vue'

const squareClassRef = ref(Array.from(Array(8), () => Array(8).fill('')))
const cellRef = ref(Array.from(Array(8), () => Array(8).fill('')))
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
const pieceValues = {'P':100, 'N':320, 'B':330, 'R':500, 'Q':900, 'K':20000}
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
  playerToMove: 'W',
  cells: Array.from(Array(8), () => Array(8).fill('')),
  moveDrawStack: [0],
  enPassantStack: [null],
  castleWKStack: [true],
  castleWQStack: [true],
  castleBKStack: [true],
  castleBQStack: [true],
  moves: [],
  captures: [],
}
let pieceHighlight = null
let highlightedMoves = {}
let realMoves = []
let evaluations = 0
let searchDepth = 4
let searchDepthNarrow = 6

function last(arr) {
  return arr[arr.length - 1]
}

function getOppositeColor(color) {
  return (color == 'W') ? 'B' : 'W'
}

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
/*
  board.cells[5][0] = 'BK'
  board.cells[4][1] = 'BP'
  board.cells[5][1] = 'BP'
  board.cells[6][1] = 'BP'
  board.cells[7][1] = 'BP'
  board.cells[4][7] = 'WK'
  board.cells[1][4] = 'WR'
  */
  updateMoves(board)
}

function updateMoves() {
  let moves = []
  if (last(board.moveDrawStack) >= 100) {
    board.moves = moves
    return
  }
  for (let y=0;y<8;y++) {
    for (let x=0;x<8;x++) {
      if (board.cells[x][y][0] == board.playerToMove) {
        for (let move of getMovesFromPiece(x, y)) {
          moves.push(move)
        }
      }
    }
  }
  board.moves = moves
}

function updateCaptures() {
  let captures = []
  if (last(board.moveDrawStack) >= 100) {
    board.captures = captures
    return
  }
  for (let y=0;y<8;y++) {
    for (let x=0;x<8;x++) {
      if (board.cells[x][y][0] == board.playerToMove) {
        for (let capture of getCapturesFromPiece(x, y)) {
          captures.push(capture)
        }
      }
    }
  }
  board.captures = captures
}

function getMovesFromPiece(x, y) {

  //Flag dictionary:
  //X_ = Capture of _ piece, DP = Double pawn move, EP = En passant capture, P_ = Pawn promotion to _ piece, CK = King side castle, CQ = Queen side castle
  //Capture flags applied when verifying moves

  const halfMoves = []
  let color = board.cells[x][y][0]
  let type = board.cells[x][y][1]

  if (type == 'P') { //Pawn moves
    if (color == 'W') { //White pawns
      if (!board.cells[x][y-1]) { //Single move
        const flags = []
        if (y == 1)
          flags.push('PQ')
        halfMoves.push({xTo:x, yTo:y-1, flags:flags})
        if (y == 6 && !board.cells[x][4]) { //Double move
          halfMoves.push({xTo:x, yTo:4, flags:['DP']})
        }
      }
      if (x > 0 && board.cells[x-1][y-1][0] == 'B') { //Capture left
        const flags = []
        if (y == 1)
          flags.push('PQ')
        halfMoves.push({xTo:x-1, yTo:y-1, flags:flags})
      }
      if (x < 7 && board.cells[x+1][y-1][0] == 'B') { //Capture right
        const flags = []
        if (y == 1)
          flags.push('PQ')
        halfMoves.push({xTo:x+1, yTo:y-1, flags:flags})
      }
      if (last(board.enPassantStack) !== null) { //Capture en passant
        let enPassantFile = last(board.enPassantStack)
        if (y == 3 && (x == enPassantFile-1 || x == enPassantFile+1)){
          halfMoves.push({xTo:enPassantFile, yTo:2, flags:['EP']})
        }
      }
    } else { //Black pawns
      if (!board.cells[x][y+1]) { //Single move
        const flags = []
        if (y==6)
          flags.push('PQ')
        halfMoves.push({xTo:x, yTo:y+1, flags:flags})
        if (y == 1 && !board.cells[x][3]) { //Double move
          halfMoves.push({xTo:x, yTo:3, flags:['DP']})
        }
      }
      if (x > 0 && board.cells[x-1][y+1][0] == 'W') { //Capture left
        const flags = []
        if (y==6)
          flags.push('PQ')
        halfMoves.push({xTo:x-1, yTo:y+1, flags:flags})
      }
      if (x < 7 && board.cells[x+1][y+1][0] == 'W') { //Capture right
        const flags = []
        if (y==6)
          flags.push('PQ')
        halfMoves.push({xTo:x+1, yTo:y+1, flags:flags})
      }
      if (last(board.enPassantStack) != null) { //Capture en passant
        let enPassantFile = last(board.enPassantStack)
        if (y == 4 && (x == enPassantFile-1 || x == enPassantFile+1)){
          halfMoves.push({xTo:enPassantFile, yTo:5, flags:['EP']})
        }
      }
    }
  }

  if (type == 'R' || type == 'Q') { //Rook+Queen moves
    if (x > 0) {
      for (let i=x-1;i>=0;i--) {
        if (board.cells[i][y]) {
          if (board.cells[i][y][0] != color) {
            halfMoves.push({xTo:i, yTo:y, flags:[]})
          }
          break
        }
        halfMoves.push({xTo:i, yTo:y, flags:[]})
      }
    }
    if (x < 7) {
      for (let i=x+1;i<=7;i++) {
        if (board.cells[i][y]) {
          if (board.cells[i][y][0] != color) {
            halfMoves.push({xTo:i, yTo:y, flags:[]})
          }
          break
        }
        halfMoves.push({xTo:i, yTo:y, flags:[]})
      }
    }
    if (y > 0) {
      for (let i=y-1;i>=0;i--) {
        if (board.cells[x][i]) {
          if (board.cells[x][i][0] != color) {
            halfMoves.push({xTo:x, yTo:i, flags:[]})
          }
          break
        }
        halfMoves.push({xTo:x, yTo:i, flags:[]})
      }
    }
    if (y < 7) {
      for (let i=y+1;i<=7;i++) {
        if (board.cells[x][i]) {
          if (board.cells[x][i][0] != color) {
            halfMoves.push({xTo:x, yTo:i, flags:[]})
          }
          break
        }
        halfMoves.push({xTo:x, yTo:i, flags:[]})
      }
    }
  }

  if (type == 'B' || type == 'Q') { //Bishop+Queen moves
    for (let xIncrement of [-1, 1]) {
      for (let yIncrement of [-1, 1]) {
        let i = x + xIncrement
        let j = y + yIncrement
        while (i >= 0 && i <= 7 && j >= 0 && j <= 7) {
          if (board.cells[i][j]) {
            if (board.cells[i][j][0] != color) {
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
      if (i >= 0 && i <= 7 && j >= 0 && j <= 7 && board.cells[i][j][0] != color) {
        halfMoves.push({xTo:i, yTo:j, flags:[]})
      }
    }
  }

  if (type == 'K') { //King moves
    //Regular moves
    const steps = [[1, 1], [1, 0], [1, -1], [0, -1], [-1, -1], [-1, 0], [-1, 1], [0, 1]]
    for (let step of steps) {
      let i = x + step[0]
      let j = y + step[1]
      if (i >= 0 && i <= 7 && j >= 0 && j <= 7 && board.cells[i][j][0] != color) {
        halfMoves.push({xTo:i, yTo:j, flags:[]})
      }
    }
    if (color == 'W') { //White king castling
      if (last(board.castleWKStack) && !board.cells[5][7] && !board.cells[6][7] && !isCellThreatened(5, 7, 'B') && !isCellThreatened(4, 7, 'B')) { //King side
        halfMoves.push({xTo:6, yTo:7, flags:['CK']})
      }
      if (last(board.castleWQStack) && !board.cells[1][7] && !board.cells[2][7] && !board.cells[3][7] && !isCellThreatened(3, 7, 'B') && !isCellThreatened(4, 7, 'B')) { //Queen side
        halfMoves.push({xTo:2, yTo:7, flags:['CQ']})
      }
    } else { //Black king castling
      if (last(board.castleBKStack) && !board.cells[5][0] && !board.cells[6][0] && !isCellThreatened(5, 0, 'W') && !isCellThreatened(4, 0, 'W')) { //King side
        halfMoves.push({xTo:6, yTo:0, flags:['CK']})
      }
      if (last(board.castleBQStack) && !board.cells[1][0] && !board.cells[2][0] && !board.cells[3][0] && !isCellThreatened(3, 0, 'W') && !isCellThreatened(4, 0, 'W')) { //Queen side
        halfMoves.push({xTo:2, yTo:0, flags:['CQ']})
      }
    }
  }
  
  //Verify potential moves, discarding those that place the moving player's king under threat
  //Apply capture flags
  const verifiedMoves = []
  for (let halfMove of halfMoves) {
    if (board.cells[halfMove.xTo][halfMove.yTo])
      halfMove.flags.push('X' + board.cells[halfMove.xTo][halfMove.yTo][1])
    let move = {xFrom:x, yFrom:y, ...halfMove}
    makeMove(move)
    if (!isKingInCheck(color)) {
      verifiedMoves.push(move)
    }
    unmakeMove(move)
  }

  return verifiedMoves
}

function getCapturesFromPiece(x, y) {
  const halfMoves = []
  let color = board.cells[x][y][0]
  let type = board.cells[x][y][1]

  if (type == 'P') { //Pawn moves
    if (color == 'W') { //White pawns
      if (x > 0 && board.cells[x-1][y-1][0] == 'B') { //Capture left
        const flags = []
        if (y == 1)
          flags.push('PQ')
        halfMoves.push({xTo:x-1, yTo:y-1, flags:flags})
      }
      if (x < 7 && board.cells[x+1][y-1][0] == 'B') { //Capture right
        const flags = []
        if (y == 1)
          flags.push('PQ')
        halfMoves.push({xTo:x+1, yTo:y-1, flags:flags})
      }
      if (last(board.enPassantStack) !== null) { //Capture en passant
        let enPassantFile = last(board.enPassantStack)
        if (y == 3 && (x == enPassantFile-1 || x == enPassantFile+1)){
          halfMoves.push({xTo:enPassantFile, yTo:2, flags:['EP']})
        }
      }
    } else { //Black pawns
      if (x > 0 && board.cells[x-1][y+1][0] == 'W') { //Capture left
        const flags = []
        if (y==6)
          flags.push('PQ')
        halfMoves.push({xTo:x-1, yTo:y+1, flags:flags})
      }
      if (x < 7 && board.cells[x+1][y+1][0] == 'W') { //Capture right
        const flags = []
        if (y==6)
          flags.push('PQ')
        halfMoves.push({xTo:x+1, yTo:y+1, flags:flags})
      }
      if (last(board.enPassantStack) != null) { //Capture en passant
        let enPassantFile = last(board.enPassantStack)
        if (y == 4 && (x == enPassantFile-1 || x == enPassantFile+1)){
          halfMoves.push({xTo:enPassantFile, yTo:5, flags:['EP']})
        }
      }
    }
  }

  if (type == 'R' || type == 'Q') { //Rook+Queen moves
    if (x > 0) {
      for (let i=x-1;i>=0;i--) {
        if (board.cells[i][y]) {
          if (board.cells[i][y][0] != color) {
            halfMoves.push({xTo:i, yTo:y, flags:[]})
          }
          break
        }
      }
    }
    if (x < 7) {
      for (let i=x+1;i<=7;i++) {
        if (board.cells[i][y]) {
          if (board.cells[i][y][0] != color) {
            halfMoves.push({xTo:i, yTo:y, flags:[]})
          }
          break
        }
      }
    }
    if (y > 0) {
      for (let i=y-1;i>=0;i--) {
        if (board.cells[x][i]) {
          if (board.cells[x][i][0] != color) {
            halfMoves.push({xTo:x, yTo:i, flags:[]})
          }
          break
        }
      }
    }
    if (y < 7) {
      for (let i=y+1;i<=7;i++) {
        if (board.cells[x][i]) {
          if (board.cells[x][i][0] != color) {
            halfMoves.push({xTo:x, yTo:i, flags:[]})
          }
          break
        }
      }
    }
  }

  if (type == 'B' || type == 'Q') { //Bishop+Queen moves
    for (let xIncrement of [-1, 1]) {
      for (let yIncrement of [-1, 1]) {
        let i = x + xIncrement
        let j = y + yIncrement
        while (i >= 0 && i <= 7 && j >= 0 && j <= 7) {
          if (board.cells[i][j]) {
            if (board.cells[i][j][0] != color) {
              halfMoves.push({xTo:i, yTo:j, flags:[]})
            }
            break
          }
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
      if (i >= 0 && i <= 7 && j >= 0 && j <= 7 && board.cells[i][j] && board.cells[i][j][0] != color) {
        halfMoves.push({xTo:i, yTo:j, flags:[]})
      }
    }
  }

  if (type == 'K') { //King moves
    const steps = [[1, 1], [1, 0], [1, -1], [0, -1], [-1, -1], [-1, 0], [-1, 1], [0, 1]]
    for (let step of steps) {
      let i = x + step[0]
      let j = y + step[1]
      if (i >= 0 && i <= 7 && j >= 0 && j <= 7 && board.cells[i][j] && board.cells[i][j][0] != color) {
        halfMoves.push({xTo:i, yTo:j, flags:[]})
      }
    }
  }
  
  //Verify potential moves, discarding those that place the moving player's king under threat
  //Apply capture flags
  const verifiedCaptures = []
  for (let halfMove of halfMoves) {
    if (!halfMove.flags.includes('EP'))
      halfMove.flags.push('X' + board.cells[halfMove.xTo][halfMove.yTo][1])
    let move = {xFrom:x, yFrom:y, ...halfMove}
    makeMove(move)
    if (!isKingInCheck(color)) {
      verifiedCaptures.push(move)
    }
    unmakeMove(move)
  }

  return verifiedCaptures
}

function isKingInCheck(color) {
  for (let y=0;y<8;y++) {
    for (let x=0;x<8;x++) {
      if (board.cells[x][y] == color+'K') {
        return isCellThreatened(x, y, getOppositeColor(color))
      }
    }
  }
}

function isCellThreatened(x, y, byColor) {
  if (byColor == 'B' && x > 1) {
    if ((x > 0 && board.cells[x-1][y-1] == 'BP') || (x < 7 && board.cells[x+1][y-1] == 'BP'))
      return true
  }
  if (byColor == 'W' && y < 6) {
    if ((x > 0 && board.cells[x-1][y+1] == 'WP') || (x < 7 && board.cells[x+1][y+1] == 'WP'))
      return true
  }
  let knightSteps = [[2, 1], [2, -1], [1, 2], [1, -2], [-2, 1], [-2, -1], [-1, 2], [-1, -2]]
  let bishopSteps = [[1, 1], [1, -1], [-1, 1], [-1, -1]]
  let rookSteps = [[1, 0], [-1, 0], [0, 1], [0, -1]]
  let kingSteps = [[1, 1], [1, 0], [1, -1], [0, -1], [-1, -1], [-1, 0], [-1, 1], [0, 1]]
  for (let step of knightSteps) {
    let i = x + step[0]
    let j = y + step[1]
    if (i < 0 || i > 7 || j < 0 || j > 7)
      continue
    if (board.cells[i][j][1] == 'N' && board.cells[i][j][0] == byColor)
      return true
  }
  for (let step of bishopSteps) {
    let i = x + step[0]
    let j = y + step[1]
    while (i >= 0 && i <= 7 && j >= 0 && j <= 7) {
      if (board.cells[i][j]) {
        if ((board.cells[i][j][1] == 'B' || board.cells[i][j][1] == 'Q') && board.cells[i][j][0] == byColor)
          return true
        break
      }
      i += step[0]
      j += step[1]
    }
  }
  for (let step of rookSteps) {
    let i = x + step[0]
    let j = y + step[1]
    while (i >= 0 && i <= 7 && j >= 0 && j <= 7) {
      if (board.cells[i][j]) {
        if ((board.cells[i][j][1] == 'R' || board.cells[i][j][1] == 'Q') && board.cells[i][j][0] == byColor)
          return true
        break
      }
      i += step[0]
      j += step[1]
    }
  }
  for (let step of kingSteps) {
    let i = x + step[0]
    let j = y + step[1]
    if (i < 0 || i > 7 || j < 0 || j > 7)
      continue
    if (board.cells[i][j][1] == 'K' && board.cells[i][j][0] == byColor)
      return true
  }
  return false
}

function makeMove(move) {
  //Keep track of moves for the 50 move rule
  if (board.cells[move.xFrom][move.yFrom][1] == 'P' || board.cells[move.xTo][move.yTo]) {
    board.moveDrawStack.push(0)
  } else  {
    board.moveDrawStack.push(last(board.moveDrawStack) + 1)
  }
  //Keep track of en passant vulnerabilities
  if (move.flags.includes('DP')) {
    board.enPassantStack.push(move.xFrom)
  } else {
    board.enPassantStack.push(null)
  }
  //Keep track of castling priveleges
  if ((move.xFrom == 7 && move.yFrom == 7) || (move.xTo == 7 && move.yTo == 7) || (move.xFrom == 4 && move.yFrom == 7)) {
    board.castleWKStack.push(false)
  } else {
    board.castleWKStack.push(last(board.castleWKStack))
  }
  if ((move.xFrom == 0 && move.yFrom == 7) || (move.xTo == 0 && move.yTo == 7) || (move.xFrom == 4 && move.yFrom == 7)) {
    board.castleWQStack.push(false)
  } else {
    board.castleWQStack.push(last(board.castleWQStack))
  }
  if ((move.xFrom == 7 && move.yFrom == 0) || (move.xTo == 7 && move.yTo == 0) || (move.xFrom == 4 && move.yFrom == 0)) {
    board.castleBKStack.push(false)
  } else {
    board.castleBKStack.push(last(board.castleBKStack))
  }
  if ((move.xFrom == 0 && move.yFrom == 0) || (move.xTo == 0 && move.yTo == 0) || (move.xFrom == 4 && move.yFrom == 0)) {
    board.castleBQStack.push(false)
  } else {
    board.castleBQStack.push(last(board.castleBQStack))
  }
  //Resolve en passant moves
  if (move.flags.includes('EP')) {
    if (board.playerToMove == 'W') {
      board.cells[move.xTo][3] = ''
    } else {
      board.cells[move.xTo][4] = ''
    }
  }
  //Resolve castling moves
  if (move.flags.includes('CK')) {
    if (board.playerToMove == 'W') {
      board.cells[7][7] = ''
      board.cells[5][7] = 'WR'
    } else {
      board.cells[7][0] = ''
      board.cells[5][0] = 'BR'
    }
  }
  if (move.flags.includes('CQ')) {
    if (board.playerToMove == 'W') {
      board.cells[0][7] = ''
      board.cells[3][7] = 'WR'
    } else {
      board.cells[0][0] = ''
      board.cells[3][0] = 'BR'
    }
  }
  //Move the piece and resolve pawn promotions
  if (move.flags.includes('PQ')) {
    board.cells[move.xTo][move.yTo] = board.playerToMove + 'Q'
  } else {
    board.cells[move.xTo][move.yTo] = board.cells[move.xFrom][move.yFrom]
  }
  board.cells[move.xFrom][move.yFrom] = ''
  //Change who moves next
  board.playerToMove = getOppositeColor(board.playerToMove)
}

function unmakeMove(move) {
  //Change who moves next
  board.playerToMove = getOppositeColor(board.playerToMove)
  //Move the piece and resolve pawn promotions
  if (move.flags.includes('PQ')) {
    board.cells[move.xFrom][move.yFrom] = board.playerToMove + 'P'
  } else {
    board.cells[move.xFrom][move.yFrom] = board.cells[move.xTo][move.yTo]
  }
  //Replace opponent piece if one was taken
  let replacement = ''
  for (let flag of move.flags) {
    if (flag[0] == 'X')
      replacement = getOppositeColor(board.playerToMove) + flag[1]
  }
  board.cells[move.xTo][move.yTo] = replacement
  //Resolve castling moves
  if (move.flags.includes('CK')) {
    if (board.playerToMove == 'W') {
      board.cells[7][7] = 'WR'
      board.cells[5][7] = ''
    } else {
      board.cells[7][0] = 'BR'
      board.cells[5][0] = ''
    }
  }
  if (move.flags.includes('CQ')) {
    if (board.playerToMove == 'W') {
      board.cells[0][7] = 'WR'
      board.cells[3][7] = ''
    } else {
      board.cells[0][0] = 'BR'
      board.cells[3][0] = ''
    }
  }
  //Resolve en passant moves
  if (move.flags.includes('EP')) {
    if (board.playerToMove == 'W') {
      board.cells[move.xTo][3] = 'BP'
    } else {
      board.cells[move.xTo][4] = 'WP'
    }
  }
  //Keep track of misc states
  board.moveDrawStack.pop()
  board.enPassantStack.pop()
  board.castleWKStack.pop()
  board.castleWQStack.pop()
  board.castleBKStack.pop()
  board.castleBQStack.pop()
  
}

function findBestMove(depth) {
  let alpha = -Infinity
  let beta = Infinity
  evaluations = 0
  
  let bestMove = board.moves[0]

  if (board.playerToMove == 'W') {
    for (let move of board.moves) {
      makeMove(move)
      let newEval = deepEvaluate(alpha, beta, depth-1)
      unmakeMove(move)
      if (newEval > alpha) {
        alpha = newEval
        bestMove = move
      }
    }
    evaluation.value = alpha
  } else {
    for (let move of board.moves) {
      makeMove(move)
      let newEval = deepEvaluate(alpha, beta, depth-1)
      unmakeMove(move)
      if (newEval < beta) {
        beta = newEval
        bestMove = move
      }
    }
    evaluation.value = beta
  }
  return bestMove
}

function deepEvaluate(alpha, beta, depth) {
  if (depth == 0) {
    return narrowEvaluate(alpha, beta, searchDepthNarrow)
  }
  updateMoves()
  if (!board.moves.length) {
    if (board.playerToMove == 'W' && isKingInCheck('W'))
      return -100
    if (board.playerToMove == 'B' && isKingInCheck('B'))
      return 100
    return 0
  }
  if (board.playerToMove == 'W') {
    let bestEval = -Infinity
    for (let move of board.moves) {
      makeMove(move)
      let newEval = deepEvaluate(alpha, beta, depth-1)
      unmakeMove(move)
      bestEval = Math.max(bestEval, newEval)
      alpha = Math.max(alpha, bestEval)
      if (beta <= alpha)
        break
    }
    return bestEval
  } else {
    let bestEval = Infinity
    for (let move of board.moves) {
      makeMove(move)
      let newEval = deepEvaluate(alpha, beta, depth-1)
      unmakeMove(move)
      bestEval = Math.min(bestEval, newEval)
      beta = Math.min(beta, bestEval)
      if (beta <= alpha)
        break
    }
    return bestEval
  }
}

function narrowEvaluate(alpha, beta, depth) {
  if (depth == 0) {
    return evaluate()
  }
  updateCaptures()
  board.captures.sort((a, b) => pieceValues[board.cells[b.xTo][b.yTo][1]] + pieceValues[board.cells[a.xFrom][a.yFrom][1]] - pieceValues[board.cells[a.xTo][a.yTo][1]] - pieceValues[board.cells[b.xFrom][b.yFrom][1]])
  if (!board.captures.length) {
    return evaluate()
  }
  if (board.playerToMove == 'W') {
    let standPat = evaluate()
    if (standPat >= beta)
      return standPat
    if (standPat > alpha)
      alpha = standPat
    let bestEval = -Infinity
    for (let capture of board.captures) {
      makeMove(capture)
      let newEval = narrowEvaluate(alpha, beta, depth-1)
      unmakeMove(capture)
      bestEval = Math.max(bestEval, newEval)
      alpha = Math.max(alpha, bestEval)
      if (beta <= alpha)
        break
    }
    return bestEval
  } else {
    let standPat = evaluate()
    if (standPat <= alpha)
      return standPat
    if (standPat < beta)
      beta = standPat
    let bestEval = Infinity
    for (let capture of board.captures) {
      makeMove(capture)
      let newEval = narrowEvaluate(alpha, beta, depth-1)
      unmakeMove(capture)
      bestEval = Math.min(bestEval, newEval)
      beta = Math.min(beta, bestEval)
      if (beta <= alpha)
        break
    }
    return bestEval
  }
}

function evaluate() {
  evaluations++
  let evaluation = 0
  for (let y=0;y<8;y++) {
    for (let x=0;x<8;x++) {
      if (board.cells[x][y]) {
        let color = board.cells[x][y][0]
        let type = board.cells[x][y][1]
        if (color == 'W') {
          evaluation += pieceValues[type] + pieceTables[type][x][y]
        } else {
          evaluation -= pieceValues[type] + pieceTables[type][x][7-y]
        }
      }
    }
  }
  return evaluation / 100
}

function clickCell(event, cellNum) {

  if (!board.moves.length)
    return

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
    const move = highlightedMoves[[x, y]]
    makeRealMove(move)
  } else {
    resetSquareClassRef()
    let piece = board.cells[x][y]
    if (piece[0] == board.playerToMove) {
      squareClassRef.value[x][y] = 'HighlightSquare'
      pieceHighlight = [x, y]
      const moves = getMovesFromPiece(x, y)
      highlightedMoves.length = 0
      for (let move of moves) {
        squareClassRef.value[move.xTo][move.yTo] = 'ThreatSquare'
        highlightedMoves[[move.xTo, move.yTo]] = move
      }
    }
  }
}

function makeRealMove(move) {
  //Apply move to board
  makeMove(move)
  updateMoves()
  resetSquareClassRef()
  updateCellRef()
  pieceHighlight = null
  squareClassRef.value[move.xFrom][move.yFrom] = 'HighlightSquare'
  squareClassRef.value[move.xTo][move.yTo] = 'HighlightSquare'
  realMoves.push(move)
  
  //Check for game end conditions
  if (!board.moves.length) {
    if (isKingInCheck(board.playerToMove)) {
    message.value = 'Checkmate! '+((board.playerToMove == 'W') ? 'Black' : 'White')+' wins!'
    } else {
      message.value = 'Stalemate! Game ends in a draw.'
    }
    return
  }

  //Let AI take turn
  if ((board.playerToMove == 'B' && blackAI.value) || (board.playerToMove == 'W' && whiteAI.value)) {
    setTimeout(makeAIMove, 100)
  }
}

function unmakeRealMove() {
  unmakeMove(realMoves.pop())
  unmakeMove(realMoves.pop())
  updateMoves()
  updateCaptures()
  resetSquareClassRef()
  updateCellRef()
}

function makeAIMove() {
  let minMoveTime = 0.5
  let time1ms = Date.now()
  const bestMove = findBestMove(searchDepth)
  let time2ms = Date.now()
  let time = (time2ms - time1ms) / 1000
  console.log('Move search time: '+time+' seconds. Positions evaluated: '+evaluations)
  if (time < minMoveTime)
    setTimeout(() => makeRealMove(bestMove), (minMoveTime-time)*1000)
  else makeRealMove(bestMove)
}

function resetSquareClassRef() {
  for (let y=0;y<8;y++) {
    for (let x=0;x<8;x++) {
      squareClassRef.value[x][y] = (x+y)%2 ? 'EvenSquare' : 'OddSquare'
    }
  }
}

function updateCellRef() {
  cellRef.value = structuredClone(board.cells)
}

onMounted(() => {
  resetBoardState()
  resetSquareClassRef()
  updateCellRef()
  if (whiteAI.value) {
    makeAIMove()
  }
})

</script>

<template>
  <div class='Center'>
    <div class='Board' @click='resetSquareClassRef'>
      <div v-for='index in 64' :class='squareClassRef[(index-1)%8][Math.floor((index-1)/8)]' :key='index-1' @click.stop='(e) => clickCell(e, index-1)'><img :src='pieceImages[cellRef[(index-1)%8][Math.floor((index-1)/8)]]'></div>
    </div>
  </div>
  <p class='Center'>
    {{ board.playerToMove=='W' ? 'White' : 'Black' }} to move. Eval: {{ (evaluation >= 0 ? '+' : '')+evaluation }}<br>
    {{ message }}
  </p>
  <div class='Center'><button class='Center' v-if='realMoves.length > 1' @click='unmakeRealMove'>Undo Move</button></div>
</template>

<style>

* {
  background-color: rgb(45, 59, 77);
  color: white;
}

img {
  background-color: transparent;
}

.Center {
  display: flex;
  justify-content: center;
  align-items: center;
}

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