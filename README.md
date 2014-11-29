adjustablejs
============



/*This module is for a set projects related to AdjustableShape
 *adjustableShape.js v1.2
 *Developed by Lapiz
 *
 *this version is modified for this activity 
 *
 * Options:
 *  -   shapeData* - Array
 *      struct > {drag:true, lengths:[], midPoint:{x:X,y:Y}, pointOfRotation:{x:X,y:Y} angles:[], points:[], angle:Angle, label:["A", "B", "C"]}
 *  -   isDrawGrid - boolean, to draw grid
 *  -   gridProp - to draw background grid
 *          minX, minY, maxX, maxY, xInt, yInt
 *
 *
 * Operation Modes
 *  -   translate
 *  -   rotate
 *  -   reflect
 *  -   vertexControl
 *  -   createShape
 *
 *  -   select
 *  -   drawCircle
 *  -   plotPoint
 *  -   drawLine
 *  -   dragLnP
 *
 *  
 *
*/
var VertexPoints;
window.AdjustableShape = function(canvasHolder, historyContainer, options) {
    this.options = options ? options : {};
    this.shapeData = this.options.shapeData ? this.options.shapeData : [];
    this.plotterData = this.options.plotterData ? this.options.plotterData : {lines:[], points:[], circles:[], relation:[]};
    this.staticPlotters = this.options.staticPlotters ? this.options.staticPlotters : {lines:[], points:[], circles:[], relation:[]};
    this.history = [];
    this.canvasHolder = canvasHolder;
    this.historyContainer = historyContainer;
    
    //Private vars
    this.relationTemplate = JSON.stringify({pId:-2, lIds:[], ends:[]});
    this.canvasSet = [];
    this.contextSet = [];
    this.operation = "";
    this.touchedShapeIndex = -1;
    this.touchedObjectIndex = -1;
    this.touchedObjectProp = {};
    this.selectedItemType = "";
    this.selectedObjectIndex = -1;
    this.selectedShapeIndex = -1;
    this.tempData = {};
    this.downPosition = null;
    this.canvasArea = {x:0, y:0};
    this.gridLocked = true;
    this.touchedObject = "";
    this.lineOfReflection;
    this.historyScroller;
    this.historyIndex = [];
    this.initShapeData = [];
    this.maxHistoryCount = 20;
    this.orginalLineOfRef = null;
    this.baseAngle = 180;
    this.baseSideLen = 3;
    this.drawingShape = [];
    this.tempGraphDetails = null;
    this.gridPropCopy;
    this.plotterDataCopy;
    this.xOperation = "";
    
    if (this.shapeData) {
        this.options.defaultFontSize = this.options.defaultFontSize ? this.options.defaultFontSize : 12;
        this.options.defaultFontFamily = this.options.defaultFontFamily ? this.options.defaultFontFamily : "arial";
        this.options.contextProp = this.options.contextProp ? this.options.contextProp :
            {shapes:{strokeStyle:"#000", fillStyle:"rgba(153, 204, 255, 0.8)", lineWidth:2, lineJoin:"round", lineCap:"round" },
            shapeVertex:{strokeStyle:"#000", fillStyle:"rgba(255, 0, 0, 0.6)"},
            vertexControl:{strokeStyle:"transparent", fillStyle:"blue"},
            drawingShape:{strokeStyle:"rgba(0,200,0,1)", fillStyle:"rgba(0,200,0,0.2)", lineWidth:2, lineJoin:"round", lineCap:"round"},
            drawingVertex:{strokeStyle:"transparent", fillStyle:"rgba(0,200,0,1)"},
            selectedDrawingVertex:{strokeStyle:"transparent", fillStyle:"rgba(0,100,0,1)"},
            shapeLabels:{fillStyle:"#000", textAlign:"center", textBaseline:"middle", font:"14px arial"},
            selectedShape:{shadowColor:"rgba(255, 0, 0, 0.6)" },
            pointOfRot:{strokeStyle:"transparent", fillStyle:"rgba(0,0,0,0.3)"},
            pointOfRotLine:{strokeStyle:"#000", setLineDash:[2,4]},
            grid:{strokeStyle:"rgba(0,0,0,0.25)", lineWidth:1},
            gridAxis:{strokeStyle:"#000", fillStyle:"#000", lineWidth:2},
            lineOfReflection:{strokeStyle:"rgba(0,0,255,0.5)", lineWidth:2},
            pointsOfLOR:{strokeStyle:"transparent", fillStyle:"#8080ff"},
            plotterLine:{strokeStyle:"red", lineWidth:2},
            plotterPoint:{strokeStyle:"transparent", fillStyle:"rgba(200,0,0,1)"},
            plotterCircle:{strokeStyle:"rgba(200,0,0,1)", fillStyle:"rgba(200,0,0,0.2)"},
            plotterLabel:{font:this.options.defaultFontSize+"px "+this.options.defaultFontFamily, textAlign:"center", textBaseline:"middle", fillStyle:"#000"},
            /*selectedPlotterLine:{shadowColor:"rgba(0, 0, 0, 1)", shadowBlur:10},*/
            selectedPlotterPoint:{shadowColor:"rgba(0, 0, 0, 1)", shadowBlur:10},
            selectedPlotterCircle:{shadowColor:"rgba(0, 0, 0, 1)", shadowBlur:10}
        };
        if (this.options.gridProp && this.options.isDrawGrid != false) this.options.isDrawGrid = true;
        this.options.isDrawGrid = this.options.isDrawGrid ? true : false;
        this.options.isVarPointOfRot = this.options.isVarPointOfRot == false ? false : true;
        this.options.isResetPOR = this.options.isResetPOR == false ? false : true;
        this.options.lineTouchSense = this.options.lineTouchSense ? this.options.lineTouchSense : 15;
        this.options.plotPointRad = this.options.plotPointRad ? this.options.plotPointRad : 7;
        this.options.pointOfRotRad = this.options.pointOfRotRad ? this.options.pointOfRotRad : 7;
        this.options.pointOfLORRad = this.options.pointOfLORRad ? this.options.pointOfLORRad : 7;
        this.options.drawingPointRad = this.options.drawingPointRad ? this.options.drawingPointRad : 7;
        this.options.vertexRad = this.options.vertexRad ? this.options.vertexRad : 7;
        this.options.vertexControlRad = this.options.vertexControlRad ? this.options.vertexControlRad : 7;
        this.options.isVisibleVertex = this.options.isVisibleVertex == false ? false : true;
        this.options.snapEveryValue = isNaN(this.options.snapEveryValue) ? 0 : this.options.snapEveryValue;
        this.options.snapTolerence = isNaN(this.options.snapTolerence) ? 0 : this.options.snapTolerence;
        this.options.linePointSnapVal = this.options.linePointSnapVal ? this.options.linePointSnapVal : 0.4;
        this.options.gridProp = this.options.gridProp ? this.options.gridProp : {minX:-10, maxX:10, minY:-10, maxY:10, xInterval:1, yInterval:1, tickLength: 10, tick:{isActive:true, xMin:-10, xMax:10, yMin:-10, yMax:10, xInt:1, yInt:1}, ratio:1};
        this.options.margins = this.options.margins ? this.options.margins : {left: 10, top: 10, bottom: 5, right: 10};
        this.options.lblDistFrmShape = this.options.lblDistFrmShape ? this.options.lblDistFrmShape : 16;
        this.options.minTicks = this.options.minTicks ? this.options.minTicks : {x: 4, y:4};
        this.options.maxTicks = this.options.maxTicks ? this.options.maxTicks : {x: 20, y:20};
        this.options.pointLabelOffset = this.options.pointLabelOffset ? this.options.pointLabelOffset : {x: 0, y: 20};
        this.gridPropCopy = JSON.parse(JSON.stringify(this.options.gridProp));
        this.plotterDataCopy = JSON.parse(JSON.stringify(this.plotterData));
        this.axisSkipData = this.options.axisSkipData ? this.options.axisSkipData : {level1:16, level2:20};
        this.axisCountCheck = this.options.axisCountCheck ? this.options.axisCountCheck : 10;
        this.init();
    }
};

