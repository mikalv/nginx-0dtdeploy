#!/usr/bin/env node

process.title = 'nginx-0dtdeploy';

var argv = require('optimist').argv;
var fs = require('fs');

if (argv.f === undefined || argv.f === true) {
  console.log('We need a file to edit :) Set it with -f <pathtothefile>');
  return -2;
}

if (argv.h === undefined) {
  console.log('We need hosts defined :) Set it with -h <upstream_server>');
  return -4;
}

if (isArray(argv.h)) {
  if (argv.h.length == 0) {
    console.log('Error: No hosts defined! Use -h to define (multiple) server(s)');
    return -1;
  }
}

if (!fs.existsSync(argv.f)) {
  console.log('File don\'t exists!');
	return -3;
}

function isArray(testObj) {
  if (Object.prototype.toString.call(testObj) === '[object Array]' ) return true;
  return false;
}

var data = "";

if (isArray(argv.h)) {
  argv.h.forEach(function(e,i){
    data = data.concat("server ".concat(e.concat(";\n")));
  });
} else {
  data = data.concat("server ".concat(argv.h.concat(";\n")));
}

var filename = argv.f;

console.log('Output: ');
console.log(data);
