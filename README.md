# jrep
[![Version](https://img.shields.io/npm/v/jrep1.svg)](https://npmjs.org/package/jrep1)
[![CircleCI](https://circleci.com/gh/sohale/jrep/tree/main.svg?style=shield)](https://circleci.com/gh/sohale/jrep/tree/main)
[![Downloads/week](https://img.shields.io/npm/dw/jrep.svg)](https://www.npmjs.com/package/jrep1)
[![License](https://img.shields.io/npm/l/jrep1.svg)](https://github.com/sohale/jrep/blob/main/package.json)

A new generation `grep` written in node/javascript.
Pragmatic post-processing for various commands, in Node/Javascript's RegExp `/.../`.
Think of `awk` & `sed` with javascript syntax. 

* Very useful for bash scripting and devops works.
* Handy and extermly flexible.
* js-friednly
* No installation needed! Just use ` npx jrep1 `

Handy examples I regularly use:
```bash
# Generate a `date -r ` command for each modified file in your `git status`:
git status |  npx jrep1  '"date -r " + /modified: +(.*)/.exec(x)[1]'

# sort dates of most recently changed files (recursively)
ls -Ralth  FOLDER1 FOLDER2  |  npx jrep1  '/(Dec *\d+ [0-9\:]+.*)/.exec(x)[1]' |grep -w rs | sort |uniq

# sort dates of most recent changed files
ls -Ralth MYFOLDER1/ MYFOLDER2/ MYFILE3   | npx jrep1 '/((?:Feb|Jan) *(\d+) [0-9\:]+.*)/.exec(x).map(x=>x+"").reverse().join("")'
```

<!-- 🧤 🧰 ☘️ -->
## 🧤 Usage
> `npx jrep1`   \<map1\>   \<map2\> ...

or

> `jrep`   \<map1\>   \<map2\> ...

where each arg, is a Jacascript Expression with `x`,
`x` is each input line.

Each arg is in form os a **free-form js expression**.
For example:
   * `x`
   * `/REGEXP/.exec(x)[1]`
   * `x.substring(20)`
   * `x.substring(20).toUpperCase()`
   * 'x.replace(/hello/g, "okthxbye")'

The args can be chained.
\<map1\>   \<map2\>

In other wrods,
* Each arg is a string to string trnsformation, that is applied one line at a time.
  * is a string transformation (a *map* callback)
  * is the body of a **javascript** function with argument `x`
* The commandline args, <map>s, are applied sequentially on each line in the standard input content (eg piped content).
* In case of exception in any of the <filter>s, that line is ignored (not output error or exception is shown).

### 🎩 Built-in primitives
While the expressions can be any expression with `x`, there are some predefined functions
<!-- ### **RE1** -->
<!-- [^1] -->

| primitive      |meaning| usage |
| ----------- |----| ----------- |
| ✨ RE1 ✨     |find/replace regexp| `RE1(`  \<regular expression with parentheses\>  `)`       |
| ✨ RER ✨    |find + custom replace| `RER(`  regexp. `,` a string with `p[1]`,`p[2]`, ...   `)`       |
|             ||        |
### ⌨️ Examples
```bash
uname -a | npx jrep1  'x.replace(/a/g, "O")'
```
```bash
uname -a | npx jrep1  'x.replace(/[a-z]/g, "a").replace(/[A-Z]/g, "A").replace(/[0-9]/g, "0")'
```
```txt
Aaaaa aaaaaa-a-0aaaa-0aa-aaa0-00-aaaaaaa 0.00.0-000-aaaaaaa #000-Aaaaaa AAA Aaa Aaa 00 00:00:00 AAA 0000 a00_00 a00_00 a00_00 AAA/Aaaaa
```

#### ⛑ Practical examples
* Extract *pid*s <br/>
`ps aux|  npx jrep1  'x.substring(12,24)'`
* Extract the filenames in a find command:
```bash
find .|  npx jrep1  '/([^\/]*)$/.exec(x)[1]'
```
* Using env for filter names: Another way to write it is to use bash env vars for filter names. It is will be also more reusable. A more readable notation:
```bash
EXTRACT_BASENAME='/([^\/]*)$/.exec(x)[1]'
find / |  npx jrep1  $EXTRACT_BASENAME | head
```
* The `PSAUX_PID` can extract the PID from the output of the commonly used `ps aux` command (tested on MacOS). Is can be used to extract other columns 
```bash
PSAUX_PID='[.../([^ ]*) * (\d*) *(\d+\.\d+) *(\d+\.\d+) *(\d+) *(\d+) *([^ ]+) +([^ ]+) +([^ ]+) +([^ ]+) +(.*)$/.exec(x)].slice(1)[1]'
ps aux |  npx jrep1  "$PSAUX_PID"
```
* Another usage of `PSAUX_PID`:
```bash
ps aux | grep -v grep | grep http-server |  npx jrep1  "$PSAUX_PID" | xargs kill
```
* Emulate `cut` command: <br/>
`ps aux|  npx jrep1  'x.substring(12,24)'`
* Pairs of pid and their running time:
```bash
ps aux|  npx jrep1  "{time=x.substring(70,78); pid=x.substring(15,24); return pid+':'+time;}"
```
* Sort PIDs based on runing time:
```bash
ps aux|  npx jrep1  "{time=x.substring(70,78); pid=x.substring(15,24); return time + ':' + pid;}"|sort
```
* Replace letters with frog: (such a useful scenario)
```bash
uname -a |  npx jrep1  'x.replace(/[a-z]/g, "🐸")'
```
* Using ✨ **built-in primitives: `RE1`** ✨. The following two are equivalent:
```bash
ps aux |  npx jrep1  'RE1(".*(python.*)")'
ps aux |  npx jrep1  '/.*(python.*)/.exec(x)[1]'
```
* The following three are equivalent
```bash
ps aux |  npx jrep1  'RE1("(.*python.*)")'
ps aux |  npx jrep1  '/(.*python.*)/.exec(x)[1]'
ps aux | grep -e python
```
* Using ✨ **built-in primitive** RER ✨. Replace `python` in filenames with emojies.
```bash
find .. |  npx jrep1  'RER("(.*)python(.*)", "p[1]+\"🐍🐍\"+p[2]")'
```

### 🐻 Find jrep on npm: [jrep1](https://www.npmjs.com/package/jrep1)

### 💡 Some suggested use cases
* Eliminate usage of obscure aommands such as `awk`, `sed`, `perl`
* Unified solution without usual tools `cut`, `grep`
* Replace a matched regular expression with given custom ccombination (See `RER`)
* Extract part of an RE1 pattern (See `RE1`)

## 🤝
* 👋 Feel free to send Pull Requests.
* 👋 Feel free to request features.

<!-- ✨ Features -->
#### ✨ Pros
* 👍 transform (map) text in linux pipes
* 👍 filter text in linux pipes
* 👍 concise
* 👍 versatile
* 👍 exteremly flexible
* 👍 customisable
* 👍 prebuilt primitive operations
* 👍 Super lightweight
* 👍 Zero npm dependencies
* 👍 Docker version available
#### Cons
* 👎 Needs `node` installed on the system. 👍 If you don't want to install node, an alternative is to use `docker`.
#### 📌 Requirements
* NodeJS (tested on node 12 and  node v16.15.0)

## 💻 Installation
Thanks to `npx`,
**No installation needed**! Just use ` npx jrep1 `

Alternatiely, for a proper `jrep` command:
* 🐻 npm
```bash
npm i -g jrep1
# test:
uname -a | jrep 'x.replace("a", "O")'
```

* 💻 MacOS , Linux
```bash
git clone https://github.com/sohale/jrep.git
./jrep/scripts/install-macos.bash
```
<!-- Mac/Linux directly download the .js file from raw using curl -->

* 🐻 npm [jrep1 on npm](https://www.npmjs.com/package/jrep1)
```bash
npm install -g jrep
```
* 🐱 yarn
```bash
yarn install -g jrep
```

## 📚 Example Usage
Also see [test/e2e-test.bash](test/e2e-test.bash)

* 🐳 docker
No need to install Node
```bash
find / | \
    docker run -i sohale/jrep:latest \
       '/\/([^\/]*\.py)$/.exec(x)[1]'
```

<!-- 🐻🦦⌨️ -->
* ⌨️ npx
```bash
uname -a | npx jrep1 'x.replace("a", "O")'
```

* 📡 bash (on the fly)
```bash
find .. | \
    node -e "$(curl -L https://raw.githubusercontent.com/sohale/jrep/main/src/jrep.js | tail -n +2))" '' \
       '/\/([^\/]*\.py)$/.exec(x)[1]'
```
 * requires NodeJS (tested on node 12) to be installed on your system.

### 💻 Development ⌨️
[docs/internals.md](docs/internals.md)

## 📚 Tutorial
(comming soon)
