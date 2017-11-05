---
layout:     post
title:      Image processing (part 1)
date:       2015-02-11
summary:    Image processing in jacascript
categories: javascsript
thumbnail: javascript
tags:
 - javascript
 - image processing

---

# Image processing

  


Front-end applications often contain lot of images. Sometimes we want to process images e.g. when user uploads his avatar we want to give him basic photo editing tools. Processing image on backend would cause massive data transfer between browser and server. We moving more and more functionalities to browser side so we can do in this case. Using ready library is great solution but what about implementing our own solution from scratch? How hard could be making image brighter or adding B&W filter.

Staring from how image is represented in memory after reading from file. To simplify we start with black and with image. Which is matrix where each number is in range from 0 to 255, where 0 means black and 255 white.
<en-media style="height: auto;" type="image/png" hash="a63d7f65592c4bbc8ba650893cfe183d"/>

  


Wait, but what about color images? They are made from 3 matrixes: red, blue and green (RGB). The combination of this three colours in different density create every other color.
<en-media style="height: auto;" type="image/png" hash="06503faded68749421e8e0f945919696"/>

For these examples we need to create boilerplate code which will read for us image from file and process it to format on which we can start work. Sadly, JavaScript does not supports Matrixes natively so all values are stored in one dimensional array with offset.

```javascript
red   = imgData.data[0];
green = imgData.data[1];
blue  = imgData.data[2];
alpha = imgData.data[3];   

```

  


Iteration over each pixel is done by changing index by 4. The main part to focus on is *"for"* loop. In this loop we iterate over each pixel and update pixels values. This will case that our image will be brighter but we will burn pixels which now are above 255. Let canvas take care of it, but in other case we would need to cover it.

```javascript
function handleFileSelect() {
  var file = document.getElementById('file').files[0]; // FileList object
  var reader = new FileReader();
  reader.onload = function(e) {
    var img = document.getElementById('output');
    img.setAttribute('src', e.target.result);
    var canvas = document.getElementById('canvas');
    var w = img.width, h = img.height;
    canvas.width = w;
    canvas.height = h;
    var ctx = canvas.getContext('2d');
    ctx.drawImage(img, 0, 0);
    var imageData = ctx.getImageData(0,0,w,h);
    for(var i = 0; i < imageData.data.length; i += 4){
      var newPixels = processPixels(imageData.data[i], imageData.data[i + 1], imageData.data[i + 2]);
      imageData.data[i] = newPixels.R;
      imageData.data[i + 1] = newPixels.G;
      imageData.data[i + 2] = newPixels.B;
    }
    ctx.putImageData(imageData, 0, 0);
  };
  reader.readAsDataURL(file);
}

document.getElementById('change').addEventListener('click', handleFileSelect, false);

```

  


Starting with a simple example lets change brightness of an image. To do this we need to increment value of each colour (red, green, blue) by using some constant. Here is an example code, which we evaluate over every pixel on our image:

```javascript
function brighter (R, G, B) {
  return {
     R: R+45,
     G: G+45,
     B: B+45
  };
}

```

  


**R** <\- value of red ( *0-255* )

  


**G** <\- value of green ( *0-255* )

  


**B** <\- value of blue ( *0-255* )
<en-media style="height: auto;" type="image/png" hash="3e0dc0568202a081d98b06fe8de37586"/>

Next one is transfering our picture to black and white. To achieve that we need to calculate illuminance of each pixel by calculation of [Relative luminance](https://en.wikipedia.org/wiki/Relative_luminance).

```javascript
Y = 0.2126 R + 0.7152 G + 0.0722 B
R = G = B = Y

```

```javascript
function blackAndWhite(R, G, B) {
  var value = 0.2126*R + 0.7152*G + 0.0722*B;
  return {
    R: value,
    G: value,
    B: value
  };
}

```

It gives us: 
<en-media style="height: auto;" type="image/png" hash="90a312b223dc0745b7172d9b9aaaec50"/>

Next easy action we perform by updating each pixel is changing contrast. To do this we can use function: 

```javascript
function updateContrast(R, G, B) {
  var update = function (pixel) {
    return pixel*1.2+10;
  };
  return {
    R: update(R),
    G: update(G),
    B: update(B)
  };
}

```

Now we can control contrast with it. Here is result of this action: 
<en-media style="height: auto;" type="image/png" hash="d1065ae084b108782c68bf50c78b911c"/>
