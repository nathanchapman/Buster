# Busted
[![npm version](https://img.shields.io/npm/v/busted.svg)](https://www.npmjs.org/package/busted)
[![npm downloads](https://img.shields.io/npm/dt/busted.svg)](https://www.npmjs.org/package/busted)
[![Travis CI](https://travis-ci.org/nathanchapman/busted.svg)](https://travis-ci.org/nathanchapman/busted)
[![Coverage Status](https://coveralls.io/repos/github/nathanchapman/busted/badge.svg?branch=master)](https://coveralls.io/github/nathanchapman/busted?branch=master)
[![Code Climate Grade](https://codeclimate.com/github/nathanchapman/busted/badges/gpa.svg)](https://codeclimate.com/github/nathanchapman/busted)
[![bitHound Overall Score](https://www.bithound.io/github/nathanchapman/busted/badges/score.svg)](https://www.bithound.io/github/nathanchapman/busted)
[![GitHub license](https://img.shields.io/github/license/nathanchapman/busted.svg)](https://github.com/nathanchapman/busted/blob/master/LICENSE)

<img align="right" width="300" src="http://www.dafont.com/forum/attach/orig/5/5/554705.png">

An npm package that detects improper iframe busting code and incorrect HTTP headers. "Clickjacking" is a malicious technique of tricking users into clicking on invisible iframes, and thus performing sensitive actions like sharing data or bank transfers without their knowledge. This tool attempts to find and offer suggestions to patch these vulnerabilities in your web applications. Also included are an [Electron](http://electron.atom.io/) application, [Chrome extension](https://chrome.google.com/webstore/detail/busted/nhlicbgagiecfehanppnblhoglonondm), and an [Arachni](http://www.arachni-scanner.com/) check.


## Install npm package
```sh
$ npm install busted
```

## Usage
In a Node.js application (no DOM access to iframes, so only headers test is functional):
```javascript
var busted = require('busted');
var URL = 'http://www.example.com';
busted.headersTest(URL, function(url, passed) {
  console.log(url + (passed ? ' passed ' : ' failed ') + 'the headers test.');
});
```

In any Electron application (has DOM access so iframes can be tested):
```javascript
window.BUSTED = require('busted.js');

var URL = 'http://www.example.com';
var iframe = document.getElementById('frame');
iframe.src = URL;

window.BUSTED.headersTest(URL, function(url, passed) {
  console.log(url + (passed ? ' passed ' : ' failed ') + 'the headers test.');
});

iframe.onload = function() {
  var passed = window.BUSTED.iframeTest(URL, iframe);
  console.log(URL + (passed ? ' passed ' : ' failed ') + 'the iframe test.');
}
```

## Using our Electron app
Our Electron app allows for scanning and testing URLs for the vulnerabilities.
#### Install
```sh
$ git clone https://github.com/nathanchapman/busted.git && cd busted/electron && npm install
```
#### Starting the app
From `busted/electron` run:
```sh
$ npm start
```

## Arachni Web Scanner Check
An Arachni check that tests headers and parses JavaScript for improper frame-busting code.
#### Install Arachni
Download and install [Arachni](http://www.arachni-scanner.com/download/).
#### Install Busted
```sh
$ git clone https://github.com/nathanchapman/busted.git
```
Move the `arachni/clickjacking.rb` file to:
`[ARACHNI_DIRECTORY]/system/gems/gems/arachni-1.4/components/checks/passive`
from within `[ARACHNI_DIRECTORY]` run the following:
```sh
$ bin/arachni --checks=clickjacking --scope-page-limit 1 --browser-cluster-pool-size 1 SOME_URL_OR_FILE
```
for more information, see the [Arachni CLI guide](https://github.com/Arachni/arachni/wiki/Command-line-user-interface).


## Clickjacking Prevention
There are other attacks that are site-specific or are possible if the attacker controls certian domain names. These attacks cannot be tested here, but we will attempt to prevent them with this two-fold solution.

__Headers:__ Set X-Frame-Options header to DENY or SAMEORIGIN

__Frame Busting Code:__ Use the following frame busting code as a solution for older browsers that do not support X-Frame-Options or Content-Security-Policy headers.
```html
<!-- Source: http://w2spconf.com/2010/papers/p27.pdf -->
<style>
html { display: none; }
</style>
<script>
if (self == top) {
  document.documentElement.style.display = 'block';
} else {
  top.location = self.location;
}
</script>
```
