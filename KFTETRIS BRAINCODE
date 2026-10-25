const COLORS = {
  1:'#3B82F6',
  2:'#E11D48',
  3:'#10B981',
  4:'#8B5CF6', 
  5:'#F59E0B',
  6:'#EC4899',
  7:'#14B8A6'
};

const BG_COLOR = '#EEEEEE';

const INTERVAL = 500;

const ROWS = 20;

const COLS = 10;

const PIECES = [
  [[1,1,1,1]], //straight
  [[2,2],[2,2]],//box
  [[0,3,0],[3,3,3]],//T shaped
  [[4,0,0],[4,4,4]],//L shaped
  [[0,0,5],[5,5,5]],//j shaped
  [[6,6,0],[0,6,6]],//S shaped
  [[0,7,7],[7,7,0]],//z shaped
];

let board = [];

let currentPiece = null;

let currentPieceIndex = 0;

let pieceRow = 0;

let pieceCol = 0;

let score = 0;

let finished = false;

function startGame() {
  
  initializeBoard();
  resetValues();
  showSidebar();
  spawnPiece();

  
  finished = false;
  let startTime = 0;
  let passedTime = 0;

  do {
    
    let restart = PropertiesService.getScriptProperties().getProperty('restart') == 'true';
    if (restart) {
      finished = true;
      score = 0;
    }

   
    startTime = new Date();
    gameTick();
    passedTime = new Date() - startTime;
    if (INTERVAL - passedTime > 0) {
      waiting(INTERVAL - passedTime);
    }
  } while (!finished);
}

function initializeBoard() {
  
  board = Array.from({ length: ROWS }, () => Array(COLS).fill(0));
 
  const sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
 
  const range = sheet.getRange(1, 1, ROWS, COLS);

  range.setBackground(BG_COLOR);

  range.clearContent();

 
  const cellSize = 30;

  for (let i = 1; i <= COLS; i++) {
    sheet.setColumnWidth(i, cellSize);
  }

  for (let j = 1; j <= ROWS; j++) {
    sheet.setRowHeight(j, cellSize);
  }
}


function spawnPiece() {
  currentPieceIndex = Math.floor(Math.random() * PIECES.length);
  
  currentPiece = PIECES[currentPieceIndex];
  
  pieceRow = 0;
  
  pieceCol = Math.floor((COLS - currentPiece[0].length) / 2);
  
  if (!isValidMove(currentPiece, pieceRow, pieceCol)) {
  
    SpreadsheetApp.getActiveSpreadsheet().toast('Game Over!', 'Tetris');
    finished = true;
    return;
  }
  drawPiece(currentPiece, pieceRow, pieceCol);
}

function drawPiece(piece, row, col) {
  
  const updates = [];
  
  piece.forEach((r, i) => {
  
    r.forEach((cell, j) => {
  
      if (cell) {
  
        updates.push({ row: row + i + 1, col: col + j + 1, color: COLORS[currentPieceIndex] });
      }
    });
  });
  batchUpdateCells(updates);
}

function clearPiece(piece, row, col) {
  
  const updates = [];
  
  piece.forEach((r, i) => {
  
    r.forEach((cell, j) => {
  
      if (cell) {
  
        updates.push({ row: row + i + 1, col: col + j + 1, color: BG_COLOR });
      }
    });
  });
  batchUpdateCells(updates);
}

function batchUpdateCells(updates) {
  
  const sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
  
  updates.forEach(update => {
  
    sheet.getRange(update.row, update.col).setBackground(update.color);
  });
}

function isValidMove(piece, row, col) {
  
  return piece.every((r, i) => r.every((cell, j) => {
  
    const x = row + i;
  
    const y = col + j;
  
    return (
  
      cell === 0 ||
      (x >= 0 && x < ROWS && y >= 0 && y < COLS && board[x][y] === 0)
    );
  }));
}

function gameTick() {
  
  let moveQueue = JSON.parse(PropertiesService.getScriptProperties().
  getProperty('moveQueue') || '[]');
  
  PropertiesService.getScriptProperties().setProperty('moveQueue', JSON.stringify([]));

  clearPiece(currentPiece, pieceRow, pieceCol);

  moveQueue.forEach(move => {
  
    if (move === 'left' && isValidMove(currentPiece, pieceRow, pieceCol - 1)) {
      pieceCol--;
    
    } else if (move === 'right' && isValidMove(currentPiece, pieceRow, pieceCol + 1)) {
      pieceCol++;
    
    } else if (move === 'down' && isValidMove(currentPiece, pieceRow + 1, pieceCol)) {
      pieceRow++;
    
    } else if (move === 'rotate') {
      const rotated = rotatePiece(currentPiece);
      if (isValidMove(rotated, pieceRow, pieceCol)) {
        currentPiece = rotated;
      }
    }
  });

  if (isValidMove(currentPiece, pieceRow + 1, pieceCol)) {
    pieceRow++;
    drawPiece(currentPiece, pieceRow, pieceCol);
  
  } else {
    lockPiece();
    clearLines();
    drawBoard();
    spawnPiece();
  }
}

function lockPiece() {
  
  currentPiece.forEach((r, i) => {
  
    r.forEach((cell, j) => {
  
      if (cell) {
        board[pieceRow + i][pieceCol + j] = currentPieceIndex + 1;
      }
    });
  });
}

function clearLines() {
  
  let prevScore = score;
  
  for (let r = ROWS - 1; r >= 0; r--) {
  
    if (board[r].every(cell => cell > 0)) {
  
      board.splice(r, 1);
      board.unshift(Array(COLS).fill(0));
      r++;

      score += 100;
    }
  }

  if (prevScore < score) {
    PropertiesService.getScriptProperties().setProperty('score', score);
  }
}

function drawBoard() {

  const updates = [];

  for (let r = 0; r < ROWS; r++) {

    for (let c = 0; c < COLS; c++) {

      const color = board[r][c] > 0 ? COLORS[board[r][c] - 1] : BG_COLOR;
      updates.push({ row: r + 1, col: c + 1, color });
    }
  }
  batchUpdateCells(updates);
}

function handleMove(direction) {

  let moveQueue = JSON.parse(PropertiesService.getScriptProperties().getProperty('moveQueue') || '[]');
  moveQueue.push(direction);
  
  PropertiesService.getScriptProperties().setProperty('moveQueue', JSON.stringify(moveQueue));
}

function rotatePiece(piece) {
  
  return piece[0].map((val, index) => piece.map(row => row[index]).reverse());
}

function onOpen() {
  
  SpreadsheetApp.getUi().createMenu('Tetris')
    .addItem('Start Game', 'startGame')
    .addToUi();
}

function resetValues() {
  
  PropertiesService.getScriptProperties().setProperty('score', score);
  
  PropertiesService.getScriptProperties().setProperty('moveQueue', JSON.stringify([]));
  
  PropertiesService.getScriptProperties().setProperty('restart', false);
}

function getScore() {
  
  return parseInt(PropertiesService.getScriptProperties().getProperty('score'));
}

function restartGame() {
  
  PropertiesService.getScriptProperties().setProperty('restart', true);
  waiting(2000);
  
  startGame();
}

function showSidebar() {
  
  const html = HtmlService.createHtmlOutputFromFile('Sidebar')
    .setTitle('Tetris Controls')
    .setWidth(300);
  
  SpreadsheetApp.getUi().showSidebar(html);
}

function waiting(msec) {
 
  SpreadsheetApp.flush();
 
  Utilities.sleep(msec);
 
  SpreadsheetApp.flush();
}
