# https-github.io
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width" />
  <title>ゲマテトリス</title>
  <link rel="stylesheet" href="style.css" />
</head>
<body>
  <h1>ゲマテトリス</h1>
  <div id="test"></div>

  <script src="img/phaser.min.js"></script>
  <script src="script.js"></script>
</body>
</html>
1: <!DOCTYPE html>
  2: <html>
  3:   <head>
  4:     <meta charset="UTF-8">
  5:     <title>テトリス風ゲーム</title>
  6:     <meta name="viewport" content="width=device-width,initial-scale=1">
  7:     <link rel="stylesheet" type="text/css" href="css/tetris.css">
  8:     <script type="text/javascript" src="js/tetris.js"></script>
  9:   </head>
 10:   <body>
 11:     <h1>テトリス風ゲーム</h1>
 12:     <div class="wrapper-container">
 13:         <span class="tetris-container">
 14:             <canvas id="stage" width="250px" height="500px" style="background-color:black;">
 15:             </canvas>
 16:             <span class="tetris-panel-container">
 17:               <p>Next:</p>
 18:                 <canvas id="next" width="150px" height="150px" style="background-color:black;">
 19:                 </canvas>
 20:               <p>LINES:<span id="lines">0</span></p>
 21:               <p><span id="message"></span></p>
 22:               <div class="tetris-panel-container-padding"></div>
 23:               <table class="tetris-button-panel">
 24:                 <tr>
 25:                   <td></td>
 26:                   <td id="tetris-rotate-button" class="tetris-button">↻</td>
 27:                   <td></td>
 28:                 </tr>
 29:                 <tr>
 30:                   <td id="tetris-move-left-button"class="tetris-button">←</td>
 31:                   <td id="tetris-fall-button"class="tetris-button">↓</td>
 32:                   <td id="tetris-move-right-button"class="tetris-button">→</td>
 33:                 </tr>
 34:               </table>
 35:             </span>
 36:         </span>
 37:     </div>
 38:     <script>
 39:       var tetris = new Tetris();
 40:       tetris.startGame();
 41:     </script>
 42:   </body>
 43: </html>
  1: html {
  2:     touch-action: manipulation;
  3: }
  4: 
  5: .wrapper-container {
  6:     display: inline-block;
  7: }
  8: 
  9: .tetris-container {
 10:     display: flex;
 11:     flex-direction: row;
 12:     margin: 10px;
 13:     background-color: #333333;
 14: }
 15: 
 16: .tetris-panel-container {
 17:     display: flex;
 18:     padding-left: 10px;
 19:     padding-right: 10px;
 20:     flex-direction: column;
 21:     color: white;
 22:     background-color: #333333;
 23: }
 24: 
 25: .tetris-panel-container-padding {
 26:     flex-grow: 1;
 27: }
 28: 
 29: .tetris-panel-container p {
 30:    margin: 0;
 31:    padding: 0;
 32: }
 33: 
 34: .tetris-button-panel {
 35:     border-style: none;
 36:     width: 100%;
 37: }
 38: 
 39: .tetris-button {
 40:     padding-top: 10px;
 41:     padding-bottom: 10px;
 42:     text-align: center;
 43:     background: #444444;
 44: }
