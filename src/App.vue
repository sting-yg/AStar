<template>
    <div>
        <svg :width="width" :height="height">
            <g v-for="(cell, rowIndex) in grid" :key="rowIndex">
                <rect
                    :x="cell.col * cellSize"
                    :y="cell.row * cellSize"
                    :width="cellSize"
                    :height="cellSize"
                    :fill="getColor(cell)"
                    stroke="#000"
                    stroke-width="1"
                    @click.left="getLeftClicked(cell.row, cell.col)"
                />
            </g>
            <text
                v-if="start != null"
                :x="start.col* cellSize + cellSize / 2"
                :y="start.row * cellSize + cellSize / 2"
                text-anchor="middle"
                dominant-baseline="middle"
                fill="black"
                @click.left="getLeftClicked(start.row, start.col)"
            >
                {{ 'start' }}
            </text>
            <text
                v-if="end != null"
                :x="end.col * cellSize + cellSize / 2"
                :y="end.row * cellSize + cellSize / 2"
                text-anchor="middle"
                dominant-baseline="middle"
                fill="black"
                @click.left="getLeftClicked(end.row, end.col)"
            >
                {{ 'end' }}
            </text>
            <g v-for="(p, index) in path" :key="index">
                <line :x1="p.x1" :y1="p.y1" :x2="p.x2" :y2="p.y2" stroke="blue" stroke-width="5" />
            </g>
        </svg>     
    </div>
    <div>
        <button @click="getFindPath(start_x,start_y,end_x,end_y)" style="background-color: aquamarine;">FindPath</button>
    </div>
</template>
<script>

export default{
  
    data(){
        return {
            grid: [],
            width: 1000,
            height: 720,
            cols: 20,
            rows: 20,
            cellSize: 40,    
            wall: 100,    
            start: null,
            end: null,      
            path: []  
        }
    },
    created(){
        document.oncontextmenu = (e) =>{
                e.preventDefault();
            }
        this.initializeGrid();
    },

    methods:{
        initializeGrid(){       
            for(let i = 0; i < this.rows; i++){
                for(let j = 0; j <this.cols; j++){
                    this.grid.push({row: i, col: j, closed: false, start: false, end: false, h: 0, g:0, parent: null, weight: Math.random()},);    
                }
            }          

            for(let i = 0; i <this.wall; i++){
                let randomRow , randomCol
                do{
                    randomRow = Math.floor(Math.random() * this.rows)
                    randomCol = Math.floor(Math.random() * this.cols)
                }while(this.grid[randomRow * this.cols + randomCol].weight > 99)
                this.grid[randomRow * this.cols + randomCol].weight = 100;
            }
        },

        getCell(row, col) {
            if(row < 0 || row >= this.rows) return null;
            if(col < 0 || col >= this.cols) return null;
            
            const index = row * this.cols + col;
            if(index < 0  || index >= this.grid.length) {
                return null;
            }
            return this.grid[index];
        },

        getColor(cell)
        {          
            if(cell.weight < 0.2){
                return "#BFEFFF"
            }
            else if(cell.weight < 0.4){
                return "#ADD8E6"        
            }
            else if(cell.weight < 0.6){
                return "#87CEEB"
            }
            else if(cell.weight < 0.8){
                return "#87CEFA"
            }
            else if(cell.weight < 1.0){
                return "#00BFFF"
            }
            else {
                return "#FF0000"
            }
        },

        getLeftClicked(row, col)
        {          
            console.log("getLeftClicked", row, col, this.start, this.end); 
            const cell = this.getCell(row, col);
            if(this.start == null)
            {
                cell.start = true;
                this.start = cell;
            }
            else
            {
                if(this.start  == cell) {
                    cell.start = false;
                    this.start = null;
                }
                else {
                    if(this.end != null) {
                        this.end.end = false;
                    }

                    cell.end = true;
                    this.end = cell;
                    this.getFindPath();
                }
            }
                
        },

        getSurroundPoint(cell)
        {
            return [
                this.getCell(cell.row-1, cell.col),
                this.getCell(cell.row+1, cell.col),
                this.getCell(cell.row, cell.col-1),
                this.getCell(cell.row, cell.col+1)].filter(x => x != null).filter(x => x.closed == false);
        },

        getFindPath()
        {
            if(this.start == null || this.end == null)
            {
                alert("no start or end point");
                return;
            }
            console.log('start', this.start);
            console.log('end', this.end);

            this.path = [];
            for(let i=0; i<this.grid.length; i++) {
                this.grid[i].g = 10000;
                this.grid[i].closed = false;
                this.grid[i].parent = null;
                this.grid[i].h = this.calcH(this.grid[i]);
            }

            this.start.g = 0;
            const openList = [this.start];
            while(openList.length > 0)
            {
                openList.sort(this.sortF);//这一步是为了循环回去的时候，找出 F 值最小的, 将它从 "开启列表" 中移掉
                const current = openList.pop(); // openList 에서 삭제  
                current.closed = true;                
                if(current == this.end) {
                    break;
                }

                const surruoundPoints = this.getSurroundPoint(current);
                for(let i=0; i<surruoundPoints.length; i++)
                {
                    const neighbor = surruoundPoints[i];
                    const tempg = current.g + neighbor.weight;
                    if (tempg < neighbor.g ) {
                        neighbor.g = tempg;
                        neighbor.parent = current;

                        if(openList.indexOf(neighbor) < 0) {
                            openList.push(neighbor);
                        }
                    }
                }
            }

            const result = [];
            result.push(this.end);
            while(result[result.length-1].parent != null) {
                result.push(result[result.length-1].parent);
            }
            result.reverse();

            console.log("result", result);
            for(let i=0; i<result.length-1; i++) {
                this.path.push({
                    x1: (result[i].col + 0.5) * this.cellSize,
                    y1: (result[i].row + 0.5) * this.cellSize,
                    x2: (result[i+1].col + 0.5) * this.cellSize,
                    y2: (result[i+1].row + 0.5) * this.cellSize,
                });
            }
        },

        sortF(a,b) {
            return (b.g + b.h) - (a.g + a.h);
        },

        calcH(cell) {
            return 0;
            //return Math.abs(this.end.row - point.row) + Math.abs(this.end.col - point.col)/this.cols;
        }
    }
}

</script>

<style>
text {
font-size: 20px;
}
</style>