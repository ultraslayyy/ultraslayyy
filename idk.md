From my experience. Most of these are from memory (besides helper funcs) so if there's small mistakes pls forgive me, because I didn't test too much. These may still be a thing on other targets, or on very specific setup, but I'll only share what I used to get this result.

## Decorators

<details>
    <summary>relevant tsconfig.json options used</summary>
    <pre>
        {
            "compilerOptions": {
                "target": "es2020",
                "experimentalDecorators": true
            }
        }
    </pre>
</details>

**Source TypeScript**
```ts
function Log(target: any, key: string) {
    console.log(`${key} called`);
}

class Example {
    @Log
    Method() {}
}
```

**Output JavaScript**
```js
var __decorate = (this && this.__decorate) || function (decorators, target, key, desc) {
    var c = arguments.length, r = c < 3 ? target : desc === null ? desc = Object.getOwnPropertyDescriptor(target, key) : desc, d;
    if (typeof Reflect === "object" && typeof Reflect.decorate === "function") r = Reflect.decorate(decorators, target, key, desc);
    else for (var i = decorators.length - 1; i >= 0; i--) if (d = decorators[i]) r = (c < 3 ? d(r) : c > 3 ? d(target, key, r) : d(target, key)) || r;
    return c > 3 && r && Object.defineProperty(target, key, r), r;
};
function Log(target, key) {
    console.log(`${key} called`);
}
class Example {
    Method() {}
}
__decorate([
    Log
], Example.prototype, "Method", null);
```

## Private fields ('#')
-# Pre-ESNext or ES2022

<details>
    <summary>relevant tsconfig.json options used</summary>
    <pre>
        {
            "compilerOptions": {
                "target": "es2020"
            }
        }
    </pre>
</details>

**Source TypeScript**
```ts
class Test {
    constructor(name: string, desc: string) {

    }
}

class Example {
    test: Test;
    #name: string;
    #desc: string;

    constructor(options: { name: string, desc: string }) {
        this.#name = options.name;
        this.#desc = options.desc;

        this.test = new Test(this.#name, this.#desc);
    }
}
```

**Output JavaScript**
```js
var __classPrivateFieldSet = (this && this.__classPrivateFieldSet) || function (receiver, state, value, kind, f) {
    if (kind === "m") throw new TypeError("Private method is not writable");
    if (kind === "a" && !f) throw new TypeError("Private accessor was defined without a setter");
    if (typeof state === "function" ? receiver !== state || !f : !state.has(receiver)) throw new TypeError("Cannot write private member to an object whose class did not declare it");
    return (kind === "a" ? f.call(receiver, value) : f ? f.value = value : state.set(receiver, value)), value;
};
var __classPrivateFieldGet = (this && this.__classPrivateFieldGet) || function (receiver, state, kind, f) {
    if (kind === "a" && !f) throw new TypeError("Private accessor was defined without a getter");
    if (typeof state === "function" ? receiver !== state || !f : !state.has(receiver)) throw new TypeError("Cannot read private member from an object whose class did not declare it"); return kind === "m" ? f : kind === "a" ? f.call(receiver) : f ? f.value : state.get(receiver);
};
var _Example_name, _Example_desc;
class Test {
    constructor(name, desc) {
    }
}
class Example {
    constructor(options) {
        _Example_name.set(this, void, 0);
        _Example_desc.set(this, void, 0);
        __classPrivateFieldSet(this, _Example_name, options.name, "f");
        __classPrivateFieldSet(this, _Example_desc, options.desc, "f");
        this.test = new Test(__classPrivateFieldGet(this, _Example_name, "f"), __classPrivateFieldGet(this, _Example_desc, "f"));
    }
}
_Example_name = new WeakMap();
_Example_desc = new WeakMap();
```

**Why?**
So, `#varname` does exist in both TypeScript and JavaScript **today**. But this is es2020. `#varname` syntax was added to JavaScript as part of ECMAScript **2022**, so they didn't exist yet. For TypeScript to be able to compile back to pre-es2022 targets, without causing errors, it has to add these helpers to recreate a similar functionality. This is a trend you may see in alot of examples.

## Some dynamic import idk (to commonjs)

<details>
    <summary>relevant tsconfig.json options used</summary>
    <pre>
        {
            "compilerOptions": {
                "target": "es2020",
                "module": "commonjs"
            }
        }
    </pre>
</details>

