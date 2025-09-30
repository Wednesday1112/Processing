# 目錄
- [摺痕圖分析](#摺痕圖分析)
- [貪吃蛇](#貪吃蛇)
  - [主程式](#主程式)
  - [主畫面](#主畫面)
  - [暫停](#暫停)
  - [敵人](#敵人)
  - [遊戲失敗](#遊戲失敗)
  - [重新開始](#重新開始)

# 摺痕圖分析
```java
HashMap<PVector, Integer> p2v = new HashMap<PVector, Integer>();
ArrayList<PVector> vertices = new ArrayList<PVector>();
ArrayList<Integer> triangles = new ArrayList<Integer>();
ArrayList<Edge> edges = new ArrayList<Edge>();
ArrayList<Triangle> myTriangles = new ArrayList<Triangle>();
int n = 0;
int n1 = 0;
class Edge{
  ArrayList<Integer> faces = new ArrayList<Integer>();
  int v0, v1, lineType;
  Edge(int _v0, int _v1, int _type){
    v0 = _v0;
    v1 = _v1;
    lineType = _type;
  }
  void display(){
    if(faces.size()==1){ //黑線
      stroke(0); strokeWeight(4);
    }else if(faces.size()==2){ //Hinge 橘線
      stroke(#FC8300); strokeWeight(4);
    } else return; //不要畫
    PVector p0 = vertices.get(v0), p1 = vertices.get(v1);
    line(p0.x, p0.y, p1.x, p1.y);
    strokeWeight(1);
  }
}

class Triangle{
  int v0, v1, v2, triangleID;
  Triangle(int _0, int _1, int _2, int _id){
    triangleID = _id;
    v0 = _0; v1 = _1; v2 = _2;
  }
  Triangle(int _0, int _1, int _2){
    v0 = _0;
    v1 = _1;
    v2 = _2;
  }
  void display(){
    PVector p0 = vertices.get(v0), p1 = vertices.get(v1), p2 = vertices.get(v2);
    fill(0,255,0, 128);
    beginShape(TRIANGLES);
      vertex(p0.x, p0.y);
      vertex(p1.x, p1.y);
      vertex(p2.x, p2.y);
    endShape();
    fill(255,0,0);
    PVector center = PVector.add(PVector.add(p0,p1),p2).div(3);
    text("i:"+triangleID, center.x, center.y);
  }
}

int helper(float x, float y){
  if(abs(x)<0.000001) x = 0;
  if(abs(y)<0.000001) y = 0;
  PVector now = new PVector(x, y);
  if(p2v.containsKey(now)) {
    return p2v.get(now);
  }
  int ans = vertices.size();
  vertices.add(now);
  PVector deviceNow = new PVector();
  deviceNow.x = -now.x / 100;
  deviceNow.y = -now.y / 100;
  println(String.format("vertices[%d] = new Vector3(%ff, %ff, %ff);", n, deviceNow.x, deviceNow.y, deviceNow.z));
  n++;
  p2v.put(now, ans);
  return ans;
}

void setup() {
  size(500, 500);
  String [] lines = loadStrings("dog.cp");
  for(String line : lines) {
    String [] t = split(line, ' ');
    int type = int(t[0]);
    float x0 = float(t[1]), y0 = float(t[2]);
    float x1 = float(t[3]), y1 = float(t[4]);
    int v0 = helper(x0,y0), v1 = helper(x1,y1);
    edges.add(new Edge(v0,v1,type));
  }
}

void draw() {
  background(#FFFFF2);
  translate(width/2, height/2);
  for(Edge e : edges) {
    PVector p00 = vertices.get(e.v0), p11 = vertices.get(e.v1);
    int type = e.lineType;
    if(type == 1) stroke(0);
    if(type == 2) stroke(255, 0, 0);
    if(type == 3) stroke(0, 0, 255);
    line(p00.x,p00.y, p11.x,p11.y);
  }
  for(int i=0; i<vertices.size(); i++) {
    PVector p = vertices.get(i);
    stroke(0); fill(255);
    ellipse(p.x, p.y, 30, 30);
    fill(0); textSize(25); textAlign(CENTER, CENTER);
    text(""+i, p.x, p.y);
  }
  for(Triangle t : myTriangles){
    t.display();
  }
  for(Edge e : edges){
    e.display();
  }
}

void mousePressed(){
  for(int i=0; i<vertices.size(); i++){
    PVector p = vertices.get(i);
    if( dist(p.x+width/2, p.y+height/2, mouseX, mouseY) <= 15 ){
      triangles.add(i);
    }
  }
  if(triangles.size()>0 && triangles.size()%3==0){
    int i = triangles.size()/3 - 1;
    int v0 = triangles.get(i*3+0), v1 = triangles.get(i*3+1), v2 = triangles.get(i*3+2);
    myTriangles.add( new Triangle(v0, v1, v2, i) );
    n1+=6;
    print(String.format("%2d,%2d,%2d,%2d,%2d,%2d, ", v0,v1,v2, v2,v1,v0));
    for(Edge e : edges){
      int found=0;
      if(e.v0==v0) found++;
      if(e.v0==v1) found++;
      if(e.v0==v2) found++;
      if(e.v1==v0) found++;
      if(e.v1==v1) found++;
      if(e.v1==v2) found++;
      //println(found);
      if(found==2){
        e.faces.add(i);
        //println("found");
      }
    }
  }
}
void keyPressed(){
  if(key == ENTER) println();
  if(key == SHIFT) println(n1);
}
```

# 貪吃蛇
## 主程式
```java
import processing.sound.*;
SoundFile start, welcome, eat, gameover,gaming;
ArrayList<PVector>pt=new ArrayList<PVector>();  //食物的點
ArrayList<PVector>body=new ArrayList<PVector>();  //蛇身
ArrayList<PVector>rgb = new ArrayList<PVector>();  //改變身體顏色
int snakelen = 1;
int s=1;
int snake=1;
float sizeX=-1000, sizeY=-800;
int speed=5;
float mapSpeed=1.5;
float sizeFood=20, sizeSnake=30;
float x=250, y=250, dir=0;
float dx, dy, d;
int status=0;
boolean glow=true;
float glowSize=1.25;
int time=0;
boolean playstart=false,playwelcome=false,playeat=false,playgameover=false,playgaming=false;

void setup() {
  size(1000, 800);
  welcome = new SoundFile(this, "game.mp3");
  start = new SoundFile(this, "startGame.mp3");
  eat = new SoundFile(this, "eat.mp3");
  gameover = new SoundFile(this, "gameOver.mp3");
  gaming = new SoundFile(this, "gaming.mp3");
  for (int i=0; i<1000; i++)
    pt.add(new PVector(random(sizeX, sizeX+3000), random(sizeY, sizeY+2400)));
  rgb.add(new PVector(random(255), random(255), random(255)));
}

void draw()
{
  if (status==0) welcome();
  if (status==1)
  {
    if (playwelcome==true) {
      welcome.stop();
      playwelcome=false;
    }
    if (playstart==true) {
      start.stop();
      playstart=false;
    }
    if(playgaming==false) gaming.play();
    playgaming=true;
    background(0);
    drawmap();
    fill(255);
    dx = mouseX-x;
    dy = mouseY-y;
    d = sqrt(dx * dx + dy * dy)*3;
    stroke(0);
    snake2.drawsnake();
    snake3.drawsnake();
    snake4.drawsnake();
    snake5.drawsnake();
    drawbody();
    drawhead(x, y, dir);
    move();
    
    eat(x, y, 1);
    
    mapMove();
    hit();
  }
  if (status==2) {
    gameOver();
  }
  if (status==4) {
    stopGame();
  }
}

//限制畫布範圍
void drawmap() {
  fill(60);
  stroke(255, 0, 0);
  strokeWeight(10);
  rect(sizeX, sizeY, 3000, 2400);
  stroke(75);
  strokeWeight(2);
  for (int i=50; i<2400; i+=50) line(sizeX, sizeY+i, sizeX+3000, sizeY+i);
  for (int i=50; i<3000; i+=50) line(sizeX+i, sizeY, sizeX+i, sizeY+2400);

  time+=1;
  if (glowSize<1.65 && glow && time==4) glowSize+=0.05;
  if (glowSize>=1.65) glow=false;
  if (glowSize>1.25 && !glow && time==4) glowSize-=0.05;
  if (glowSize<=1.25) glow=true;
  if (time==4) time=0;

  for (PVector p : pt) {
    noStroke();
    fill(255, 255, 0, 100);
    ellipse(p.x, p.y, sizeFood*glowSize, sizeFood*glowSize);
    fill(255, 255, 0);
    ellipse(p.x, p.y, sizeFood, sizeFood);
    fill(225, 255, 0);
    ellipse(p.x, p.y, sizeFood*0.7, sizeFood*0.7);
  }

  for (int i = pt.size() - 1; i >= 0; i--) {
    PVector p = pt.get(i);
    if (p.x < sizeX || p.x > sizeX + 3000 || p.y < sizeY || p.y > sizeY + 2400) {
      pt.remove(i);
      p.x=random(sizeX, sizeX+3000);
      p.y=random(sizeY, sizeY+2400);
      rgb.add(new PVector(random(255), random(255), random(255)));
    }
  }
}

//畫布移動
void mapMove() {
  float sx = 0.0, sy = 0.0;

  if (x < 100) {
    x = 100;
    sizeX += mapSpeed;
    sx = mapSpeed;
  } else if (x > 900) {
    x = 900;
    sizeX -= mapSpeed;
    sx = -mapSpeed;
  }

  if (y < 100) {
    y = 100;
    sizeY += mapSpeed;
    sy = mapSpeed;
  } else if (y > 700) {
    y = 700;
    sizeY -= mapSpeed;
    sy = -mapSpeed;
  }

  for (PVector p : pt) {
    p.x += sx;
    p.y += sy;
  }

  for (PVector b : body) {
    b.x += sx;
    b.y += sy;
  }
  snake2.moveBody(sx, sy);
  snake3.moveBody(sx, sy);
  snake4.moveBody(sx, sy);
  snake5.moveBody(sx, sy);
}

void drawbody() {
  int j=0;
  for (int i=0; i<body.size(); i+=8) {
    PVector b = body.get(i);
    noStroke();
    fill(rgb.get(j).x, rgb.get(j).y, rgb.get(j).z);
    ellipse(b.x, b.y, sizeSnake, sizeSnake);
    line(b.x, b.y, b.x-15*cos(b.z), b.y-15*sin(b.z) );
    j+=1;
  }
}

void drawhead(float x, float y, float dir) {
  noStroke();
  ellipse(x, y, sizeSnake, sizeSnake);
  line(x, y, x+15*cos(dir), y+15*sin(dir));
  fill(255);
}

//移動
void move() {
  if (dist(mouseX, mouseY, x, y)<20) {
    x+=1*cos(dir);
    y+=1*sin(dir);
    dir+=0.1;
  } else {
    x += dx / d*speed;
    y += dy / d*speed;
  }
  if (body.size()<snakelen*8) {
    body.add( new PVector(x, y, dir) );
    if (sizeFood>5) sizeFood-=0.01;
    sizeSnake+=0.1;
  } else {
    for (int i=0; i<body.size()-1; i++) {
      body.get(i).x = body.get(i+1).x;
      body.get(i).y = body.get(i+1).y;
      body.get(i).z = body.get(i+1).z;
    }
    body.get(body.size()-1).x = x;
    body.get(body.size()-1).y = y;
    body.get(body.size()-1).z = dir;
  }
}

//吃到食物偵測
void eat(float x, float y, int snake) {
  for (PVector p : pt) {
    if (p.x<=x+sizeSnake/2 && p.x>=x-sizeSnake/2 && p.y<=y+sizeSnake/2 && p.y>=y-sizeSnake/2) {
      if (snake==1){
        eat.play();
        snakelen++;
      }
      else if (snake==2)
        snake2.eat(x, y);
      else if (snake==3)
        snake3.eat(x, y);
      else if (snake==4)
        snake4.eat(x, y);
      else if (snake==5)
        snake5.eat(x, y);
      p.x=random(sizeX, sizeX+3000);
      p.y=random(sizeY, sizeY+2400);
      rgb.add(new PVector(random(255), random(255), random(255)));
    }
  }
}

void keyPressed() {
  if (key=='s') status=1;
  if (status==1 && key==' ') status=4;
  if (status==2 && key=='r') {
    if (playgameover==true) gameover.stop();
    if (playstart==false) start.play();
    playstart=true;
    reset();
  }
}

void keyReleased() {
  key='>';  //隨便丟一個未使用的值做初始化
}

void mousePressed() {
  speed=8;
  mapSpeed=2.5;
}

void mouseReleased() {
  speed=5;
  mapSpeed=1.5;
}
```

## 主畫面
```java
PImage img1;
void welcome()
{
  if (playwelcome==false) welcome.play();
  playwelcome=true;
  img1 = loadImage("main.png");
  img1.resize(1000, 800);
  background(60);
  stroke(75);
  strokeWeight(2);
  for (int i=50; i<2400; i+=50) line(sizeX, sizeY+i, sizeX+3000, sizeY+i);
  for (int i=50; i<3000; i+=50) line(sizeX+i, sizeY, sizeX+i, sizeY+2400);
  image(img1, 0, 0);
}
```

## 暫停
```java
PImage img2;
void stopGame()
{
  img2 = loadImage("stop.png");
  img2.resize(1000, 800);
  background(60);
  image(img2, 0, 0);
  if (key=='s') status=1;
}
```

## 敵人
```java
//每一隻蛇
class Snake {
  ArrayList<PVector> body;
  int snakelen;
  float x, y, dir;

  Snake(float x, float y) {
    this.body = new ArrayList<PVector>();
    this.snakelen = 1;
    this.x = x;
    this.y = y;
    this.dir = 0;
  }

  void reset(float x, float y) {
    body.clear();
    snakelen=1;
    dir=0;
    x=x;
    y=y;
  }

  void drawsnake() {
    for (int i = 0; i < body.size(); i += 8) {
      PVector b = body.get(i);
      noStroke();
      fill(random(200, 255), random(100, 200), random(100, 200));
      ellipse(b.x, b.y, 35, 35);
      line(b.x, b.y, b.x - 15 * cos(b.z), b.y - 15 * sin(b.z));
    }

    // 改變方向
    dir += random(-0.2, 0.2);
    x += cos(dir) * 0.6;
    y += sin(dir) * 0.6;

    if (body.size() < snakelen * 8) {
      body.add(new PVector(x, y, dir));
    } else {
      for (int i = 0; i < body.size() - 1; i++) {
        body.get(i).x = body.get(i + 1).x;
        body.get(i).y = body.get(i + 1).y;
        body.get(i).z = body.get(i + 1).z;
      }
      body.get(body.size() - 1).x = x;
      body.get(body.size() - 1).y = y;
      body.get(body.size() - 1).z = dir;
    }
    eat(x, y);

    //碰到邊界消失重置
    if (this.x < sizeX || this.x > sizeX + 3000 || this.y < sizeY || this.y > sizeY + 2400) {
      this.reset(random(sizeX, sizeX + 3000), random(sizeY, sizeY + 2400));
    }
  }

  void eat(float x, float y) {
    for (PVector p : pt) {
      if (p.x <= x + sizeSnake / 2 && p.x >= x - sizeSnake / 2 && p.y <= y + sizeSnake / 2 && p.y >= y - sizeSnake / 2) {
        snakelen++;
        p.x = random(sizeX, sizeX + 3000);
        p.y = random(sizeY, sizeY + 2400);
        rgb.add(new PVector(random(255), random(255), random(255)));
      }
    }
  }

  void moveBody(float dx, float dy) {
    for (PVector b : body) {
      b.x += dx;
      b.y += dy;
    }
  }

  boolean check(float x, float y) {
  for (int i = 0; i < body.size(); i++) {
    if (dist(x, y, body.get(i).x, body.get(i).y) < 25) {
      return true;
    }
  }
  return false;
}
}

Snake snake2 = new Snake(random(300, 1000), random(300, 1000));
Snake snake3 = new Snake(random(1500, 2000), random(0, 1000));
Snake snake4 = new Snake(random(2000, 3000), random(500, 800));
Snake snake5 = new Snake(random(100, 500), random(200, 800));

void hit() {
  if (snake2.check(x, y) || snake3.check(x, y) || snake4.check(x, y) || snake5.check(x, y)) {
    status=2;
  }

  if (x<=sizeX+sizeSnake/5 || y<=sizeY+sizeSnake/5 || x>=sizeX+3000-sizeSnake/5 || y>=sizeY+2400-sizeSnake/5) {
    status=2;
  }
}
```

## 遊戲失敗
```java
PImage img3;
void gameOver()
{
  gaming.stop();
  playgaming=false;
  if (playgameover==false) gameover.play();
  playgameover=true;
  img3 = loadImage("gameover.png");
  img3.resize(1000, 800);
  background(60);
  image(img3, 0, 0);
}
```

## 重新開始
```java
void reset() {
  pt.clear();
  body.clear();
  rgb.clear();
  snake2.reset(random(300, 1000), random(300, 1000));
  snake3.reset(random(1500, 2000), random(0, 1000));
  snake4.reset(random(2000, 3000), random(500, 800));
  snake5.reset(random(100, 500), random(200, 800));
  snakelen = 1;
  s = 1;
  snake = 0;
  speed = 5;
  mapSpeed = 1.5;
  sizeFood = 20;
  sizeSnake = 30;
  sizeX=-1000;
  sizeY=-800;
  x = 250;
  y = 250;
  dir = 0;
  status = 1;
  playstart=false;
  playwelcome=false;
  playeat=false;
  playgameover=false;
  playgaming=false;
  for (int i=0; i<1000; i++)
    pt.add(new PVector(random(sizeX, sizeX+3000), random(sizeY, sizeY+2400)));
  rgb.add(new PVector(random(255), random(255), random(255)));
}
```
