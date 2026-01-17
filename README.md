# chess-pro
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Pro Chess</title>

<style>
body{
  margin:0;
  height:100vh;
  display:flex;
  justify-content:center;
  align-items:center;
  background:radial-gradient(circle at top,#1e293b,#020617);
  font-family:"Segoe UI",Arial;
  color:#f8fafc;
}
.screen{display:none;text-align:center}
.active{display:block}

/* MENU */
.menu{
  padding:40px;
  border-radius:20px;
  background:rgba(0,0,0,0.35);
  box-shadow:0 25px 60px rgba(0,0,0,0.9);
}
.menu h1{font-size:42px;margin-bottom:8px}
.menu p{opacity:.85;margin-bottom:30px}
.btn{
  width:260px;
  padding:14px 0;
  margin:12px auto;
  font-size:18px;
  border:none;
  border-radius:12px;
  cursor:pointer;
  display:block;
  transition:.25s;
}
.btn:hover{transform:scale(1.05)}
.ai{background:#2563eb;color:#fff}
.friends{background:#16a34a;color:#fff}

/* GAME */
#board{
  width:480px;height:480px;
  display:grid;
  grid-template-columns:repeat(8,1fr);
  border:10px solid #3f2a14;
  box-shadow:0 20px 50px rgba(0,0,0,.8);
}
.square{
  width:60px;height:60px;
  display:flex;
  align-items:center;
  justify-content:center;
  font-size:42px;
  cursor:pointer;
  user-select:none;
}
.light{background:#f0d9b5}
.dark{background:#b58863}
.selected{background:#facc15!important}
.move{background:#86efac!important}
.white{color:#fff;text-shadow:0 2px 4px #000}
.black{color:#000;text-shadow:0 2px 4px #fff}
.game-btn{
  margin:10px;
  padding:8px 16px;
  border:none;
  border-radius:6px;
  background:#2563eb;
  color:#fff;
  cursor:pointer;
}
</style>
</head>

<body>

<!-- MENU -->
<div id="menu" class="screen active">
  <div class="menu">
    <h1>♟️ Welcome to Pro Chess</h1>
    <p>Think • Plan • Checkmate</p>
    <button class="btn ai" onclick="startGame()">🤖 Play with AI</button>
    <button class="btn friends" onclick="alert('Friends mode coming soon')">
      👥 Play with Friends
    </button>
  </div>
</div>

<!-- GAME -->
<div id="game" class="screen">
  <h2>Chess vs AI</h2>
  <div>Turn: <span id="turn">White</span></div>
  <div id="board"></div>
  <button class="game-btn" onclick="restart()">Restart</button>
  <button class="game-btn" onclick="back()">Back</button>
</div>

<script>
const menu = document.getElementById("menu");
const game = document.getElementById("game");
const boardEl = document.getElementById("board");
const turnEl = document.getElementById("turn");

const symbols={
r:"♜",n:"♞",b:"♝",q:"♛",k:"♚",p:"♟",
R:"♖",N:"♘",B:"♗",Q:"♕",K:"♔",P:"♙"
};

let board, turn, selected, moves, gameOver;

const isWhite = p => p === p.toUpperCase();
const set = (row,i,ch)=>row.slice(0,i)+ch+row.slice(i+1);

function startGame(){
  menu.classList.remove("active");
  game.classList.add("active");
  restart();
}

function back(){
  game.classList.remove("active");
  menu.classList.add("active");
}

function restart(){
  board=[
    "rnbqkbnr","pppppppp","........","........",
    "........","........","PPPPPPPP","RNBQKBNR"
  ];
  turn="white";
  selected=null;
  moves=[];
  gameOver=false;
  turnEl.textContent="White";
  draw();
}

function draw(){
  boardEl.innerHTML="";
  board.forEach((row,r)=>{
    [...row].forEach((p,c)=>{
      const sq=document.createElement("div");
      sq.className="square "+((r+c)%2?"dark":"light");
      sq.onclick=()=>click(r,c,sq);
      if(p!="."){
        const s=document.createElement("span");
        s.textContent=symbols[p];
        s.className=isWhite(p)?"white":"black";
        sq.appendChild(s);
      }
      boardEl.appendChild(sq);
    });
  });
}

function click(r,c,sq){
  if(gameOver||turn!=="white")return;
  clear();
  const p=board[r][c];

  if(selected){
    if(moves.some(m=>m.r===r&&m.c===c)){
      move(selected.r,selected.c,r,c);
      setTimeout(aiMove,400);
    }
    selected=null;
    moves=[];
  }else if(p!="."&&isWhite(p)){
    selected={r,c};
    sq.classList.add("selected");
    moves=getMoves(r,c,p);
    highlight(moves);
  }
}

function move(fr,fc,tr,tc){
  const target=board[tr][tc];
  if(target==="K"||target==="k"){
    gameOver=true;
    alert("Checkmate! "+turn.toUpperCase()+" wins ♟️");
    return;
  }
  board[tr]=set(board[tr],tc,board[fr][fc]);
  board[fr]=set(board[fr],fc,".");
  turn=turn==="white"?"black":"white";
  turnEl.textContent=turn[0].toUpperCase()+turn.slice(1);
  draw();
}

function getMoves(r,c,p){
  const M=[], dir=isWhite(p)?-1:1;
  const add=(nr,nc)=>{
    if(nr<0||nr>7||nc<0||nc>7)return;
    if(board[nr][nc]=="."||isWhite(board[nr][nc])!==isWhite(p))
      M.push({r:nr,c:nc});
  };

  if(p.toLowerCase()=="p"){
    if(board[r+dir]?.[c]==".") add(r+dir,c);
    [[dir,-1],[dir,1]].forEach(d=>{
      let nr=r+d[0],nc=c+d[1];
      if(nr>=0&&nr<8&&nc>=0&&nc<8 &&
         board[nr][nc]!="." &&
         isWhite(board[nr][nc])!==isWhite(p))
        M.push({r:nr,c:nc});
    });
  }

  if(p.toLowerCase()=="r"||p.toLowerCase()=="q")
    [[1,0],[-1,0],[0,1],[0,-1]].forEach(d=>{
      let nr=r+d[0],nc=c+d[1];
      while(nr>=0&&nr<8&&nc>=0&&nc<8){
        if(board[nr][nc]==".") M.push({r:nr,c:nc});
        else{
          if(isWhite(board[nr][nc])!==isWhite(p))
            M.push({r:nr,c:nc});
          break;
        }
        nr+=d[0]; nc+=d[1];
      }
    });

  if(p.toLowerCase()=="b"||p.toLowerCase()=="q")
    [[1,1],[1,-1],[-1,1],[-1,-1]].forEach(d=>{
      let nr=r+d[0],nc=c+d[1];
      while(nr>=0&&nr<8&&nc>=0&&nc<8){
        if(board[nr][nc]==".") M.push({r:nr,c:nc});
        else{
          if(isWhite(board[nr][nc])!==isWhite(p))
            M.push({r:nr,c:nc});
          break;
        }
        nr+=d[0]; nc+=d[1];
      }
    });

  if(p.toLowerCase()=="n")
    [[2,1],[2,-1],[-2,1],[-2,-1],[1,2],[1,-2],[-1,2],[-1,-2]]
      .forEach(d=>add(r+d[0],c+d[1]));

  if(p.toLowerCase()=="k")
    [[1,0],[-1,0],[0,1],[0,-1],[1,1],[1,-1],[-1,1],[-1,-1]]
      .forEach(d=>add(r+d[0],c+d[1]));

  return M;
}

function aiMove(){
  if(gameOver||turn!=="black")return;
  let all=[];
  board.forEach((row,r)=>[...row].forEach((p,c)=>{
    if(p!="."&&!isWhite(p))
      getMoves(r,c,p).forEach(m=>all.push({fr:r,fc:c,...m}));
  }));
  if(!all.length)return;
  const m=all[Math.floor(Math.random()*all.length)];
  move(m.fr,m.fc,m.r,m.c);
}

function highlight(M){
  document.querySelectorAll(".square").forEach((s,i)=>{
    const r=Math.floor(i/8),c=i%8;
    if(M.some(m=>m.r===r&&m.c===c))
      s.classList.add("move");
  });
}
function clear(){
  document.querySelectorAll(".square")
    .forEach(s=>s.classList.remove("selected","move"));
}
</script>

</body>
</html>
