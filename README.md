# ea-json
If you JSON.stringify instances of Buffer or Date, it may work. Yet parsing the stringified data doesn't result in recreating the proper type. This module's parse method will look for a static fromJSON method whenever a json property contains a "type" and "data" tree, where type is the class name and data the data passed to the fromJSON method.

### Install
` npm install ea-json --save `

### Example 
```js

var json = require("ea-json")

// NOTE: usually you woud just define those methods inside some modules you require
// ----
Buffer.fromJSON = function(json) {
	return new Buffer(json,'base64');
}
Buffer.prototype.toJSON = function() {
	return {
		type: "Buffer",
		data: this.toString('base64')
	}
}

RegExp.fromJSON = ([source,global])=>new RegExp(source||'',(global?"g":""));
RegExp.prototype.toJSON = function() { return ({type:"RegExp", data:[this.source,this.global]}) }

Date.fromJSON = function(json) {
    return new Date(json);
}

Date.prototype.toJSON = function() { 
    return {type: 'Date', data: this.toISOString()}; 
}

// Sample Object
var sample = { 
    user: {
        name: 'earendel',
        email: 'kasia99@gmx.de'
    },
    // have added this to show you can (by some chance or at will) still 
    // have a node with a type and data key
    node: {
        type: 'SpaceShuttle',
        data: '\\_----@>'
    },
    date: new Date(),
    buffer: new Buffer('hello world')
};

//works!
var recreated = json.parse(json.stringify(sample));

console.log(require("util").inspect(recreated,{depth:null}));

```


### Appendix


```
//to be implemented 
//*
interface JSON.Serializable
{
    ({type: string, data: string}) toJSON(): [Function],
    ({instance of type}) static fromJSON(data): [Function]
}
/* */

//toJSON() and fromJSON() are functions to be implement by contract of the interface
```

```js
// a) require[type] = class constructor implementing JSON.serializable 
exports.require = {};

// b) like above but in global[type] 

// c) var T = eval(type); (T && typeof(T) == "function" && T.fromJSON) ..use required class constructors in local vars   

// d) alternatively just a function in: fromJSON[type] = (data) => {} 
exports.fromJSON = {};



//As initially intended JSON.parseTyped investigates class methods looking for a static .fromJSON() as a pendant to nativly lookup for customized prototype.toJSON() method. 
//if it finds a .fromJSON() for the v.type it invokes with v.data as argument  (if there is no {type} OR {data} or proper factory found, default JSON.parse is used on the entire tree v)    
var parse = exports.parse = (json,custom) => {
    return JSON.parse(json, (k,v) => {
        var type = v.type;
        var data = v.data;
		if((type !== undefined) && (data !== undefined)) { 
            var construct = exports.require[type] || global[type] || (/^[a-z_]+[a-z0-9_]*$/i.test(type) && eval(`typeof(${type})=="function" && ${type}`)); 
			if(construct && construct.fromJSON && typeof(construct.fromJSON) == "function" ) {
                return construct.fromJSON(data);
			} else if(exports.fromJSON[type] && typeof(exports.from[type]) == "function") {
				return exports.fromJSON[type](data);
            }
        }; 
        if(custom && typeof(custom)=="function") {
            return custom(k,v)
        } else {
            return v;
        }
    });
};  
var stringify = exports.stringify = (obj) => JSON.stringify(obj);


exports.sample = { 
    user: {
        name: 'earendel',
        email: 'kasia99@gmx.de'
    },
    // have added this to show you can (by some chance or at will) still 
    // have a node with a type and data key
    node: {
        type: 'SpaceShuttle',
        data: '\\_----@>'
    },
    date: new Date(),
    buffer: new Buffer('hello world')
};

```