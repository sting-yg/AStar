<template>
    <div>
        <svg :width="width" :height="height">
            <g v-for="(row, rowIndex) in grid" :key="rowIndex">
                <g v-for="(cell, colIndex) in row" :key="colIndex">
                    <rect
                        :x="colIndex * cellSize"
                        :y="rowIndex * cellSize"
                        :width="cellSize"
                        :height="cellSize"
                        :fill="getColor(cell)"
                        stroke="#000"
                        stroke-width="1"
                        @click.left="getLeftClicked(rowIndex, colIndex)"
                    />                    
                </g>
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
            cols: 15,
            rows: 15,
            cellSize: 40,    
            wall: 30,    
            start: null,
            end: null,      
            path: []  
        }
    },
    created(){
        document.oncontextmenu = (e) =>{
                e.preventDefault();
            }
        this.initializeSvg();
    },

    methods:{
        initializeSvg(){       
            for(let i = 0; i < this.rows; i++){
                let row = [];
                for(let j = 0; j <this.cols; j++){
                    row.push({isPath:false, isWall: false, isClicked: false, isStart: false, isEnd: false, F: 0, H: 0, isParent: false, parentPoint: null, G: Math.random().toFixed(1)},);    
                }
                this.grid.push(row);
            }          

            for(let i = 0; i <this.wall; i++){
                let randomRow , randomCol
                do{
                    randomRow = Math.floor(Math.random() * this.rows)
                    randomCol = Math.floor(Math.random() * this.cols)
                }while(this.grid[randomRow][randomCol].weight > 9999)
                this.grid[randomRow][randomCol].weight = 10000;
            }
        },
        getColor(cell){

            const weight = cell.weight
            if (cell.isPath) {
                return 'white'; // 路径用白色显示
            }
            else
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
                    return "FF0000"
                }
            }
        },

        getLeftClicked(row, col)
        {          
            console.log("getLeftClicked", row, col, this.start, this.end); 
            if(this.start == null)
            {
                this.grid[row][col].isStart = true;
                this.start = {row, col};
            }
            else
            {
                if(this.start.row == row && this.start.col == col) {
                    this.start = null;
                    this.grid[row,col].isStart = false;
                }
                else {
                    if(this.end == null) {
                        this.end = {row, col};
                        this.grid[row,col].isEnd = true;
                    }
                    else if(this.end.row == row && this.end.col == col) {
                        this.end = null;
                        this.grid[row,col].isEnd = false;
                    }
                }
            }
                
        },

        getFindPath(){

            const openList = [];
            const closeList = [];
            const result = [];          
            var result_index;

            // early exit
            if(this.start == null || this.end == null)
            {
                alert("no start or end point");
                return;
            }

            console.log('start', this.start);
            console.log('end', this.end);

            openList.push(grid[start.row][start.col]); // start point  openList        

            console.log('openList', openList);

            while(openList.length > 0)
            {
                const openPop = openList.pop(); // openList 에서 삭제  
                console.log('openPop', openPop);
                openPop.isParent = true;
                closeList.push(openPop); // closeList 에 푸시
                console.log('closeList', closeList);
                const surruoundPoint = this.getSurroundPoint(openPop);
                console.log("surroundPoint", surruoundPoint)

                for (var i in surruoundPoint){

                    var item  =  surruoundPoint[i];
                    console.log("item" , item)
                    console.log("surroundPoint[i]", surruoundPoint[i])

                    if  (item.x >=0 && 
                        item.y >= 0 &&
                        item.x < this.cols &&
                        item.y < this.rows &&
                        !this.grid[item.x][item.y].isWall &&
                        !this.existList(item, closeList)
                        )
                    {
                        var g = openPop.G + 10
                        if (!this.existList(item, openList)) {       
                            
                            item.H = Math.abs(end_x - item.x) * 10 + Math.abs(end_y - item.y) * 10;
                            console.log("item.H" , item)
                            item.G = g;
                            console.log("item.G" , item)  
                            item.F = item.H + item.G;
                            item.isParent = openPop;
                            openList.push(item);
                        }
                        else {                                  
                            var index = this.existList(item, openList);                           
                            if (g < openList[index].G) {
                                openList[index].isParent = openPop;
                                openList[index].G = g;
                                openList[index].F=g+openList[index].H;

                                console.log("openList[index].G" , openList);
                                console.log("openList[index].isParent" , openList); 
                            }
                        }
                    }
                }
                //如果开启列表空了，没有通路，结果为空
                if(openList.length==0) {
                    break;
                }
                openList.sort(this.sortF);//这一步是为了循环回去的时候，找出 F 值最小的, 将它从 "开启列表" 中移掉
                console.log("openList: " + openList);
            }while(!(result_index=this.existList({x:end_x,y:end_y},openList)));
            //判断结果列表是否为空
            if(!result_index) {
                result=[];
            }
            else {
                var currentObj=openList[result_index];
                do{
                    //把路劲节点添加到result当中
                    result.unshift({
                        x:currentObj.x,
                        y:currentObj.y
                    });
                    currentObj=currentObj.parent;
                }while (currentObj.x!=start_x || currentObj.y!=start_y);

            }
            return result;

        },
        sortF(a,b){
            return b.F- a.F;
        },

        getSurroundPoint(openPop){
            var x = openPop.x;
            var y = openPop.y;

            return [            
                {x:x+1,y:y},   
                {x:x,y:y+1},
                {x:x-1,y:y},
                {x:x,y:y-1},
            ]
        },

        existList(point, list){
            console.log("point", point);
            console.log("list", list);
            for(var i in list) {
                if(point.x==list[i].x && point.y==list[i].y) {
                    return i;
                }
            }
            return false;
        }
    }
}

</script>

<style>
text {
font-size: 20px;
}
</style>