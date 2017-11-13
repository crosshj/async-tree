# async-tree   
Make it possible to depth-first traverse an async tree of events.

### example
 - Call #1, return a list of integers   
 - Call #2, for each integer make another call which returns another list of integers    
 - Call #3, acts like Call #2 but uses integers got from each Call #2 response   

This could be accomplished using async.waterfall and forEach on results at each level.  However, when using this approach, all Call #2's must complete before any Call #3 begins and there will be no results until all calls complete.  In addition, keeping track of results at each level when processing a tree is a bit tedious and verbose.   

With async-tree, a priority queue is created and the results of previous calls are passed with priority on first finishing each branch.   

The effect achieved is like processing a tree using async.waterfall, but traversal is handled in a more granular way and code is more terse and easier to test.

### usage
```
var asyncTree= require('./async-tree');

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

### notes
this.item is available in each function to track what arguments are passed to that function   
this.results contains results from previous calls in branch   