AdjustableShape.prototype={
    init:function(){
        var canvasString = "<canvas style=\"position:absolute; top:0px; left:0px; width:100%; height:100%;\" />";
        for(var c=0; c<2; c++){
            var canvas = $(canvasString);
            this.canvasHolder.append(canvas);
            this.canvasSet.push(canvas);
            var context = canvas[0].getContext("2d");
            this.contextSet.push(context);
            if (!context.setLineDash) context.setLineDash = function () {}
            this.canvasArea = {x:this.canvasHolder.width(), y:this.canvasHolder.height()};
            this.setHiDPICanvas(canvas[0], this.canvasArea.x, this.canvasArea.y);
        }
        var _this = this;
        if (this.isTouchDevice()) {
            this.canvasHolder.bind("touchstart", function(event){_this.onStart(event);});
            $(window).bind("touchmove", function(event){_this.onMove(event);});
            $(window).bind("touchend", function(event){_this.onEnd(event);});
        }
        else {
            this.canvasHolder.bind("mousedown", function(event){_this.onStart(event);});
            $(window).bind("mousemove", function(event){_this.onMove(event);});
            $(window).bind("mouseup", function(event){_this.onEnd(event);});
        }
        
        var graphDetails = this.graphDetails();
        
        var LoRPointX = this.pointToX(0, graphDetails, 0, false);
        var LoRPointY_1 = this.pointToY(8, graphDetails, 0, false);
        var LoRPointY_2 = this.pointToY(-8, graphDetails, 0, false);
        
        this.lineOfReflection = {point1:{x:LoRPointX, y:LoRPointY_1}, point2:{x:LoRPointX, y:LoRPointY_2}, midPoint:{x:LoRPointX, y:(LoRPointY_1+LoRPointY_2)/2}};
        this.orginalLineOfRef = JSON.parse(JSON.stringify(this.lineOfReflection));
        if (this.historyContainer) {
            this.historyScroller = $("<div class=\"historyScroller\"></div>");
            this.historyScroller.css({"overflow-y":"scroll","height":"100%","max-height":"100%","width":"100%"});
            this.historyContainer.append(this.historyScroller);
        }
        this.initShapeData = JSON.parse(JSON.stringify(this.shapeData));
        for(var i=0; i<this.initShapeData.length; i++)
            this.historyIndex[i] = -1;
        this.drawAll(true);
    },
    addShape:function(data) {
        var shapeData = {};
        var isGridBasedValues = (data.coords != null);
        var isValues = (data.points != null);
        shapeData.drag = data.drag==false?false:true;
        if (isValues || isGridBasedValues) {
            shapeData.points = [];
            shapeData.lengths = [];
            if (isGridBasedValues) {
                var graphDetails = this.graphDetails();
                for(var c=0; c<data.coords.length; c++){
                    var coord = data.coords[c];
                    var x = coord.x;
                    var y = coord.y;
                    var point = this.pointToPos(x, y, graphDetails, 0, false);
                    shapeData.points.push(point);
                }
            }
            else if(isValues) shapeData.points = data.points;
            shapeData.midPoint = this.midPoint(shapeData.points);
            shapeData.pointOfRotation = JSON.parse(JSON.stringify(shapeData.midPoint));
            shapeData.angles = [];
            this.updateAngles(shapeData);
            shapeData.angle = this.baseAngle;
            shapeData.angleOfRotation = 0;
            shapeData.distFrmCP = 0;
            shapeData.label = data.label ? data.label : [];
            shapeData.PORLabel = data.PORLabel ? data.PORLabel : null;
            shapeData.customProp = data.customProp ? data.customProp : null;
            this.shapeData.push(shapeData);
            if (!this.initShapeData) this.initShapeData = [];
            this.initShapeData[this.shapeData.length-1] = JSON.parse(JSON.stringify(shapeData));
            this.historyIndex[this.shapeData.length-1] = -1;
            this.drawAll(true);
        }
    },
    addDrawingAsShape:function(){
        if (this.drawingShape.length >= 3) {
           
            var points = JSON.parse(JSON.stringify(this.drawingShape));
            this.drawingShape = [];
            this.addShape({points:points});
            this.triggerChange();
        }
    },
    updateAngles:function(shapeData){
        var angle = 0;
        var cumAngles = 0;
        for(var p=0; p<shapeData.points.length; p++){
            var _relp = p+1;
            var relP = this.correctIndex(_relp, shapeData.points.length);
            shapeData.lengths[p] = this.distance(shapeData.points[p], shapeData.points[relP]);
            var _ang;
            if (_relp != relP) _ang = this.angleBetweenPoints(shapeData.points[relP], shapeData.points[p])*180/Math.PI;
            else _ang = this.angleBetweenPoints(shapeData.points[p], shapeData.points[relP])*180/Math.PI;
            angle = this.correctAngle(_ang-cumAngles);
            shapeData.angles[p] = angle;
            cumAngles += angle;
        }
    },
    correctAngle:function(angle){
        if (angle > 360) angle = angle%360;
        while (angle < 0) angle = angle+360;
        return angle;
    },
    correctIndex:function(index, length){
        if (index >= length) index = index-length;
        if (index < 0) index = index+length;
        return index;
    },
    drawAll:function(isReCalculate, isOnlyCalculate, isMoving){
        if (this.options.isDrawGrid) this.drawGrid(true);
        this.drawShapes(true, isOnlyCalculate);
        this.formHistory();
        
    },
    formHistory:function(){
        if (this.historyScroller) {
            var history = "Select shape to view history.";
            if (this.historyScroller && this.selectedShapeIndex >= 0 && this.selectedItemType == "shape") {
                if (!this.history[this.selectedShapeIndex]) this.history[this.selectedShapeIndex]= [];
                history = this.historyItem(-1);
                for(var i=0; i<this.history[this.selectedShapeIndex].length; i++) history += this.historyItem(i);
            }
            this.historyScroller.html(history);
            var _this = this;
            this.historyScroller.find(".historyItem").bind("mousedown", function(event){
                var target = $(this);
                var value = parseInt(target.attr("value"));
                _this.loadHistory(value);
            });
        }
    },
    loadHistory:function(index){
        if (this.selectedShapeIndex >= 0 && this.selectedItemType == "shape") {
            var dataToLoad = null;
            if(index < 0) dataToLoad = this.initShapeData[this.selectedShapeIndex];
            else dataToLoad = this.history[this.selectedShapeIndex][index].shapeData;
            this.historyIndex[this.selectedShapeIndex] = index;
            if (dataToLoad)
                this.shapeData[this.selectedShapeIndex] = JSON.parse(JSON.stringify(dataToLoad));
            this.drawAll(true);
        }
    },
    historyItem:function(item) {
        var isCurrent = false;
        if(this.historyIndex[this.selectedShapeIndex] == item) isCurrent = true;
        var value = "<div class=\"historyItem"+(isCurrent?" current":"")+"\" value=\""+item+"\">";
        if (item < 0) value += "Inital state";
        else value += this.history[this.selectedShapeIndex][item].change.subject;
        value += "</div>";
        return value;
    },
    storeAsHistory:function(details){
        if (this.selectedShapeIndex >= 0 && this.selectedItemType == "shape") {
            if (!this.history[this.selectedShapeIndex]) this.history[this.selectedShapeIndex] = [];
            this.history[this.selectedShapeIndex].splice(this.historyIndex[this.selectedShapeIndex]+1);
            this.historyIndex[this.selectedShapeIndex] = this.history[this.selectedShapeIndex].length;
            this.history[this.selectedShapeIndex].push({shapeData:JSON.parse(JSON.stringify(this.shapeData[this.selectedShapeIndex])), change:details});
            var historyLength = this.history[this.selectedShapeIndex].length;
            if (historyLength > this.maxHistoryCount) {
                var extraHistory = historyLength-this.maxHistoryCount;
                this.history[this.selectedShapeIndex].splice(0, extraHistory);
                this.historyIndex[this.selectedShapeIndex] -= extraHistory;
            }
        }
    },
    drawGrid:function(forceDraw){
        var canvasIndex = 0;
        var graphDetails = this.graphDetails();
        var tickCount = graphDetails.count;
        var maxTickX = tickCount.x;
        var maxTickY = tickCount.y;
        this.totalWidth = Math.floor(graphDetails.tickSpace.x * maxTickX);
        this.totalHeight = Math.floor(graphDetails.tickSpace.y * maxTickY);
        var ctx = this.contextSet[canvasIndex];
        var graphW = this.totalWidth;
        var graphH = this.totalHeight;
        ctx.clearRect(0, 0, this.canvasSet[canvasIndex].width(), this.canvasSet[canvasIndex].height());
        var cntxtProp = this.options.contextProp.grid;
        ctx.save();
        this.applycontextProp(ctx, cntxtProp);
        var xValue = graphDetails.data.minX;
        var yValue = graphDetails.data.minY;
        var adjThickness = -0.5;
        var left = ctx.lineWidth + this.options.margins.left + adjThickness;
        var right = graphW + this.options.margins.left + adjThickness;
        var top = ctx.lineWidth + this.options.margins.top + adjThickness;
        var bottom = graphH + this.options.margins.top + adjThickness;
        var xAxisYPos = this.pointToY(0, graphDetails, 0, false);
        var yAxisXPos = this.pointToX(0, graphDetails, 0, false);
        var tickLength = graphDetails.data.tickLength;
        
        var xAxisYMinPos, xAxisYMaxPos, yAxisXMinPos, yAxisXMaxPos;
        xAxisYMinPos = this.pointToY(graphDetails.data.maxY, graphDetails, adjThickness, true);
        xAxisYMaxPos = this.pointToY(graphDetails.data.minY, graphDetails, adjThickness, true);
        yAxisXMinPos = this.pointToX(graphDetails.data.minX, graphDetails, adjThickness, true);
        yAxisXMaxPos = this.pointToX(graphDetails.data.maxX, graphDetails, adjThickness, true);
        
        if (!this.gridLocked || forceDraw) {
            var tickData = graphDetails.data.tick;
            if (tickData.isActive) {
                tickData.xMin = tickData.xInt * graphDetails.data.minX;
                tickData.xMax = tickData.xInt * graphDetails.data.maxX;
                tickData.yMin = tickData.yInt * graphDetails.data.minY;
                tickData.yMax = tickData.yInt * graphDetails.data.maxY;
            }
            var sampleValue1 = this.pointToPos(0, 0, graphDetails, 0, false);
            var sampleValue2 = this.pointToPos(1, 1, graphDetails, 0, false);
            var xVariation = Math.abs(sampleValue1.x - sampleValue2.x);
            var yVariation = Math.abs(sampleValue1.y - sampleValue2.y);
            var axisThickness = 2;
            
            for (var r = 0; r <= maxTickY; r++) {
                var thickLine = false;
                //if (yValue == 0) {
                //    ctx.save();
                //    this.applycontextProp(ctx, this.options.contextProp.gridAxis);
                //    thickLine = true;
                //}
                var yPos = this.pointToY(yValue, graphDetails, adjThickness, thickLine);
                ctx.beginPath();
                var additional = ctx.lineWidth;
                ctx.moveTo(left - additional, yPos);
                ctx.lineTo(right + additional, yPos);
                ctx.stroke();
                if (graphDetails.data.tick.isActive && (yAxisXMinPos <= yAxisXPos && yAxisXMaxPos >= yAxisXPos)) {
                    ctx.save();
                   
                    ctx.beginPath();
                     ctx.setLineDash([5]);
                    this.applycontextProp(ctx, this.options.contextProp.gridAxis);
                    ctx.stroke();
                    ctx.moveTo(yAxisXPos - tickLength, yPos);
                    ctx.lineTo(yAxisXPos, yPos);
                    ctx.stroke();
                    ctx.restore();
                }
                if (thickLine) ctx.restore();
                yValue += graphDetails.data.yInterval;
            }
            for (var c = 0; c <= maxTickX; c++) {
                var thickLine = false;
                //if (xValue == 0) {
                //    ctx.save();
                //    this.applycontextProp(ctx, this.options.contextProp.gridAxis);
                //    thickLine = true;
                //}
                var additional = ctx.lineWidth;
                var xPos = this.pointToX(xValue, graphDetails, adjThickness, thickLine);
                ctx.beginPath();
                ctx.setLineDash([5]);
                ctx.moveTo(xPos, top - additional);
                ctx.lineTo(xPos, bottom + additional);
                ctx.stroke();
                if (graphDetails.data.tick.isActive && (xAxisYMinPos <= xAxisYPos && xAxisYMaxPos >= xAxisYPos)) {
                    ctx.save();
                    ctx.beginPath();
                   
                    this.applycontextProp(ctx, this.options.contextProp.gridAxis);
                    ctx.stroke();
                    ctx.moveTo(xPos, xAxisYPos);
                    ctx.lineTo(xPos, xAxisYPos + tickLength);
                    ctx.stroke();
                    ctx.restore();
                }
                if (thickLine) ctx.restore();
                xValue += graphDetails.data.xInterval;
            }
            if (graphDetails.data.tick.isActive) {
                var xGrayout = false, yGrayout = false, isPrintAllYLabels = false;
                var txtDrawYAxisXPos = yAxisXPos;
                var txtDrawXAxisYPos = xAxisYPos;
                if (txtDrawYAxisXPos < yAxisXMinPos) {
                    txtDrawYAxisXPos = yAxisXMinPos;
                    yGrayout = true;
                }
                else if (txtDrawYAxisXPos > yAxisXMaxPos) {
                    txtDrawYAxisXPos = yAxisXMaxPos + (this.options.withTickMargin.right + tickLength/2);
                    yGrayout = true;
                }
                if (txtDrawXAxisYPos < xAxisYMinPos) {
                    txtDrawXAxisYPos = xAxisYMinPos - (this.options.withTickMargin.top + tickLength/2);
                    xGrayout = true;
                }
                else if (txtDrawXAxisYPos > xAxisYMaxPos) {
                    txtDrawXAxisYPos = xAxisYMaxPos;
                    xGrayout = true;
                }
                
                if(graphDetails.data.minX > -2) isPrintAllYLabels = true;
                
                var grayoutOpacity = 0.5;
                ctx.restore();
                ctx.save();
                this.applycontextProp(ctx, this.options.contextProp.gridAxis);
                
                var isXSci = false, isYSci = false;
                for (var times = 0; times < 2; times++){
                    xValue = graphDetails.data.minX;
                    yValue = graphDetails.data.minY;
                    var isCheck = times == 0;
                    var res;
                    
                    ctx.globalAlpha = 1;
                    if (yGrayout) ctx.globalAlpha = grayoutOpacity;
                    
                    for (var r = 0; r <= maxTickY; r++) {
                        var yPos = this.pointToY(yValue, graphDetails, 0, 0);
                        res = this.drawNumber(ctx, graphDetails.data.tick.yInt * yValue, txtDrawYAxisXPos, yPos, "y", yValue, maxTickY, yVariation, isCheck, isYSci, !yGrayout, !isPrintAllYLabels);
                        if (isCheck) {
                            isYSci = res;
                            if (isYSci && isCheck) break;
                        }
                        yValue += graphDetails.data.yInterval;
                    }
                    
                    ctx.globalAlpha = 1;
                    if (xGrayout) ctx.globalAlpha = grayoutOpacity;
                    
                    for (var c = 0; c <= maxTickX; c++) {
                        var xPos = this.pointToX(xValue, graphDetails, 0, 0);
                        res = this.drawNumber(ctx, graphDetails.data.tick.xInt * xValue, xPos, txtDrawXAxisYPos, "x", xValue, maxTickX, xVariation, isCheck, isXSci, !xGrayout);
                        xValue += graphDetails.data.xInterval;
                        if (isCheck) {
                            isXSci = res;
                            if (isXSci && isCheck) break;
                        }
                    }
                }
                ctx.restore();
            }
        }
    },
    graphDetails: function () {
        var details = {};
        var data = this.options.gridProp;
        details.data = data;
        details.count = this.getTickCount();
        var width = this.canvasArea.x - this.options.margins.left - this.options.margins.right;
        var height = this.canvasArea.y - this.options.margins.top - this.options.margins.bottom;
        details.tickSpace = { x: (width - this.options.contextProp.grid.lineWidth * 2) / details.count.x, y: (height - this.options.contextProp.grid.lineWidth * 2) / details.count.y };
        return details;
    },
    getTickCount: function () {
        var counts = {};
        var data = this.options.gridProp;
        counts.x = data.maxX - data.minX;
        counts.y = data.maxY - data.minY;
        return counts;
    },
    drawNumber: function (ctx, number, xPos, yPos, axis, axisIndex, axisCount, tickDistance, initialCheck, isSci, originOverlapChance, overlapChance) {
        ctx.fillStyle = this.options.axisColor;
        var fSize = this.options.defaultFontSize;
        ctx.font = fSize + "px "+this.options.defaultFontFamily;
        var pow = 0;
        var buffer = 2;
        var normalPowLimit = 300;
        var numberStr = number.toString();
        var dotTxt = "\u00B7";
        var powTxt = "10";
        var powBufferWidth = ctx.measureText(dotTxt + powTxt).width;
        var powPowBufferWidth = ctx.measureText("8").width;
        var align = "left";
        var vAlign = "bottom";
        if (axis == "y") {
            align = "right";
            vAlign = "middle";
            xPos -= this.options.gridProp.tickLength * 1.2;
        }
        else if (axis == "x") {
            align = "center";
            vAlign = "top";
            yPos += this.options.gridProp.tickLength * 1.2;
            if (axisCount > this.axisSkipData.level2){
                if (Math.abs(axisIndex) % 2 == 1) return;
                tickDistance *= 2;
            }
        }
        
        //To OverRide Overlapping
        if (axis == "y") {
            if (axisIndex == 0 && originOverlapChance) return;
            if (axisCount > this.axisSkipData.level1 && axisIndex == -1 && overlapChance) return;
            if (axisCount > this.axisSkipData.level2 && axisIndex == -2 && overlapChance) return;
        }
        
        var size, width, retry = false;
        do {
            numberStr = this.powerFormat(number, pow);
            var toNum = parseInt(numberStr.replace(/\,/g, "").replace("-", ""));
            width = ctx.measureText(numberStr).width;
            var totalBufferW = buffer - (pow > 0 ? powBufferWidth + powPowBufferWidth : 0);
            if (toNum > Math.pow(10, 6) - 1 || (isSci && !retry && toNum != 0)) {
                pow++;
                retry = true;
            }
            else if (xPos - totalBufferW < width) {
                fSize--;
                ctx.font = fSize + "px "+this.options.defaultFontFamily;
                retry = true;
            }
            else if (pow > 0 && toNum.toString().length > 1) {
                pow++;
                retry = true;
            }
            else if (axis == "x" && tickDistance - totalBufferW < width && Math.abs(number) > Math.pow(10, 3) - 1) {
                fSize--;
                ctx.font = fSize + "px "+this.options.defaultFontFamily;
                retry = true;
            }
            else retry = false;
            if (pow > 1000) return;
        }
        while (retry);
        
        if (axisIndex == 0 && axis == "x" && originOverlapChance) {
            yPos -= this.options.gridProp.tickLength * 0.8;
            xPos -= width;
        }
        else if (axisIndex == 0) return;
        
        if (pow > 0) {
            if (numberStr == "1") numberStr = "";
            else numberStr += dotTxt;
            numberStr += powTxt;
            if (axis == "y")
                xPos -= powPowBufferWidth;
        }
        if (initialCheck) return (pow>0);
        else{
            this.fillTxtClrBg(ctx, numberStr, xPos, yPos, width + (pow > 0 ? powBufferWidth : 0), fSize, align, vAlign);
            var microFSize = (fSize * 0.75);
            if (pow > 0) {
                ctx.font = microFSize + "px "+this.options.defaultFontFamily;
                if (axis == "y")
                    this.fillTxtClrBg(ctx, pow, xPos, yPos, width, microFSize, "left", "bottom");
                else if (axis == "x")
                    this.fillTxtClrBg(ctx, pow, xPos + (width + powBufferWidth) / 2, yPos + microFSize, width, microFSize, "left", "bottom");
            }
        }
    },
    fillTxtClrBg: function (context, text, xPos, yPos, width, height, align, vAlign) {
        if (!align) align = "left";
        if (!vAlign) vAlign = "bottom";
        context.textAlign = align;
        context.textBaseline = vAlign;
        var x, y;
        width = width * 1;
        height = height * 1.25;
        if (align == "left") x = xPos;
        else if (align == "center") x = xPos - width / 2;
        else if (align == "right") x = xPos - width;
        if (vAlign == "top") y = yPos;
        else if (vAlign == "middle") y = yPos - height / 2;
        else if (vAlign == "bottom") y = yPos - height;
        context.clearRect(x, y, width, height);
        context.fillText(text, xPos, yPos);
    },
    roundTo:function(value, length){
        var _v = Math.pow(10,length);
        return Math.round(value*_v)/_v;
    },
    powerFormat: function (number, power) {
        if (!power) power = 0;
        var isNeg = number < 0;
        var numberStr = Math.abs(number) / Math.pow(10, power);
        numberStr = Math.round(numberStr*10)/10;
        numberStr = numberStr.toString();
        if (numberStr.length > 3) {
            var commaPos = [];
            for (var i = 0; i < numberStr.length; i++) {
                if (i % 3 == 0 && i > 2) commaPos.push(numberStr.length - i);
            }
            var addPos = 0;
            for (i = commaPos.length - 1; i >= 0; i--) {
                var actPos = commaPos[i];
                numberStr = numberStr.splice(actPos + addPos, 0, ",");
                addPos++;
            }
        }
        if (isNeg) numberStr = "-" + numberStr;
        return numberStr;
    },
    pointToPos: function (x, y, graphDetails, adjThickness, thickLine) {
        var xPos = 0, yPos = 0;
        return { x: this.pointToX(x, graphDetails, adjThickness, thickLine), y: this.pointToY(y, graphDetails, adjThickness, thickLine) };
    },
    pointToX: function (val, graphDetails, adjThickness, thickLine, isDistBased) {
        if(!isDistBased) val -= graphDetails.data.minX;
        return this.options.margins.left + Math.round(graphDetails.tickSpace.x * val) + (thickLine ? 0 : adjThickness);
    },
    pointToY: function (val, graphDetails, adjThickness, thickLine, isDistBased) {
        if(!isDistBased) val -= graphDetails.data.minY;
        return this.options.margins.top + Math.round(graphDetails.tickSpace.y * (graphDetails.count.y - val)) + (thickLine ? 0 : adjThickness);
    },
    coordsToPoint:function(coords, graphDetails, isDistBased){
        return {x:this.xToPoint(coords.x, graphDetails, isDistBased), y:this.yToPoint(coords.y, graphDetails, isDistBased)};
    },
    xToPoint: function (val, graphDetails, isDistBased) {
        if(isDistBased) val += this.options.margins.left;
        var xPos = ((val - this.options.margins.left) / graphDetails.tickSpace.x) + graphDetails.data.minX;
        if(isDistBased) xPos -= graphDetails.data.minX;
        return xPos*graphDetails.data.tick.xInt;
    },
    yToPoint: function (val, graphDetails, isDistBased) {
        if(isDistBased) val += this.options.margins.top;
        this.totalHeight = Math.floor(graphDetails.tickSpace.y * graphDetails.count.y);
        var yPos = ((this.totalHeight - val + this.options.margins.top) / graphDetails.tickSpace.y) + graphDetails.data.minY;
        if(isDistBased) yPos -= graphDetails.data.maxY;
        return yPos*graphDetails.data.tick.yInt;
    },
    drawShapes:function(isReCalculate, isOnlyCalculate, isMoving) {
        var canvasIndex = 1;
        var context = this.contextSet[canvasIndex];
        var shapes = this.shapeData;
        context.clearRect(0,0,this.canvasSet[canvasIndex].width(),this.canvasSet[canvasIndex].height());
	for(var t=0; t<shapes.length; t++){
            var shape = shapes[t];
            if (!shape.angles) shape.angles = [];
            if (!shape.points) shape.points = [];
            var angle=shape.angle;
            var angles=shape.angles;
            var points=shape.points;
            var labels=shape.label;
            var lengths=shape.lengths;
            var sideLen=Math.max(lengths.length, points.length);
            var maxAngle = this.baseAngle*(sideLen-2);
            var gd = this.graphDetails();
            var tempPoint = {x:this.pointToX(0, gd, 0, false), y:this.pointToY(0, gd, 0, false)};
            var pointVariation = {x:0, y:0};
            if (isReCalculate) {
                var pointAngle = angle;
                points[0] = tempPoint;
                var prevPoint = tempPoint;
                for (var p=1; p<sideLen; p++) {
                    var lIndx = this.correctIndex(p-1, sideLen);
                    var aIndx = this.correctIndex(p-1, sideLen);
                    pointAngle += angles[aIndx];
                    points[p] = this.APLtoPoint(pointAngle, prevPoint, lengths[lIndx]);
                    prevPoint = points[p];
                }
                
                var currMid = this.midPoint(points);
                pointVariation.x = shape.midPoint.x-currMid.x;
                pointVariation.y = shape.midPoint.y-currMid.y;
                
                for (p=0; p<points.length; p++) {
                    points[p].x += pointVariation.x;
                    points[p].y += pointVariation.y;
                }
            }
            
            if (!isOnlyCalculate) {
                context.save();
                this.applycontextProp(context, this.options.contextProp.shapes);
                if(this.selectedShapeIndex == t && this.selectedItemType == "shape") this.applycontextProp(context, this.options.contextProp.selectedShape);
                if(shape.customProp) this.applycontextProp(context, shape.customProp);
                this.drawShape(points, context, true);
                context.restore();
                
                                  
                var isVertCntrl = this.operation == "vertexControl" && shape.drag;
                if(this.options.isVisibleVertex || isVertCntrl) {
                    context.save();
                   // if (isVertCntrl) this.applycontextProp(context, this.options.contextProp.vertexControl);
                   // else
                    this.applycontextProp(context, this.options.contextProp.shapeVertex);
                    var rad = isVertCntrl?this.options.vertexControlRad:this.options.vertexRad;
                    this.drawVertex(points, rad, context);
                    context.restore();
                }
                context.save();
                this.applycontextProp(context, this.options.contextProp.shapeLabels);
                var labeledPoints = JSON.parse(JSON.stringify(points));
                if(shape.PORLabel) labeledPoints.push({x: shape.pointOfRotation.x, y: shape.pointOfRotation.y});
                for(p=0; p<labeledPoints.length; p++){
                    var text = labels[p];
                    if(shape.PORLabel && p==labeledPoints.length-1) text = shape.PORLabel;
                    if (text) {
                        var tDist = this.distance(labeledPoints[p],shape.midPoint)+this.options.lblDistFrmShape;
                        var tAng = this.angleBetweenPoints(labeledPoints[p],shape.midPoint)*180/Math.PI;
                        var tPos = this.APLtoPoint(tAng, shape.midPoint, tDist);
                        context.fillText(text, tPos.x, tPos.y);
                    }
                }
                context.restore();
                if (shape.drag)
                    if (this.options.isVarPointOfRot && this.operation == "rotate") {
                        context.save();
                        this.applycontextProp(context, this.options.contextProp.pointOfRotLine);
                        context.beginPath();
                      
                        context.moveTo(shape.midPoint.x, shape.midPoint.y);
                        context.lineTo(shape.pointOfRotation.x, shape.pointOfRotation.y);
                        context.stroke();
                        context.restore();
                        context.save();
                        this.applycontextProp(context, this.options.contextProp.pointOfRot);
                        context.beginPath();
                        //context.arc(shape.pointOfRotation.x, shape.pointOfRotation.y, this.options.pointOfRotRad, 0, Math.PI*2);
                        context.fill();
                        context.stroke();
                        context.restore();
                    }
            }
 	}
        
        if (this.drawingShape.length && this.operation == "createShape") {
            context.save();
            this.applycontextProp(context, this.options.contextProp.drawingShape);
            this.drawShape(this.drawingShape, context);
            context.restore();
            context.save();
            this.applycontextProp(context, this.options.contextProp.drawingVertex);
            this.drawVertex(this.drawingShape, this.options.drawingPointRad, context, this.touchedObjectIndex, this.options.contextProp.selectedDrawingVertex);
            context.restore();
        }
        
        if (!isOnlyCalculate){
            var graphDetails = this.graphDetails();
            if(this.staticPlotters) this.drawPlotterData(context, graphDetails, this.staticPlotters, false);
            if(this.plotterData) this.drawPlotterData(context, graphDetails, this.plotterData, true);
        }
        
        if (!isOnlyCalculate && this.lineOfReflection && this.operation == "reflect") {
            var maxDiagonal = Math.sqrt(Math.pow(this.canvasArea.x,2)+Math.pow(this.canvasArea.y,2));
            context.save();
            this.applycontextProp(context, this.options.contextProp.lineOfReflection);
            context.beginPath();
            var angleOfReflection = this.angleBetweenPoints(this.lineOfReflection.point1, this.lineOfReflection.point2)*180/Math.PI;
            this.lineOfReflection.angle = angleOfReflection;
            var midPoint = this.midPoint([this.lineOfReflection.point1, this.lineOfReflection.point2]);
            this.lineOfReflection.midPoint = midPoint;
            var drawPoint1 = this.APLtoPoint(angleOfReflection, midPoint, maxDiagonal);
            var drawPoint2 = this.APLtoPoint(angleOfReflection, midPoint, -maxDiagonal);
            var gridLineWidth = this.options.contextProp.grid.lineWidth*2;
            context.rect(this.options.margins.left, this.options.margins.top, this.canvasArea.x-this.options.margins.left-this.options.margins.right-gridLineWidth, this.canvasArea.y-this.options.margins.top-this.options.margins.bottom-gridLineWidth);
            context.clip();
            context.beginPath();
             
            context.moveTo(drawPoint1.x, drawPoint1.y);
            context.lineTo(drawPoint2.x, drawPoint2.y);
            context.stroke();
            context.restore();
            context.save();
            
            this.applycontextProp(context, this.options.contextProp.pointsOfLOR);
            context.beginPath();
            context.arc(this.lineOfReflection.point1.x, this.lineOfReflection.point1.y, this.options.pointOfLORRad, 0, Math.PI*2);
            context.fill();
            context.stroke();
            context.beginPath();
            context.arc(this.lineOfReflection.point2.x, this.lineOfReflection.point2.y, this.options.pointOfLORRad, 0, Math.PI*2);
            context.fill();
            context.stroke();
            context.restore();
        }
        
        this.drawShapeLen(shapes);
        
    },
    drawShapeLen:function(shapes)
    {
    for(var t=0; t<shapes.length; t++){
            var shape = shapes[t];
            if (!shape.angles) shape.angles = [];
            if (!shape.points) shape.points = [];
            var angle=shape.angle;
            var angles=shape.angles;
            var points=shape.points;
            var labels=shape.label;
            var lengths=shape.lengths;
            var sideLen=Math.max(lengths.length, points.length);
            var maxAngle = this.baseAngle*(sideLen-2);
            var gd = this.graphDetails();
           
            var shapepoints=[];
            var shapeposition=[];
	        shapelen=[];	
                for(var i=0;i<points.length;i++)
                {   var cod={x:points[i].x,y:points[i].y};
                point=adjShape.coordsToPoint(cod,adjShape.graphDetails(),false);
                point={x:point.x,y:point.y};
                shapepoints.push(point);
                shapeposition.push(cod); 
                }
               
            shapepoints.push(shapepoints[0]);
            shapeposition.push(shapeposition[0]);
           
             d= Math.sqrt(Math.pow(this.shapeData[0].points[0].x-this.shapeData[0].points[1].x,2)+Math.pow(this.shapeData[0].points[0].y-this.shapeData[0].points[1].y,2));
             shapelen.push(d/37.795276);
             d= Math.sqrt(Math.pow(this.shapeData[0].points[1].x-this.shapeData[0].points[2].x,2)+Math.pow(this.shapeData[0].points[1].y-this.shapeData[0].points[2].y,2));
             shapelen.push(d/37.795276);
             d= Math.sqrt(Math.pow(this.shapeData[0].points[2].x-this.shapeData[0].points[3].x,2)+Math.pow(this.shapeData[0].points[2].y-this.shapeData[0].points[3].y,2));
             shapelen.push(d/37.795276);
             d= Math.sqrt(Math.pow(this.shapeData[0].points[3].x-this.shapeData[0].points[0].x,2)+Math.pow(this.shapeData[0].points[3].y-this.shapeData[0].points[0].y,2));
             shapelen.push(d/37.795276);
             
            
            var context = this.contextSet[1];
           
            rand=10;
            rand1=10;
            rand3=25;
            rand4=-55;
            rand5=-10
            for(var i=0;i<shapelen.length;i++)
            {
            context.save();
            if (this.operation=="Result"){rand=Math.random()*10;rand1=Math.random()*10;}
            context.font="14px Arial";
            if (i==0) {
                context.fillText(Math.round((shapelen[0])*10)+" mm",(shapeposition[0].x+shapeposition[1].x)/2+rand ,(shapeposition[0].y+shapeposition[1].y)/2+ rand1 );
            }
            else if(i==1){
            context.fillText(Math.round((shapelen[1])*10)+" mm",(shapeposition[1].x+shapeposition[2].x)/2 ,(shapeposition[1].y+shapeposition[2].y)/2 +rand3);
            }
             else if(i==2){
            context.fillText(Math.round((shapelen[2])*10)+" mm",(shapeposition[2].x+shapeposition[3].x)/2+rand4 ,(shapeposition[2].y+shapeposition[3].y)/2 );
            }
              else if(i==3){
            context.fillText(Math.round((shapelen[3])*10)+" mm",(shapeposition[3].x+shapeposition[4].x)/2+rand5 ,(shapeposition[3].y+shapeposition[4].y)/2+rand5 );
            }
            context.restore();    
            }
            
            
          
 	}
      
        
    },
    drawPlotterData:function(context, graphDetails, data, isLive){
        var lines = data.lines;
        var points = data.points;
        var circles = data.circles;
        for (var l=lines.length-1; l>=0; l--) {
            var line = lines[l];
            context.save();
            this.applycontextProp(context, this.options.contextProp.plotterLine);
            if(line.contextProp) this.applycontextProp(context, line.contextProp);
            var isSelected = false;
            if (this.selectedItemType == "line" && this.selectedObjectIndex == l && isLive){
                isSelected = true;
                context.save();
                this.applycontextProp(context, this.options.contextProp.selectedPlotterLine);
            }
            var fromPoint = this.pointToPos(line.from.x/graphDetails.data.tick.xInt, line.from.y/graphDetails.data.tick.yInt, graphDetails, false, 0);
            var toPoint = this.pointToPos(line.to.x/graphDetails.data.tick.xInt, line.to.y/graphDetails.data.tick.yInt, graphDetails, false, 0);
            
            /*var getdistance=[];
            for (var k=0;k<this.shapeData[0].points.length;k++) {
                getdistance.push(Math.sqrt(Math.pow(fromPoint.x-this.shapeData[0].points[k].x,2)+Math.pow(fromPoint.y-this.shapeData[0].points[k].y,2)));
            }
            
         
            for (var k=0;k<this.shapeData[0].points.length;k++) {
         var pointsLineDistane=(Math.min(getdistance[0],getdistance[1],getdistance[2],getdistance[3]));
              }
              var minIndex=$.inArray(pointsLineDistane,getdistance)
             this.drawLinefrom={};
            this.drawLinefrom=this.shapeData[0].points[minIndex];
            
            drawLineto=[];
             this.lineconnectTo={};
           for (var m=0;m<this.shapeData[0].points.length;m++) {
            
            drawLineto.push(Math.abs(toPoint.x-this.shapeData[0].points[m].x)<40 && Math.abs(toPoint.y-this.shapeData[0].points[m].y)<40);
            if (drawLineto[m]==true){
            this.lineconnectTo=this.shapeData[0].points[m]
            var test=true;
           
            }}
            
            */
          
            context.beginPath();
             context.setLineDash([5]);
            context.moveTo(fromPoint.x, fromPoint.y);
            
            //if (test==true) {
            //
            //  context.lineTo(this.lineconnectTo.x, this.lineconnectTo.y);
            //  }
            //
            //else{
            context.lineTo(toPoint.x, toPoint.y);
            //}
            context.stroke();
            if (isSelected) context.restore();
            context.restore();
        }
        
        for (var p=points.length-1; p>=0; p--) {
            var point = points[p];
            context.save();
            this.applycontextProp(context, this.options.contextProp.plotterPoint);
            if(point.contextProp) this.applycontextProp(context, point.contextProp);
            var isSelected = false;
            if (this.selectedItemType == "point" && this.selectedObjectIndex == p && isLive){
                isSelected = true;
                context.save();
                this.applycontextProp(context, this.options.contextProp.selectedPlotterPoint);
            }
            var pointPos = this.pointToPos(point.from.x/graphDetails.data.tick.xInt, point.from.y/graphDetails.data.tick.yInt, graphDetails, false, 0);
            context.beginPath();
            context.arc(pointPos.x, pointPos.y, this.options.plotPointRad, 0, Math.PI*2);
            context.fill();
            context.stroke();
            if (isSelected) context.restore();
            var labelText = point.label;
            if (labelText) {
                this.applycontextProp(context, this.options.contextProp.plotterLabel);
                context.fillText(labelText, pointPos.x+this.options.pointLabelOffset.x, pointPos.y+this.options.pointLabelOffset.y);
            }
            context.restore();
        }
        for (var c=circles.length-1; c>=0; c--) {
            var circle = circles[c];
            context.save();
            this.applycontextProp(context, this.options.contextProp.plotterCircle);
            if(circle.contextProp) this.applycontextProp(context, circle.contextProp);
            var isSelected = false;
            if (this.selectedItemType == "circle" && this.selectedObjectIndex == c && isLive) {
                isSelected = true;
                context.save();
                this.applycontextProp(context, this.options.contextProp.selectedPlotterCircle);
            }
            var anotherPointPos;
            var midPoint = this.pointToPos(circle.midPoint.x/graphDetails.data.tick.xInt, circle.midPoint.y/graphDetails.data.tick.yInt, graphDetails, false, 0);
            if (circle.anotherPoint){
                anotherPointPos = this.pointToPos(circle.anotherPoint.x/graphDetails.data.tick.xInt, circle.anotherPoint.y/graphDetails.data.tick.yInt, graphDetails, false, 0);
                circle.radius = this.distance(midPoint, anotherPointPos);
            }
            if (circle.radius) {
                context.save();
                var gridLineWidth = this.options.contextProp.grid.lineWidth*2;
                context.rect(this.options.margins.left, this.options.margins.top, this.canvasArea.x-this.options.margins.left-this.options.margins.right-gridLineWidth, this.canvasArea.y-this.options.margins.top-this.options.margins.bottom-gridLineWidth);
                context.clip();
                context.beginPath();
                context.arc(midPoint.x, midPoint.y, circle.radius, 0, Math.PI*2);
                context.fill();
                context.stroke();
                context.restore();
            }
            context.fillStyle = context.strokeStyle;
            context.beginPath();
            context.arc(midPoint.x, midPoint.y, this.options.plotPointRad, 0, Math.PI*2);
            context.fill();
            context.stroke();
            if (anotherPointPos) {
                context.beginPath();
                context.arc(anotherPointPos.x, anotherPointPos.y, this.options.plotPointRad, 0, Math.PI*2);
                context.fill();
                context.stroke();
            }
            if (isSelected) context.restore();
            context.restore();
        }
    },
    drawVertex:function(points, radius, context, selIndex, selCntxtProp){
        for(p=0; p<points.length; p++) {
           
            context.beginPath();
            context.arc(points[p].x, points[p].y,radius, 0, Math.PI*2);
            if (p == selIndex) {
                context.save();
                this.applycontextProp(context, selCntxtProp);
            }
            context.fill();
            context.stroke();
            if (p == selIndex) context.restore();
        }
    },
    drawShape:function(points, context, isClosePath){
        context.beginPath();
        for(var p=0; p<points.length; p++){
            if (p==0) context.moveTo(points[p].x, points[p].y);
            else context.lineTo(points[p].x, points[p].y);
        }
        if(isClosePath) context.closePath();
        context.fill();
        context.stroke();
    },
    applycontextProp:function(context, ctxProp) {
        if (ctxProp)
            for (var k in ctxProp) {
                if (k == "setLineDash") context.setLineDash(ctxProp[k]);
                else context[k] = ctxProp[k];
            }
    },
    APLtoPoint:function(angle, point, length){
	angle = angle*Math.PI/180;
	var newPoint = {x: 0, y:0};
	newPoint.x = (Math.cos(angle)*length)+point.x;
	newPoint.y = (Math.sin(angle)*length)+point.y;
	return newPoint;
    },
    sidesToAngle:function(sideA, sideB, reqSide){
	var angle = Math.acos((Math.pow(sideA, 2)+Math.pow(sideB, 2)-Math.pow(reqSide, 2))/(2*sideA*sideB))*180/Math.PI;
	return angle;
    },
    setOperation:function(operation){
        this.operation = operation;
        if (this.options.isResetPOR) this.resetPointOfRotation();
        this.drawAll();
    },
    resetPointOfRotation:function(){
        var shapes = this.shapeData;
	for(var t=0; t<shapes.length; t++){
            var shape = shapes[t];
            shape.pointOfRotation = JSON.parse(JSON.stringify(shape.midPoint));
        }
    },
    hexToRGB: function (hexCol) {
        var str = hexCol.replace("#", "");
        var newCol = hexCol;
        var red, green, blue;
        if (str.length == 3) {
            red = str[0]; green = str[1]; blue = str[2];
        }
        else if (str.length == 6) {
            red = str[0] + str[1]; green = str[2] + str[3]; blue = str[4] + str[5];
        }
        return {r:parseInt(red, 16), g:parseInt(green, 16), b:parseInt(blue, 16)};
    },
    getSelectedObject:function(){
        if (this.selectedObjectIndex >= 0 && this.selectedItemType != "")
            return this.plotterData[this.selectedItemType+"s"][this.selectedObjectIndex];
    },
    select:function(touchPos) {
        var touchedElemProp = {touched:false};
        if(this.plotterData){
            var graphDetails = this.graphDetails();
            this.tempGraphDetails = graphDetails;
            var lines = this.plotterData.lines;
            var points = this.plotterData.points;
            var circles = this.plotterData.circles;
            
            if (!touchedElemProp.touched)
                for (var p=0; p<points.length; p++) {
                    var point = points[p];
                    var pointPos = this.pointToPos(point.from.x/this.tempGraphDetails.data.tick.xInt, point.from.y/this.tempGraphDetails.data.tick.yInt, graphDetails, false, 0);
                    if (this.distance(pointPos, touchPos) < this.options.plotPointRad*2){
                        touchedElemProp.touched = true;
                        touchedElemProp.touchedObj = "point";
                        touchedElemProp.touchedObjIndex = p;
                        break;
                    }
                }
            
            if (!touchedElemProp.touched)
                for (var l=0; l<lines.length; l++) {
                    var line = lines[l];
                    var fromPoint = this.pointToPos(line.from.x/this.tempGraphDetails.data.tick.xInt, line.from.y/this.tempGraphDetails.data.tick.yInt, graphDetails, false, 0)
                    var toPoint = this.pointToPos(line.to.x/this.tempGraphDetails.data.tick.xInt, line.to.y/this.tempGraphDetails.data.tick.yInt, graphDetails, false, 0)
                    var lineLength = this.distance(fromPoint, toPoint);
                    var fromLength = this.distance(fromPoint, touchPos);
                    var toLength = this.distance(toPoint, touchPos);
                    var lineAngle = this.angleBetweenPoints(fromPoint, toPoint)*180/Math.PI;
                    var sense = this.options.lineTouchSense;
                    var frm1 = this.APLtoPoint(lineAngle+90, fromPoint, sense);
                    var frm2 = this.APLtoPoint(lineAngle-90, fromPoint, sense);
                    var to1 = this.APLtoPoint(lineAngle+90, toPoint, sense);
                    var to2 = this.APLtoPoint(lineAngle-90, toPoint, sense);
                    var isTouched = this.pointInShape(touchPos, [frm1, frm2, to2, to1]);
                    var lenDiv = lineLength/3;
                    if (isTouched) {
                        touchedElemProp.touched = true;
                        touchedElemProp.touchedObj = "line";
                        if (fromLength < lenDiv) touchedElemProp.touchedInternalObj = "fromPoint";
                        else if (toLength < lenDiv) touchedElemProp.touchedInternalObj = "toPoint";
                        else touchedElemProp.touchedInternalObj = "midPoint";
                        touchedElemProp.touchedObjIndex = l;
                        break;
                    }
                }
                
            if (!touchedElemProp.touched)
                for (var c=0; c<circles.length; c++) {
                    var circle = circles[c];
                    var anotherPointPos;
                    var midPoint = this.pointToPos(circle.midPoint.x/this.tempGraphDetails.data.tick.xInt, circle.midPoint.y/this.tempGraphDetails.data.tick.yInt, graphDetails, false, 0);
                    if (circle.anotherPoint) {
                        anotherPointPos = this.pointToPos(circle.anotherPoint.x/this.tempGraphDetails.data.tick.xInt, circle.anotherPoint.y/this.tempGraphDetails.data.tick.yInt, graphDetails, false, 0);
                        if (this.distance(anotherPointPos, touchPos) < this.options.plotPointRad*2){
                            touchedElemProp.touched = true;
                            touchedElemProp.touchedInternalObj = "anotherPoint";
                            touchedElemProp.touchedObj = "circle";
                            touchedElemProp.touchedObjIndex = c;
                            break;
                        }
                    }
                    if (!touchedElemProp.touched)
                        if (this.distance(midPoint, touchPos) < (circle.radius ? circle.radius : this.options.plotPointRad)) {
                            touchedElemProp.touched = true;
                            touchedElemProp.touchedInternalObj = "midPoint";
                            touchedElemProp.touchedObj = "circle";
                            touchedElemProp.touchedObjIndex = c;
                            break;
                        }
                }
        }
        return touchedElemProp;
      },
 plotPoint:function(point){
        this.tempGraphDetails = this.graphDetails();
        var p=0;
        var index;
        for(p=0; p<this.plotterData.points.length; p++) {
            var _p = this.plotterData.points[p].from;
            var pointPos = this.pointToPos(_p.x/this.tempGraphDetails.data.tick.xInt, _p.y/this.tempGraphDetails.data.tick.yInt, this.tempGraphDetails, 0, false);
            if (this.distance(pointPos, point) < this.options.plotPointRad*2) {
                this.touchedObjectProp.object = this.plotterData.points[p];
                break;
            }
        }
        if (!this.touchedObjectProp.object) {
            var res = this.isWithinArea(point, 0);
            point.x += res._x;
            point.y += res._y;
            var pointVal = {x:this.xToPoint(point.x, this.tempGraphDetails), y:this.yToPoint(point.y, this.tempGraphDetails)};
            pointVal = this.correctPlotPosition(pointVal, true);
            var pointData = {from:pointVal};
            this.plotterData.points.push(pointData);
            this.touchedObjectProp.object = pointData;
            index = this.plotterData.points.length-1;
        }
        else index=p;
        this.selectedItemType = "point";
        this.selectedObjectIndex = index;
        this.touchedObjectProp.data = {touchedInternalObj: "midPoint", touchedObj: "point", touchedObjIndex: index};
        this.tempData = JSON.parse(JSON.stringify(this.touchedObjectProp.object));
    },
    drawLineAndCirlce:function(point){
        
        this.tempGraphDetails = this.graphDetails();
        var res = this.isWithinArea(point, 0);
        point.x += res._x;
        point.y += res._y;
        var pointVal = {x:this.xToPoint(point.x, this.tempGraphDetails), y:this.yToPoint(point.y, this.tempGraphDetails)};
        pointVal = this.correctPlotPosition(pointVal, true);
        var object;
        if (this.operation == "drawLine") {
            
            if(this.options.slctLineOnDT) {
                this.tapCount++;
                this.dragLnP(point);
                var _this = this;
                setTimeout(function(){ _this.tapCount = 0; },this.doubleTapTime);
                if (this.xOperation) {
                    if(this.tapCount >= 1) return;
                    else if(this.xselectedItemType == this.selectedItemType && this.xselectedObjectIndex == this.selectedObjectIndex) return;
                }
            }
            this.xOperation = "";
            object = {from:pointVal, to:JSON.parse(JSON.stringify(pointVal)), label:""};
            this.plotterData.lines.push(object);
            this.touchedObjectProp.data = {touchedInternalObj: "midPoint", touchedObj: "line", touchedObjIndex: this.plotterData.lines.length-1};
            this.selectedItemType = "line";
            this.selectedObjectIndex = this.plotterData.lines.length-1;
        }
         if (this.operation == "drawLine") {
                    object = {from:pointVal, to:JSON.parse(JSON.stringify(pointVal))};
                    if (chosenLineColor)
                        object.contextProp=chosenLineColor;
                    
                    
                    this.plotterData.lines.push(object);
                    this.touchedObjectProp.data = {touchedInternalObj: "midPoint", touchedObj: "line", touchedObjIndex: this.plotterData.lines.length-1};
                    this.selectedItemType = "line";
                    this.selectedObjectIndex = this.plotterData.lines.length-1;
                }
        this.touchedObjectProp.object = object;
        this.tempData = JSON.parse(JSON.stringify(object));
    },
    dragLnP:function(point){
        var selectRes = this.select(point);
        if (selectRes.touched && selectRes.touchedObj) {
            var touchedObject = this.plotterData[selectRes.touchedObj+"s"][selectRes.touchedObjIndex];
            this.touchedObjectProp.data = selectRes;
            this.touchedObjectProp.addData = {};
            this.selectedItemType = selectRes.touchedObj;
            this.selectedObjectIndex = selectRes.touchedObjIndex;
            if (selectRes.touchedInternalObj) this.touchedObjectProp.addData.internalObj = selectRes.touchedInternalObj;
            this.tempData = JSON.parse(JSON.stringify(touchedObject));
            this.drawShapes(false);
            this.xOperation = "dragLnP";
        }
        else this.xOperation = "";
    },
    tryDeleteLine:function(){
        this.setOperation("delete");
        this.deleteMode = "byLine";
    },
    tryDeleteLinkedLines:function(){
        this.setOperation("delete");
        this.deleteMode = "byShape";
    },
    deleteLine:function(point){
        var selectRes = this.select(point);
        if (selectRes.touched) {
            this.plotterData[selectRes.touchedObj+"s"].splice(selectRes.touchedObjIndex, 1);
            this.updateLineToLineRelation();
        }
        this.drawAll();
    },
    deleteLinkedLines:function(point){
        var selectRes = this.select(point);
        if (selectRes.touched) {
            var linkedLines = [];
            this.getLinkedLineIndices(selectRes.touchedObjIndex, linkedLines);
            for(var i=linkedLines.length-1; i>=0; i--) this.plotterData[selectRes.touchedObj+"s"].splice(linkedLines[i], 1);
            this.updateLineToLineRelation();
        }
        this.drawAll();
    },
    sortNumber:function(a, b){ return a - b; },
    getLinkedLineIndices:function(lineIndex, array){
        this._getLinkedLineIndices(lineIndex, array);
        array.sort(this.sortNumber);
    },
    storeShapeDetails:function()
    {        
     shapeDetails=[];
     var linkedLinesPairs=[];
     var lines=[];
     var shapesInfo=[];
     
     //get linked pairs
     for (var i=0;i<this.plotterData.lines.length;i++) {
        var selectRes={touched: true, touchedObj: "line", touchedInternalObj: "midPoint", touchedObjIndex:i} 
        if (lines.indexOf(i)==-1) {
        if (selectRes.touched) {
            var linkedLines = [];
            this.getLinkedLineIndices(selectRes.touchedObjIndex, linkedLines);
            linkedLinesPairs.push(linkedLines);
           var linepoints=[];
            info={"links":linkedLines,"endone":-1,"endtwo":-1,"linksWithPoints":[]};
            for(var i=0; i<linkedLines.length; i++)
            {
                lines.push(linkedLines[i]);
                var pointVal=this.plotterData.lines[linkedLines[i]];
                delete pointVal.label;
                 pnt={"point":pointVal,"index":linkedLines[i]};
                info.linksWithPoints.push(pnt)         
            }
            shapesInfo.push(info);
           }
        }
 }
 

    var canProceed=false;
    for(var i=0; i<linkedLinesPairs.length; i++){
               flag=0;
                   for(var j=0; j<linkedLinesPairs[i].length; j++){
                       if(this.plotterData.secRelation[linkedLinesPairs[i][j]] &&linkedLinesPairs[i].length==3)
                       if(this.plotterData.secRelation.length>0 && (this.plotterData.secRelation[linkedLinesPairs[i][j]].from.length==0 || this.plotterData.secRelation[linkedLinesPairs[i][j]].to.length==0))
                       { if(this.plotterData.secRelation[linkedLinesPairs[i][j]].to.length==0)side="to";else side="from";
                           
                           if(!flag){shapesInfo[i].endone=linkedLinesPairs[i][j]; shapesInfo[i].endoneside=side;}
                           if(flag){shapesInfo[i].endtwo=linkedLinesPairs[i][j];shapesInfo[i].endtwoside=side;}
                           flag++;
                           canProceed=true;
                       }
                   }
               }
            
        
 
        if (canProceed) {
           
        for(var iterLine=0;iterLine<this.staticPlotters.lines.length;iterLine++)
        {
        for(var iterShape=0; iterShape<shapesInfo.length; iterShape++){
        if(shapesInfo[iterShape].links.length==3)  
        if ((this.staticPlotters.lines[iterLine].from.x==this.plotterData.lines[shapesInfo[iterShape].endone][shapesInfo[iterShape].endoneside].x &&
            this.staticPlotters.lines[iterLine].from.y==this.plotterData.lines[shapesInfo[iterShape].endone][shapesInfo[iterShape].endoneside].y
            ||this.staticPlotters.lines[iterLine].to.x==this.plotterData.lines[shapesInfo[iterShape].endone][shapesInfo[iterShape].endoneside].x &&
            this.staticPlotters.lines[iterLine].to.y==this.plotterData.lines[shapesInfo[iterShape].endone][shapesInfo[iterShape].endoneside].y 
            )&&
            (this.staticPlotters.lines[iterLine].from.x==this.plotterData.lines[shapesInfo[iterShape].endone][shapesInfo[iterShape].endoneside].x &&
            this.staticPlotters.lines[iterLine].from.y==this.plotterData.lines[shapesInfo[iterShape].endone][shapesInfo[iterShape].endoneside].y
            ||this.staticPlotters.lines[iterLine].to.x==this.plotterData.lines[shapesInfo[iterShape].endone][shapesInfo[iterShape].endoneside].x &&
            this.staticPlotters.lines[iterLine].to.y==this.plotterData.lines[shapesInfo[iterShape].endone][shapesInfo[iterShape].endoneside].y 
            )
            )
            {   //sort according to end points
               
               
        var t1,t2,t3;
        var foundIndices=[];
         for (var i=0;i<3;i++) {
         if(shapesInfo[iterShape].endone==shapesInfo[iterShape].linksWithPoints[i].index){ t1=shapesInfo[iterShape].linksWithPoints[i].point; foundIndices.push(i)};
         if(shapesInfo[iterShape].endtwo==shapesInfo[iterShape].linksWithPoints[i].index){ t3=shapesInfo[iterShape].linksWithPoints[i].point;foundIndices.push(i)};           
        }
        for (var i=0;i<3;i++) 
        if(foundIndices[0]!=shapesInfo[iterShape].linksWithPoints[i].index && foundIndices[1]!=shapesInfo[iterShape].linksWithPoints[i].index )
        t2=shapesInfo[iterShape].linksWithPoints[i].point;foundIndices.push(i)
        shapesInfo[iterShape].linksWithPoints[0]=t1;
        shapesInfo[iterShape].linksWithPoints[1]=t2;
        shapesInfo[iterShape].linksWithPoints[2]=t3;
        shapesInfo[iterShape].completeLineIndex=iterLine;
        shapesInfo[iterShape].linksWithPoints.push(this.staticPlotters.lines[iterLine]);
        shapeDetails.push(shapesInfo[iterShape]);     
     }
    }
  }
}
 
 
 
 },
    _getLinkedLineIndices:function(lineIndex, array){
        if (array.indexOf(lineIndex) < 0) {
            array.push(lineIndex);
            var rel = this.plotterData.secRelation[lineIndex];
            if (rel) {
                for (var i=0; i<rel.from.length; i++) this.getLinkedLineIndices(rel.from[i].index, array);
                for (var i=0; i<rel.to.length; i++) this.getLinkedLineIndices(rel.to[i].index, array);
            }
        }
        //this.plotterData[selectRes.touchedObj+"s"].splice(selectRes.touchedObjIndex, 1);
    },
   
    onStart:function(event){
	this.touchedShapeIndex = -1;
        this.touchedObjectIndex = -1;
        this.selectedShapeIndex = -1;
        this.touchedObjectProp = {};
        this.selectedItemType = "";
        this.selectedObjectIndex = -1;
        this.xOperation = "";
        var touches = this.touchCoordsFromEvent(event, true);
        var shapes = this.shapeData;
        this.downPosition = null;
        if (touches.length == 1) {
            var xPos = touches[0].x-this.canvasHolder.offset().left;
            var yPos = touches[0].y-this.canvasHolder.offset().top;
            this.downPosition = {x:xPos, y:yPos};
            if (this.operation == "plotPoint") {
                this.tempGraphDetails = this.graphDetails();
                var point = {x:xPos, y:yPos};
                var p=0;
                var index;
                for(p=0; p<this.plotterData.points.length; p++) {
                    var _p = this.plotterData.points[p].from;
                    var pointPos = this.pointToPos(_p.x/this.tempGraphDetails.data.tick.xInt, _p.y/this.tempGraphDetails.data.tick.yInt, this.tempGraphDetails, 0, false);
                    if (this.distance(pointPos, point) < this.options.plotPointRad*2) {
                        this.touchedObjectProp.object = this.plotterData.points[p];
                        break;
                    }
                }
                if (!this.touchedObjectProp.object) {
                    var res = this.isWithinArea(point, 0);
                    point.x += res._x;
                    point.y += res._y;
                    var pointVal = {x:this.xToPoint(point.x, this.tempGraphDetails), y:this.yToPoint(point.y, this.tempGraphDetails)};
                    pointVal = this.correctPlotPosition(pointVal, true);
                    var pointData = {from:pointVal};
                    this.plotterData.points.push(pointData);
                    this.touchedObjectProp.object = pointData;
                    index = this.plotterData.points.length-1;
                }
                else index=p;
                this.selectedItemType = "point";
                this.selectedObjectIndex = index;
                this.touchedObjectProp.data = {touchedInternalObj: "midPoint", touchedObj: "point", touchedObjIndex: index};
                this.tempData = JSON.parse(JSON.stringify(this.touchedObjectProp.object));
            }
            else if (this.operation == "drawCircle" || this.operation == "drawLine") {
                
                
                
                var point = {x:xPos, y:yPos};
                this.tempGraphDetails = this.graphDetails();
                var res = this.isWithinArea(point, 0);
                point.x += res._x;
                point.y += res._y;
                var pointVal = {x:this.xToPoint(point.x, this.tempGraphDetails), y:this.yToPoint(point.y, this.tempGraphDetails)};
                pointVal = this.correctPlotPosition(pointVal, true);
                var object;
                if (this.operation == "drawLine") {
                    object = {from:pointVal, to:JSON.parse(JSON.stringify(pointVal))};
                    this.plotterData.lines.push(object);
                    this.touchedObjectProp.data = {touchedInternalObj: "midPoint", touchedObj: "line", touchedObjIndex: this.plotterData.lines.length-1};
                    this.selectedItemType = "line";
                    this.selectedObjectIndex = this.plotterData.lines.length-1;
                    this.snapToShape(object, false, false);
                }
                else if (this.operation == "drawCircle"){
                    object = {midPoint:pointVal, anotherPoint:JSON.parse(JSON.stringify(pointVal))};
                    this.plotterData.circles.push(object);
                    this.selectedItemType = "circle";
                    this.selectedObjectIndex = this.plotterData.circles.length-1;
                }
                this.touchedObjectProp.object = object;
                this.tempData = JSON.parse(JSON.stringify(object));
            }
            else if (this.operation == "dragLnP") {
                var selectRes = this.select(this.downPosition);
                if (selectRes.touched && selectRes.touchedObj) {
                    var touchedObject = this.plotterData[selectRes.touchedObj+"s"][selectRes.touchedObjIndex];
                    this.touchedObjectProp.data = selectRes;
                    this.touchedObjectProp.addData = {};
                    this.selectedItemType = selectRes.touchedObj;
                    this.selectedObjectIndex = selectRes.touchedObjIndex;
                    if (selectRes.touchedInternalObj) this.touchedObjectProp.addData.internalObj = selectRes.touchedInternalObj;
                    this.tempData = JSON.parse(JSON.stringify(touchedObject));
                    this.drawShapes(false);
                }
            }
            else if (this.operation == "createShape") {
                for(var p=0; p<this.drawingShape.length; p++)
                    if(this.distance(this.downPosition, this.drawingShape[p]) < this.options.vertexControlRad) {
                        this.touchedObjectIndex = p;
                        this.touchedObject = "createVertex";
                        break;
                    }
                if (this.touchedObjectIndex < 0) {
                    var point = {x:xPos, y:yPos};
                    var res = this.isWithinArea(point, 0);
                    point.x += res._x;
                    point.y += res._y;
                    this.drawingShape.push(point);
                }
                this.triggerChange();
                this.onChange();
            }
            else {
                var isRefPoint1, isRefPoint2;
                if (this.operation == "reflect") {
                    isRefPoint1 = this.distance(this.lineOfReflection.point1, this.downPosition) < this.options.pointOfLORRad;
                    if (!isRefPoint1) isRefPoint2 = this.distance(this.lineOfReflection.point2, this.downPosition) < this.options.pointOfLORRad;
                }
                
                if (isRefPoint1 || isRefPoint2) {
                    this.touchedObject = "reflectionLine";
                    if (isRefPoint1) this.touchedObjectIndex = 1;
                    else this.touchedObjectIndex = 2;
                }
                else {
                    for(var i=0; i<shapes.length; i++){
                        var shape = shapes[i];
                        if (shape.drag){
                            
                            if (this.operation=="vertexControl") {
                                for(var p=0; p<shape.points.length; p++)
                                if(this.distance(this.downPosition, shape.points[p]) < this.options.lineTouchSense) {//this.options.vertexControlRad
                                    this.touchedShapeIndex = i;
                                    this.touchedObjectIndex = p;
                                    this.touchedObject = "vertexControl";
                                    
                                    break;
                                }
                                else if(this.pointInShape(this.downPosition, shape.points)) {
                                    this.touchedShapeIndex = i;
                                    this.touchedObject = "";
                                    this.xOperation = "translate";
                                 
                                    break;
                                }
                            }
                            else {
                                if (this.operation=="rotate" && this.distance(this.downPosition, shape.pointOfRotation) < this.options.pointOfRotRad) {
                                    this.touchedShapeIndex = i;
                                    this.touchedObject = "rotationPoint";
                                    break;
                                }
                                else if(this.pointInShape(this.downPosition, shape.points)) {
                                    this.touchedShapeIndex = i;
                                    this.touchedObject = "";
                                    break;
                                }
                            }
                        }
                    }
                    if (this.touchedShapeIndex >= 0) {
                        this.selectedShapeIndex = this.touchedShapeIndex;
                        this.selectedItemType = "shape";
                        this.formHistory();
                        var shape = shapes[this.touchedShapeIndex];
                        this.tempData = JSON.parse(JSON.stringify(shape));
                        shape.distFrmCP = this.distance(shape.pointOfRotation, shape.midPoint);
                        shape.angleOfRotation = this.angleBetweenPoints(shape.pointOfRotation, shape.midPoint);
                        if (this.operation=="reflect") this.reflect(shape);
                        this.onChange();
                    }
                }
            }
        }
            
            //if (this.selectedItemType == "shape") {
            //this.setOperation("translate");
            //}
            //else
            //{
            //    this.setOperation("vertexcontrol");
            //}
            //  
            
               
       
    },
    reflect:function(shape){
        for(var i=0; i<shape.points.length; i++) shape.points[i] = this.reflectPoint(shape.points[i]);
        shape.midPoint = this.reflectPoint(shape.midPoint);
        var _ang = shape.angle+180;
        shape.angle = shape.angle-_ang;
        this.updateAngles(shape);
        this.drawShapes(true, true);
        var newPosition = this.rePosition(shape.points);
        shape.midPoint = newPosition.midPoint;
        shape.points = newPosition.points;
        this.drawShapes();
    },
    reflectPoint:function(point){
        var dist = this.distance(point, this.lineOfReflection.midPoint);
        var _angle = this.angleBetweenPoints(point, this.lineOfReflection.midPoint)*180/Math.PI
        var angle = this.lineOfReflection.angle-_angle;
        angle = this.lineOfReflection.angle+angle;
        return this.APLtoPoint(angle, this.lineOfReflection.midPoint, dist);
    },
    distance:function(point1, point2) {
	return Math.sqrt(Math.pow(point1.x-point2.x,2) + Math.pow(point1.y-point2.y,2));
    },
    angleBetweenPoints:function(point1, point2) {
	return Math.atan2(point1.y-point2.y, point1.x-point2.x);
    },
    midPoint:function(points){
        var xSum = 0, ySum = 0;
        for (var i=0; i<points.length; i++) {
            xSum += points[i].x;
            ySum += points[i].y;
        }
        return {x:xSum/points.length, y:ySum/points.length};
    },
    correctPlotPosition:function(existingPoint, isPrimaryPoint, data, addData){
        this.tempGraphDetails = this.graphDetails();
        var newPoint = {x:existingPoint.x, y:existingPoint.y};
        if(this.options.snapEveryValue){
            var xSnapValue = this.options.snapEveryValue*this.tempGraphDetails.data.tick.xInt;
            var ySnapValue = this.options.snapEveryValue*this.tempGraphDetails.data.tick.yInt;
            newPoint.x = (Math.round((newPoint.x-this.options.margins.left)/xSnapValue)*xSnapValue)+this.options.margins.left;
            newPoint.y = (Math.round((newPoint.y-this.options.margins.top)/ySnapValue)*ySnapValue)+this.options.margins.top;
        }
        else if (this.options.snapTolerence) {
            var xFloor = Math.floor(newPoint.x);
            var yFloor = Math.floor(newPoint.y);
            if (xFloor){
                if (newPoint.x-xFloor < this.options.snapTolerence) newPoint.x = xFloor;
                else if (newPoint.x-xFloor > 1-this.options.snapTolerence) newPoint.x = xFloor+1;
            }
            if (yFloor){
                if (newPoint.y-yFloor < this.options.snapTolerence) newPoint.y = yFloor;
                else if (newPoint.y-yFloor > 1-this.options.snapTolerence) newPoint.y = yFloor+1;
            }
        }
        
        if (newPoint.x < this.tempGraphDetails.data.tick.xMin) newPoint.x = this.tempGraphDetails.data.tick.xMin;
        else if (newPoint.x > this.tempGraphDetails.data.tick.xMax) newPoint.x = this.tempGraphDetails.data.tick.xMax;
        if (newPoint.y < this.tempGraphDetails.data.tick.yMin) newPoint.y = this.tempGraphDetails.data.tick.yMin;
        else if (newPoint.y > this.tempGraphDetails.data.tick.yMax) newPoint.y = this.tempGraphDetails.data.tick.yMax;
        
        if (data) {
            if (addData == "point") {
                for(var r=0; r<this.plotterData.relation.length; r++){
                    var relation = this.plotterData.relation[r];
                    if (relation.pId == data.touchedObjIndex) {
                        for(var l=0; l<relation.lIds.length; l++){
                            var lId = relation.lIds[l];
                            var end = relation.ends[l];
                            var rLPoint = adjShape.plotterData.lines[lId][end];
                            rLPoint.x = newPoint.x;
                            rLPoint.y = newPoint.y;
                        }
                    }
                }
            }
        }
        return newPoint;
    },
    onMove:function(event){
        var shapes = this.shapeData;
        var touches = this.touchCoordsFromEvent(event, false);
        if (touches.length != 1) return;
        var xPos = touches[0].x-this.canvasHolder.offset().left;
        var yPos = touches[0].y-this.canvasHolder.offset().top;
	if (this.downPosition) {
            var posVariation = {x:this.downPosition.x-xPos, y:this.downPosition.y-yPos};
            if (!this.tempGraphDetails) this.tempGraphDetails = this.graphDetails();
            var posVar = {x: this.xToPoint(posVariation.x, this.tempGraphDetails, true), y: this.yToPoint(posVariation.y, this.tempGraphDetails, true)};
            if (this.operation == "plotPoint") {
                var point = this.touchedObjectProp.object.from;
                point.x = this.tempData.from.x - posVar.x;
                point.y = this.tempData.from.y - posVar.y;
                var newPos = this.correctPlotPosition(point, true);
                point.x = newPos.x;
                point.y = newPos.y;
                this.drawShapes();
            }
            else if (this.operation == "drawCircle" || this.operation == "drawLine") {
                var object = this.touchedObjectProp.object;
                if (this.operation == "drawLine") {
                    object.to.x = this.tempData.to.x - posVar.x;
                    object.to.y = this.tempData.to.y - posVar.y;
                    var newPos = this.correctPlotPosition(object.to, true);
                    object.to.x = newPos.x;
                    object.to.y = newPos.y;
                    this.snapToShape(object, false, false);
                }
                else {
                    object.anotherPoint.x = this.tempData.anotherPoint.x - posVar.x;
                    object.anotherPoint.y = this.tempData.anotherPoint.y - posVar.y;
                    var newPos = this.correctPlotPosition(object.anotherPoint, true);
                    object.anotherPoint.x = newPos.x;
                    object.anotherPoint.y = newPos.y;
                }
                this.drawShapes();
            }
            else if (this.operation == "dragLnP") {
                var data = this.touchedObjectProp.data;
                if (data) {
                    var object = this.plotterData[data.touchedObj+"s"][data.touchedObjIndex];
                    if (object) {
                        var pointsToUpdate = [];
                        if (data.touchedObj == "line") {
                            var point = data.touchedInternalObj.toLowerCase().replace("point", "");
                            if (data.touchedInternalObj == "fromPoint" || data.touchedInternalObj == "toPoint")
                            pointsToUpdate.push({name:point, addData:"line"});
                            else{
                                pointsToUpdate.push({name:"from", addData:"line"});
                                pointsToUpdate.push({name:"to", addData:"line"});
                            }
                        }
                        else if (data.touchedObj == "circle") {
                            if (data.touchedInternalObj == "midPoint") pointsToUpdate.push({name:"midPoint"});
                            pointsToUpdate.push({name:"anotherPoint"});
                        }
                        else if (data.touchedObj == "point") pointsToUpdate.push({name:"from",addData:"point"});
                        var xRed=0; yRed=0;
                        for (var oP=0; oP<pointsToUpdate.length; oP++) {
                            var point = pointsToUpdate[oP];
                            object[point.name].x = this.tempData[point.name].x - posVar.x;
                            object[point.name].y = this.tempData[point.name].y - posVar.y;
                            var newPos = this.correctPlotPosition(object[point.name], true, data, point.addData);
                            var _xRed = newPos.x-object[point.name].x;
                            var _yRed = newPos.y-object[point.name].y;
                            if (Math.abs(_xRed) > Math.abs(xRed)) xRed = _xRed;
                            if (Math.abs(_yRed) > Math.abs(yRed)) yRed = _yRed;
                        }
                        if (xRed || yRed)
                            for (var oP=0; oP<pointsToUpdate.length; oP++){
                                var point = pointsToUpdate[oP];
                                object[point.name].x += xRed;
                                object[point.name].y += yRed;
                            }
                        this.drawShapes();
                    }
                }
            }
            else if(this.operation == "createShape" && this.drawingShape.length > 0) {
                var pointToMoveIndex;
                pointToMoveIndex = this.drawingShape.length-1;
                if (this.touchedObjectIndex >= 0 && this.touchedObject == "createVertex") pointToMoveIndex = this.touchedObjectIndex;
                var pointToMove = {x:xPos, y:yPos};
                var res = this.isWithinArea(pointToMove, 0);
                pointToMove.x += res._x;
                pointToMove.y += res._y;
                this.drawingShape[pointToMoveIndex] = pointToMove;
                this.drawShapes();
            }
            else if ((this.touchedShapeIndex >= 0 && this.touchedShapeIndex < shapes.length) || this.touchedObjectIndex >= 0) {
                var shape = shapes[this.touchedShapeIndex];
                if (this.touchedShapeIndex >= 0 && (this.operation == "translate" || this.operation == "rotate" || this.xOperation == "translate")) {
                    if (shape.drag) {
                        if (this.operation=="translate" && this.tempData.points || this.xOperation == "translate") {
                            this.translateTo(posVariation, false, shape);
                            this.snapToShape(this.plotterData.lines[0], false, true);
                        }
                        else if (this.operation=="rotate"){
                            if(this.touchedObject == "rotationPoint"){
                                //shape.pointOfRotation.x = this.tempData.pointOfRotation.x - posVariation.x;
                              //  shape.pointOfRotation.y = this.tempData.pointOfRotation.y - posVariation.y;
                              //  this.setPointOfRotation(shape.pointOfRotation, shape);
                            }
                            else{
                               // var angle = this.angleBetweenPoints({x:xPos, y:yPos},shape.pointOfRotation)*180/Math.PI;
                               // this.rotateTo(angle, shape);
                            }
                        }
                    }
                }
                else if (this.operation=="reflect" && this.touchedObjectIndex > 0) {
                    var touchedPoint = this.lineOfReflection["point"+this.touchedObjectIndex];
                    touchedPoint.x = xPos;
                    touchedPoint.y = yPos;
                    var moveResult = this.isWithinArea(touchedPoint, this.options.pointOfLORRad);
                    if (moveResult.result) {
                        touchedPoint.x += moveResult._x;
                        touchedPoint.y += moveResult._y;
                    }
                    
                    this.drawShapes();
                }
                else if(this.operation=="vertexControl" && this.touchedObjectIndex >= 0){
                    var newPoint = JSON.parse(JSON.stringify(this.tempData.points[this.touchedObjectIndex]));
                    
                    newPoint.x -= posVariation.x;
                    newPoint.y -= posVariation.y;
                    var res = this.isWithinArea(newPoint, 0);
                    if(res.result){
                        newPoint.x += res._x;
                        newPoint.y += res._y;
                    }
                    
                    shape.points[this.touchedObjectIndex] = newPoint;
                  
                    var midPoint = this.midPoint(shape.points);
                    shape.midPoint = midPoint;
                    //try{
                        this.snapToShape(this.plotterData.lines[0], false, true);
                    //}catch(e){}
                    
                    
                    this.updateAngles(shape);
                    this.drawShapes();
                }
            }
	}
    },
    isWithinArea:function(point, addMargin) {
        addMargin = addMargin ? addMargin : 0;
        var canvas = this.canvasSet[1];
	var result = {result: false, _x:0, _y:0};
        var margins = this.options.margins;
        var l = margins.left+addMargin, r = canvas.width()-margins.right-addMargin, t = margins.top+addMargin, b = canvas.height()-margins.bottom-addMargin;
	if (point.x < l) result._x = l-point.x;
	else if (point.x > r) result._x = r-point.x;
	if (point.y < t) result._y = t-point.y;
	else if (point.y > b) result._y = b-point.y;
	if (result._x || result._y) result.result = true;
	return result;
    },
    onEnd:function(event) {
        if (this.operation != "rotate" && this.options.isResetPOR) this.resetPointOfRotation();
        if (this.touchedShapeIndex >= 0 && this.downPosition) this.recordChange();
        if (this.operation == "dragLnP" || this.operation == "drawLine" || this.operation == "plotPoint") {
            this.snapPlottings();
            if (this.operation == "dragLnP" && this.downPosition) {
                this.snapToShape(this.plotterData.lines[0], true);
            }
        }
        if (this.operation == "drawCircle") {
            if (!this.tempGraphDetails) this.tempGraphDetails = this.graphDetails();
            var circle = this.plotterData.circles[this.selectedObjectIndex];
            if (circle)
                if (circle.midPoint.x == circle.anotherPoint.x && circle.midPoint.y == circle.anotherPoint.y) {
                    var anotherPoint_bak = JSON.parse(JSON.stringify(circle.anotherPoint));
                    circle.anotherPoint.x += this.tempGraphDetails.data.tick.xInt;
                    circle.anotherPoint.y += this.tempGraphDetails.data.tick.yInt;
                    var res = this.isWithinArea(this.pointToPos(circle.anotherPoint.x, circle.anotherPoint.y, this.tempGraphDetails, 0, false), 0);
                    if (res._x) circle.anotherPoint.x = anotherPoint_bak.x-this.tempGraphDetails.data.tick.xInt;
                    if (res._y) circle.anotherPoint.y = anotherPoint_bak.y-this.tempGraphDetails.data.tick.yInt;
                }
        }
        if (this.touchedShapeIndex >= 0 || this.touchedObjectIndex >= 0 || this.downPosition) {
            this.onChange();
        }
        this.formHistory();
        this.touchedShapeIndex = -1;
        this.touchedObjectIndex = -1;
        this.tempData = {};
        this.touchedObjectProp = {};
        this.downPosition = null;
        this.drawShapes();
    },
    removeSelected:function(){
        var itemType = this.selectedItemType;
        var index = this.selectedObjectIndex;
        this.removePlotData(itemType, index);
    },
    removePlotData:function(itemType, index){
        try{
            if (itemType && index >= 0) {
                this.plotterData[itemType+"s"].splice(index, 1);
                for (var r=this.plotterData.relation.length-1; r>=0; r--) {
                    var relation = this.plotterData.relation[r];
                    if (itemType == "line")
                        for (var l=relation.lIds.length-1; l>=0; l--) {
                            if(relation.lIds[l] == index){
                                relation.lIds.splice(l, 1);
                                relation.ends.splice(l, 1);
                            }
                            else if (relation.lIds[l] > index) relation.lIds[l]--;
                        }
                    else if (itemType == "point"){
                        if(relation.pId == index) this.plotterData.relation.splice(r, 1);
                        else if(relation.pId > index) relation.pId --;
                    }
                }
            }
            this.selectedItemType = "";
            this.selectedObjectIndex = -1;
            this.drawAll(true);
            this.onChange();
        }
        catch(e) { this.warn(e); }
    },
    garbageCollector:function(){
        for (var l=this.plotterData.lines.length-1; l>=0; l--) {
            var line = this.plotterData.lines[l];
            if (line.from.x == line.to.x && line.from.y == line.to.y)
                this.removePlotData("line", l);
        }
    },
    sortByKey: function(array, key) {
        return array.sort(function(a, b) {
            var x = a[key]; var y = b[key];
            return ((x < y) ? -1 : ((x > y) ? 1 : 0));
        });
    },
    snapToShape:function (linePoints, canRemove, shouldStick){
        //var data = this.touchedObjectProp.data;
        if (linePoints) {
            var object = linePoints;
            var internalPoint;
            if (!shouldStick) {
            this.fromPoints = [];
            this.toPoints = [];
            for (var s=0; s<this.shapeData.length; s++) {
                for (var sp=0; sp<this.shapeData[s].points.length; sp++) {
                    var _p1 = object.from;
                    var pointPos1 = this.pointToPos(_p1.x/this.tempGraphDetails.data.tick.xInt, _p1.y/this.tempGraphDetails.data.tick.yInt, this.tempGraphDetails, 0, false);
                    var pos1ToPoint = this.coordsToPoint(this.shapeData[s].points[sp],this.graphDetails(),false);
                    this.fromPoints.push({"distance": this.distance(pointPos1, this.shapeData[s].points[sp]),"point":pos1ToPoint, "shape":s, "snapPoint":sp});
                    
                    var _p2 = object.to;
                    var pointPos2 = this.pointToPos(_p2.x/this.tempGraphDetails.data.tick.xInt, _p2.y/this.tempGraphDetails.data.tick.yInt, this.tempGraphDetails, 0, false);
                    var pos2ToPoint = this.coordsToPoint(this.shapeData[s].points[sp],this.graphDetails(),false);
                    this.toPoints.push({"distance": this.distance(pointPos2, this.shapeData[s].points[sp]),"point":pos2ToPoint, "shape":s, "snapPoint":sp});
                }
            }
              
            this.fromPoints = this.sortByKey(this.fromPoints, 'distance');
            
            this.toPoints = this.sortByKey(this.toPoints, 'distance');
            
            this.plotterData.lines[0].from.x = parseFloat(this.fromPoints[0].point.x.toFixed(2));
            this.plotterData.lines[0].from.y = parseFloat(this.fromPoints[0].point.y.toFixed(2));
            this.fromPoints[0].shouldStick = true;
            if (this.toPoints[0].distance<40) {
                this.plotterData.lines[0].to = this.toPoints[0].point;
                this.toPoints[0].shouldStick = true;
            }
            else{
                this.plotterData.lines[0].to = this.plotterData.lines[0].to;
                this.toPoints[0].point = this.plotterData.lines[0].to;
                this.toPoints[0].shouldStick = false;
            }
            if (this.fromPoints[0].point.x == this.toPoints[0].point.x && this.fromPoints[0].point.y == this.toPoints[0].point.y && canRemove) {
                this.plotterData.lines.length = 0;
            }
            }
            else {
                var pos1ToPoint = this.coordsToPoint(this.shapeData[this.fromPoints[0].shape].points[this.fromPoints[0].snapPoint],this.graphDetails(),false);
                this.plotterData.lines[0].from = pos1ToPoint;
                
                var pos2ToPoint = this.coordsToPoint(this.shapeData[this.toPoints[0].shape].points[this.toPoints[0].snapPoint],this.graphDetails(),false);
                if (this.toPoints[0].shouldStick==true) {
                    this.plotterData.lines[0].to = pos2ToPoint;
                }else {
                    this.plotterData.lines[0].to = this.plotterData.lines[0].to;
                }
                
            }
        }
    },
    snapPlottings:function(){
        var data = this.touchedObjectProp.data;
        if (data) {
            var object = this.plotterData[data.touchedObj+"s"][data.touchedObjIndex];
            var internalPoint;
            if (data.touchedInternalObj)
           
                internalPoint = data.touchedInternalObj.toLowerCase().replace("point","");
            if (data.touchedObj == "line") {
                for (var r=this.plotterData.relation.length-1; r>=0; r--) {
                    var relation = this.plotterData.relation[r];
                    var index = relation.lIds.indexOf(data.touchedObjIndex);
                    if (index >= 0)
                        if(relation.ends[index] == internalPoint || !internalPoint || internalPoint == "mid"){
                            relation.lIds.splice(index, 1);
                            relation.ends.splice(index, 1);
                        }
                }
            }
            var relatedObj = [];
            if (data.touchedObj == "line") {
                this.garbageCollector();
                var sourcePoints = [];
                if(internalPoint == "mid") sourcePoints = [{name:"from", point:object["from"]}, {name:"to", point:object["to"]}];
                else sourcePoints = [{name:internalPoint, point:object[internalPoint]}];
                for (var p=0; p<this.plotterData.points.length; p++) {
                    var point = this.plotterData.points[p].from;
                    for (var s=0; s<sourcePoints.length; s++) {
                        var sourcePoint = sourcePoints[s].point;
                        if (Math.abs(point.x-sourcePoint.x) < this.options.linePointSnapVal && Math.abs(point.y-sourcePoint.y) < this.options.linePointSnapVal) {
                            relatedObj.push({pId:p,lId:data.touchedObjIndex,end:sourcePoints[s].name});
                            break;
                        }
                    }
                }
            }
            else if (data.touchedObj == "point") {
                var sourcePoint = object.from;
                for (var l=0; l<this.plotterData.lines.length; l++) {
                    var linePoints = [{name:"from", point:this.plotterData.lines[l].from}, {name:"to", point:this.plotterData.lines[l].to}];
                    
                    for (var s=0; s<linePoints.length; s++) {
                        var point = linePoints[s].point;
                        if (Math.abs(point.x-sourcePoint.x) < this.options.linePointSnapVal && Math.abs(point.y-sourcePoint.y) < this.options.linePointSnapVal) {
                            relatedObj.push({pId:data.touchedObjIndex,lId:l,end:linePoints[s].name});
                            break;
                        }
                    }
                }
            }
            if (relatedObj.length >= 0){
                for(var r=0; r<relatedObj.length; r++) {
                    var rO = relatedObj[r];
                    var lineAlreadyCommitted = false;
                    for(var _r=0; _r<this.plotterData.relation.length; _r++){
                        var _rel = this.plotterData.relation[_r];
                        for(var _l=0; _l<_rel.lIds.length; _l++)
                            if(_rel.lIds[_l] == rO.lId && _rel.ends[_l] == rO.end){
                                lineAlreadyCommitted = true;
                                break;
                            }
                    }
                    if (!lineAlreadyCommitted) {
                        var existingRelObjIndex = this.indexOfByObjsElement(this.plotterData.relation, ["pId"], [rO.pId]);
                        var relationObj;
                        if (existingRelObjIndex.length >= 1) relationObj = this.plotterData.relation[existingRelObjIndex[0]];
                        else {
                            relationObj = JSON.parse(this.relationTemplate);
                            relationObj.pId = rO.pId;
                            this.plotterData.relation.push(relationObj);
                        }
                        if (relationObj.lIds.indexOf(rO.lId)<0) {
                            relationObj.lIds.push(rO.lId);
                            relationObj.ends.push(rO.end);
                        }
                    }
                }
            }
        }
    },
    recordChange:function(){
        if (this.selectedItemType == "shape") {
            var shape = this.shapeData[this.selectedShapeIndex];
            var compResult = this.compareDatas(this.tempData, shape);
            if (compResult.change){
                this.triggerChange();
                this.storeAsHistory(compResult);
            }
        }
    },
    compareDatas:function(data1, data2){
        var result = {change:false, subject:""};
        try {
            if (this.operation == "rotate") {
                if (!result.change) {
                    var a1 = Math.round(data1.angle);
                    var a2 = Math.round(data2.angle);
                    var angularVar = a2-a1;
                    if(angularVar) {
                        result.change = true;
                        result.subject = "Angle, " + (angularVar >= 0 ? "+":"-") + Math.abs(angularVar);
                    }
                }
            }
            else if (this.operation=="reflect" || this.operation=="translate") {
                if (!result.change) {
                    var x1 = data1.midPoint.x;
                    var y1 = data1.midPoint.y;
                    var x2 = data2.midPoint.x;
                    var y2 = data2.midPoint.y;
                    var xVar = x2-x1, yVar = y2-y1;
                    if(this.options.isDrawGrid){
                        var graphDetails = this.graphDetails();
                        xVar = this.roundTo(xVar/graphDetails.tickSpace.x, 2);
                        yVar = this.roundTo(yVar/graphDetails.tickSpace.y, 2);
                    }
                    if(xVar || yVar) {
                        result.change = true;
                        if (this.operation == "translate") result.subject = "Position, (" + (xVar >= 0 ? "+":"-") + Math.abs(xVar) + " , " + (yVar >= 0 ? "+":"-") + Math.abs(yVar) + ")";
                        else if (this.operation == "reflect") result.subject = "Reflect";
                    }
                    
                }
            }
            if(this.operation=="vertexControl") {
                for(var p=0; p<data1.points.length; p++) {
                    var pointOf1 = data1.points[p];
                    var pointOf2 = data2.points[p];
                    if (Math.abs(pointOf1.x-pointOf2.x) > 0 || Math.abs(pointOf1.y-pointOf2.y) > 0) {
                        result.change = true;
                        result.subject = "Shape modified";
                    }
                }
            }
        }
        catch(e){ }
        return result;
    },
    addValueToPoints:function(points, xVal, yVal){
        for(var p=0; p<points.length; p++){
            points[p].x += xVal;
            points[p].y += yVal;
        }
    },
    rePosition:function(inPoints) {
        var points = JSON.parse(JSON.stringify(inPoints));
        var max_x = null, max_y = null, min_x = null, min_y = null;
        var isOut = false;
        for(var p=0; p<points.length; p++){
            var res = this.isWithinArea(points[p]);
            if (res.result) isOut = true;
            if (max_x==null || max_x < res._x) max_x = res._x;
            if (max_y==null || max_y < res._y) max_y = res._y;
            if (min_x==null || min_x > res._x) min_x = res._x;
            if (min_y==null || min_y > res._y) min_y = res._y;
        }
	var midPoint = null;
	var change = false;
	if (isOut) {
            change = true;
            if (Math.abs(max_x) > Math.abs(min_x)) this.addValueToPoints(points, max_x, 0);
            else this.addValueToPoints(points, min_x, 0);
            
            if (Math.abs(max_y) > Math.abs(min_y)) this.addValueToPoints(points, 0, max_y);
            else this.addValueToPoints(points, 0, min_y);
	}
	midPoint = this.midPoint(points);
	return {points:points, midPoint:midPoint, change: change};
    },
    pointInShape:function(p, points) {
        var nvert = points.length;
        var i, j, isInShape = false;
        for(i=0,j=nvert-1;i<nvert;j=i++)
            if(((points[i].y>p.y)!=(points[j].y>p.y))&&(p.x<(points[j].x-points[i].x)*(p.y-points[i].y)/(points[j].y-points[i].y )+points[i].x )) isInShape = !isInShape;
        return isInShape;
    },
    pointsToOrientation:function(a, b, p) {
        var o = ((b.x - a.x) * (p.y - a.y)) - ((p.x - a.x) * (b.y - a.y));
        if (o > 0) return 1;
        else if (o < 0) return -1;
        else return 0;
    },
    translateTo:function(posVariation, isGridBased, shape, isRead) {
        var isRecord = false;
        if (!shape || isRead){
            if (!shape) shape = this.getSelectedShape();
            if (shape) {
                isRecord = true;
                this.tempData = JSON.parse(JSON.stringify(shape));
                var graphDetails = this.graphDetails();
                posVariation.x *= graphDetails.tickSpace.x;
                posVariation.y *= graphDetails.tickSpace.y;
            }
            
        }
        if (shape && (posVariation.x || posVariation.y)) {
            var shapePos = shape.midPoint;
            var newPoints = JSON.parse(JSON.stringify(shape.points));
            for(var p=0; p<newPoints.length; p++) {
                newPoints[p].x = this.tempData.points[p].x - posVariation.x;
                newPoints[p].y = this.tempData.points[p].y - posVariation.y;
            }
            shapePos.x -= posVariation.x;
            shapePos.y -= posVariation.y;
            var newPosition = this.rePosition(newPoints);
            shape.midPoint = newPosition.midPoint;
            shape.points = newPosition.points;
            this.snapToShape(this.plotterData.lines[0], false, true);
            this.drawShapes();
            if (isRecord) this.recordChange();
        }
    },
    rotateTo:function(angle, shape, isRead){
        var isRecord = false;
        if (!shape || isRead){
            if (!shape) shape = this.getSelectedShape();
            if (shape) {
                isRecord = true;
                this.tempData = JSON.parse(JSON.stringify(shape));
                shape.distFrmCP = this.distance(shape.pointOfRotation, shape.midPoint);
                shape.angleOfRotation = this.angleBetweenPoints(shape.pointOfRotation, shape.midPoint);
                this.tempData._storedAngle = 0;
            }
        }
        if (shape && angle) {
            var shapeCopy = JSON.parse(JSON.stringify(shape));
            var centerPoint = shape.pointOfRotation;
            var prevAngle = shape.angle;
            if (angle <= 0) angle += 360;
            if(this.tempData._storedAngle == null) this.tempData._storedAngle = angle;
            var anglarVar = angle-this.tempData._storedAngle;
            shape.angle = this.tempData.angle+anglarVar;
            var pointOfRotVar = {};
            var distFrmMid = shape.distFrmCP;
            var proposedPos = JSON.parse(JSON.stringify(shape.midPoint));
            if (distFrmMid) {
                var angleInRad = shape.angleOfRotation+(anglarVar*Math.PI/180);
                proposedPos.x = centerPoint.x-Math.cos(angleInRad)*distFrmMid;
                proposedPos.y = centerPoint.y-Math.sin(angleInRad)*distFrmMid;
            }
            shape.midPoint = proposedPos;
            this.drawShapes(true, true);
            var newPosition = this.rePosition(shape.points);
            if (Math.abs(newPosition.midPoint.x-shape.midPoint.x) > 1 || Math.abs(newPosition.midPoint.y-shape.midPoint.y)>1) this.shapeData[this.selectedShapeIndex] = shapeCopy;
            else {
                shape.midPoint = newPosition.midPoint;
                shape.points = newPosition.points;
                if (isRecord) this.recordChange();
            }
            this.drawShapes();
        }
    },
    setPointOfRotation:function(pos, shape, isRead){
        if (!shape || isRead){
            if (!shape) shape = this.getSelectedShape();
            if (shape) {
                this.tempData = JSON.parse(JSON.stringify(shape));
                var graphDetails = this.graphDetails();
                pos = this.pointToPos(pos.x, pos.y, graphDetails, 0, false);
            }
        }
        if (shape) {
            shape.pointOfRotation = pos;
            var result = this.isWithinArea(shape.pointOfRotation, 0);
            if (result) {
                shape.pointOfRotation.x += result._x;
                shape.pointOfRotation.y += result._y;
            }
            this.drawShapes();
        }
    },
    getSelectedShape:function(){
        var shape;
        var shapeIndex = this.touchedShapeIndex;
        if(shapeIndex < 0 && this.selectedShapeIndex >= 0) shapeIndex = this.selectedShapeIndex;
        if (shapeIndex >= 0 && shapeIndex < this.shapeData.length) shape = this.shapeData[shapeIndex];
        else{
            var editableShapes = this.indexOfByObjsElement(this.shapeData, ["drag"], [true]);
            if (editableShapes.length == 1){
                this.touchedShapeIndex = editableShapes[0];
                shape = this.shapeData[shapeIndex];
            }
        }
        this.selectedShapeIndex = shapeIndex;
        return shape;
    },
    saveGridSetting: function () {
        var data = this.options.gridProp;
        var tickData = data.tick;
        if (tickData.isActive) {
            var zoomState = this.getTickCount();
            var xZoomState = zoomState.x, yZoomState = zoomState.y;
            var xMin = Math.round(tickData.xMin);
            var xMax = Math.round(tickData.xMax);
            var yMin = Math.round(tickData.yMin);
            var yMax = Math.round(tickData.yMax);
            if (!tickData.isActive) {
                if (xMin > 0) xMin = 0;
                if (yMin > 0) yMin = 0;
                if (xMax < 0) xMax = 0;
                if (yMax < 0) yMax = 0;
            }
            
            var x_var = xMax - xMin;
            var y_var = yMax - yMin;
            if(!x_var) x_var = 1;
            if(!y_var) y_var = 1;
            
            if (x_var >= this.options.minTicks.x && x_var <= this.options.maxTicks.x) xZoomState = x_var;
            else xZoomState = this.axisCountCheck;
            if (y_var >= this.options.minTicks.y && y_var <= this.options.maxTicks.y) yZoomState = y_var;
            else yZoomState = this.axisCountCheck;
            
            /*var commonZoomState = Math.max(xZoomState, yZoomState);
            var otherCommonZoom = Math.min(xZoomState, yZoomState);
            if (commonZoomState > this.axisCountCheck && otherCommonZoom == this.axisCountCheck) commonZoomState = this.options.maxTicks.x;
            
            var xRat = 1;
            var yRat = 1;
            if (this.options.gridProp.ratio < 1) yRat = this.options.gridProp.ratio;
            else xRat = 1/this.options.gridProp.ratio;
            
            xZoomState = commonZoomState/xRat;
            yZoomState = commonZoomState/yRat;*/
            
            var xInt = Math.ceil(x_var / xZoomState);
            var yInt = Math.ceil(y_var / yZoomState);
            var xStart = Math.round(xZoomState * (xMin / x_var));
            var xEnd = xStart + xZoomState;
            var yStart = Math.round(yZoomState * (yMin / y_var));
            var yEnd = yStart + yZoomState;
            
            data.minX = xStart;
            data.maxX = xEnd;
            data.minY = yStart;
            data.maxY = yEnd;
            tickData.xInt = this.roundByJump(xInt);
            tickData.yInt = this.roundByJump(yInt);
            this.triggerChange();
        }
        this.drawAll(true, false);
    },
    roundByJump: function (val) {
        var len = val.toString().length;
        var mul = Math.pow(10, len - 1);
        val = Math.ceil(val / mul);
        if (val <= 1) val = 1;
        else if (val <= 2) val = 2;
        else if (val <= 5) val = 5;
        else val = 10;
        return (val * mul);
    },
    hardReset:function(){
        this.initShapeData = [];
        this.reset();
    },
    reset:function(){
        this.selectedShapeIndex = -1;
        this.touchedShapeIndex = -1;
        this.touchedObjectIndex = -1;
        this.tempData = {};
        this.downPosition = null;
        this.shapeData = JSON.parse(JSON.stringify(this.initShapeData));
        this.historyIndex = [];
        this.history = [];
        for(var i=0; i<this.shapeData.length; i++) this.historyIndex[i] = -1;
        this.lineOfReflection = JSON.parse(JSON.stringify(this.orginalLineOfRef));
        this.drawingShape = [];
        this.options.gridProp = JSON.parse(JSON.stringify(this.gridPropCopy));
        this.plotterData = JSON.parse(JSON.stringify(this.plotterDataCopy));
        this.fromPoints = [];
        this.toPoints = [];
        this.selectedShapeIndex = -1;
        this.selectedObjectIndex = -1;
        this.selectedItemType = "";
        this.drawAll(true);
    },
    cloneArray: function (array) {
        var newArray = array.slice(0);
        return newArray;
    },
    indexOfByObjsElement: function (array, keys, values, exKeys, exValues) {
        var indexes = [];
        if (!exKeys) exKeys = [];
        if (!keys) keys = [];
        for (var i = array.length - 1; i >= 0; i--) {
            var pass = true;
            for (key in array[i]) {
                if (pass) {
                    var keyIndx = keys.indexOf(key);
                    if (keyIndx >= 0 && pass) {
                        if (array[i][key] == values[keyIndx]) pass = true;
                        else pass = false;
                    }
                    keyIndx = exKeys.indexOf(key);
                    if (keyIndx >= 0 && pass) {
                        if (array[i][key] != exValues[keyIndx]) pass = true;
                        else pass = false;
                    }
                }
            }
            if (pass) indexes.push(i);
        }
        return indexes;
    },
    touchCoordsFromEvent:function(event, isDown){
        var touches = [];
        if (isDown) {
            if (this.isTouchDevice())
                for (var tInd = 0; tInd < event.originalEvent.touches.length; tInd++)
                    touches[tInd] = { x: event.originalEvent.touches[tInd].pageX, y: event.originalEvent.touches[tInd].pageY };
            else touches[0] = { x: event.pageX, y: event.pageY };
        }
        else{
            if (this.isTouchDevice())
                for (var tInd = 0; tInd < event.originalEvent.changedTouches.length; tInd++)
                    touches[tInd] = { x: event.originalEvent.changedTouches[tInd].pageX, y: event.originalEvent.changedTouches[tInd].pageY };
            else touches[0] = { x: event.pageX, y: event.pageY };
        }
        return touches;
    },
    isTouchDevice: function () {
        return "ontouchstart" in document;
    },
    PixelRatio: function () {
        var ctx = document.createElement("canvas").getContext("2d"),
            dpr = window.devicePixelRatio || 1,
            bsr = ctx.webkitBackingStorePixelRatio ||
                  ctx.mozBackingStorePixelRatio ||
                  ctx.msBackingStorePixelRatio ||
                  ctx.oBackingStorePixelRatio ||
                  ctx.backingStorePixelRatio || 1;

        return dpr / bsr;
    },
    setHiDPICanvas: function (can, w, h, ratio) {
        if (!ratio) { ratio = this.PixelRatio(); }
        can.width = w * ratio;
        can.height = h * ratio;
        can.style.width = w + "px";
        can.style.height = h + "px";
        can.getContext("2d").setTransform(ratio, 0, 0, ratio, 0, 0);
    },
    triggerChange: function () { },
    onChange: function () { },
    warn:function(message){
        
    }
}

