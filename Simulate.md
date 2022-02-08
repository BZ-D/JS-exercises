#### 1、照明网格

**哈希表+模拟**

![image-20220208164434045](assets/image-20220208164434045.png)

![image-20220208164459104](assets/image-20220208164459104.png)

```js
var gridIllumination = function(n, lamps, queries) {
    // xi => 行，用xi唯一标识被照亮的行
    // yi => 列，用yi唯一标识被照亮的列
    // xi - yi => 正对角线，用xi-yi唯一标识被照亮的正对角线
    // xi + yi => 反对角线，用xi+yi唯一标识被照亮的反对角线
    
    // 用哈希表标记当前行/列/正反对角线上的灯的数目
    let rows = new Map(),
        cols = new Map(),
        dj1 = new Map(),
        dj2 = new Map();
    // 集合用来存放有灯的点
    let points = new Set();

    for (let lamp of lamps) {
        const x = lamp[0], y = lamp[1];
        if (points.has(point2Str(x, y))) {
            // 若出现重复的灯，不予处理
            continue;
        }
		// 加入灯的坐标，并在行、列、正反对角线对应的映射灯数加一
        points.add(point2Str(x, y));
        rows.set(x, (rows.get(x) || 0) + 1);
        cols.set(y, (cols.get(y) || 0) + 1);
        dj1.set(x - y, (dj1.get(x - y) || 0) + 1);
        dj2.set(x + y, (dj2.get(x + y) || 0) + 1);
    }

    let ans = new Array(queries.length).fill(0);
    // 用于某次查询后，对九宫格内的灯进行熄灭操作
    const dirs = [ [0,0], [1,0], [-1,0], [0,1], [0,-1], [1,1], [-1,1], [1,-1], [-1,-1] ];
    for (let [i, [x, y]] of queries.entries()) {
        if (rows.get(x) || cols.get(y) || dj1.get(x - y) || dj2.get(x + y)) {
            // 若当前点已被照亮，设置为1
            ans[i] = 1;
        }
        for (let dir of dirs) {
            // 九个方向（包括自身点）进行灭灯
            const nx = x + dir[0], ny = y + dir[1];
            if (nx >= 0 && nx < n && ny >= 0 && ny < n) {
                if (points.has(point2Str(nx, ny))) {
                    points.delete(point2Str(nx, ny));
                    rows.set(nx, rows.get(nx) - 1);
                    cols.set(ny, cols.get(ny) - 1);
                    dj1.set(nx - ny, dj1.get(nx - ny) - 1);
                    dj2.set(nx + ny, dj2.get(nx + ny) - 1);
                }
            }
        }
    }

    return ans;
};

function point2Str(x, y) {
    return x + ',' + y;
}
```

