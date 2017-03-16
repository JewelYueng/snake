# snake
a Vue webPage for snake game

这是一款用vue.js实现的贪吃蛇页游。下面介绍一下实现时的思想。

PS：逻辑运算都放在script标签内

# -设计数据结构
- **Step 1：**根据贪吃蛇的运行情况，大致分成三个数据结构，分别是Fruit（随机出现的水果）、Snake（蛇）、Game（负责管理游戏的运行状态）
- **Step 2：**细化Fruit和Snake数据结构内部。

```javascript
	class Fruit {
        constructor(x, y){
            // 确定水果的坐标，也可添加别的参数，例如水果的分值，不设置就默认为1
            this.x = x;
            this.y = y;
        }
    }
    class Snake {

        constructor(sHead){
            // 蛇身的坐标数组
            let point = {x: 0,y: 0};
            let point_array = [];
            for(let i = 0; i < 3 ; i++ ){
                point = {x : sHead.x , y : sHead.y + i};
                point_array[i] = point;
            }
            this.point_array = point_array;
        }

        length(){
        	//获得蛇的长度
            return this.point_array.length;
        }
		//蛇吃了水果以后长大的实现函数
        growUp(direction){
            let newHead = {x : this.point_array[0].x + direction.x , y : this.point_array[0].y + direction.y};
            this.point_array = [newHead].concat(this.point_array);
        }
		//蛇往前走时，把蛇尾的小方格删掉，在蛇头多添一个小方格
        goAhead(direction){
            let newHead = {x : this.point_array[0].x + direction.x , y : this.point_array[0].y + direction.y};
            this.point_array = [newHead].concat(this.point_array.slice(0,-1));
        }
		//获取蛇身坐标点的数组
        getPointArray(){
            return this.point_array;
        }
    }
```
- **Step 3：**根据游戏运行状态，细化Game类

```javascript
//每一局游戏都有一条蛇，每个时间点存在的唯一的水果，以及蛇目前的方向，以及游戏盘的大小
class Game{
        constructor(snake,dir,fruit,width,height){
            this.snake = snake;
            this.direction = dir;
            this.fruit = fruit;
            this.width = width;
            this.height = height;
        }
		//nextStep()是一层很关键的抽象
        nextStep(){
            const n = this.check();
            switch (n) {
                case 1: return this.gameAhead();
                case 2: return this.growUp();
                case 3: return this.die();
            }

        }

        setDirection(dir){
            this.direction = dir;
        }

        setFruit(){
            let fruitX = parseInt(Math.random() * this.width);
            let fruitY = parseInt(Math.random() * this.height);
            this.fruit.x = fruitX;
            this.fruit.y = fruitY;
        }


        gameAhead(){
            this.snake.goAhead(this.direction);
            return {alive: true, map: this.draw(), length: this.snake.length()};
        }

        growUp(){
            this.snake.growUp(this.direction);
            return {alive: true, map: this.draw(), length: this.snake.length()};
        }

        die(){
            return {alive: false, map: this.draw(), length: this.snake.length()};
        }

        draw(){
//            Initialize the board
            let board = [];
            let isEmpty = true;
            for (let i = 0;i < 30;i++ ){
                let boardLine = [];
                board.push(boardLine);
                for (let j = 0;j < 30;j++){
                    boardLine.push(0);
                }
            }
            for(let y = 0;y < this.width;y++){
                for(let x = 0;x < this.height;x++){
                    isEmpty = true;
                    for(let k = 0; k < this.snake.length();k++){
                        if(y == this.snake.point_array[0].y && x == this.snake.point_array[0].x){
                            board[y][x] = 1;
                            isEmpty = false;true
                            continue;
                        }
                        if(y == this.snake.point_array[k].y && x == this.snake.point_array[k].x){
                            board[y][x] = 2;
                            isEmpty = false;
                        }
                        if(y == this.fruit.y && x == this.fruit.x){
                            board[y][x] = 3;
                            isEmpty = false;
                        }
                    }
                    if (isEmpty)
                        board[y][x] = 0;
                }
            }

            return board;
        }
        check(){
            if(!this.checkSnake()) return 3;
            else {
                if(this.eatFruit()){
                    do {
                        this.setFruit();
                    }while(this.checkFruit());
                    return 2;
                }

                return 1;
            }
        }

        checkFruit(){
            for(let j = 0 ; j < this.snake.length() ; j++){
                if(this.fruit.x == this.snake.point_array[j].x && this.fruit.y == this.snake.point_array[j].y){
                    this.setFruit();
                    return false;
                }
            }
            return true;
        }
        checkSnake(){
            let head = this.snake.point_array[0];
            let newHead = {x : head.x + this.direction.x , y : head.y + this.direction.y};
            let point_array = this.snake.point_array;
            point_array = [newHead].concat(point_array.slice(0,-1));
//            touch the border
            if(newHead.x >= this.width || newHead.x < 0 || newHead.y < 0 || newHead.y >= this.height) return false;
//            kill itself
            for (let i = 1;i < this.snake.length();i++){
                if(point_array[i].x == newHead.x && point_array[i].y == newHead.y)
                    return false;
            }
            return true;
        }

        eatFruit(){
            return (this.snake.point_array[0].x == this.fruit.x && this.snake.point_array[0].y == this.fruit.y);
        }

    }
```

- **Step 4：**初始化你的游戏状态
	
	```javascript
	function GameInit() {
        let width = 30;
        let height = 30;
        let snakeHead = {x : 14 , y : 14};
        let fruit = {x:15,y:15};
        let snake = new Snake(snakeHead);
        let newGame = new Game(snake,UP,fruit,width,height);
        return newGame;
    }
	```	