**Source TypeScript**
```ts
class Test {
    async Method(filename: string) {
        return await import(`./files/${filename}.js`);
    }
}
```

**Outputted JavaScript**
```js
var __createBinding = (this && this.__createBinding) || (Object.create ? (function(o, m, k, k2) {
    if (k2 === undefined) k2 = k;
    var desc = Object.getOwnPropertyDescriptor(m, k);
    if (!desc || ("get" in desc ? !m.__esModule : desc.writable || desc.configurable)) {
      desc = { enumerable: true, get: function() { return m[k]; } };
    }
    Object.defineProperty(o, k2, desc);
}) : (function(o, m, k, k2) {
    if (k2 === undefined) k2 = k;
    o[k2] = m[k];
}));
var __setModuleDefault = (this && this.__setModuleDefault) || (Object.create ? (function(o, v) {
    Object.defineProperty(o, "default", { enumerable: true, value: v });
}) : function(o, v) {
    o["default"] = v;
});
var __importStar = (this && this.__importStar) || (function () {
    var ownKeys = function(o) {
        ownKeys = Object.getOwnPropertyNames || function (o) {
            var ar = [];
            for (var k in o) if (Object.prototype.hasOwnProperty.call(o, k)) ar[ar.length] = k;
            return ar;
        };
        return ownKeys(o);
    };
    return function (mod) {
        if (mod && mod.__esModule) return mod;
        var result = {};
        if (mod != null) for (var k = ownKeys(mod), i = 0; i < k.length; i++) if (k[i] !== "default") __createBinding(result, mod, k[i]);
        __setModuleDefault(result, mod);
        return result;
    };
})();
class Test {
    async Method(filename) {
        return await Promise.resolve(`${`./files/${filename}.js`}`).then(s => __importStar(require(s)));
    }
}
```

## Default import (with commonjs)

<details>
    <summary>relevant tsconfig.json options used</summary>
    <pre>
        {
            "compilerOptions": {
                "module": "commonjs"
            }
        }
    </pre>
</details>

**Source TypeScript**
```ts
import something from './module';
```

**Outputted JavaScript**
```js
var __importDefault = (this && this.__importDefault) || function (mod) {
    return (mod && mod.__esModule) ? mod : { "default": mod };
};
var module_1 = __importDefault(require("./module"));
```

## pre-es2015 class inheritence

<details>
    <summary>relevant tsconfig.json options used</summary>
    <pre>
        {
            "compilerOptions": {
                "target": "es5",
            }
        }
    </pre>
</details>

**Source TypeScript**
```ts
class A {}
class B extends A {}
```

**Outputted JavaScript**
```js
var __extends = (this && this.__extends) || function (d, b) {
    for (var p in b) if (Object.prototype.hasOwnProperty.call(b, p)) d[p] = b[p];
    function __() { this.constructor = d; }
    d.prototype = b === null ? Object.create(b) : (__.prototype = b.prototype, new __());
};

var A = /** @class */ (function () {
    function A() {}
    return A;
}());
var B = /** @class */ (function (_super) {
    __extends(B, _super);
    function B() {
        return _super !== null && _super.apply(this, arguments) || this;
    }
    return B;
}(A));
```

## Async / Await and Generators (ES5/ES2015)

<details>
    <summary>relevant tsconfig.json options used</summary>
    <pre>
        {
            "compilerOptions": {
                "module": "es2015",
                "target": "es5"
            }
        }
    </pre>
</details>

**Source TypeScript**
```ts
async function test() {
    await new Promise(r => setTimeout(r, 100));
}
```

