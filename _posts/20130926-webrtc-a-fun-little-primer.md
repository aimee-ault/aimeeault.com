title: WebRTC: A Fun Little Primer
link: http://aimeeault.com/2013/09/26/webrtc-a-fun-little-primer/
author: aimeeault
description: 
post_id: 168
created: 2013/09/26 21:41:06
created_gmt: 2013/09/26 21:41:06
comment_status: open
post_name: webrtc-a-fun-little-primer
status: publish
post_type: post

# WebRTC: A Fun Little Primer

A couple of months ago, I posted a "call to action" regarding the state of Google's MediaStream API, which is meant to be used with cool new web technology like WebRTC. You've probably seen WebRTC already. It allows you to record video and audio through your webcam without having to use Flash or some sort of third-party technology--it's all handled through the browser. This is particularly cool because it can allow for things like real-time video chatting without all parties having to download extra software (for example, Chat Roulette operates on Flash... if you hate Flash, you're fucked). With WebRTC, as long as your browser supports it, you're in. A lot of people don't know about WebRTC though and kind of float away from it because of that, so I'm going to share my experience with it in a non-complainy context because it's kind of awesome!  Right now, the backend of the API for MediaStream is unimplemented and as a result, most of the web applications using it today are only using it for exporting single-frame images. For example, if you are a Tumblr user, you've probably seen WebRTC in use! From your dashboard, if you click to add a photo, and then select "Take a Photo," Tumblr will ask your permission to use your camera through your browser. It'll then give you the option to take a single frame photo or build an animated GIF. ![](https://s3.amazonaws.com/aimeeault.com/screen_shot_2013_09_26_at_12_16_56_pm_by_fartprincess-d6nzdm3.png) I'm such a nice young lady. This is kind of mysterious though, right? Is the NSA spying on me through my webcam every time I log in to RedTube and watch porn? I mean, I don't know about you guys, but I do some pretty weird crap when I'm left alone with my laptop, like make weird dinosaur faces and yell at things and sing songs that I've adlibbed, so I don't want iSight to just randomly start recording and sending my shit off to some rogue website. But it's not really mysterious if you look into it some. 

# getUserMedia

So here's where we begin: 
    
    
    //crossbrowser support for getUserMedia
    navigator.getUserMedia = navigator.getUserMedia || navigator.webkitGetUserMedia || 
                             navigator.mozGetUserMedia || navigator.msGetUserMedia;

For the sake of avoiding hairy spaghetti code, I'm condensing all the different browsers' implementations of getUserMedia, which are vendor-prefixed into navigator.getUserMedia so that I can reuse that. getUserMedia is an HTML5 API that has been implemented. This is the API which will fetch input from your webcam (or other video input device) or microphone. If you give the request permission and it is successfully initiated, it will return a stream object to the success callback. The most obvious thing to do with this stream object is to give the user some affirmation that it has been successfully initiated by showing it back to them. Your browser will ask for your permission to let websites access your camera every single time it tries to do so, unless the site has an SSL certificate. In which case, you are given the option to permanently allow it access for that one site. So for the sake of simplifying things, let's pretend we're just recording video! 

# CreateObjectUrl
    
    
    var self = this;
    navigator.getUserMedia({video: true}, function (stream) {
      self.streamURL = window.URL.createObjectURL(stream);
      self.videoRecorder.src = self.streamURL;
    });

We unfortunately can't just blindly manipulate this stream object that getUserMedia has returned to us into anything that is useful by any element on a webpage. If we were using NodeJS, we could feed it into a peer-to-peer connection and do some super radtastic video conferencing, but in this example, we're not using NodeJS, we don't have mulithreading, nor are we dealing with any server-side languages. We're just using Javascript and HTML5. So what is that going to get us? Well. We can use the HTML5 File API to take our stream and turn it into a URI that is usable by elements on the webpage, using URL.createObjectURL. This is pretty nifty and can take either a file or a blob object (which streams are!). In my example above, self.videoRecorder is a reference to a video element on the page: ![screen_shot_2013_09_26_at_12_46_10_pm_by_fartprincess-d6nzi60](https://s3.amazonaws.com/aimeeault.com/screen_shot_2013_09_26_at_12_46_10_pm_by_fartprincess-d6nzi60.png) So, the video element is now playing a live stream through a blob it has been fed through our Javascript.Fancy. But I can't really do anything with this, can I? I mean, it's just a read-only video stream at this point. That's incredibly useless unless I just want a mirror to look at while I put on makeup or whatever. So what now? **This is when it gets interesting and kind of hackishly amusing.**

# Canvas Exporting to an Array. Oh god, why.

You can create an empty canvas element in your Javascript and draw frames from the video onto the canvas and export them. 
    
    
    var self = this;
    function drawVideoFrame_(time) {
      self.rafId = requestAnimationFrame(drawVideoFrame_);
      var canvas = document.createElement('canvas');
      var ctx = canvas.getContext('2d');
      ctx.drawImage(self.videoRecorder, 0, 0, 320, 240);
      var url = canvas.toDataURL('image/webp', 1);
      self.webPframes.push(url);
    }

This is a recursively called function that serves as the callback for requestAnimationFrame for as long as the user is "recording." It draws an individual frame of the stream, as it is played inside of the video element onto our canvas (which is not attached to the page's DOM anywhere or displayed) and converts those individual canvas images to data URLs, stored in webp format. toDataURL is a method of canvas. You could pass it another format other than webp, such as png or jpeg. It'll default to png if you don't pass in that parameter. This code stores all of the frames inside of an array. If we were just trying to take a photo with the webcam, we'd be done... we'd have our one image, no need to recursively call this method over and over again. But we're recording a video, which has several frames so we still have more work ahead of us. If you know anything about Javascript profiling or canvas, you are probably thinking, "This sounds like it will have shoddy performance results leading to a very low framerate and being just all-around super inefficient." I initially thought this myself, but have found the results to be somewhat impressive, given how hackish it seems--but will lower myself to admit that it is on the lesser of side of "efficiency." If you recall my mention of Tumblr building an animated GIF above, they take this same approach, limiting the number of frames captured to 4, pausing briefly between each capture so that framerate is not an issue. **But now we just have an array of still frames. "What the hell am I going to do with that, Aimee?" you ask.** We're only halfway done, chill. 

# My Blobby Babby

We want to push our array of webp images back into a blob once more. I've used [Whammy.js](https://github.com/antimatter15/whammy) to do that. It's probably the most popular tool for encoding webm using JS alone. 
    
    
    ,stop: function () {
      cancelAnimationFrame(this.rafId);
      this.webmBlob = Whammy.fromImageArray(this.webPframes, 1000/60);
      
      url = window.URL.createObjectURL(this.webmBlob);
      
      this.videoRecorder.controls = true;
      this.videoRecorder.src = url;
      this.stream = null;
      this.videoRecorder.pause();
    }