# async-tree
Make it possible to depth-first traverse an async tree of events.

## example
Call #1 returns a list of integers
Call #2 for each integer make another call which returns multiple integers for each integer
Call #3 acts like Call #2 but uses integers got from each Call #2 response

This could be accomplished using async waterfall, but all Call #2's must complete before all Call #3's and the same occurs for all Call #3's.
With async-tree, a priority queue is created and results of each Call #2 is fed to Call #3 with priority on finishing each branch.

The effect achieved is like async.waterfall, but traversal is handled in a more granular way.

## usage
```
var asyncTree= require('./asyncTree');

function getLevelOne(callback){
    var initialArgs = [0, 1, 2, 3, 4];
    callback(null, initialArgs);
}

function getLevelTwoItem(callback){
    var item = this.item;

    const results = new Array(item||1).fill().map(x => 2*item);
    setTimeout(function(){
        callback(null, results);
    }, [0, 2000, 2000, 3000, 1, 1][item]);
}

function getLevelThreeItem(callback){
    var item = this.item;
    console.log('---- ',this.results);
    setTimeout(function(){
        const results = new Array(item/2||1).fill().map(x => 2*item);
        callback(null, results);
    }, Math.floor(Math.random() * 3000));
}

var finalResultsArray = [];
function doneCallback(){
    console.log(`finished with ${finalResultsArray.length} results`);
}

function eachCallback(err, result){
    finalResultsArray.push(this.results);    
    console.log(result);
}

var functionArray = [
    getLevelOne,
    getLevelTwoItem,
    getLevelThreeItem
];

asyncTree({
    functionArray,
    concurrency: 2,
    delay: 100, //in ms
    doneCallback,
    eachCallback
});

```
