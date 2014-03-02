# bitjs: Binary Tools for JavaScript

=Introduction
A set of tools to handle binary data in JS (using Typed Arrays).

=Example Usage
==bitjs.io
`This namespace includes stream objects for reading and writing binary data at the bit and byte level: BitStream, ByteStream.

var bstream = new bitjs.io.BitStream(someArrayBuffer, true, offset, length);
var crc = bstream.readBits(12); // read in 12 bits as CRC, advancing the pointer
var flagbits = bstream.peekBits(6); // look ahead at next 6 bits, but do not advance the pointer`

==bitjs.archive
`This namespace includes objects for unarchiving binary data in popular archive formats (zip, rar, tar) providing unzip, unrar and untar capabilities via JavaScript in the browser. The unarchive code depends on browser support of Web Workers. See the design doc.

function updateProgressBar(e) { ... update UI element ... }
function displayZipContents(e) { ... display contents of the extracted zip file ... }

var unzipper = new bitjs.archive.Unzipper(zipFileArrayBuffer);
unzipper.addEventListener("progress", updateProgressBar);
unzipper.addEventListener("finish", displayZipContents);
unzipper.start();
`

=Archive
The plan is to migrate decode.js, unzip.js, unrar.js and untar.js from kthoom into BitJS. The code is a little messy right now and not very easily extractable in library form, so the following is a design proposal for the API.

==Details
This code will live in the namespace: bitjs.archive

==UnarchiveEvent

The first thing to note is that decode.js currently depends on being in its own worker thread. Even though this is probably the preferred means of using the unarchiving code, I'd really like the code to not depend upon this (i.e. it should be possible to untar in the main window JS context).

The right way to handle this is to define events (bitjs.archive.UnarchiveEvent) that the unarchiving code sends out to listeners. Some UnarchiveEvent types:

`Start Event
Progress Event - sends progress update information
Extract Event - sends the extracted file
Finish Event
Error Event
Client code would then listen for unarchiving events and act accordingly. For example, the event listeners in a web worker could postMessage().
`

==UnarchivedFile

We'll define a bits.archive.UnarchivedFile object that has the following two properites:

filename - returns the full filename
fileData - returns a TypedArray representing the file's binary contents
These objects will be returned in Extracted File Events to the client code's listener.

==Unarchiver
We will define a base abstract class called bitjs.archive.Unarchiver:

the constructor accepts a Typed Array
an addEventListener() method
a removeEventListener() method
a run() method, which will start the unarchive process and return immediately
an abstract run() method which must be implemented by a subclass
Unzipper, Unrarrer, Untarrer
We will have three concrete subclasses of Unarchiver, one for each supported file format: zip, rar, tar that will each live in separate files. This helps save on JS load time/memory, since the web application only needs to include the files it needs.

NOTE: kthoom's file sniffing will necessarily have to be done first (using a bitjs.io stream, naturally) to determine which Unarchiver subclass to create and kick off.

=Credits
== Original developer: Jeff Schiller, http://www.codedread.com/
== Github maintainer: Varun Mayya, http://www.varunmayya.com