//Adding DOM Methods
Element.prototype.remove = function() {
    this.parentElement.removeChild(this);
}
NodeList.prototype.remove = HTMLCollection.prototype.remove = function() {
    for(var i = 0, len = this.length; i < len; i++) {
        if(this[i] && this[i].parentElement) {
            this[i].parentElement.removeChild(this[i]);
        }
    }
}
Array.prototype.move = function (oldIndex, newIndex) {
    if (newIndex >= this.length) {
        var k = newIndex - this.length;
        while ((k--) + 1) {
            this.push(undefined);
        }
    }
    this.splice(newIndex, 0, this.splice(oldIndex, 1)[0]);
    return this;
};
String.prototype.splice = function(idx, rem, s) {
    return (this.slice(0,idx) + s + this.slice(idx + Math.abs(rem)));
};



activity.js
===========


 adjShape = new AdjustableShape(graphContainer, null, {isDrawGrid:false, snapEveryValue:0.5, maxTicks:{x:20, y:20}, gridProp : {minX:0, maxX:20, minY:0, maxY:20, xInterval:1, yInterval:1, tickLength: 10, tick:{isActive:true, xMin:0, xMax:20, yMin:0, yMax:20, xInt:1, yInt:1}, ratio:1}, snapEveryValue:0.5});
    
    adjShape.addShape({drag:true,coords:[{x:15,y:13},{x:14,y:8},{x:8,y:8},{x:5,y:13}]});
    
     adjShape.setOperation("vertexControl");
    adjShape.setOperation("translate");