1: class Tetris {
  2:     constructor() {
  3:         this.stageWidth = 10;
  4:         this.stageHeight = 20;
  5:         this.stageCanvas = document.getElementById("stage");
  6:         this.nextCanvas = document.getElementById("next");
  7:         let cellWidth = this.stageCanvas.width / this.stageWidth;
  8:         let cellHeight = this.stageCanvas.height / this.stageHeight;
  9:         this.cellSize = cellWidth < cellHeight ? cellWidth : cellHeight;
 10:         this.stageLeftPadding = (this.stageCanvas.width - this.cellSize * this.stageWidth) / 2;
 11:         this.stageTopPadding = (this.stageCanvas.height - this.cellSize * this.stageHeight) / 2;
 12:         this.blocks = this.createBlocks();
 13:         this.deletedLines = 0;
 14: 
 15:         window.onkeydown = (e) => {
 16:             if (e.keyCode === 37) {
 17:                 this.moveLeft();
 18:             } else if (e.keyCode === 38) {
 19:                 this.rotate();
 20:             } else if (e.keyCode === 39) {
 21:                 this.moveRight();
 22:             } else if (e.keyCode === 40) {
 23:                 this.fall();
 24:             }
 25:         }
 26: 
 27:         document.getElementById("tetris-move-left-button").onmousedown = (e) => {
 28:             this.moveLeft();
 29:         }
 30:         document.getElementById("tetris-rotate-button").onmousedown = (e) => {
 31:             this.rotate();
 32:         }
 33:         document.getElementById("tetris-move-right-button").onmousedown = (e) => {
 34:             this.moveRight();
 35:         }
 36:         document.getElementById("tetris-fall-button").onmousedown = (e) => {
 37:             this.fall();
 38:         }
 39:     }
 40: 
 41:     createBlocks() {
 42:         let blocks = [
 43:             {
 44:                 shape: [[[-1, 0], [0, 0], [1, 0], [2, 0]],
 45:                         [[0, -1], [0, 0], [0, 1], [0, 2]],
 46:                         [[-1, 0], [0, 0], [1, 0], [2, 0]],
 47:                         [[0, -1], [0, 0], [0, 1], [0, 2]]],
 48:                 color: "rgb(0, 255, 255)",
 49:                 highlight: "rgb(255, 255, 255)",
 50:                 shadow: "rgb(0, 128, 128)"
 51:             },
 52:             {
 53:                 shape: [[[0, 0], [1, 0], [0, 1], [1, 1]],
 54:                         [[0, 0], [1, 0], [0, 1], [1, 1]],
 55:                         [[0, 0], [1, 0], [0, 1], [1, 1]],
 56:                         [[0, 0], [1, 0], [0, 1], [1, 1]]],
 57:                 color: "rgb(255, 255, 0)",
 58:                 highlight: "rgb(255, 255, 255)",
 59:                 shadow: "rgb(128, 128, 0)"
 60:             },
 61:             {
 62:                 shape: [[[0, 0], [1, 0], [-1, 1], [0, 1]],
 63:                         [[-1, -1], [-1, 0], [0, 0], [0, 1]],
 64:                         [[0, 0], [1, 0], [-1, 1], [0, 1]],
 65:                         [[-1, -1], [-1, 0], [0, 0], [0, 1]]],
 66:                 color: "rgb(0, 255, 0)",
 67:                 highlight: "rgb(255, 255, 255)",
 68:                 shadow: "rgb(0, 128, 0)"
 69:             },
 70:             {
 71:                 shape: [[[-1, 0], [0, 0], [0, 1], [1, 1]],
 72:                         [[0, -1], [-1, 0], [0, 0], [-1, 1]],
 73:                         [[-1, 0], [0, 0], [0, 1], [1, 1]],
 74:                         [[0, -1], [-1, 0], [0, 0], [-1, 1]]],
 75:                 color: "rgb(255, 0, 0)",
 76:                 highlight: "rgb(255, 255, 255)",
 77:                 shadow: "rgb(128, 0, 0)"
 78:             },
 79:             {
 80:                 shape: [[[-1, -1], [-1, 0], [0, 0], [1, 0]],
 81:                         [[0, -1], [1, -1], [0, 0], [0, 1]],
 82:                         [[-1, 0], [0, 0], [1, 0], [1, 1]],
 83:                         [[0, -1], [0, 0], [-1, 1], [0, 1]]],
 84:                 color: "rgb(0, 0, 255)",
 85:                 highlight: "rgb(255, 255, 255)",
 86:                 shadow: "rgb(0, 0, 128)"
 87:             },
 88:             {
 89:                 shape: [[[1, -1], [-1, 0], [0, 0], [1, 0]],
 90:                         [[0, -1], [0, 0], [0, 1], [1, 1]],
 91:                         [[-1, 0], [0, 0], [1, 0], [-1, 1]],
 92:                         [[-1, -1], [0, -1], [0, 0], [0, 1]]],
 93:                 color: "rgb(255, 165, 0)",
 94:                 highlight: "rgb(255, 255, 255)",
 95:                 shadow: "rgb(128, 82, 0)"
 96:             },
 97:             {
 98:                 shape: [[[0, -1], [-1, 0], [0, 0], [1, 0]],
 99:                         [[0, -1], [0, 0], [1, 0], [0, 1]],
100:                         [[-1, 0], [0, 0], [1, 0], [0, 1]],
101:                         [[0, -1], [-1, 0], [0, 0], [0, 1]]],
102:                 color: "rgb(255, 0, 255)",
103:                 highlight: "rgb(255, 255, 255)",
104:                 shadow: "rgb(128, 0, 128)"
105:             }
106:         ];
107:         return blocks;
108:     }
109: 
110:     drawBlock(x, y, type, angle, canvas) {
111:         let context = canvas.getContext("2d");
112:         let block = this.blocks[type];
113:         for (let i = 0; i < block.shape[angle].length; i++) {
114:             this.drawCell(context,
115:                      x + (block.shape[angle][i][0] * this.cellSize),
116:                      y + (block.shape[angle][i][1] * this.cellSize),
117:                      this.cellSize,
118:                      type);
119:         }
120:     }
121: 
122:     drawCell(context, cellX, cellY, cellSize, type) {
123:         let block = this.blocks[type];
124:         let adjustedX = cellX + 0.5;
125:         let adjustedY = cellY + 0.5;
126:         let adjustedSize = cellSize - 1;
127:         context.fillStyle = block.color;
128:         context.fillRect(adjustedX, adjustedY, adjustedSize, adjustedSize);
129:         context.strokeStyle = block.highlight;
130:         context.beginPath();
131:         context.moveTo(adjustedX, adjustedY + adjustedSize);
132:         context.lineTo(adjustedX, adjustedY);
133:         context.lineTo(adjustedX + adjustedSize, adjustedY);
134:         context.stroke();
135:         context.strokeStyle = block.shadow;
136:         context.beginPath();
137:         context.moveTo(adjustedX, adjustedY + adjustedSize);
138:         context.lineTo(adjustedX + adjustedSize, adjustedY + adjustedSize);
139:         context.lineTo(adjustedX + adjustedSize, adjustedY);
140:         context.stroke();
141:     }
142: 
143:     startGame() {
144:         let virtualStage = new Array(this.stageWidth);
145:         for (let i = 0; i < this.stageWidth; i++) {
146:             virtualStage[i] = new Array(this.stageHeight).fill(null);
147:         }
148:         this.virtualStage = virtualStage;
149:         this.currentBlock = null;
150:         this.nextBlock = this.getRandomBlock();
151:         this.mainLoop();
152:     }
153: 
154:     mainLoop() {
155:         if (this.currentBlock == null) {
156:             if (!this.createNewBlock()) {
157:                 return;
158:             }
159:         } else {
160:             this.fallBlock();
161:         }
162:         this.drawStage();
163:         if (this.currentBlock != null) {
164:             this.drawBlock(this.stageLeftPadding + this.blockX * this.cellSize,
165:                 this.stageTopPadding + this.blockY * this.cellSize,
166:                 this.currentBlock, this.blockAngle, this.stageCanvas);
167:         }
168:         setTimeout(this.mainLoop.bind(this), 500);
169:     }
170: 
171:     createNewBlock() {
172:         this.currentBlock = this.nextBlock;
173:         this.nextBlock = this.getRandomBlock();
174:         this.blockX = Math.floor(this.stageWidth / 2 - 2);
175:         this.blockY = 0;
176:         this.blockAngle = 0;
177:         this.drawNextBlock();
178:         if (!this.checkBlockMove(this.blockX, this.blockY, this.currentBlock, this.blockAngle)) {
179:             let messageElem = document.getElementById("message");
180:             messageElem.innerText = "GAME OVER";
181:             return false;
182:         }
183:         return true;
184:     }
185: 
186:     drawNextBlock() {
187:         this.clear(this.nextCanvas);
188:         this.drawBlock(this.cellSize * 2, this.cellSize, this.nextBlock,
189:             0, this.nextCanvas);
190:     }
191: 
192:     getRandomBlock() {
193:         return  Math.floor(Math.random() * 7);
194:     }
195: 
196:     fallBlock() {
197:         if (this.checkBlockMove(this.blockX, this.blockY + 1, this.currentBlock, this.blockAngle)) {
198:             this.blockY++;
199:         } else {
200:             this.fixBlock(this.blockX, this.blockY, this.currentBlock, this.blockAngle);
201:             this.currentBlock = null;
202:         }
203:     }
204: 
205:     checkBlockMove(x, y, type, angle) {
206:         for (let i = 0; i < this.blocks[type].shape[angle].length; i++) {
207:             let cellX = x + this.blocks[type].shape[angle][i][0];
208:             let cellY = y + this.blocks[type].shape[angle][i][1];
209:             if (cellX < 0 || cellX > this.stageWidth - 1) {
210:                 return false;
211:             }
212:             if (cellY > this.stageHeight - 1) {
213:                 return false;
214:             }
215:             if (this.virtualStage[cellX][cellY] != null) {
216:                 return false;
217:             }
218:         }
219:         return true;
220:     }
221: 
222:     fixBlock(x, y, type, angle) {
223:         for (let i = 0; i < this.blocks[type].shape[angle].length; i++) {
224:             let cellX = x + this.blocks[type].shape[angle][i][0];
225:             let cellY = y + this.blocks[type].shape[angle][i][1];
226:             if (cellY >= 0) {
227:                 this.virtualStage[cellX][cellY] = type;
228:             }
229:         }
230:         for (let y = this.stageHeight - 1; y >= 0; ) {
231:             let filled = true;
232:             for (let x = 0; x < this.stageWidth; x++) {
233:                 if (this.virtualStage[x][y] == null) {
234:                     filled = false;
235:                     break;
236:                 }
237:             }
238:             if (filled) {
239:                 for (let y2 = y; y2 > 0; y2--) {
240:                     for (let x = 0; x < this.stageWidth; x++) {
241:                         this.virtualStage[x][y2] = this.virtualStage[x][y2 - 1];
242:                     }
243:                 }
244:                 for (let x = 0; x < this.stageWidth; x++) {
245:                     this.virtualStage[x][0] = null;
246:                 }
247:             let linesElem = document.getElementById("lines");
248:                 this.deletedLines++;
249:                 linesElem.innerText = "" + this.deletedLines;
250:             } else {
251:                 y--;
252:             }
253:         }
254:     }
255: 
256:     drawStage() {
257:         this.clear(this.stageCanvas);
258: 
259:         let context = this.stageCanvas.getContext("2d");
260:         for (let x = 0; x < this.virtualStage.length; x++) {
261:             for (let y = 0; y < this.virtualStage[x].length; y++) {
262:                 if (this.virtualStage[x][y] != null) {
263:                     this.drawCell(context,
264:                         this.stageLeftPadding + (x * this.cellSize),
265:                         this.stageTopPadding + (y * this.cellSize),
266:                         this.cellSize,
267:                         this.virtualStage[x][y]);
268:                 }
269:             }
270:         }
271:     }
272: 
273:     moveLeft() {
274:         if (this.checkBlockMove(this.blockX - 1, this.blockY, this.currentBlock, this.blockAngle)) {
275:             this.blockX--;
276:             this.refreshStage();
277:         }
278:     }
279: 
280:     moveRight() {
281:         if (this.checkBlockMove(this.blockX + 1, this.blockY, this.currentBlock, this.blockAngle)) {
282:             this.blockX++;
283:             this.refreshStage();
284:         }
285:     }
286: 
287:     rotate() {
288:         let newAngle;
289:         if (this.blockAngle < 3) {
290:             newAngle = this.blockAngle + 1;
291:         } else {
292:             newAngle = 0;
293:         }
294:         if (this.checkBlockMove(this.blockX, this.blockY, this.currentBlock, newAngle)) {
295:             this.blockAngle = newAngle;
296:             this.refreshStage();
297:         }
298:     }
299: 
300:     fall() {
301:         while (this.checkBlockMove(this.blockX, this.blockY + 1, this.currentBlock, this.blockAngle)) {
302:             this.blockY++;
303:             this.refreshStage();
304:         }
305:     }
306: 
307:     refreshStage() {
308:         this.clear(this.stageCanvas);
309:         this.drawStage();
310:         this.drawBlock(this.stageLeftPadding + this.blockX * this.cellSize,
311:                 this.stageTopPadding + this.blockY * this.cellSize,
312:                 this.currentBlock, this.blockAngle, this.stageCanvas);
313:     }
314: 
315:     clear(canvas) {
316:         let context = canvas.getContext("2d");
317:         context.fillStyle = "rgb(0, 0, 0)";
318:         context.fillRect(0, 0, canvas.width, canvas.height);
319:     }
320: }
