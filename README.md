node-transcode -- Media transcoding and streaming for node.js
====================================

node-transcode is a library for enabling both offline and real-time media
transcoding. In addition to enabling the manipulation of the input media,
utilities are provided to ease serving of the output.

Currently supported features:

* Nothing!

Coming soon (maybe):

* Everything!

## Quickstart

    npm install transcode
    node
    > var transcode = require('transcode');
    > transcode.process('input.flv', 'output.m4v',
        transcode.profiles.APPLE_TV_2, function(err, sourceInfo, targetInfo) {
          console.log('completed!');
        });

## Installation

With [npm](http://npmjs.org):

    npm install transcode

From source:

    cd ~
    git clone https://benvanik@github.com/benvanik/node-transcode.git
    npm link node-transcode/

### Dependencies

node-transcode requires the command-line `ffmpeg` tool to run. Make sure
it's installed and on your path.

#### Source

    ./configure \
        --enable-gpl --enable-nonfree --enable-pthreads \
        --enable-libfaac --enable-libfaad --enable-libmp3lame \
        --enable-libx264
    sudo make install

#### Mac OS X

The easiest way to get ffmpeg is via [MacPorts](http://macports.org).
Install it if needed and run the following from the command line:

    sudo port install ffmpeg +gpl +lame +x264 +xvid

You may also need to add the MacPorts paths to your `~./profile`:

    export C_INCLUDE_PATH=$C_INCLUDE_PATH:/opt/local/include/
    export CPLUS_INCLUDE_PATH=$CPLUS_INCLUDE_PATH:/opt/local/include/
    export LIBRARY_PATH=$LIBRARY_PATH:/opt/local/lib/

#### FreeBSD

    sudo pkg_add ffmpeg

#### Linux

    # HAHA YEAH RIGHT GOOD LUCK >_>
    sudo apt-get install ffmpeg

## API

### Media Information

Whenever 'info' is used, it refers to a MediaInfo object that looks something
like this:

    {
      container: 'flv',
      duration: 126,         // seconds
      start: 0,              // seconds
      bitrate: 818000,       // bits/sec
      streams: [
        {
          type: 'video',
          codec: 'h264',
          resolution: { width: 640, height: 360 },
          bitrate: 686000,
          fps: 29.97
        }, {
          type: 'audio',
          language: 'eng',
          codec: 'aac',
          sampleRate: 44100, // Hz
          channels: 2,
          bitrate: 131000
        }
      ]
    }

### Querying Media Information

To quickly query media information (duration, codecs used, etc) use the
`queryInfo` API:

    var transcode = require('transcode');
    transcode.queryInfo(source, function(err, info) {
      // Completed
    });

### Transcoding Profiles

Transcoding requires a ton of parameters to get the best results. It's a pain in
the ass. So what's exposed right now is a profile set that tries to set the
best options for you. Pick your profile and pass it into the transcoding APIs.

    var transcode = require('transcode');
    for (var profileName in transcode.profiles) {
      var profile = transcode.profiles[profileName];
      console.log(profileName + ':' + util.inspect(profile));
    }

### Simple Transcoding

If you are doing simple offline transcoding (no need for streaming, extra
options, progress updates, etc) then you can use the `process` API:

    var transcode = require('transcode');
    transcode.process(source, target, transcode.profiles.APPLE_TV_2, function(err, sourceInfo, targetInfo) {
      // Completed
    });

Note that this effectively just wraps the advanced API, without the need to
track events.

### Advanced Transcoding

    var transcode = require('transcode');
    var task = transcode.createTask(source, target, {
      profile: transcode.profiles.APPLE_TV_2
    });

    task.on('begin', function(sourceInfo, targetInfo) {
      // Transcoding beginning, info available
      console.log('transcoding beginning...');
      console.log('source:');
      console.log(util.inspect(sourceInfo));
      console.log('target:');
      console.log(util.inspect(targetInfo));
    });
    task.on('progress', function(timestamp, duration) {
      // New progress made, currrently at timestamp out of duration
      console.log('progress ' + progress.timestamp + ' / ' + progress.duration);
    });
    task.on('error', function(err) {
      // Error occurred, transcoding ending
      console.log('error: ' + err);
    });
    task.on('end', function() {
      // Transcoding has completed
      console.log('finished');
    });

    // Start transcoding
    task.start();

    // At any time, abort transcoding
    task.stop();