# Portals
Final Project for Intro to Interactive Media
```javascript
//setup to get data from webcam
import processing.video.*;
Serial camPort;

Capture cam;

//creates floats for the dimensions of the portals and their placement on the screen
float portalWidth;
float portalHeight;
float sliceX1;
float sliceY1;
float sliceX2;
float sliceY2;

//setup to get data from arduino
import processing.serial.*;
Serial arduinoPort;

//floats for slider position values and if the button has been pressed or not (from arduino)
float slider1;
float slider2;
float slider3;
float slider4;
float slider5;
float slider6;
int takePicture;

void setup() {

  //initial positions and dimensions of the portals
  portalWidth=200;
  portalHeight=300;
  sliceX1= 0;
  sliceY1=0;
  sliceX2=width/2;
  sliceY2=0;

//gets data from Arduino
  String portName= "COM5";
  arduinoPort=new Serial(this, portName, 9600);
  arduinoPort.bufferUntil('\n');

//draws canvas
  size(640, 480);

//checks the different cameras that are available and prints them
  String[] cameras = Capture.list();

  if (cameras.length == 0) {
    //println("There are no cameras available for capture.");
    exit();
  } else {
    //println("Available cameras:");
    for (int i = 0; i < cameras.length; i++) {
      //println(cameras[i]);
    }

//the input from camera 42 is read
    cam = new Capture(this, cameras[42]);
    cam.start();
  }
}

void draw() {
  
  //reads frame from camera
  if (cam.available() == true) {
    cam.read();
  }

//flips canvas horizontally to make the image on the screen mirror the real world
  pushMatrix();
  translate(cam.width, 0);
  scale(-1, 1);
  image(cam, 0, 0);
  popMatrix();

//adds greyscale filter
  filter(GRAY);

//maps values from the sliders onto the Y position of the portals
  float myslider1= map (slider1, 0, 1023, 0, height-portalHeight+10);
  float myslider2= map(slider2, 0, 1023, 0, height-portalHeight+10);

//maps values from the sliders onto the X position of the portals
  float myslider3= map (slider3, 0, 1023, 0, width/2-portalWidth);
  float myslider4= map(slider4, 0, 1023, width/2, width-portalWidth);

//maps values from the sliders onto the width and height of the portals
  float myslider5= map (slider5, 0, 1023, 50, 200);
  float myslider6= map(slider6, 0, 1023, 50, 300);

  //println (myslider1 + myslider2+ myslider3+ myslider4);

//assigns values from the sliders to where the images should be copied from in order to move them
  sliceY1=myslider1;
  sliceY2=myslider2;

  sliceX1=myslider3;
  sliceX2=myslider4;

  portalWidth=myslider5;
  portalHeight=myslider6;

//saves frame
  saveFrame("frame.jpeg");  

//reinserts frame and copies the image from within one portal space and draws it in the other portal
  PImage frame = loadImage("frame.jpeg");
  image(frame, 0, 0, width, height);
  copy(frame, int(sliceX1), int(sliceY1), int(portalWidth), int(portalHeight), int(sliceX2), int(sliceY2), int(portalWidth), int(portalHeight));
  copy(frame, int(sliceX2), int(sliceY2), int(portalWidth), int(portalHeight), int(sliceX1), int(sliceY1), int(portalWidth), int(portalHeight));

//draws portal
  drawportals();

   //saves frame if the button to take a picture has been pressed
  if (takePicture==1) {
    saveFrame("photo-####.jpeg");
  }

}



void serialEvent(Serial arduinoPort) {
  //reads strings from the arduino
  String message=arduinoPort.readString();
  //splits the string where there are commas into an array
  message.trim();
  String[] newmessage = split(message, ",");
  
  //println(newmessage);

//assigns values from the array of data from arduino to slider variables
  slider1 = float(newmessage[0]);
  slider2 = float(newmessage[1]);
  slider3 = float(newmessage[2]);
  slider4 = float(newmessage[3]);
  slider5 = float(newmessage[4]);
  slider6 = float(newmessage[5]);
  takePicture = int(newmessage[6].trim());
  

}

//instructions to draw reactangles as the boundary around each portal
void drawportals() {
  strokeWeight (5);
  noFill();
  stroke(255, 255, 0);
  rect(sliceX1, sliceY1, portalWidth, portalHeight);
  stroke(0, 0, 153);
  rect(sliceX2, sliceY2, portalWidth, portalHeight);

}
```