**Outputted JavaScript**
```js
var __awaiter = (this && this.__awaiter) || function (thisArg, _arguments, P, generator) {
    function adopt(value) { return value instanceof P ? value : new P(function (resolve) { resolve(value); }); }
    return new (P || (P = Promise))(function (resolve, reject) {
        function fulfilled(value) { try { step(generator.next(value)); } catch (e) { reject(e); } }
        function rejected(value) { try { step(generator["throw"](value)); } catch (e) { reject(e); } }
        function step(result) { result.done ? resolve(result.value) : adopt(result.value).then(fulfilled, rejected); }
        step((generator = generator.apply(thisArg, _arguments || [])).next());
    });
};
var __generator = (this && this.__generator) || function (thisArg, body) {
    var _ = { label: 0, sent: function() { if (t[0] & 1) throw t[1]; return t[1]; }, trys: [], ops: [] }, f, y, t, g = Object.create((typeof Iterator === "function" ? Iterator : Object).prototype);
    return g.next = verb(0), g["throw"] = verb(1), g["return"] = verb(2), typeof Symbol === "function" && (g[Symbol.iterator] = function() { return this; }), g;
    function verb(n) { return function (v) { return step([n, v]); }; }
    function step(op) {
        if (f) throw new TypeError("Generator is already executing.");
        while (g && (g = 0, op[0] && (_ = 0)), _) try {
            if (f = 1, y && (t = op[0] & 2 ? y["return"] : op[0] ? y["throw"] || ((t = y["return"]) && t.call(y), 0) : y.next) && !(t = t.call(y, op[1])).done) return t;
            if (y = 0, t) op = [op[0] & 2, t.value];
            switch (op[0]) {
                case 0: case 1: t = op; break;
                case 4: _.label++; return { value: op[1], done: false };
                case 5: _.label++; y = op[1]; op = [0]; continue;
                case 7: op = _.ops.pop(); _.trys.pop(); continue;
                default:
                    if (!(t = _.trys, t = t.length > 0 && t[t.length - 1]) && (op[0] === 6 || op[0] === 2)) { _ = 0; continue; }
                    if (op[0] === 3 && (!t || (op[1] > t[0] && op[1] < t[3]))) { _.label = op[1]; break; }
                    if (op[0] === 6 && _.label < t[1]) { _.label = t[1]; t = op; break; }
                    if (t && _.label < t[2]) { _.label = t[2]; _.ops.push(op); break; }
                    if (t[2]) _.ops.pop();
                    _.trys.pop(); continue;
            }
            op = body.call(thisArg, _);
        } catch (e) { op = [6, e]; y = 0; } finally { f = t = 0; }
        if (op[0] & 5) throw op[1]; return { value: op[0] ? op[1] : void 0, done: true };
    }
};
function test() {
    return __awaiter(this, void 0, void 0, function () {
        return __generator(this, function (_a) {
            switch (_a.label) {
                case 0: return [4 /*yield*/, new Promise(function (r) { return setTimeout(r, 100); })];
                case 1:
                    _a.sent();
                    return [2 /*return*/];
            }
        });
    });
}
```

## Iterables / Spread on Iterables

<details>
    <summary>relevant tsconfig.json options used</summary>
    <code>
        {
            "compilerOptions": {
                "module": "es2015",
                "target": "es5",
                "downlevelIteration": true
            }
        }
    </code>
</details>

**Source TypeScript**
```ts
for (const x of new Set([1, 2, 3])) console.log(x);
```

**Outputted JavaScript**
```js
var __values = (this && this.__values) || function(o) {
    var s = typeof Symbol === "function" && Symbol.iterator, m = s && o[s], i = 0;
    if (m) return m.call(o);
    if (o && typeof o.length === "number") return {
        next: function () {
            if (o && i >= o.length) o = void 0;
            return { value: o && o[i++], done: !o };
        }
    };
    throw new TypeError(s ? "Object is not iterable." : "Symbol.iterator is not defined.");
};
var e_1, _a;
try {
    for (var _b = __values(new Set([1, 2, 3])), _c = _b.next(); !_c.done; _c = _b.next()) {
        var x = _c.value;
        console.log(x);
    }
}
catch (e_1_1) { e_1 = { error: e_1_1 }; }
finally {
    try {
        if (_c && !_c.done && (_a = _b.return)) _a.call(_b);
    }
    finally { if (e_1) throw e_1.error; }
}
```

## Object spread

<details>
    <summary>relevant tsconfig.json options used</summary>
    <code>
        {
            "compilerOptions": {
                "module": "es2015",
                "target": "es5"
            }
        }
    </code>
</details>

**Source TypeScript**
```ts
const merged = { ...{ a: 1 }, b: 2 }
```

**Outputted JavaScript**
```js
var __assign = (this && this.__assign) || function () {
    __assign = Object.assign || function(t) {
        for (var s, i = 1, n = arguments.length; i < n; i++) {
            s = arguments[i];
            for (var p in s) if (Object.prototype.hasOwnProperty.call(s, p))
                t[p] = s[p];
        }
        return t;
    };
    return __assign.apply(this, arguments);
};
var merged = __assign({ a: 1 }, { b: 2 });
```